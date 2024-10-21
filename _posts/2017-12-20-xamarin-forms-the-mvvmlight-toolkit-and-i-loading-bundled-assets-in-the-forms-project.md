---
id: 5405
title: 'Xamarin Forms, the MVVMLight Toolkit and I: loading bundled assets in the Forms project'
date: '2017-12-20T11:24:40+01:00'
author: 'Marco Siccardi'
excerpt: "A common practice is bundling resource files like images or other files and ship them with the application. Apps written with\_Xamarin.Forms\_make no difference here. This post is showing a simple way to access those bundled resources/assets from within the Xamarin.Forms project."
layout: post
permalink: /xamarin-forms-the-mvvmlight-toolkit-and-i-loading-bundled-assets-in-the-forms-project/
image: /assets/img/2017/12/bundle.jpg
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - Android
    - asset
    - assets
    - bundle
    - bundled
    - 'Dependency Injection'
    - 'dependency service'
    - interface
    - iOS
    - native
    - 'platform implementation'
    - resources
    - specific
    - uwp
    - xamarin
    - 'xamarin forms'
---

## The scenario …

The reason I came up with this is that I am writing on an `Xamarin.Forms` web reader app. It is an app that uses a `WebView` to display the contents of web articles. Of course, I am using a CSS-file to style the content that gets displayed. I am using the default font of every platform, plus some platform specific settings in there. The easiest way to get it working right is to give every platform its own CSS-file. In the `Xamarin.Forms` project however, I just want to call one method that gets the thing done.

For this scenario, [the well documented Files access in Xamarin.Forms does not work](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/files/).

This post will not yet be reflected in my ongoing [XfMvvmLight project on Github](https://github.com/MSiccDev/XfMvvmLight) as I have another one building on top of this in my queue. Once the second one is written, the project will show these changes, too. This post will contain the full classes however, so you could C&amp;P them if you want/need.

## DependencyService and another interface

If you are following this series already, you might already know that the easiest way to achieve my goal is to use the built-in `Xamarin.Forms``DependencyService` and the needed interface with the native implementations.

So let’s start with the interface:

``` csharp
 namespace XfMvvmLight.Abstractions
{
    public interface IAssetPathHelper
    {
        string GetResourceFolderPath(string folderName, bool forWeb = false);

        string GetResourcePath(bool forWeb = false);

        string GetResourceFilePath(string folder, string fileName, bool forWeb = false);
    }
}
```
 
The interface dictates three string-returning methods that will either return the base path of the platform resources, a specific folder or the full path to the bundled file. This interface covers most usage scenarios I came across. Feel free to leave any feedback if I am missing out a common one.

The only thing left to do is to register the interface in our `ViewModelLocator`, like we did already before in the `RegisterServices()` method:

``` csharp
 var assetPathHelper = DependencyService.Get();
SimpleIoc.Default.Register(()=> assetPathHelper);
```
 
We are getting the platform implementation via the built in `DependecyService` and assign in to our `Xamarin.Forms` interface ([like we have done already before]({% post_url 2017-06-02-xamarin-forms-the-mvvmlight-toolkit-and-i-dependecy-injection %}/)). By registering it with our `SimpleIoc` instance, we can now use it wherever we want in our `Xamarin.Forms` project.

## Platform implementations

### Android

If you add files in the Resources folder, you can easily access them via the Resource class in your Android project. However, files like CSS-files are normally placed within the Assets folder of your Xamarin.Android project.

Depending on the usage scenario, we have two ways to access the files in the ‘Assets’ folder. If we are residing in the Xamarin.Android project and want to access the content of those bundled assets, we are able to access them using the [Android.App.Context.Assets](https://developer.xamarin.com/api/property/Android.Content.Context.Assets/) property and assign it to the [Android.Content.Res.AssetManager class](https://developer.xamarin.com/api/type/Android.Content.Res.AssetManager/). We can then use streams to get the data contained in those files.

This does not help however if we want to access those files from a `WebView` (both in the Android and the `Xamarin.Forms` project), that’s why we have to use the ‘*file:///android\_asset*‘ uri-scheme. Here is the platform implementation:

``` csharp
 using XfMvvmLight.Abstractions;
using XfMvvmLight.Droid.PlatformImplementation;
using System.Diagnostics;
using System.IO;

[assembly: Xamarin.Forms.Dependency(typeof(AssetPathHelper))]
namespace XfMvvmLight.Droid.PlatformImplementation;
{
    public class AssetPathHelper : IAssetPathHelper
    {
        public string GetResourceFolderPath(string folderName, bool forWeb = false)
        {
            return Path.Combine(GetResourcePath(),folderName);
        }

        public string GetResourcePath(bool forWeb = false)
        {
            //reminding ourselves to double check if this way is really necessary
            if (!forWeb)
            {
                Debug.WriteLine("**********************************");
                Debug.WriteLine("You should consider using AssetManager if you are not using this in a WebView.");
                Debug.WriteLine("See: https://developer.xamarin.com/guides/android/application_fundamentals/resources_in_android/part_6_-_using_android_assets/");
                Debug.WriteLine("**********************************");
            }
            
            //but we are always returning the uri scheme 
            return $"file:///android_asset"; 
        }

        public string GetResourceFilePath(string folder, string fileName, bool forWeb = false)
        {
            var folderPath = string.IsNullOrEmpty(folder) ? GetResourcePath() : GetResourceFolderPath(folder);

            return Path.Combine(folderPath,fileName);
        }
    }
}
```
 
The implementation is pretty straight forward. Although we could call all three methods, the one we use probably the most is the `GetResourceFilePath` method. It will give us the complete path to the resource file, which we can then use in our calling code of our `Xamarin.Forms` project.

By using the `Path.Combine`[ method](https://msdn.microsoft.com/en-us/library/fyy7a5kt(v=vs.110).aspx) we make sure we get a valid file path string, which is exactly what we need if we are accessing assets in this way. As most of the scenarios for accessing assets could be easily covered by using `AssetManager` (see above), I am printing a reminder message that it exists to the output window of VisualStudio.

**Important**: you have to make sure the Build Action of your files is set to `AndroidAsset`, otherwise you’ll see nothing, in some scenarios it will even throw exceptions. This accounts for the `AssetManager` as well as for the `AssetPathHelper` implementations.

### iOS

On iOS, we are able to access bundled assets via the `NSBundle` [class](https://developer.xamarin.com/api/type/MonoTouch.Foundation.NSBundle/). The implementation is even easier than the one for Android, as this is the only way to get those assets. That’s why we are ignoring the `forWeb` parameter in this case. Here is the implementation:

``` csharp
 using System.IO;
using Foundation;
using XfMvvmLight.Abstractions;
using XfMvvmLight.iOS.PlatformImplementation;

[assembly: Xamarin.Forms.Dependency(typeof(AssetPathHelper))]
namespace XfMvvmLight.iOS.PlatformImplementation
{
    //forWeb is ignored on iOS!
    public class AssetPathHelper : IAssetPathHelper
    {
        public string GetResourceFolderPath(string folderName, bool forWeb = false)
        {
            return Path.Combine(GetResourcePath(), folderName);
        }

        public string GetResourcePath(bool forWeb = false)
        {
            return NSBundle.MainBundle.BundlePath;
        }

        public string GetResourceFilePath(string folder, string fileName, bool forWeb = false)
        {
            var folderPath = string.IsNullOrEmpty(folder) ? GetResourcePath() : GetResourceFolderPath(folder);

            return Path.Combine(folderPath, fileName);
        }
    }
}
```
 
**Important**: Make sure your files have the Build Action set to `BundleResource`, because otherwise you will once again get some errors flying around your head.

### UWP

The implementation of the UWP Assets is once again the one with the most places involved. Let’s have a look at the implementation itself first:

``` csharp
 using System.IO;
using XfMvvmLight.Abstractions;

namespace XfMvvmLight.UWP.PlatformImplementations
{
    public class AssetPathHelper : IAssetPathHelper
    {
        public string GetResourceFolderPath(string folderName, bool forWeb = false)
        {
            return Path.Combine(GetResourcePath(forWeb),folderName);
        }

        public string GetResourcePath(bool forWeb = false)
        {
            if (forWeb)
            {
                return $"ms-appx-web:///";
            }
            else
            {
                return $"ms-appx:///";
            }
        }

        public string GetResourceFilePath(string folder, string fileName, bool forWeb = false)
        {
            var folderPath = string.IsNullOrEmpty(folder) ? GetResourcePath(forWeb) : GetResourceFolderPath(folder, forWeb);

            return Path.Combine(folderPath,fileName);
        }
    }
}
```
 
The UWP platform uses a separate uri-scheme for all web related things. That’s where the `forWeb` parameter comes in handy. If we are not loading a bundled asset for the web, we can use this implementation for other resources as well (bundled placeholder images are a good example here).

The next step is to add the assembly again to the list of assemblies that must be included, like we have done before in the `OnLaunched` method within `App.xaml.cs`:

``` csharp
 //modified for .NET Compile
//see https://developer.xamarin.com/guides/xamarin-forms/platform-features/windows/installation/universal/#Target_Invocation_Exception_when_using_Compile_with_.NET_Native_tool_chain
List<Assembly> assembliesToInclude =
    new List<Assembly>
    {
        typeof(OsVersionService).GetTypeInfo().Assembly,
        typeof(PlatformDialogService).GetTypeInfo().Assembly, 
        typeof(AssetPathHelper).GetTypeInfo().Assembly
    };
```
 
The last step involved in the UWP project is to register the implementation with the `DependencyService`  
**after** the `Xamarin.Forms` framework is initialized:

``` csharp
 Xamarin.Forms.DependencyService.Register<AssetPathHelper>();
```
 
The resources should be packed with a Build Action of `Content` for the UWP platform.

## Back to the Xamarin.Forms project

Now that we have everything in place on our platform projects as well as our `Xamarin.Forms` project, we finally can start using these methods. Here is an example of loading a CSS-file into a string. We can pass this string together with an HTML-string into a [HtmlWebViewSource:](https://developer.xamarin.com/api/type/Xamarin.Forms.HtmlWebViewSource/)

``` csharp
 private static string GetCssString(string cssFileName)
{
    var resourcePath = SimpleIoc.Default.GetInstance<IAssetPathHelper>().GetResourceFolderPath("HtmlResources", true);

    return $"<link rel=\"stylesheet\" href=\"{resourcePath}/{cssFileName}\">";
}
```
 
## Summary

Using the `DependencyService` of `Xamarin.Forms` allows us once again to use platform specific functionality very easily. When we are working with `WebView` and HTML, this comes in handy. If you have other valid scenarios for this implementations or even ideas on how to improve them, feel free to leave a comment below or ping me on my social network accounts. Otherwise, I hope this post is helpful for some of you.

As this is the last post before xmas, I wish you all a merry xmas in addition to my traditional

Happy coding, everyone!