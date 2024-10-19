---
id: 6605
title: 'MSicc&#8217;s Blog version 1.6.0 out now for Android and iOS'
date: '2020-02-14T10:00:00+01:00'
author: 'Marco Siccardi'
excerpt: 'Besides updating my WordPressReader library and making some minor visual changes to my blog, I also updated both the Android and iOS apps for my blog.'
layout: post
permalink: /msiccs-blog-version-1-6-0-out-now-for-android-and-ios/
image: /assets/img/2020/02/msiccsblog_v16_blog_title.jpg
categories:
    - Android
    - Azure
    - 'Dev Stories'
    - iOS
    - Xamarin
tags:
    - Android
    - app
    - Azure
    - iOS
    - notification
    - 'notification hub'
    - push
    - reference
    - xamarin
---

Here are the new features:

##### Push Notifications 

With version 1.6.0 of the app, you can opt-in to receive push notifications once I publish a new blog post. I use an Azure Function (v1, for the ease of bindings – at least for now), and of course, an Azure NotificationHub. The Function gets called from a WebHook (via a plugin on WordPress), which triggers it to run (the next blog posts I write will be about how I achieved this, btw.)

##### New Design using Xamarin.Forms Shell

I also overhauled the design of the application. Initially, it was a MasterDetail app, but I never felt happy with that. Using Xamarin.Forms.Shell, I optimized the app to only show the last 30 posts I wrote. If you need older articles, you’ll be able to search for them within the app. The new design is a “v1” and will be constantly improved along with new features.

##### Bugs fixed in this release

- fixed a bug where code snippets were not correctly displayed
- fixed a bug where the app did not refresh posts after cleaning the cache
- other minor fixes and improvements

I hope some of you will use the application and give me some feedback.

*You can download the app using these links:*  
[iOS ](https://apps.apple.com/app/id1359113195) | [Android](https://play.google.com/store/apps/details?id=com.msiccdev.msiccsblog)

#### Until the next post, happy coding, everyone!