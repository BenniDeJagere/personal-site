+++
date = "2016-09-04T16:29:47+02:00"
draft = false
title = "Diagnosing memory issues with the Xamarin profiler"
tags = ["Xamarin", "app performance", "profiling", "iOS", "Android"]

+++

The Xamarin profiler is a must-have tool for every Xamarin developer. The profiler comes with a Xamarin business or enterprise license and is available as a standalone installer at [xamarin.com/profiler](https://www.xamarin.com/profiler).

To get started, make sure a debug version of your app is installed on a device or a simulator (works both for Android and iOS). Another way to start the profiler is to open the profiler via *Run > Start Profiling* in Xamarin Studio or *Analyze > Xamarin Profiler* in Visual Studio. Choose the *Memory* profiling template, make sure the correct app/activity is selected and press the red record button to start profiling.

## Allocations and cyclic references

Memory issues are easy to detect and to fix, the hardest part is pinpointing the exact code causing the issue. The memory profiling template gives you two diagnostics: allocations and cyclic references.

Allocations are literally bits of memory used by your app. Every object needs some memory and the more objects stay in memory, the more memory your app uses. So from time to time, the system (called the Garbage Collector or GC) needs to remove old, unused objects from the memory. The allocations tab lets you know how many instances of an object are living in the memory and how much space they occupate.

So how should the Garbage Collector know that you no longer need a particular instance of an object? It's simple. Objects stay in memory as long as another object has a reference to it. In the worst case scenario an object never dies because of a cyclic reference. A cyclic reference occurs when two objects have (indirect) references to each other. This should always be avoided.

The tab with the cyclic references stays empty until the profiling is stopped. The allocations tab should give an indication where or when something goes wrong while running the app.

## How to collect useful information: snapshots

Pinpointing the exact cause of an issue can be hard, but luckily the Xamarin profiler comes with some tools that can help you if you use them wisely. I'd recommend writing down a few scenarios with small steps that make you go through the most performance intensive parts of your app. E.g. in an app about recipes, you could have the following scenario:

1. Open the app
1. Swipe left twice to the third tab with the ingredients
1. Sort the ingredients by name by pressing the sort button
1. Swipe back twice to the first tab

The scenario should always produce the same result while you're profiling your app and every step should consist of a single action. If the above scenario was about an Android app with a `ViewPager` with the default settings, the list with the ingredients should no longer reside in the memory after the fourth step as the default `ViewPager` only keeps the current and the neighboring fragments in memory.

The camera button in the Xamarin profiler allows you to take snapshots of the current memory usage. Taking a snapshot also makes the garbage collector run so that you know for certain which objects still have references to them and which don't. Take a snapshot after every step in your scenario and press the stop button after you've collected your last snapshot.

When you're done collecting data, you can save the collected data to a file (.mlpd) so you can share it with your colleagues or analyse it when you have the time.

By default the dataset contains objects of every type. The data becomes clearer when you use the filter box at the bottom to only show objects in the namespace of your project.

## Analysing the cyclic references

The tab with the cyclic references don't take the snapshots in account and is just a list of all the cyclic references in your code. When you select one, you can see the path the references follow as most cyclic references are indirect. This list can get long if you use dependency injection.

## Analysing the allocations

Under the allocations, you'll find a tab called *Snapshots* which tells you how much new objects are kept in memory since the previous snapshot. Use the filter box to only look for objects in your namespace and sort by object or size growth to find the biggest culprits. The number of the snapshot should tell you where in your app the objects were created. A lot of new objects is no problem as long as die in one of the following snapshots.

## References to native objects

The Xamarin profiler only tells you a part of the story. Xamarin.Android apps have two runtimes: the Mono runtime and the Android RunTime (ART) or Dalvik (Android 4.4 or older). ART also has its own garbage collector and collects the native objects. Both Xamarin.Android and Xamarin.iOS apps have references to native objects. As long as these references are not removed, the native objects also stay in memory.

## More resources

There are a lot of useful resources about optimizing Xamarin apps for performance and I'll add some to my blog. In the meantime, check out the following resources:

* Xamarin University: Diagnosing Memory Issues [XAM370]
* Xamarin University: GC Fundamentals [CSC270]
* Xamarin University: Managing non-memory resources [CSC271]
* [Cross-Platform Performance](https://developer.xamarin.com/guides/cross-platform/deployment,_testing,_and_metrics/memory_perf_best_practices/)
* [Introduction to the Xamarin Profiler](https://developer.xamarin.com/guides/cross-platform/deployment,_testing,_and_metrics/xamarin-profiler/)