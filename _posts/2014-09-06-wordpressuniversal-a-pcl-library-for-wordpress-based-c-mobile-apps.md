---
id: 4140
title: 'WordPressUniversal &#8211; a PCL library for WordPress based C# mobile apps'
date: '2014-09-06T07:33:38+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /wordpressuniversal-a-pcl-library-for-wordpress-based-c-mobile-apps/
categories:
    - Archive
tags:
    - 'C#'
    - library
    - 'mobile apps'
    - 'news reader'
    - 'Windows 8'
    - 'Windows Phone'
    - WordPress
    - WP8
    - WP81
    - xamarin
---

![WP_CSharp_Lib](/assets/img/2014/09/WP_CSharp_Lib.png "WP_CSharp_Lib")

As I have already developed a news reader app for my blog, I got asked quite a few times if I want to share my code to help others creating their apps. As I need to redesign my blog reader to match the latest OS features of Windows and Windows Phone, I decided to create my WordPressUniversal library.

Automattic, the company behind WordPress, integrated a powerful JSON API into their [JetPack](http://jetpack.me/) plugin. My library is based on this API. Please make sure the JSON API is active on the blog you are developing your app for.

The library currently provides the following features:

- getting a list posts or pages
- getting a list of all categories
- getting a list of comments for the site and single posts
- supports Windows Phone 8 &amp; 8.1 Silverlight, Windows Phone 8.1 RT, Windows 8, Xamarin.iOS and Xamarin.Android

The library deserializes all data into C# objects you can immediately work with.

It is still a work in progress, but as it provides already a bunch of options to get you started, I am making this public already.

I am constantly working on the library, so be sure to follow the project along.

Note: JetPack’s JSON API does not support guest commenting at the moment. I already reached out to Automattic to (hopefully) find a solution for this. If you cannot wait, [Disqus](https://help.disqus.com/customer/portal/articles/1222036-windows-phone-sdk-pre-release-) has a portable solution for Windows Phone apps.

Please make sure to see the documentation [on my GitHub page](http://bit.ly/WordPressUniversal).

If you have any questions, idea, wishes for me, just post a comment here or ping on Twitter.

Happy coding everyone!