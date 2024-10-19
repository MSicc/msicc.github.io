---
id: 5394
title: '[Updated] A workaround for Xamarin Forms 2.5 bug that prevents resource declaration in App.xaml'
date: '2017-12-14T09:42:41+01:00'
author: 'Marco Siccardi'
excerpt: 'Xamarin Forms 2.5 introduced a bug that prevents resource declaration in App.xaml like many of us are used to. This post shows a possible workaround.'
layout: post
permalink: /a-workaround-for-xamarin-forms-2-5-bug-that-prevents-resource-declaration-in-app-xaml/
image: /assets/img/2017/12/ex_resourcedic_xf25b.png
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - app
    - Bug
    - Bugzilla
    - dictionary
    - ResourceDictionary
    - resources
    - workaround
    - xamarin
    - 'xamarin forms'
    - XAML
    - XF2.5
---

Update: Xamarin appearently solved this problem with [Service Release 3 for Xamarin Forms 2.5](https://developer.xamarin.com/releases/xamarin-forms/xamarin-forms-2.5/2.5.0-sr3/). I can confirm it works in the app that caused me to write this post.

Additional note: the `forms:`prefix is no longer needed, just insert the `<ResourceDictionary>`tag.

---

If you have a Windows background like I do, one of the most normal things for applications is to create keyed Resources in App.xaml to make them available throughout the app. Something like this should look familiar:

``` xml
 <forms:ResourceDictionary >
    <viewModels:ViewModelLocator x:Key="Locator"></viewModels:ViewModelLocator>
    <forms:Color x:Key="MainAccentColor">#1e73be</forms:Color>
    <forms:Color x:Key="LightAccentColor">#61a1f1</forms:Color>
    <forms:Color x:Key="DarkAccentColor">#00488d</forms:Color>
    <forms:Color x:Key="MainBackgroundColor">#f4f4f4</forms:Color>
</forms:ResourceDictionary>
```
 
This is also possible in Xamarin.Forms. Sadly, Xamarin.Forms 2.5 introduced an ugly bug where this declarations throw an ArgumentException, telling us the key(s) already exist in the dictionary ([see Bugzilla here](https://bugzilla.xamarin.com/show_bug.cgi?id=60788)). I can confirm that this bug affects at least UWP, Android and iOS applications which use such an implementation.

As this is a show-stopping bug, I had to find a way to work around it for the moment. In such cases, I always try to find a way that has only very little impact. For this particular bug, I just moved the declaration of the resources into the code-behind file, which keeps the rest of my code unchanged. I just created a method that does the work I originally had in the .xaml-file:

``` csharp
 //needed because of Xamarin Bug  https://bugzilla.xamarin.com/show_bug.cgi?id=60788
private void CreateResourceDictionary()
{
    //making sure there is only one dictionary
    if (this.Resources == null)
        this.Resources = new ResourceDictionary();

    //making sure there is only one key
    if (!this.Resources.ContainsKey("Locator"))
    {
        this.Resources.Add("Locator", ViewModels.ViewModelLocator.Instance);
    }

    if (!this.Resources.ContainsKey("MainAccentColor"))
    {
        this.Resources.Add("MainAccentColor", Color.FromHex("#1e73be"));
    }

    if (!this.Resources.ContainsKey("LightAccentColor"))
    {
        this.Resources.Add("LightAccentColor", Color.FromHex("#61a1f1"));
    }

    if (!this.Resources.ContainsKey("DarkAccentColor"))
    {
        this.Resources.Add("DarkAccentColor", Color.FromHex("#00488d"));
    }

    if (!this.Resources.ContainsKey("MainBackgroundColor"))
    {
        this.Resources.Add("MainBackgroundColor", Color.FromHex("#f4f4f4"));
    }
}
```
 
This makes the application running again like it did before. Once the bug in Xamarin.Forms is fixed, I just have to delete this method and uncomment the XAML-declarations to get back to the state where I was prior to Xamarin.Forms 2.5.

If you are experiencing the same bug, I recommend to also comment on the Bugzilla-Entry ([link](https://bugzilla.xamarin.com/show_bug.cgi?id=60788)).

As always, I hope this post is helpful for some of you.

Happy coding!