---
id: 5523
title: 'WordPressReader (Standard) library is now available on Nuget'
date: '2018-03-17T06:42:36+01:00'
author: 'Marco Siccardi'
excerpt: 'With this post, I am happy to announce my new WordPressReader (Standard) library is now available on Nuget.'
layout: post
permalink: /wordpressreader-standard-library-is-now-available-on-nuget/
image: /assets/img/2018/03/WP_Hand.jpg
categories:
    - 'Dev Stories'
tags:
    - .netStandard
    - library
    - Nuget
    - OpenSource
    - OSS
    - Standard
    - WordPress
    - 'WordPress API'
---

I am happy to announce my new WordPressReader (Standard) library that is available on [NuGet](https://www.nuget.org/packages/WordPressReader/) now. It is written in .netStandard 1.4 which enables you to [use it in a lot of of places](https://docs.microsoft.com/en-us/dotnet/standard/net-standard). As the name suggests, it focuses on reading tasks against the [WordPress API](https://developer.wordpress.org/rest-api/).

Features:

- get, search and filter posts
- get pages
- get and filter categories
- get, search and filter tags
- get comments
- post anonymous comments (needs additional work on the WordPress site)
- get basic user info

The library uses a generic `WordPressEntity `model implementation, which makes it easy to implement and extend. You can always get the raw json-value as well. The additional `Error `property on every model makes it easy to handle API errors properly. You can read the full documentation in the GitHub [Wiki](https://github.com/MSiccDev/WordPressReaderStd/wiki).

If you have problems with the library or want to contribute, you can do so on the [GitHub repo](https://github.com/MSiccDev/WordPressReaderStd).

Happy coding, everyone!

P.S.  
If you want to see the library in a live app, you can download my official blog reader app (which is written around it) here:

[iOS](https://itunes.apple.com/de/app/msiccs-blog/id1359113195) [Android ](https://play.google.com/store/apps/details?id=com.msiccdev.msiccsblog) [Windows 10](https://www.microsoft.com/store/apps/9WZDNCRDPQLK)