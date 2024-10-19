---
id: 6565
title: 'WordPressReader (Standard) updated to version 2.1.0'
date: '2020-02-01T07:00:51+01:00'
author: 'Marco Siccardi'
excerpt: 'WordPressReader (Standard) has been updated to version 2.1.0. Read on to learn what''s new in this update.'
layout: post
permalink: /wordpressreader-standard-updated-to-version-2-1-0/
image: /assets/img/2018/03/WP_Hand.jpg
categories:
    - 'Dev Stories'
tags:
    - .netStandard
    - Nuget
    - 'Nuget package'
    - 'REST API'
    - WordPress
    - 'WordPress API'
---

Currently, I am working on an update for my blog reader app on iOS to bring it on par with the Android version (which snagged push notifications for new posts already). As my WordPressReader library sits at the core of the application, I made also some updates to this library.

#### Here is what’s new:

- based on .netStandard 2.1 (still works across UWP, WPF, Xamarin)
- activated nullable reference types
- because of the nullable reference types, updated the entity classes to always include null values (for type safety)
- fixed a bug where comments in response to pingbacks lead to a crash
- renamed `CreateAnonymousComment` to `CreateAnonymousCommentAsync`
- other minor fixes

The update to the app is already [available via Nuget](https://www.nuget.org/packages/WordPressReader/), while you can check the [source code in the Github repo](https://github.com/MSiccDev/WordPressReaderStd).

#### What’s next:

I am currently exploring further improvements using C# 8.0, most notably `IAsyncEnumerable<T>` and `ValueTask`.

If you have any feedback for the library, feel free to sound off in the comments, open a Github issue or contact me via my social media accounts.

##### Until the next post, happy coding, everyone!

Title Image by [Kevin Phillips](https://pixabay.com/users/27707-27707/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=589121) from [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=589121)