+++
date = "2016-02-22T18:59:37+01:00"
draft = false
title = "Dependency injection with Autofac and MVVM Light in Xamarin"
tags = ["MVVM", "MVVM Light", "Xamarin", "Autofac", "Android", "iOS", "Windows RT", "C#"]
+++

## You gotta have MVVM

A developer and his tools are inseparable. We all like [SOLID](https://sites.google.com/site/unclebobconsultingllc/getting-a-solid-start) and every (.NET) developer has his or her favourite dependency injection tool. There is [a lot](http://www.hanselman.com/blog/ListOfNETDependencyInjectionContainersIOC.aspx) to choose from. I like Autofac because of the way it handles modules, the lifetime of a type and how it registers types.

At the moment I am working on an app for Android, iOS and Windows Phone with Xamarin and when you’re developing an app in C#, you’ll really want to use MVVM. You can either go the hard way and use the built-in classes, you can go the easy way and use a framework like [Caliburn Micro](http://caliburnmicro.com/) or you can go the comfortable way and use [MVVM Light](http://mvvmlight.net). MVVM Light is a toolkit. It comes with everything you need and nothing more. It doesn’t force a pattern upon you, you can use the parts you like and safely ignore everything else. Want to get started with MVVM Light? Make sure to read [Nico’s practical guide](http://www.spikie.be/blog/category/MVVM-Light.aspx).

MVVM Light comes with an IoC container called SimpleIoC. And that’s what it is: a dead-simple IoC container. As I said: you don’t have to use the parts you don’t like. So let me replace SimpleIoC with my dependency injector of choice: Autofac.

## Architecture

This is an overview of how I usually structure my solution:

```
Legend:
* project
  - namespace
    + class/interface


* MyApp (PCL)
  - MyApp.Utilities
    + MyApp.Utilities.ViewModelLocator
    + MyApp.Utilities.CrossPlatformModule
  - MyApp.ViewModels
  - MyApp.Services
    + MyApp.Services.ICrossPlatformService
    + MyApp.Services.IPlatformSpecificService
    + MyApp.Services.MyCrossPlatformServiceImplementation
    + ...
  - ...
* MyApp.Android
  - MyApp.Android.Utilities
    + MyApp.Android.Utilities.PlatformModule
  - MyApp.Android.Services
    + MyApp.Android.Services.MyPlatformSpecificServiceImplementation
    + ...
  - ...
  + App
* MyApp.iOS
  - MyApp.iOS.Utilities
    + MyApp.iOS.Utilities.PlatformModule
  - MyApp.iOS.Services
    + MyApp.iOS.Services.MyPlatformSpecificServiceImplementation
    + ...
  - ...
  + Main
* MyApp.WindowsPhone
  - MyApp.WindowsPhone.Utilities
    + MyApp.WindowsPhone.Utilities.PlatformModule
  - MyApp.WindowsPhone.Services
    + MyApp.WindowsPhone.Services.MyPlatformSpecificServiceImplementation
    + ...
  - ...
  + App
* MyApp.UnitTests
  - ...
    + ...
```

### Where to put what?

* Registrations of viewmodels and cross-platform service implementations: MyApp.Utilities.CrossPlatformModule
* Registrations of platform-specific services: MyApp.Platform.Utilities.PlatformModule
* Static properties referring to viewmodels: MyApp.Utilities.ViewModelLocator
* Autofac initialization: MyApp.Utilities.ViewModelLocator
* ViewModelLocator initialization: App/Main

## How it all ties together

First off, you start by creating interfaces for all the services you need. Next up, you can start defining implementations for the services and put them in the correct namespaces.

When that's done, it's time to create our modules. Now, assembly scanning sometimes causes exceptions on certain platforms. Also, PCL's don't have the methods you're used to from ASP.NET or other types of projects for assembly scanning. I know it makes things incredibly easy, but I'd advise against it for Xamarin projects. You'll have to register type by type in the modules. Usually I create an array of types and throw them in `builder.RegisterTypes(types)`. The platform-specific modules should contain registrations for the platform-specific services. Don't forget the ones that come with Autofac by default.

### Example of a platform-specific service

```C#
using System;
using Autofac;
using MyApp.Android.Services;
using GalaSoft.MvvmLight.Views;

namespace MyApp.Android.Utilities
{
    public class PlatformModule : Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            var navigationService = new NavigationService();
            // navigationService setup...
            builder.RegisterInstance(navigationService).AsImplementedInterfaces();

            Type[] types =
            {
                typeof (DialogService),
                typeof (MyPlatformSpecificServiceImplementation)
            };
            builder.RegisterTypes(types).AsImplementedInterfaces();
        }
    }
}
```

I think you get my point. The module in your PCL should contain all the cross-platform services and the ViewModels. Don't forget to use `.SingleInstance()` where you think it's applicable (e.g. where you use `HttpClient` or with some ViewModels).

When that's done, it's time to use a little bit of magic to make sure the right implementations are registered in the right platform. This can be a little bit tricky and it isn't a very clean solution, but It Does The Job (tm).

Laurent, the creator of MVVM Light, gave [a talk at Xamarin Evolve](http://blog.galasoft.ch/posts/2014/10/my-xamarinevolve-talk-is-online-for-your-viewing-pleasure/) explaining how he makes it work on Android, iOS and Windows Phone. On Android, you make a singleton class called `App` while you use the `Application` and `App` classes on Windows Phone and iOS.

But first, we need to create our `ViewModelLocator`. Microsoft's ServiceLocator and Autofac's extra's make things easier so all you need is `nuget Install-Package Autofac.Extras.CommonServiceLocator`.

```C#
public class ViewModelLocator
{
    // you only need this if you'd like to use design-time data which is only supported on XAML-based platforms
    static ViewModelLocator()
    {
        if (!ServiceLocator.IsLocationProviderSet)
        {
            RegisterServices(registerFakes: true);
        }
    }

    public MyViewModel MyViewModel => ServiceLocator.Current.GetInstance<MyViewModel>();

    public static void RegisterServices(ContainerBuilder registrations = null, bool registerFakes = false)
    {
        var builder = new ContainerBuilder();

        // you only need this if-clause if you'd like to use design-time data which is only supported on XAML-based platforms
        if (ViewModelBase.IsInDesignModeStatic || registerFakes)
        {
            builder.RegisterModule<FakeServiceModule>();
        }
        else
        {
            // just use this one if you don't use design-time data
            builder.RegisterModule<CrossPlatformModule>();
        }

        var container = builder.Build();
        registrations?.Update(container);

        ServiceLocator.SetLocatorProvider(() => new AutofacServiceLocator(container));
    }
}
```

Now in all of the mentioned app initializers mentioned above I have a method that looks like this:

```CSharp
private static void RegisterServices()
{
    var builder = new ContainerBuilder();
    builder.RegisterModule<PlatformModule>();
    ViewModelLocator.RegisterServices(builder);
}
```

That method is then called at the moment I initialize the `ViewModelLocator`. Laurent's talk goes in depth on how to do that so I won't cover that part.

## ServiceLocator

Whenever you need an instance of a registered type, you can use the `ServiceLocator` like so:

```C#
var myService = ServiceLocator.Current.GetInstance<IMyService>();
```

You'll only need this in the (usually empty) code-behind parts like activities (Android), ViewControllers (iOS) or the page classes (Windows Phone).

You can even use [factories](http://docs.autofac.org/en/latest/advanced/delegate-factories.html) as long as you register them.

```C#
var myVar = "some required constructor parameter for e.g. a ViewModel";
var factory = ServiceLocator.Current.GetInstance<MyViewModelRequiringAParameter.Factory>();
var vm = factory(myVar);
```