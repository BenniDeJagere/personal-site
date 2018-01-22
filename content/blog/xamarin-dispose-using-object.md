+++
date = "2017-06-02T16:32:42+02:00"
title = "Optimize memory usage in Xamarin apps"
draft = false
tags = ["xamarin", "csharp", "android", "ios", "xamarin.forms", "app performance"]
+++

> This post has been translated to Russian by [Denis Gordin](https://twitter.com/g0rdan). You can read the Russian version on the Russian website [TechMedia](https://habrahabr.ru/post/330854/). Thanks, Denis!

Xamarin is amazing in how it allows .NET developers to write apps for Android, iOS, MacOS... in C#. But that amazing capability comes with a prize and even the most simple apps can suffer from high memory usage. Let's find out what happens and what we can do about it. The majority of my examples are based on Xamarin.Android, but you'll quickly notice how this also applies to Xamarin.iOS.

## Garbage collection in Xamarin apps

In Xamarin apps, you actually use multiple kinds of objects. Every Xamarin app has objects living in two separate worlds:

* the managed, Mono world with objects deriving from `System.Object`
* the unmanaged, native world with objects deriving from either `NSObject` (iOS) or `Java.Lang.Object` (Android)

This also means there are two garbage collectors at work:

* the managed, Mono garbage collector named **SGEN**
* the unmanaged, native garbage collector on Android or iOS

Let's focus on SGEN first. [Xamarin University](https://university.xamarin.com/classes/track/csharp) actually has a number of very interesting classes on this topic and it's very well explained in the [documentation](https://developer.xamarin.com/guides/android/advanced_topics/garbage_collection/) too.

I won't be going into details on how SGEN works and leave this topic for a future blogpost. All you need to know for now is that you can force a full garbage collection with `GC.Collect()` and a small collection (only the most recent created objects, also known as *Generation 0*) with `GC.Collect(0)`. Most of the other commands in the `GC` class are not implemented in Mono at the time of writing. An alternative to forcing this in code is using the snapshot function in the [Xamarin Profiler](https://www.xamarin.com/profiler).

When SGEN is done collecting garbage, it also triggers a collection on the native side of things.

## Peer objects

Didn't I tell you there are two types of objects in Xamarin? Yes and no. All the objects live in either of the two worlds, but we actually use three kinds of objects:

* managed objects (Mono world)
* native objects (native world)
* peer objects (Mono world, bridges from the managed world to the unmanaged world)

We can further divide the peer objects into two categories:

* framework peers: instances of classes which are part of the Xamarin.Android or Xamarin.iOS SDKs
* user peers: instances of classes you created yourself which derive from native objects

So as a Xamarin developer you create either managed objects, either user peers.

Some examples:

* __framework peers__: `android.content.Context`, `UIViewController`...
* __user peers__: `MyCustomActivity`, `MyCustomViewHolder`, `MyCustomViewController`...

How are they different? Let's take a look from the Android side (iOS works in a similar fashion):

A framework peer is often called a *Managed Callable Wrapper* (MCW). As the name says it's:

1. Managed Callable: the object exists and is callable in the Mono world
1. Wrapper: it wraps the native Android object into a Mono object

This is also what you create when you build an Android binding project. Underneath, Xamarin generates code that invokes native methods on the Android object. To achieve that, they use JNI (*Java Native Interface*). If you want to call a method that exists in Android, but which hasn't been mapped (yet) in Xamarin, you can use JNI to call that method.

A user peer is often called an *Android Callable Wrapper* (ACW). As the name says it's:

1. Android Callable: the object exists and is callable in the Android world
1. Wrapper: it's nothing more than a wrapper which is able to invoke the methods from the Mono world

So, actually you could say every peer object actually consists of two objects effectively living in the memory: the real, native or Mono object and the wrapper object.

This structure is what makes Xamarin work and why Xamarin is so awesome. The experience for developers becomes very simple, but a lack of understanding of how this works is often the source of memory issues in Xamarin apps.

## Danger ahead! Classic example: the bitmap

The most common _big_ objects in Xamarin apps are bitmaps. Almost every app contains a few images to make it look more amazing. But not without a cost, these are most often the biggest objects in the memory of your app.

However, if you let Android load a bitmap and verify its memory usage using the tooling you know and love, you'll probably notice its size is insignificant. Even a 5 MB image merely takes up a few bytes.

How is that possible? Where did that 5 MB go? For the Mono world, the bitmap is nothing more than a wrapper pointing to the native object. It's the native object that occupies the 5 MB in memory.

Okay, fine, but how would that cause any issues and how does that fit into the context of this blogpost? Take a look at the following activity:

It loads 100 bitmaps and verifies if the image contains a unicorn and if so, shows it in the `ImageView`. We only use one bitmap at the end, so there shouldn't be any memory issues because the allocated bitmaps are going out of scope and they would be collected by the garbage collector, right?

```C#
[Activity(Label = "App1", MainLauncher = true, Icon = "@drawable/icon")]
public class MainActivity : Activity
{
    protected override void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);
        SetContentView(Resource.Layout.Main);

        for (int i = 0; i < 100; i++)
        {
            var hugeBitmap = Android.Graphics.BitmapFactory.DecodeFile($"path/to/bitmaps/{i}.png");
            if(!ImageContainsUnicorn(hugeBitmap))
            {
                continue;
            }

            var imageView = FindViewById<ImageView>(Resource.Id.SomeImageView);
            imageView.SetImageBitmap(hugeBitmap);
        }
    }
}
```

The app would crash in a matter of milliseconds due to an `OutOfMemoryException`. To find out why, let's take a look at the plumbing which Xamarin does for us.

The variable `hugeBitmap` is actually a MCW and it's size in the Mono world would be insignificant. The code above wouldn't usually trigger a garbage collection.

On the Android side of things, the droids are going crazy and the garbage collector would be doing a lot of overtime. However, it wouldn't be able to find anything to collect. The collector can't collect the bitmaps as they are still referenced from the managed wrapper objects. So as long as the managed wrappers aren't collected by SGEN, the native garbage collector can't do a thing. This results in the native side of your app going out of memory.

## What can we do to fix this?

Every peer object implements the `IDisposable` interface. Let's take a quick look at how this is implemented:

* [`NSObject` in `Foundation`](https://github.com/xamarin/xamarin-macios/blob/master/src/Foundation/NSObject2.cs#L730)
* [`Object` in `Java.Lang`](https://github.com/xamarin/xamarin-android/blob/master/src/Mono.Android/Java.Lang/Object.cs#L227)

_Note that the implemenation above for Xamarin.Android is no longer used in the latest version as they migrated to using [Java.Interop](https://github.com/xamarin/java.interop). While the implementation itself is very different, the way it works is very analogue to the old one._

We can see that a call to `Dispose()` actually breaks the bridge between the wrapper object and the wrapped object. This removes the references and after disposing the wrapper, the native object can be garbage collected during the next run as long as it doesn't have any other references in the native world.

> Great! So I should just call `Dispose()` on everything?

Close, but no cigar. We can actually improve the code above by implementing the `using` pattern. As we know, `using` immediately calls `Dispose()` at the end of its code block. 99% of the time it's perfectly okay to dispose **framework peers** immediately after you've called the methods/properties you needed. The native object will continue to live as long as necessary and you won't break anything in the running code except for the reference.

A better version of the code above would look like this:

```C#
protected override void OnCreate(Bundle bundle)
{
    base.OnCreate(bundle);
    SetContentView(Resource.Layout.Main);

    for (int i = 0; i < 1000; i++)
    {
        using(var hugeBitmap = Android.Graphics.BitmapFactory.DecodeFile($"path/to/bitmaps/{i}.png"))
        {
            if (!ImageContainsUnicorn(hugeBitmap))
            {
                continue;
            }

            using(var imageView = FindViewById<ImageView>(Resource.Id.SomeImageView))
            {
                imageView.SetImageBitmap(hugeBitmap);
            }
        }
    }
}
```

However, if I would have needed the `Imageview` in another method like `OnResume()`, a better place to dispose of it would be in the `OnDestroy()` or in the `Dispose()` method of the activity itself. You could argue that you can also call `FindViewById()` multiple times, but this is a very costly operation so it should generally be avoided. I generally use the method at the very end of the lifecycle of an object or I override the `Dispose()` method. This is not something you *have* to do, but it would certainly help to bring down memory usage in most apps.

## Short note about events

As you would suspect, event subscriptions are also references underneath. Be sure to never forgot to unsubscribe your events in the last lifecycle method or SGEN will never collect your object. If your object has references to peer objects, then those peer objects would continue to live indefinitely.

## Why you should avoid calling `Dispose()` on user peers

Xamarin makes sure to call `Dispose()` whenever it's the right time, but for us app developers, this isn't always easy to figure out. The documentation actually recommends to never manually dispose user peers, but just make sure that nothing references it so that the framework will handle disposing these objects for you.

### The constructor with `IntPtr` and `JNIHandleOwnership`

If you did dispose a user peer and the Android OS needs the disposed user peer, Mono will call the constructor below:

```C#
public MyClass(IntPtr javaRef, JniHandleOwnership transfer) : base(javaRef, transfer) { }
```

The same constructor exists in Xamarin.iOS without the `JniHandleOwnership`. Mono actually tries to reinstate the object you disposed earlier.

If this constructor doesn't exist, your app will immediately crash with a `NotSupportedException`. If Google ever decides to change the lifecycle of some kind of object and you call `Dispose()` before the end of the object's lifecycle, then this would happen.

## How `WeakReference` can help you

Using `WeakReference` instead of the regular (hard) references, avoids placing the reference on the native objects. There's a little bit of performance overhead in retrieving these objects and they can be collected by the native garbage collector at any given time. So choose your references wisely! Bitmaps that can't be immediately disposed would be good candidates for weak references, but smaller objects like a `UILabel` wouldn't matter that much.

## What about Xamarin.Forms?

Every Xamarin.Forms element has a renderer on the mobile platforms, either custom or shipped with the NuGet package. These renderers are user peers and they are treated as such. [Here's an example](https://github.com/xamarin/Xamarin.Forms/blob/master/Xamarin.Forms.Platform.Android/ViewRenderer.cs#L79) of a `Dispose()` implementation in one of the built-in Android renderers. I'd recommend to follow the same pattern in custom renderers and always dispose the native elements inside of it.

## Let Android and iOS guide you

Both Android and iOS apps contain hooks that are triggered when the memory usage becomes alarming. On iOS it's [`DidReceiveMemoryWarning()` in `UIViewController`](https://developer.apple.com/reference/uikit/uiviewcontroller/1621409-didreceivememorywarning). On Android it's a little bit more hidden and less documented: [`OnTrimMemory()` in `Application`](https://developer.android.com/reference/android/app/Application.html#onTrimMemory(int)). The logical thing to do inside of these methods is to call `GC.Collect()`. This will clean up some objects, trigger a few finalizers and call `Dispose()` on peer objects which are no longer used, so that the native garbage collector has a little bit more room to clean things up on the native side too.

## Conclusion

I think this post offers some helpful guidelines to increase the memory efficiency in your Xamarin apps but do note that there is so much more to tell about this. Over time, I will cover this with more blogposts, but in the meantime, you can continue reading through the Xamarin documentation, browse through the Xamarin source code over on GitHub or experiment with the Xamarin Profiler or the native profilers.