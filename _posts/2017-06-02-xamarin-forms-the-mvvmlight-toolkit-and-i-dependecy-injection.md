---
id: 4610
title: 'Xamarin Forms, the MVVMLight Toolkit and I: Dependecy Injection'
date: '2017-06-02T15:00:39+02:00'
author: 'Marco Siccardi'
excerpt: 'Xamarin Forms is not always able to call platform specific code without some additional work to do. Thanks to the MVVM pattern, we are already familiar with the solution for this problem: Dependency Injection (DI). This post explains everything you need to know.'
layout: post
permalink: /xamarin-forms-the-mvvmlight-toolkit-and-i-dependecy-injection/
image: /assets/img/2017/06/Injection.png
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - Android
    - 'Dependency Injection'
    - DI
    - iOS
    - 'platform implementation'
    - uwp
    - xamarin
    - 'xamarin forms'
---

## Recap

Let’s just do a small recap what Dependency Injection means. The DI pattern’s main goal is to decouple objects and their dependencies. To separate concerns, we are using this structure nearly every time:

- interface which defines the provided functionality
- service class which provides the functionality defined in the interface
- container that allows client classes/objects to use the functionality defined in the interface

## The interface, our helpful dictator

DI always involves an interface, which dictates the functionality of the implementation. In Xamarin Forms, the interface rests inside the PCL/common project:

``` csharp
 public interface IOsVersionService
{
    string GetOsVersion { get; } 
}
```
 
This interface gets the current installed version of the operating system. The next step ist to create the platform implementation, which is commonly defined as a service class.

## Platform Implementation (service class)

We need to implement the service class for each platform. The setup is pretty easy, just add a new class and implement the interface for each platform:

![implement-interface-vs2017](/assets/img/2017/06/implement-interface-vs2017.png)


*Tip:* I am using a separate folder for platform implementations and set it to be a namespace provider. This makes it easier to maintain and I keep the same structure in all platform projects.

Let’s have a look into the specific implementations:

#### Android

``` csharp
 public string GetOsVersion
{
    get
    {
        var versionNb = Build.VERSION.Release;
        var codename = Build.VERSION.Codename;
 
        return $"Android {versionNb} ({codename})";
    }
}
```
 
#### iOS

``` csharp
 public string GetOsVersion
{
    get
    {
        try
        {
            return $"iOS {UIDevice.CurrentDevice.SystemVersion} ({UIDevice.CurrentDevice.UserInterfaceIdiom})";
 
        }
        catch
        {
            return "This demo supports iOS only for the moment";
        }
    }
}
```
 
#### Windows

``` csharp
 public string GetOsVersion
{
    get
    {
        var currentOS = AnalyticsInfo.VersionInfo.DeviceFamily;
 
        var v = ulong.Parse(AnalyticsInfo.VersionInfo.DeviceFamilyVersion);
        var v1 = (v & 0xFFFF000000000000L) >> 48;
        var v2 = (v & 0x0000FFFF00000000L) >> 32;
        var v3 = (v & 0x00000000FFFF0000L) >> 16;
        var v4 = v & 0x000000000000FFFFL;
        var versionNb = $"{v1}.{v2}.{v3}.{v4}";
 
        return $"{currentOS} {versionNb} ({AnalyticsInfo.DeviceForm})";
    }
}
```
 
Now that we are able to fetch the OS Version, we need to make the implemation visible outside of the platform assemblies. On Android and iOS, this one is pretty straigt forward by adding this Attribute on top of the class:

``` csharp
 [assembly: Xamarin.Forms.Dependency(typeof(OsVersionService))]
```
 
Because Universal Windows projects compile differently, we need to go a different route on Windows. To make the implementation visible, we need to explicit declare the class as an assembly to remain included first (otherwise the .NET Toolchain is likely to strip it away):

``` csharp
 protected override void OnLaunched(LaunchActivatedEventArgs e)
{
    //other code for initialization, removed for readabilty
 
    //modified for .NET Compile
    List<Assembly> assembliesToInclude = new List<Assembly>();
    assembliesToInclude.Add(typeof(OsVersionService).GetTypeInfo().Assembly);

    Xamarin.Forms.Forms.Init(e, assembliesToInclude); 
}
```
 
Now that we have our platform implementations in place, we can go ahead and use the interface to get the OS versions.

## Xamarin Forms DependencyService

With the static DependencyService class, Xamarin Forms provides a static container that is able to resolve interfaces to their native platform implementations. Using it is, once again, pretty straight forward:

``` csharp
 private string _osVersionViaDs;
public string OsVersionViaDs
{
    get { return _osVersionViaDs; }
    set { Set(ref _osVersionViaDs, value); }
}
 
private RelayCommand _getOSVersionViaDsCommand;
 
public RelayCommand GetOsVersionViaDsCommand => _getOSVersionViaDsCommand ?? (_getOSVersionViaDsCommand = new RelayCommand(() =>
{
    OsVersionViaDs = DependencyService.Get().GetOsVersion; 
}));
```
 
In my sample application, I am using a button that fetches the OS version via Xamarin Forms DependencyService and display it into a label in my view.

#### Special case UWP, once again

To make this acutally work in an UWP application, we need to register the Service manually. Xamarin recommends to do so in the OnLaunched event, after Xamarin Forms is initialized:

``` csharp
 //in OnLaunched event (App.xaml.cs)
//manually register for DependencyService 
//AFTER Forms is initialized but BEFORE VMLocator is initialized:
Xamarin.Forms.DependencyService.Register<OsVersionService>();
```
 
Only with that extra line of code, it will actually work like it should. If you want to know more on the fact that UWP needs a separate solution, [take a look here into the Xamarin docs](https://developer.xamarin.com/guides/xamarin-forms/platform-features/windows/installation/universal/#Target_Invocation_Exception_when_using_Compile_with_.NET_Native_tool_chain).

## Why use the MVVMLight Toolkit’s Ioc?

There are several reasons. First is of course, purely personal: because I am used to write code for it. But there are also technical reasons:

- support for cunstroctor injection
- interface implementations can have parametered constructors
- MVVMLight supports additional features like named instances and immediate creation on registration
- in MVVM(Light) applications, you are very likely using DI on Xamarin Forms level, anyway (like in a NavigationService)

You see, there are some (in my opinion) good reasons to use the built in Ioc of the MVVMLight Toolkit.

## How to use SimpleIoc and DependencyService together

If you are not relying on the DI-System of Xamarin Forms, you will have to write a lot of code yourself to get the platform implementations. That is not necessary, though. As our ViewModelLocator normally is already part of the Xamarin Forms PCL project, it has access to the DependencyService and can be used to get the right implementation:

``` csharp
 //this one gets the correct service implementation from platform implementation
var osService = DependencyService.Get();
 
// which can be used to register the service class with MVVMLight's Ioc
SimpleIoc.Default.Register<IOsVersionService>(() => osService);
```
 
This works pretty well for most implementations and provides us the possibility to use all the features MVVMLight provides. The usage then matches to what we are already familiar with:

``` csharp
 private string _osVersionViaSimpleIoc;
public string OsVersionViaSimpleIoc
{
    get { return _osVersionViaSimpleIoc; }
    set { Set(ref _osVersionViaSimpleIoc, value); }
} 
 
private RelayCommand _getOSVersionViaSimpleIocCommand;
 
public RelayCommand GetOsVersionViaSimpleIocCommand => _getOSVersionViaSimpleIocCommand ?? (_getOSVersionViaSimpleIocCommand = new RelayCommand(() =>
{
    OsVersionViaSimpleIoc = SimpleIoc.Default.GetInstance().GetOsVersion; 
}));
```
 
If you do not want (or it is not possible due to complexity) register the platform implementation directly in the ViewModelLocator, you can go down another route. You could create a new interface in the Xamarin Forms PCL which references the interface with the platform implentation as a member. Your implementation of this new interface (in Xamarin Forms) will be responsible for the getting the instance of the platform implementation via the built in DepenencyService.

I already used both ways in my recent Xamarin projects, but I prefer the first way that I described above.

## Conclusion

Due to the fact that we know the DI pattern already as we (of course) follow the MVVM pattern in our applications, there is no big mystery about using the built in DependencyService of Xamarin Forms. We can easily integrate it into the MVVMLight Toolkit and combine the best of both worlds this way.

Nonetheless, I know that also some beginners are following my posts, so I tried to describe everything a bit more extended. As always, I hope this post is helpful for some of you. In my next post, I will show you how I solved the Navigation “problem” in my Xamarin Forms applications. In the meantime, you can already have a look at the [sample code on Github](https://github.com/MSiccDev/XfMvvmLight).

Happy coding, everyone!