---
id: 5578
title: 'WordPressReader (Standard) version 1.2.0 focuses on improving performance'
date: '2018-04-22T11:28:45+02:00'
author: 'Marco Siccardi'
excerpt: 'In the last few days, I was working on some improvements for my WordPressReader (Standard) library. I am happy to announce that an update to version 1.2.0 is now available.'
layout: post
permalink: /wordpressreader-standard-version-1-2-0-focuses-on-improving-performance/
image: /assets/img/2018/03/WP_Hand.jpg
categories:
    - 'Dev Stories'
tags:
    - .netStandard
    - library
    - Nuget
    - Standard
    - WordPress
    - 'WordPress API'
---

The update contains several improvements to the `HttpClient`implementation, which should improve performance of the library a lot:

- all handlers now use one static `HttpClient`instance. `HttpClient`is built to use it that way, and there is also no problem with multiple request handled by that instance. So everyone should use it in that way
- the library is now actively enforcing gzip/deflate compression to make responses faster, especially when requesting bigger lists of items (if you want do deactivate it on the managed implementation, pass in `false`with the new `useCompression`parameter with your call to the `SetupClient`method
- deserialization by default now happens directly from the `HttpResponse`stream instead of first converting it to a string
- added an API to pass in your own `HttpClient`instance instead of the managed one (you could even use one static instance for your whole app this way to improve performance even further). To do so, just use the newly added method `SetCustomClient(HttpClient client)`on your handler instances.

I made this changes working without breaking existing implementations. Just by updating the library, you will get a better performance.

I also updated the source code on [GitHub](https://github.com/MSiccDev/WordPressReaderStd). If you just want to update the library, the update is already available on [NuGet](https://www.nuget.org/packages/WordPressReader/).

Happy coding, everyone!

P.S.  
If you want to see the library in a live app, you can download my official blog reader app (which is written around it as a `Xamarin.Forms`app) here:

[iOS](https://itunes.apple.com/de/app/msiccs-blog/id1359113195) [Android ](https://play.google.com/store/apps/details?id=com.msiccdev.msiccsblog) [Windows 10](https://www.microsoft.com/store/apps/9WZDNCRDPQLK)