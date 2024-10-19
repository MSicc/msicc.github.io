---
id: 6908
title: 'Dealing with the System UI on iOS in Xamarin.Forms'
date: '2021-11-15T22:13:53+01:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I am showing you various aspects of the iOS system UI and how to manage them in your Xamarin.Forms application.'
layout: post
permalink: /dealing-with-the-system-ui-on-ios-in-xamarin-forms/
image: /assets/img/2021/11/DealingWithTheSystemUIoniOSXF.png
categories:
    - 'Dev Stories'
    - iOS
    - Xamarin
tags:
    - 'home indicator'
    - iOS
    - NavigationBar
    - SafeArea
    - StatusBar
    - SystemUI
    - UI
    - xamarin
    - 'xamarin forms'
---

Having written a few applications with `Xamarin.Forms` by now, there was always the one part where you have to go platform specific. Over time, this part got easier as the collection of [Platform-specifics in the Xamarin.Forms package](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/ios/) was growing and growing.

This post will show (my) most used implementations leveraging the comfort of Platform-specifics as well as some other gotchas I collected over time. At the end of this post, you will also find a link to a demo project on my Github.

### Large page title

Let’s start on top (literally). With iOS 11, Apple introduced large title’s that go back to small once the user is scrolling the content.

To make your app use this feature, you need two perform two steps. The first step is to tell your `NavigationPage` instance to prefer large titles. I often do this when creating my apps `MainPage` in `App.xaml.cs`:

``` csharp
 public App()
{
    InitializeComponent();

    var navigationPage = new Xamarin.Forms.NavigationPage(new MainPage())
    {
        BarBackgroundColor = Color.DarkGreen,
        BarTextColor = Color.White
    };

    navigationPage.On<iOS>().SetPrefersLargeTitles(true);

    MainPage = navigationPage; 
}
```
 
This opens the door to show large titles on all pages that are managed by this `NavigationPage` instance. Sometimes, however, you need to actively tell the page it should use the large title (mostly happened to me in my base page implementation – never was able to nail it down to a specific point. I just opted in to always explicitly handle it on every page. In the sample application for this post, you will find a switch to toggle and untoggle the large title on the app’s `MainPage`:

``` csharp
 On<iOS>().SetLargeTitleDisplay(_useLargeTitle ? 
LargeTitleDisplayMode.Always : 
LargeTitleDisplayMode.Never);
```
 
You can read more in the [documentation](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/ios/page-large-title).

### StatusBar text color

Chances are high that we are customizing the `BarBackgroundColor` and `BarTextColor` properties. Of course, it makes absolutely sense that the `StatusBar` text follows the` BarTextColor.` Luckily, there is a Platform-specific for that as well:

``` csharp
 if (this.Parent is Xamarin.Forms.NavigationPage navigationPage)
{
    navigationPage.On<iOS>().SetStatusBarTextColorMode(_statusBarTextFollowNavBarTextColor ? 
                             StatusBarTextColorMode.MatchNavigationBarTextLuminosity : 
                             StatusBarTextColorMode.DoNotAdjust);
}
```
 
The [documentation](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/ios/status-bar-text-color) ends here. However, I always need to add/change the `Info.plist` file as well:

``` csharp
 <key>UIViewControllerBasedStatusBarAppearance</key>
<false/>
```
 
Only after adding this value the above-mentioned trick for the `StatusBar` text works.

### NavigationBar Separator

On iOS, the `NavigationBar` has a separator on its bottom. If you want to hide this separator (which always disturbs the view), you can leverage another [Platform-specific](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/ios/navigation-bar-separator) on your page:

``` csharp
 if (this.Parent is Xamarin.Forms.NavigationPage navigationPage)
{
    navigationPage.On<iOS>().SetHideNavigationBarSeparator(_hideNavBarSeparator);
}
```
 
### Home indicator visibility

All iPhones after the iPhone 8 (except the SE 2) do not have the home button. Instead, they have a home indicator on the bottom of the device (at least in app). If you are trying to set the color on it, I have bad news for you: you can’t (read on to learn why).

You can hide the indicator in your app, however. Just use this [Platform-specific](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/ios/page-home-indicator):

``` csharp
 On<iOS>().SetPrefersHomeIndicatorAutoHidden(_hideHomeIndicator);
```
 
### Home indicator background color

Hiding the home indicator is a hard measure. Most users do not even really recognize the indicator if it is incorporated into the app’s UI. To better understand how the home indicator works, [I absolutely recommend to read Nathan Gitter’s great post on the topic](https://medium.com/@nathangitter/reverse-engineering-the-iphone-x-home-indicator-color-a4c112f84d34).

The home indicator is adaptive to its surroundings. Most probably using a matching background color is all it needs to integrate the indicator nicely in your app(s).

### Safe area

Thanks to the notch and the home indicator, putting content of our apps got trickier than before. However, `Xamarin.Forms` has you covered as well. All we have to do is to use the [SetUseSafeArea Platform-specific](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/ios/page-safe-area-layout) – it will allow us to just use the area where we are not covering any System UI like the `StatusBar` or the home indicator:

``` csharp
 On<iOS>().SetUseSafeArea(_useSafeArea);
```
 
### Conclusion

Even though iOS has some specialties when it comes to the System UI, `Xamarin.Forms` has the most important tools built in to deal with them. I absolutely recommend creating a base page for your applications and set the most common specifics there. You can find the promised demo project [here on Github](https://github.com/MSicc/DealingWithSystemUIoniOSWithXF). Like always, I hope this post is helpful for some of you.

#### Until the next post, happy coding, everyone!