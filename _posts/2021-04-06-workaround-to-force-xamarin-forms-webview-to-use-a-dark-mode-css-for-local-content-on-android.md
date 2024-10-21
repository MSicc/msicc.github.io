---
id: 6870
title: 'Workaround to force Xamarin.Forms WebView to use a dark mode CSS for local content on Android'
date: '2021-04-06T09:00:00+02:00'
author: 'Marco Siccardi'
excerpt: 'As the ''prefer-color-scheme''-CSS query does not work on Android with a Xamarin.Forms WebView, I needed find a workaround to support dark mode for local HTML content. In this post, I''ll show you a Xamarin.Forms only workaround.'
layout: post
permalink: /workaround-to-force-xamarin-forms-webview-to-use-a-dark-mode-css-for-local-content-on-android/
image: /assets/img/2021/04/android_webview_darkmode_xamarinforms.png
categories:
    - 'Dev Stories'
    - Android
    - Xamarin
tags:
    - Android
    - CSS
    - 'dark mode'
    - theme
    - webview
    - workaround
    - xamarin
    - 'xamarin forms'
---

Recently I updated my blog reader app to support the dark mode newer iOS and Android version support. While everything went smooth on iOS and the update is already live in the App Store, I had some more work to do on Android. One of the bigger problems: the `WebView` I use to view posts does not automatically switch to dark mode with `Xamarin.Forms`.

#### What’s causing this problem?

On part of the problem is that the `WebView` does not support the CSS query “`prefers-color-scheme`“. This works as intended on iOS however and is a problem specific to Android. You can [refer to this issue](https://github.com/xamarin/Xamarin.Forms/issues/12551) on the `Xamarin.Forms` repository on Github.

#### Workaround

I am not sure if this problem will ever get solved by the `Xamarin.Forms` team. I tried to play around with some Javascript solutions that are floating around the web to keep just one CSS file. In the end however, I went with a `Xamarin.Forms` only approach following the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle).

`Xamarin.Forms` has a working [theme detection mechanism](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/theming/system-theme-changes). Based on the return value of the `Application.Current.RequestedTheme` property, I am loading either the dark mode CSS file or the light mode CSS file (which is default in my case).

Shipping the CSS files is easy, we just need to add them to the Assets folder and set the Build action to `AndroidAsset`. This results in the following structure within the Android project:

![](/assets/img/2021/04/android_webview_asst_structure.png)
All files that are shipped that way are accessible via the `android_asset` file uri:

``` html
 <link rel="stylesheet" href="file:///android_asset/dummycss_light.css">
```
 
Now that everything is set up in the Android project, let’s head over to the `Xamarin.Forms` project. In the sample for this post, I am loading a local html file, while I generate the html dynamically in my blog reader app. The idea works the same way in both cases:

``` csharp
 private async Task SetThemeAndLoadSource()
{
    _html = await LoadHtmlFromFileAsync();

    _html = Application.Current.RequestedTheme == OSAppTheme.Dark ?
            _html.Replace("light", "dark") :
            _html.Replace("dark", "light");

    this.TestWebView.Source = new HtmlWebViewSource() { Html = _html };
}
```
 
As you can see, I just override the light and dark part of the CSS file name I load into the HTML. That’s all the “magic” that needs to happen here. Just one little thing left to add – what if the user changes the theme while the app is running? `Xamarin.Forms` has the solution built in as well – just handle the `RequestedThemeChanged` event and override the file name again, followed by setting the `HtmlWebViewSource` again:

``` csharp
 private async void Current_RequestedThemeChanged(object sender, AppThemeChangedEventArgs e)
{
    await SetThemeAndLoadSource();
}
```
 
#### Conclusion

As most of us are already used to, we sometimes need to find some workarounds when dealing with `Xamarin.Forms`. While this problem could have been solved with a bunch of Javascript and a custom CSS in a `WebViewRenderer` as well (I tried that, but didn’t like the complexity), you can achieve reliable results with the workaround above just in `Xamarin.Forms`.

You can find a working [sample here on Github.](https://github.com/MSicc/XFWebviewDarkmode) As always, I hope this post will be helpful for some of you.

##### Until the next post, happy coding, everyone! 