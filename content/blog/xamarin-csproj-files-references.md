+++
date = "2017-05-25T21:21:16+02:00"
title = "A few common issues with .csproj files in Xamarin apps"
tags = ["xamarin", "C#"]
+++

Usually you don't need to manually edit .csproj files for your apps and most developers don't even know what's going on inside this file. However, sometimes you might run into issues related to this file where Visual/Xamarin Studio can't help you.

## The case of the missing references

At my current client our solution consists of more than 10 projects:

* portable class libraries with shared non-platform specific code
* unit test projects for every PCL (!)
* 2 UI test projects
* 2 Xamarin.iOS projects
* some Xamarin.iOS extension projects
* 2 Xamarin.Android projects
* iOS class libraries with shared iOS specific code (!)
* Android class libraries with shared Android specific code (!)

The categories marked with `(!)` were causing build errors. We had to restructure our references and suddenly we couldn't build some of them anymore.

### The build errors

```
Error CS0012: The type 'System.Object' is defined in an assembly that is not referenced. You must add a reference to assembly 'System.Runtime, Version=4.0.0.0
```
```
Error CS0012: The type 'System.ValueType' is defined in an assembly that is not referenced. You must add a reference to assembly 'System.Runtime, Version=4.0.0.0
```

... and a few more for `System.Collections`, `System.Threading`, `System.IO` etc.

So suddenly C# doesn't know about `object` or enums anymore? If you open the dialog to add references in Xamarin or Visual Studio, there isn't any option to add references to `System.Runtime`.

It took us a few hours and some research but then we found a pointer in the right direction: the ones causing issues didn't have references to any other projects in our solution. You'd think those are the ones which are the least likely to cause any issues, right?

The same code worked fine in PCL projects and you _can_ add a reference to `System.Runtime` in PCL projects. Whenever we added a reference in the errored projects to a PCL containing the missing reference, the errors were gone. However, this wasn't a viable option for us.

### The simple solution: manually add a reference in the .csproj file

At the bottom of your .csproj file, you'll find a couple of lines with all the references from your project looking like `<Reference Include="...." />`.

Just add the missing references exactly like the compiler suggests. Eventually we had to add the following references (which were all missing in the Xamarin/Visual Studio dialogs):

```XML
<Reference Include="System.Collections" />
<Reference Include="System.IO" />
<Reference Include="System.Runtime" />
<Reference Include="System.Threading" />
<Reference Include="System.Threading.Tasks" />
```

## The case of the `Microsoft.Bcl.Build` NuGet package

Dan Rigby wrote a great summary of [all the supported PCL profiles](http://danrigby.com/2014/04/16/xamarin-pcl-profile-notes/) in Xamarin. We migrated to PCL profile 44 as we wanted to use a few classes in `System.ServiceModel.Http` and `System.ComponentModel`.

After the migration the compiler was warning us that we're missing out of the fun stuff included in the `Microsoft.Bcl.Build` NuGet package.

```
All projects referencing MyProject.csproj must install nuget package Microsoft.Bcl.Build
```

So I added it to the projects that showed that warning. But the warning was still there.

Another manual .csproj file edit:

```XML
<Import Project="..\..\..\packages\Microsoft.Bcl.Build.1.0.21\build\Microsoft.Bcl.Build.targets" Condition="Exists('..\..\..\packages\Microsoft.Bcl.Build.1.0.21\build\Microsoft.Bcl.Build.targets')" />
```

Every PCL we migrated to profile 44 contained the line above at the end and the warnings were gone after removing this line.

So would I recommend you to just randomly delete some code? Well no, sir. BCL stands for Base Class Libraries and this NuGet package is meant for older kinds of projects which don't include all the plumbing required for compilation if they included newer packages like `Microsoft.Net.Http`. There are cases where you actually need this NuGet or where you get an error instead of a warning about it. But then you simply wouldn't be able to compile your code without this magical NuGet.

## Some fun stuff to look into

Before my current project, I had never used custom build targets and tasks. It was my colleague [Stefan](http://www.stefandevo.com) who showed me how you can use C# to develop custom targets and make your life easier when repeatedly doing a few common tasks before/after your build. We use it to copy or edit a few files, but you could easily extend it to uploading your app to HockeyApp or sending the related DSYM file Xamarin.Insights... As always, the possibilities are endless.

* [Custom build tasks](https://msdn.microsoft.com/en-us/library/t9883dzc.aspx)

