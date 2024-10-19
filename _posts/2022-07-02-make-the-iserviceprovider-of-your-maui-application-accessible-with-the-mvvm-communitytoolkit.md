---
id: 7057
title: 'Make the IServiceProvider of your MAUI application accessible with the MVVM CommunityToolkit'
date: '2022-07-02T08:04:50+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I will show you how you can make the default IServiceProvider  of your MAUI application accessible via the MVVM CommunityToolkit''s Ioc.Default implementation.'
layout: post
permalink: /make-the-iserviceprovider-of-your-maui-application-accessible-with-the-mvvm-communitytoolkit/
image: /assets/img/2022/07/IServiceProviderMAUITitle-scaled.jpeg
categories:
    - 'Dev Stories'
    - MAUI
    - Xamarin
tags:
    - 'Dependency Injection'
    - dotNET
    - Ioc
    - IServiceProvider
    - MAUI
    - mvvm
---

As you might know, I am in the process of converting all my internal used libraries to be .NET MAUI compatible. This is quite a bigger task than initially thought, although I somehow enjoy the process. One thing I ran pretty fast into is the fact that you can’t access the MAUI app’s `IServiceProvider` by default.

### Possible solutions

As always, there is more than one solution. While [@DavidOrtinau](https://twitter.com/davidortinau) shows [one approach in the WeatherTwentyOne application](https://github.com/davidortinau/WeatherTwentyOne/blob/main/src/WeatherTwentyOne/Services/ServiceExtensions.cs) that accesses the platform implementation of the Services, I prefer another approach that uses, in fact, Dependency Injection to achieve the same goal.

### Implementation

I am subclassing the Microsoft.Maui.Controls.Application to provide my own, overloaded constructor where I inject the `IServiceProvider` used by the MAUI application. Within the constructor, I am using the [MVVM CommunityToolkit’s](https://docs.microsoft.com/en-us/windows/communitytoolkit/mvvm/introduction) `Ioc.Default.ConfigureServices `method to initialize the toolkit’s `Ioc` handler. Here is the code:

``` csharp
 using CommunityToolkit.Mvvm.DependencyInjection;

namespace MauiTestApp
{
	public class MyMauiAppImpl : Microsoft.Maui.Controls.Application
	{
		public MyMauiAppImpl(IServiceProvider services) 
		{
            Ioc.Default.ConfigureServices(services);
        }
	}
}
```
 
### Usage

Using the class is straight forward. Open your `App.xaml` file and replace the Application base with your `MyMauiAppImpl`:

``` csharp
 <local:MyMauiAppImpl
             xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:MauiTestApp"
             x:Class="MauiTestApp.App">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
                <ResourceDictionary Source="Resources/Styles/Styles.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</local:MyMauiAppImpl>
```
 
And, of course, the same goes for the code behind-file `App.xaml.cs`:

``` csharp
 namespace MauiTestApp;

public partial class App : MyMauiAppImpl
{
	public App(IServiceProvider serviceProvider) : base(serviceProvider)
	{
		InitializeComponent();

		MainPage = new AppShell;
	}

}
```
 
That’s it, you can now use the MVVM CommunityToolkit’s `Ioc.Default` implementation to access the registered Services, ViewModels and Views.

### Conclusion

In this post, I showed you a simple (and even easily reusable way) of making the `IServiceProvider` of your .NET MAUI application available. I also linked to an alternative approach, if you do not want to subclass the application object, I recommend that way.

As always, I hope this post is helpful for some of you.

##### Until the next post, happy coding, everyone!