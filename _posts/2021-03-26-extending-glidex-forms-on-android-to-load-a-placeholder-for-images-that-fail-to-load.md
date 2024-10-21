---
id: 6853
title: 'Extending glidex.forms on Android to load a placeholder for images that fail to load'
date: '2021-03-26T18:39:50+01:00'
author: 'Marco Siccardi'
excerpt: 'Whenever we load images from the web, there is a chance that loading an image fails. For better user experience, having a placeholder mechanism ready is essential. In this post, I will show you how I extended my fork of glidex.foms to achieve this goal on Android.'
layout: post
permalink: /extending-glidex-forms-on-android-to-load-a-placeholder-for-images-that-fail-to-load/
image: /assets/img/2021/03/Placeholder_Sample_Android_Title.png
categories:
    - 'Dev Stories'
    - Android
    - Xamarin
tags:
    - Android
    - glide
    - glidex.forms
    - image
    - native
    - OpenSource
    - placeholder
    - 'xamarin forms'
---

#### What is glidex.forms?

The` glidex.forms` library is a `Xamarin.Forms` implementation of [Glide](https://github.com/bumptech/glide), which is one of the quasi standards for imaging on Android (it is even recommended by Google). Luckily, [Jonathan Peppers](https://github.com/jonathanpeppers) from Microsoft has a passion to improve `Xamarin.Android`, and `Xamarin.Forms` takes a big advantage from that as well. He made the `Xamarin.Android` Binding library as well as the `Xamarin.Forms` implementation. [Like before with Xamarin.Forms.Nuke]({% post_url 2021-03-15-extending-xamarin-forms-nuke-to-load-a-placeholder-for-images-that-fail-to-load  %}), I learned about that library because I am substituting my former image caching solution with Akavache.

#### Why do we need to extend the library?

If you want to load a placeholder that is stored in the resources of your Android project – there is no need. You could just implement a custom hook into the `GlideExtensions` class of `glidex.forms` (I’ll show you that one as well). But if you want to load an image from a `Xamarin.Forms` resource or even a from a font, we’ll need to extend the `GlideExtensions` class a little bit.

On a side note, I tried to use the existing mechanism of implementing an `IGlideHandler `for these purposes, but due to timing issues I never was able to load these kinds of placeholders and I moved on by extending the `GlideExtensions` class.

#### Show me some code!

##### Use Android resource

Let’s have a look at how to load an Android resource as a placeholder first (because it is the easiest way). To do so, we just need to create a custom implementation of an `IGlideHandler` in the Android project, which will then be called by the `GlideExtensions` implementation:

``` csharp
 public class GlideWithAndroidResourcePlaceholder : IGlideHandler
{
    public GlideWithAndroidResourcePlaceholder()
    {
    }

    public bool Build(ImageView imageView, ImageSource source, RequestBuilder builder, CancellationToken token)
    {
        if (builder != null)
        {
            //easiest way - add the image to Android resources ....
            //general placeholder:
            //builder.Placeholder(Resource.Drawable.MSicc_Logo_Base_Blue_1024px_pad25).Into(imageView);

            //error placeholder:
            builder.Error(Resource.Drawable.MSicc_Logo_Base_Blue_1024px_pad25).Into(imageView);

            return true;
        }
        else
            return false;
    }
}
```
 
As you can see, the `RequestBuilder` has already a placeholder mechanism in place. We just hook into that. If you want a general placeholder (shows the image also during download), use the `Placeholder` method. For failed image loading only, use the `Error `method. That’s it.

##### Use Xamarin.Forms resouce/FontImageSource

As we want to be able to load `FontImageSource` and `Xamarin.Forms` resources as well however, we need to go another route. Like we did on iOS, we need to add a placeholder property to the `Forms` class:

``` csharp
 public static ImageSource? PlaceholderImageSource { get; private set; }
```
 
To fill our `ImageSource` with either a `Xamarin.Forms` resource or a `FontImageSource`, add these two methods as well:

``` csharp
 public static void PlaceholderFromResource (string resourceName, Assembly assembly) =>
	PlaceholderImageSource = ImageSource.FromResource (resourceName, assembly);

public static void PlaceholderFromFontImageSource (FontImageSource fontImageSource) =>
	PlaceholderImageSource = fontImageSource;
```
 
Now that we have the `PlaceholderImageSource` in place, we need to extend the `GlideExtensions` class. First, we implement a Handler for `FontImageSource`. The `Xamarin.Forms` handler for `FontImageSource` provides us a bitmap, which we are going to use as a placeholder. I only got this working with the `AsDrawable` method, the `AsBitmap` method always threw an exception because it couldn’t cast the `Bitmap` to `Drawable` (I did not investigate that further).

``` csharp
 private static async Task<RequestBuilder?> HandleFontImageSource (RequestManager request, Context? context, FontImageSource fontImageSource, CancellationToken token, Func<bool> cancelled)
{
	if (context == null)
		return null;

	var defaultHandler = new Xamarin.Forms.Platform.Android.FontImageSourceHandler ();

	var bitmap = await defaultHandler.LoadImageAsync (fontImageSource, context, token);

	if (token.IsCancellationRequested || cancelled())
		return null;

	if (bitmap == null)
		return null;

	return request.AsDrawable().Load (bitmap);
}
```
 
As for the `HandleFontImageSource` method, I also needed to change the return value of the `HandleStreamImageSource` method to use the `AsDrawable` method, otherwise we would get the same exception as with the `FontImageSource`:

``` csharp
 static async Task<RequestBuilder?> HandleStreamImageSource (RequestManager request, StreamImageSource source, CancellationToken token, Func<bool> cancelled)
{ 
        //code omitted for readability

	return request.AsDrawable().Load (memoryStream.ToArray ());
}
```
 
The final steps to make it all work are done in the `LoadViaGlide` method. First, add another `RequestBuilder?` variable and name it `errorBuilder`:

``` csharp
 RequestBuilder? errorBuilder = null;
```
 
After the switch that handles the `source` parameter, add these lines of code:

``` csharp
 switch(Forms.PlaceholderImageSource) {
	case StreamImageSource streamSource:
		errorBuilder = await HandleStreamImageSource (request, streamSource, token, () => !IsActivityAlive (imageView, Forms.PlaceholderImageSource));
		break;
	case FontImageSource fontImageSource:
		errorBuilder = await HandleFontImageSource(request, imageView.Context, fontImageSource, token, () => !IsActivityAlive (imageView, Forms.PlaceholderImageSource));
		break;
	default:
		errorBuilder = null;
		break;
}

if (errorBuilder != null)
	builder?.Error (errorBuilder);
```
 
We are handling our `PlaceholderImageSource` like we do with the `ImageSource` that we intend to load. If we have a valid `errorBuilder`, we pass that into the entire process. With that, we have already everything in place. Let’s see how to use it in your app.

#### How to use it in your Xamarin.Forms – Android project

Head over to the `MainActivity` class in your Android project and add the following method:

``` csharp
 private void AttachGlide()
{
    Android.Glide.Forms.Init(this, null, false);

    //recommended way of loading resource images=>
    //Android.Glide.Forms.Init(this, new GlideWithAndroidResourcePlaceholder(), false);

    //Xamarin Forms resource image
    //Android.Glide.Forms.PlaceholderFromResource("CachedImageTest.MSicc_Logo_Base_Blue_1024px_pad25.png", Assembly.GetAssembly(typeof(MainViewModel)));

    //FontImageSource
    Android.Glide.Forms.PlaceholderFromFontImageSource(new FontImageSource
    {
        Glyph = XfNativeCachedImages.Resources.MaterialDesignIcons.ImageBroken,
        FontFamily = "MaterialDesignIcons",
        Color = Xamarin.Forms.Color.Red
    });
}
```
 
The Glide library needs to be initialized. Depending on the method you are using, there are separate ways to do so. The method above shows them all, you just need to change the commented code to play around with it in the sample (link at the end of the post). The mechanism is like the one I used for iOS and the Nuke package.

If you followed along (or downloaded the sample), you should see similar result to this:

![Placeholder_Sample_Android_Title](/assets/img/2021/03/Placeholder_Sample_Android_Title.png)
<figcaption>Left: Single image with placeholder, Right: CollectionView placeholder</figcaption>#### Conclusion

As we did on iOS, we now also use native caching and image handling on Android. It took me quite some time to get it up and running, but it was worth the effort. The sample has more than five hundred remote images loaded into a `CollectionView`. Check how smooth the scrolling is (even with DataBinding!).

You can [find the sample here](https://github.com/MSicc/XFNativeCachedImages) on Github, while the [modified version of glidex.forms is available here](https://github.com/MSiccDev/glidex). As always, I hope this post will be helpful for some of you.

#### Until the next post, happy coding, everyone!