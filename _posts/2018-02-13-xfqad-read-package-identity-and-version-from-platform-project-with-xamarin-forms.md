---
id: 5492
title: '#XfQaD: read package identity and version from platform project with Xamarin.Forms'
date: '2018-02-13T07:00:38+01:00'
author: 'Marco Siccardi'
excerpt: 'In this second #XFQaD I am showing how to read your applications''s package identity as well as the package version using their dedicated native implementations for that task.'
layout: post
permalink: /xfqad-read-package-identity-and-version-from-platform-project-with-xamarin-forms/
image: /assets/img/2018/02/PackageIdentity.png
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
tags:
    - Android
    - iOS
    - package
    - 'package identity'
    - 'package name'
    - 'package version'
    - uwp
    - version
    - xamarin
    - 'xamarin forms'
    - XfQaD
---

All of my apps, no matter on which platform, need to know their version number (for displaying in app) and their package identifier (for opening them in their store). If you are following around for some time, you know I prefer own solutions in a lot of use cases – that’s why I created another [XfQaD]({{ site.baseurl }}{/tags/xfqad/}) for this task, even if there are plugins around for that.

## The concept

Once again, I am utilizing the built-in `Xamarin.Forms`[DependencyService](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/dependency-service/) for this task. So the concept is pretty easy:

- interface that dictates the available options
- platform implementations that execute the code and return the values I want

Let’s have a look at

## The interface

``` csharp
 namespace PackageInfo
{
    public interface IAppDataHelper
    {
        string GetApplicationPackageName();

        string GetApplicationVersion();

        string GetApplicationVersionName();
    }
}
```
 
The interface provides three string returning methods. As the versioning is different on all three platforms, I return two different version strings to cover that fact.

## UWP implementation

The UWP implementation uses the [Package](https://docs.microsoft.com/en-us/uwp/api/windows.applicationmodel.package?f1url=https%3A%2F%2Fmsdn.microsoft.com%2Fquery%2Fdev15.query%3FappId%3DDev15IDEF1%26l%3DEN-US%26k%3Dk(Windows.ApplicationModel.Package)%3Bk(TargetFrameworkMoniker-.NETCore%2CVersion%3Dv5.0)%3Bk(DevLang-csharp)%26rd%3Dtrue) class, which provides access to the package information, including those we are interested in. As the UWP has just one version type, it returns the same value for version and version name:

``` csharp
 using PackageInfo.UWP;
using Windows.ApplicationModel;
using Xamarin.Forms;

[assembly: Dependency(typeof(AppDataHelper))]
namespace PackageInfo.UWP
{
    public class AppDataHelper : IAppDataHelper
    {
        private Package _package;

        public AppDataHelper()
        {
            _package = Package.Current;
        }

        public string GetApplicationPackageName()
        {
            return _package.Id.FamilyName;
        }

        public string GetApplicationVersion()
        {
            return  $"{_package.Id.Version.Major}.{_package.Id.Version.Minor}.{_package.Id.Version.Build}.{_package.Id.Version.Revision}";
        }

        public string GetApplicationVersionName()
        {
            return $"{_package.Id.Version.Major}.{_package.Id.Version.Minor}.{_package.Id.Version.Build}.{_package.Id.Version.Revision}";
        }
    }
}
```
 
## Android implementation

The Android implementation uses the [PackageManager](https://developer.xamarin.com/api/type/Android.Content.PM.PackageManager/) class, which uses the [GetPackageInfo method](https://developer.xamarin.com/api/member/Android.Content.PM.PackageManager.GetPackageInfo/p/System.String/Android.Content.PM.PackageInfoFlags/) to provide the information about the currently installed package. As Android has a different version structure ([see more info here](https://developer.xamarin.com/guides/android/advanced_topics/building-apps/building-abi-specific-apks/)), it returns two different strings for version and version name:

``` csharp
 using Android.Content;
using Android.Content.PM;
using PackageInfo.Droid;
using Xamarin.Forms;

[assembly: Dependency(typeof(AppDataHelper))]
namespace PackageInfo.Droid
{
    public class AppDataHelper : IAppDataHelper
    {
        private readonly Context _context;
        private readonly PackageManager _packageManager;
        public AppDataHelper()
        {
            _context = Android.App.Application.Context;
            _packageManager = _context.PackageManager;
        }

        public string GetApplicationPackageName()
        {
            return _context.PackageName;
        }

        public string GetApplicationVersion()
        {
            return _packageManager.GetPackageInfo(_context.PackageName, 0).VersionCode.ToString();
        }

        public string GetApplicationVersionName()
        {
            return _packageManager.GetPackageInfo(_context.PackageName, 0).VersionName;
        }
    }
}
```
 
## iOS implementation

Even iOS provides a way to get the package identity and version. It uses the [NSBundle.MainBundle](https://developer.xamarin.com/api/property/Foundation.NSBundle.MainBundle/) implementation to get the info. To get those we are interested in, we just query the `InfoDictionary`the `MainBundle`holds:

``` csharp
 using Foundation;
using PackageInfo.iOS;
using Xamarin.Forms;

[assembly: Dependency(typeof(AppDataHelper))]
namespace PackageInfo.iOS
{
    public class AppDataHelper : IAppDataHelper
    {
        private readonly NSDictionary _infoDictionary;

        public AppDataHelper()
        {
            _infoDictionary = NSBundle.MainBundle.InfoDictionary;
        }

        public string GetApplicationPackageName()
        {
            return _infoDictionary[new NSString("CFBundleIdentifier")].ToString();
        }

        public string GetApplicationVersion()
        {
            var appVersionString = NSBundle.MainBundle.ObjectForInfoDictionary("CFBundleShortVersionString").ToString();
            var appBuildNumber = NSBundle.MainBundle.ObjectForInfoDictionary("CFBundleVersion").ToString();

            return $"{appVersionString}.{appBuildNumber}";
        }

        public string GetApplicationVersionName()
        {
            return NSBundle.MainBundle.ObjectForInfoDictionary("CFBundleShortVersionString").ToString();
        }
    }
}
```
 
That’s all it takes to get your application’s package identity and version. You can have a look yourself in [this GitHub sample](https://github.com/MSiccDev/XfQADs/tree/master/PackageInfo) and play around with it. If you want to extend and read more information, the above implementation is easily expandable.

As always, I hope this post will be helpful for some of you. Happy coding, everyone!