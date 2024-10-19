---
id: 3408
title: 'Dev Story Series (Part 3 of many): Why I use a WebBrowser/WebView to display WordPress post content'
date: '2013-01-13T00:40:24+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /dev-stories-series-part-3-of-many-why-i-use-a-webbrowserwebview-to-display-wordpress-post-content/
categories:
    - Archive
tags:
    - apps
    - CSS
    - Dev
    - HTML
    - json
    - webbrowser
    - webview
    - win8dev
    - WordPress
    - wpdev
    - XAML
---

When it comes to display the post content on a blog reader app, it starts to become a bit challenging. The post content is formatted to look great on your website. But when we pull our posts into an app, there is only the naked, HTML formatted string.

As developer, you have to think about several things now:

- What part of the content do I want to be displayed?
- How do I get the images there?
- What if there is a video in the post?
- Where do I put the Links in?
- How can I handle enumerations?
- and so on…

There is the HTMLAgilityPack out there, but I never got a satisfying result out of it. The next method would be to write a custom parser. This is what I have done before, in the old version of the app for my WordPress blog. It did work, but I had to invest a real big amount of time in it before I got a result that I was able to live with. I was also not too experienced with RegEx (and I am still not) that I could set up a perfect parser.

When I was creating the Windows 8 version of my app, I wanted to achieve a good reading experience. On the other side I wanted the code to be as reliable as possible, because there are often changes on WordPress that can have impact on my app.

As I mentioned above, the post content is already formatted. It is formatted in HTML. I decided to render the content string instead of parsing it.

It is pretty easy to do that. Just pass the content string to your desired details page, and use a WebBrowser on Windows Phone or a WebView on Windows 8. Without any parsing, just by “navigating” to the passed string, we will get a result like this:

![image](/assets/img/2013/01/image1.png "image")

So we have already a readable result, and if my app has only white background, I could leave it like it is and go on. Without any additional line of code.

Using the WebBrowser/WebView brings also additional advantages:

- Pinch-to-Zoom support
- Orientation support
- automatic image downloading without any additional control
- WebView on Windows 8 embeds videos automatically

I don’t want to hide that there are a few points that we need to handle, which will be subject of additional posts:

- styling of content to match our app colors
- Navigation to links (including a solution for video links on Windows Phone)
- Scroll direction in Windows 8 WebView

I know it might be not the best practice for displaying web content, but I am really satisfied what I achieved by using the WebBrowser and WebView element in my apps.

I hope the upcoming blog posts will be helpful for some of you to create also a good user experience by using these elements. Of course these posts will contain some code. Before starting the posts about it I just wanted to share why I used these elements.