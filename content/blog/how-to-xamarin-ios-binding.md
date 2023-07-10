+++
date = "2016-08-29T11:10:40+02:00"
draft = false
title = "Creating a Xamarin.iOS binding project for dummies"
tags = ["Xamarin", "iOS", "C#", "Objective-C"]
+++

## What you need

* Experience with Xamarin.iOS
* Xamarin Studio for Mac
* An empty binding project (just create a new project in Xamarin Studio)

## A very short intro to Objective-C for C# developers

Oh god, Obj-C, the most incomprehensible programming language in the app dev world. You simply can't create an iOS binding project without some very basic knowledge of Obj-C. So here goes, an intro to Obj-C for C# developers.

When you're developing a bindings library, you'll get the iOS project in one of the following forms:

* a .a file (static library) - basically like a .DLL
* a .framework file (Cocoa Touch framework) - basically like a .DLL with some resources
* a CocoaPod - just like NuGet
* source code with project file (containing a .xcodeproj file) - like source code including .csproj and .sln files
* source code without project file - just like a bunch of .cs files...

Obj-C projects have .h files and .m files. The headers (.h files) describe something like a class. You could think of them like interfaces. The models (.m files) are the implementations. Each .h file has one .m file. Except for those files, every project usually has an _info.plist_ file describing project settings (names, supported architectures...) and a _MyProject.xcodeproj_ file (this is actually a directory containing multiple configuration files) describing the build settings etc.

Now, there are a few concepts in the programming language itself which you should be familiar with:

### Protocols

Protocols are like interfaces or abstract classes. The main difference between C# interfaces and protocols are the optional methods. The protocol defines optional methods that the models may or may not implement. Each class in Obj-C can implement multiple protocols just like a C# class can implement multiple interfaces. Protocols are usually translated in C# as abstract classes. Note that when you're overriding from a protocol, the base method could have no implementation if the original Obj-C method was optional. This means that you generally should avoid calling the base methods. When in doubt, consult [the Apple developer documentation](https://developer.apple.com/reference/).

### Umbrella headers

Each iOS framework or library has one header that is the umbrella header for that project. It contains references to all the other headers in that project and defines the base functionality for that library. The umbrella header should have the same name as the project itself. This header is essential for our binding project. Sometimes, you'll have to modify this header to include references to other relevant headers in the project for the bindings to work properly. When that's the case, you just have to add `#import "AnotherHeader.h"` statements at the top of the file, just like the `using` statement in C#.

It may be useful to browse through [this documentation page](https://developer.xamarin.com/guides/ios/application_fundamentals/delegates,_protocols,_and_events/) at Xamarin.com.

### Static libraries and frameworks

When you want to distribute something in the .NET world, you usually just create a class library and distribute its source code or the compiled version (DLL). There are two types of libraries in the iOS world.

Cocoa Touch Static Libraries are just like the class libraries you know in .NET. The other kind, frameworks, are about the same but they also contain the headers (most of the time just the umbrella header) and media resources (like a NuGet package).

## Getting started with your binding project

To create a binding project for iOS, you'll need the Objective Sharpie tool. The latest version is available at [Xamarin's website](https://developer.xamarin.com/guides/cross-platform/macios/binding/objective-sharpie/getting-started/) and if you already have the tool installed, you can use the `sharpie update` command to make sure you have the latest version.

A binding project consist of three parts:

1. A native reference (either a .a file or a .framework file)
1. The _StructsAndEnums.cs_ file containing all the used enums
1. The _ApiDefinitions.cs_ file containing all the definitions of the classes used in the iOS framework/library

The API definitions are a list of interfaces with methods decorated with attributes. The attributes tell Xamarin how it should generate the C# API that is bound to the native code. There should be at least one interface with the name of the project itself (or the name of the umbrella header).

The following sections describe how to generate both files using Objective Sharpie from easiest to most difficult approach. In most cases, you can try multiple approaches. So if one of them doesn't properly generate the API definitions for you, just move on and try another one. You'd be surprised how much of a difference they can make, even if the framework and the CocoaPod contain the same files. I strongly recommend to read through every approach.

### When the project is available as [CocoaPod](https://cocoapods.org) (recommended approach)

.NET developers are usually familiar with NuGet packages. They always contain a bunch of .dll files and define dependencies and compatible project types. CocoaPods consist of either compiled code, either source code. To generate the API definitions from a CocoaPod, use the following commands in an empty directory:

```shell
sharpie pod init ios MyCocoaPod
sharpie pod bind -n MyNamespace
```

The CocoaPod is downloaded to the _Pods_ directory. When you browse through its files, you'll quickly notice if it contains just the source code or also a binary file (without extension), a framework or a static library (.a file).

### When the project is available as a .framework file

When your project is supplied as a framework, simply open the framework and verify if it has a compiled binary file inside. If it has one, read the part about the binary file below. Otherwise, use the source code to create the binary yourself as documented below.

Alternatively, the following command may also do the job, this doesn't always work though.

```shell
sharpie bind -o OutputDirectory -sdk iphoneos9.3 -n MyNamespace -framework MyFramework.framework
```

## Inspecting the binary file (without extension or .a file) and generating api definitions

What you have to verify is the architecture. The iOS simulator has the `i386 x86_64` architectures and iPhones and iPads have the `armv7 arm64` and possibly `armv7s` (only useful on iPhone 5/5c) architectures. You can use the `lipo` command to verify the architectures of a binary like so:

```shell
lipo -info MyFramework.framework/MyFramework
lipo -info Pods/MySDK/MySDK.framework/MySDK
lipo -info JustABinary.a
```

If you want to debug your app in the iOS simulator, you _need_  the simulator architectures. But make sure to remove these architectures **when you submit your app to the App Store** as Apple could deny your app if it includes simulator bytecode. The command to remove these architectures is the following:

```shell
lipo -remove i386 x86_64 -output Path/To/Binary/File Path/To/Binary/File
```

Note that you can't access the binary file after you've created the bindings. This means that you possibly could have to create two versions of the binding project. One with the simulator architectures and one without them. The only difference would be the referenced file in the native references. This can easily be fully automated with MSBuild targets or in your continuous integration process.

If the binary file doesn't include the architectures you require, try to build it yourself as documented below.

Now, if you have a binary, regardless of its architectures, you can use it to generate the necessary API definitions. This is what the Xamarin tooling uses to build the bindings itself.

## Building the binary file yourself

If you don't have a binary file or if your binary file doesn't support all the architectures you need, you end up building one yourself. This is the hardest part of the binding process but it's well documented on [Xamarin's website](https://developer.xamarin.com/guides/ios/advanced_topics/binding_objective-c/walkthrough/#Creating_A_Static_Library). The Xamarin guide describes how to create a static library, but I found it easier to create a framework instead. I used Xcode 7 for this part, but the minimum is Xcode 6.

If the source code already contains an Xcode project, you're golden. Just open the Xcode project and try to build it. If not, continue...

Create a new Xcode project and choose _iOS > Framework & Library > Cocoa Touch Framework_. Now you basically have to put all the files from the source code in this project. You can't replace the header that Xcode created by default, so just copy paste the code from the header in the sources into the one that Xcode created. Try to build the project. Sometimes the linking goes wrong and you have to fix the links between the .h and the .m files yourself.

### Creating the binary for multiple architectures

As mentioned before, you'll probably need a binary that contains the following architectures: `i386 x86_64 armv7 arvmv64` and you can use the `lipo -info mybinary` command to inspect the supported architectures. Your binary probably won't contain all of them, so you can use the commands below to combine build and combine multiple binaries:

#### Build for ARM devices

```shell
/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild ONLY_ACTIVE_ARCH=NO -project XcodeProject.xproj -target NameOfTheProject -sdk iphoneos -configuration Release clean build
```

#### Build for simulator

```shell
/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild ONLY_ACTIVE_ARCH=NO -project XcodeProject.xproj -target NameOfTheProject -sdk iphonesimulator -configuration Release clean build
```

#### Combine both binaries into one

```shell
lipo -create -output PathToCombinedFile PathToARMBinary PathToSimulatorBinary
```

#### Makefile which does everything for you

This Makefile also generates the binding definitions (described below) and is used for Cocoa Touch Frameworks.

```makefile
XBUILD=/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild
PROJECT_ROOT=PathToProject
PROJECT=$(PROJECT_ROOT)/NameOfXcodeProject.xcodeproj
TARGET=NameOfTheProject
BINDING_PROJECT=NamespaceOfTheBindings

all: $(TARGET).framework

$(TARGET)-simulator.framework:
	$(XBUILD) ONLY_ACTIVE_ARCH=NO -project $(PROJECT) -target $(TARGET) -sdk iphonesimulator -configuration Release clean build
	mv $(PROJECT_ROOT)/build/Release-iphonesimulator/$(TARGET).framework $(TARGET)-simulator.framework

$(TARGET)-iphone.framework:
	$(XBUILD) ONLY_ACTIVE_ARCH=NO -project $(PROJECT) -target $(TARGET) -sdk iphoneos -configuration Release clean build
	mv $(PROJECT_ROOT)/build/Release-iphoneos/$(TARGET).framework $(TARGET)-iphone.framework

$(TARGET).framework: $(TARGET)-simulator.framework $(TARGET)-iphone.framework $(BINDING_PROJECT)/Generated_ApiDefinitions.cs
	cp -R $(TARGET)-iphone.framework ./$(TARGET).framework
	rm ./$(TARGET).framework/$(TARGET)
	lipo -create -output $(TARGET).framework/$(TARGET) $(TARGET)-iphone.framework/$(TARGET) $(TARGET)-simulator.framework/$(TARGET)

$(BINDING_PROJECT)/Generated_ApiDefinitions.cs:
	 sharpie bind -p Generated_ -n $(BINDING_PROJECT) -o $(BINDING_PROJECT) $(PROJECT)

clean:
	rm -rf *.framework
	rm $(BINDING_PROJECT)/Generated_ApiDefinitions.cs
	rm $(BINDING_PROJECT)/Generated_StructsAndEnums.cs
```

## Generating the bindings from the binary file

Generating the API definitions from a binary is as simply as running a single command.

```shell
sharpie bind -o OutputDirectory -sdk iphoneos9.3 -n MyNamespace MyBinaryFile.a
sharpie bind -o OutputDirectory -sdk iphoneos9.3 -n MyNamespace MyBinaryFile
sharpie bind -o OutputDirectory -sdk iphoneos9.3 -n MyNamespace MyXcodeProject.xcodeproj
```

There will probably be a few warnings in the output, but if you got some errors, you need to choose a different approach or take a look at the Objective-C source code.

## The right sharpie SDK version

Objective Sharpie works for every version of iOS, macOS, watchOS, tvOS etc. To get a list of the available SDK versions on your system, you can run `sharpie xcode -sdks`.

## Fixing the generated definitions

I haven't done a binding project where the definitions Objective Sharpie generated were perfect. Most of the binding definition syntax is documented on [Xamarin's website](https://developer.xamarin.com/guides/cross-platform/macios/binding/binding-types-reference/).

First, let's make sure you get the point of these files. The _StructsAndEnums.cs_ file is just a collection of enums used in the project. The _ApiDefinitions.cs_ file is the keystone. This file contains a list of interfaces which are used by the Xamarin tooling to create implementations that call the native binary/framework. All we have to do is define the C# interface which is going to be available in our binding and which native method should be used for each C# method/property. A binding project generates a DLL just like any other .NET library.

### Fixing the enums

You may find the enumerations to be decorated with the `[Native]` attribute. This means that it refers to an enum used in the native code. Just make sure that the underlying type of this enum is `long`. You'll also notice that Objective Sharpie generates enums where the underlying type is `uint` which is technically impossible. I usually make the underlying type `byte` unless it has some values defined which don't fit in a byte (0-255).

```C#
[Native]
public enum ExampleEnum: long
{
    Value1,
    Value2
}
```

### Enums or interfaces decorated with the `[Verify(InferredFromMemberPrefix)]` attribute

Remove the attribute and verify the name of the interfaces or enum. The name could not be determined by Objective Sharpie and you're probably better of naming it yourself.

### Interfaces decorated with `[Category]`

This is a list of extension methods. Make sure the class only contains methods (no properties) as they will all be made static. You can easily replace a property by a method by writing the method yourself and decorating it with the `[Export("nameOfNativeMethod:")]` attribute.

### The partial constants interface

Usually you'll find a couple of these in the definitions and they're all marked by the `[ConstantsInterfaceAssociation]` attribute. Remove the attribute and put all the constants in a single interface decorated with `[Static]`. The constants themselves have the `[Field]` attribute.

### Interfaces with `[Protocol, Model]`

This generates an implementation and an interface. However, you have to declare the interface yourself. So if the native protocol would be called `AmazingService`, you'd have to add an interface called `IAmazingService`.

#### Before

```C#
[Protocol, Model]
[BaseType(typeof(NSObject))]
interface AmazingService
{
    a few methods...
}
```

#### After

```C#
public interface IAmazingService {}

[Protocol, Model]
[BaseType(typeof(NSObject))]
interface AmazingService: IAmazingService
{
    some methods...
}
```

### Interfaces decorated with `[Verify(MethodToProperty)]`

It's up to you to decide if the bindings should contain properties or methods. Often, you can replace a method with a getter and or you can combine two methods and replace them with a getter and setter. Objective Sharpie also tries to do this for you and marks them with the aforementioned attribute. You can safely remove the attribute and ignore it unless they're part of an interface which is marked with the `[Category]` attribute.

### Methods marked with `[Verify(StronglyTypedNSArray)]`

Objective Sharpie couldn't determine the type of the array passed in/out this method/property and used `NSObject` as the type. Replace it with a more specific type or let it be and remove the attribute.

### Converting delegates to events

The concept of events as we know them in the .NET framework is not available in Objective-C. Obj-C uses delegates and they resemble the concept of delegates in .NET. However, we don't use delegates in .NET anymore so we'd like our bindings to have events instead of delegates. Jim has written [a nice blog post](https://www.jimbobbennett.io/binding-ios-libraries-in-xamarin/) covering this topic (step 5).

### Mapping constructors

To let Xamarin generate a non-default constructor simply add a method named `Constructor` returning an `IntPtr`. By default, Xamarin always generates a parameterless constructor. To disable this, add `[DisableDefaultCtor]` to the interface defining the class.

### Protocols & abstract constructors

Protocols in Objective-C are mapped to an interface containing all the methods defined in the protocol, an abstract class where only optional methods are marked `abstract` and a static class with extension methods containing the optional methods. This means that you have to mark all the non-optional methods as `[Abstract]` in your API definitions.

There is one exception. If you add `[Abstract]` to a constructor in a protocol, Xamarin generates a constructor with the signature `public abstract MyProtocol(Type1 param1){ ... }`. This is invalid syntax in C#, an abstract constructor should always be `protected`. Just remove the `[Abstract]` from the constructor.

### Other `[Verify]` attributes

Run the command `sharpie verify-docs` for the latest docs on the `[Verify]` attributes you'll find. You have to remove all these attributes before you can compile your binding project.

## Congrats

You're no longer a dummy and I hope you have a working binding project. If you're having trouble generating the required binary file, I suggest you contact an iOS developer.
