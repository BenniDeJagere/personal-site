+++
date = "2016-03-22T20:02:53+01:00"
draft = false
title = "Fix common binding errors with MVVM Light on Xamarin"
tags = ["Xamarin", "MVVM", "MVVM Light", "CSharp", "Android", "iOS"]
+++

There isn't much documentation available for [MVVM Light](http://www.mvvmlight.net/) when it comes to Xamarin.Android and Xamarin.iOS. There are several overloads for the `SetBinding` method and using the wrong overload causes `TargetInvocationException` or `TargetException` like [this one](http://stackoverflow.com/q/35197870/1592358). It's also possible that your bindings don't update anymore after you set one binding using an incorrect syntax.

## Correct binding

You can only bind on properties, not on fields. You can use the new C# 6 syntax if you like (`public TextView TextView => ...`). They don't always have to be `public` but it sure helps making them `public` either way. The easiest way to create a new view property on Android is with the `mvvmdroidelement` snippet provided by [this extension](https://visualstudiogallery.msdn.microsoft.com/ee36692d-ed3f-4888-b904-281aaaeac529).

You should put your bindings in your views and make sure to respect the lifecycle of the platform you're using. Also, always keep a reference to your binding so it doesn't get garbage collected. I usually Store a list with all my bindings in my view.

### Android activities

First create your layout in the `OnCreate` method, create and store the bindings and activate/update the view model if necessary. Call `Detach()` on every binding in the `OnDestroy` method.

### Android fragments

The layout can be set up in `OnCreateView`. Use the `OnViewCreated` method to set and store the bindings and activate/update the view model if necessary. Call `Detach()` on every binding in the `OnDestroy` method.

### iOS ViewControllers

Initialize everything in the `ViewDidLoad` method. Then use the `ViewWillAppear` method to set and store the bindings. In some rare cases it helps calling `ForceUpdateValueFromSourceToTarget` in `ViewDidAppear`. Bindings should be detached using `Detach()` on the binding in `ViewWillDisappear`. You can use `DidReceiveMemoryWarning` to clean up or dispose some references.

### Static view models

To avoid the mentioned `TargetException`, I'd recommend setting up a static view model locator as [Laurent Bugnion explained](http://blog.galasoft.ch/posts/2014/10/my-xamarinevolve-talk-is-online-for-your-viewing-pleasure/) and using the view models on that locator. Injecting a view model in your view to bind on, usually causes the `TargetException`, so try to use the view models defined in the locator.

### Is the source of your binding a property in your view?

Then use this one: 

```C#
this.SetBinding(() => Path.To.Property.On.Your.View, App.Locator.MyViewModel, () => App.Locator.MyViewModel.Path.To.Property.On.Your.ViewModel, BindingMode.OneWay)
```

### Is the source of your binding a property in your view model?

Then use the following overload:

```C#
App.Locator.MyViewModel.SetBinding(() => App.Locator.MyViewModel.Path.To.Property.On.Your.ViewModel, this, () => Path.To.Property.On.Your.View, BindingMode.OneWay);
```
    
### Two-way binding

```C#
this.SetBinding(() => Path.To.Property.On.Your.View, App.Locator.MyViewModel, () => App.Locator.MyViewModel.Path.To.Property.On.Your.ViewModel, BindingMode.TwoWay);
```
    
### Binding to a target type different from the source type

```C#
App.Locator.MyViewModel.SetBinding(() => App.Locator.MyViewModel.Path.To.Property.On.Your.ViewModel, this, () => Path.To.Property.On.Your.View, BindingMode.OneWay).ConvertSourceToTarget(ConversionMethod);
```
    
You can also use a lambda, but that's harder to debug.

### Just binding to a source and updating the view yourself

```C#
App.Locator.MyViewModel.SetBinding(() => App.Locator.MyViewModel.Path.To.Property.On.Your.ViewModel).WhenSourceChanges(MyUpdateMethod);
```

I guess this should cover all cases. I wrote this post using MVVM Light v5.2, but v5.3 or v6 is in the works (probably to be released at Xamarin Evolve 2016), so your mileage may vary with these newer versions.