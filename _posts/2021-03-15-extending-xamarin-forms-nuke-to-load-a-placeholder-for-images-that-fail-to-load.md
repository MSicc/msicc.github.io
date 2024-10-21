---
id: 6833
title: 'Extending Xamarin.Forms.Nuke on iOS to load a placeholder for images that fail to load'
date: '2021-03-15T19:43:23+01:00'
author: 'Marco Siccardi'
excerpt: 'Whenever we load images from the web, there is a chance that the loading fails. For better user experience, having a placeholder mechanism ready is essential. In this post, I will show you how I extended my fork of Xamarin.Forms.Nuke to achieve this goal on iOS.'
layout: post
permalink: /extending-xamarin-forms-nuke-to-load-a-placeholder-for-images-that-fail-to-load/
image: /assets/img/2021/03/Placeholder_Sample_Title.png
categories:
    - 'Dev Stories'
    - iOS
    - Xamarin
tags:
    - image
    - iOS
    - Nuke
    - OpenSource
    - placeholder
    - 'xamarin forms'
---

#### What is Xamarin.Forms.Nuke?

[Xamarin.Forms.Nuke](https://www.sharpnado.com/xamarin-forms-nuke/) is a Xamarin.Forms implementation of [Nuke](https://github.com/kean/Nuke), one of the most advanced image libraries on iOS today. The [Xamarin.Forms implementation](https://github.com/roubachof/Xamarin.Forms.Nuke) focuses heavily on caching only, while the original library has a bunch of more features. I learned about this library when I started to look for an alternative to cache images via [Akavache](https://github.com/reactiveui/Akavache) which I used before (I never blogged about that part because it wasn’t ready for that, tbh).

#### Why do we need to extend the library?

The library currently does only one thing (pretty well) – handling the caching of web images. It uses the default settings of the native Nuke library. The `Xamarin.Forms` implementation overwrites the `ImageSourceHandler` for `UriImageSource` and `FileImageSource` (optionally), but the case of placeholder loading is not intended in the original version. As I have multiple scenarios where a placeholder comes in handy, I decided to extend the library – and maintain my own fork of it from now on. (Maybe there will be a pull request to the original source, too).

#### Show me some code, finally!

For our extension, we modify the `FormsHandler` class as well as the `ImageSourceHandler` class. Let’s have a look at the `FormsHandler` class first. We are adding a new property for the placeholder:

``` csharp
 public static ImageSource PlaceholderImageSource { get; private set; }
```
 
With that property in place, let’s add two methods. One method is for loading a placeholder image from an embedded resource file, while the second one is for specifying a `FontImageSource`.

``` csharp
 public static void PlaceholderFromResource(string resourceName, Assembly assembly) =>
    PlaceholderImageSource = ImageSource.FromResource(resourceName, assembly);

public static void PlaceholderFromFontImageSource(FontImageSource fontImageSource) =>
    PlaceholderImageSource = fontImageSource;
```
 
I have chosen the `FormsHandler` class because setting the placeholder is a global thing (in my scenarios, your mileage may vary). That’s already everything in takes in the `FormsHandler` class, so let’s have a look at the `ImageSourceHandler` class.

As we are using the default `Xamarin.Forms` `ImageSourceHandler` for the resource image (which is a `StreamImageSource`) and the `FontImageSource`, we need to add the static fields for them first:

``` csharp
 private static readonly StreamImagesourceHandler DefaultStreamImageSourceHandler = new StreamImagesourceHandler();

private static readonly FontImageSourceHandler DefaultFontImageSourcehandler = new FontImageSourceHandler();
```
 
Now let’s implement the loading of the placeholder in a separate method:

``` csharp
 private static Task<UIImage> LoadPlaceholderAsync()
{
    switch (FormsHandler.PlaceholderImageSource)
    {
        case StreamImageSource streamImageSource:
            FormsHandler.Warn($"loading placeholder from resource");
            return DefaultStreamImageSourceHandler.LoadImageAsync(streamImageSource);
                    
        case FontImageSource fontImageSource:
            FormsHandler.Warn($"loading placeholder from Font");
            return DefaultFontImageSourcehandler.LoadImageAsync(fontImageSource);
        default:
            FormsHandler.Warn($"no valid placeholder found");
            return null;
    }
}
```
 
As you can see, there is nothing overly complicated in this method. Based on the type of the placeholder set in the `FormsHandler` class, we are calling the default `Xamarin.Forms` implementation for the placeholder image. Let’s take this code into action by changing the `LoadImageAsync` method of the `ImageSourceHandler`:

``` csharp
 public async Task<UIImage> LoadImageAsync(
    ImageSource imageSource,
    CancellationToken cancellationToken = new CancellationToken(),
    float scale = 1)
{
    var result = await NukeHelper.LoadViaNuke(imageSource, cancellationToken, scale);
    if (result == null)
        result = await LoadPlaceholderAsync();

    return result;
}
```
 
As we need to know if the `Nukehelper` class is able to load the image, we are already running the code by awaiting it at this level. If the result is null, we are loading the placeholder image via our prior implemented method. That’s all we need to do in our forked `Xamarin.Forms.Nuke` repository.

#### How to use it in your Xamarin.Forms – iOS project

First, [clone my fork](https://github.com/MSiccDev/Xamarin.Forms.Nuke) (or fork it if you want) of the `Xamarin.Forms.Nuke` repository and import it into your `Xamarin.Forms` solution and reference it in your iOS project. Once that is done, we need to initialize the Nuke library (like in the original source) in the `AppDelegate`‘s `FinishedLaunching` method:

``` csharp
 Xamarin.Forms.Nuke.FormsHandler.Init(true, false);
```
 
 The second step is to define the placeholder image source. The `FontImageSource` should be defined after the `LoadApplication` method. This way, you can the `Xamarin.Forms` way of loading the font as a resource.

``` csharp
 //Resource image
Xamarin.Forms.Nuke.FormsHandler.PlaceholderFromResource("CachedImageTest.MSicc_Logo_Base_Blue_1024px_pad25.png", Assembly.GetAssembly(typeof(MainViewModel)));

//FontImageSource
Xamarin.Forms.Nuke.FormsHandler.PlaceholderFromFontImageSource(new FontImageSource
{
    Glyph = CachedImageTest.Resources.MaterialDesignIcons.ImageBroken,
    FontFamily = "MaterialDesignIcons",
    Color = Color.Red
});
```
 
Now use the `Xamarin.Forms` `Image` control like you always do. If the image from the web cannot be loaded, you will see the placeholder, like in these two examples:

![cached image placeholder sample](/assets/img/2021/03/Placeholder_Sample_Title.png)
<figcaption>Left: Single image failed loading, right image in `CollectionView `failed loading</figcaption>With a few additions to the `Xamarin.Forms.Nuke` library, we have implemented a placeholder mechanism for images that can’t be loaded. As always, I hope this post will be useful for some of you. Now that I have the iOS implementation of a fast cached image with placeholder loading in place, I will turn to Android, where I will try to achieve the same using the [Glidex.Forms](https://github.com/jonathanpeppers/glidex) library and extend it to load a placeholder. There will be a full sample once that is implemented as well. Stay tuned for that one!

##### Until the next post, happy coding, everyone!