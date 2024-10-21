---
id: 1486
title: 'Microsoft shows what Windows 8 means for developers at CeBIT'
date: '2012-03-10T22:42:26+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /microsoft-shows-what-windows-8-means-for-developers-at-cebit/
image: /assets/img/2012/03/WP_000425.jpg
categories:
    - Archive
tags:
    - CeBIT
    - developers
    - Microsoft
    - 'Windows 8'
---

[![WP_000425](/assets/img/2012/03/WP_000425.jpg "WP_000425")](/assets/img/2012/03/WP_000425.jpg)

As I told you in my earlier post from CeBIT, I visited also an event for Developers at CeBIT.

Microsoft Developer Evangelist F. Rieseberg and G. Logemann were talking about some important things while developing for Windows 8.

**Components of a modern IT**

First Microsoft told us a few words about the new form factors, which both Microsoft and we Developer have to deal with. This was to show us how important it is to use the Grid. The Grid is a huge tool to accomplish our apps to look on all devices the same, regardless of the screen resolution. The told us some areas which we should not use, as the are reserved for gestures or the back button.

**Windows RunTime (WinRT) and code languages**

[![WP_000427](/assets/img/2012/03/WP_000427.jpg "WP_000427")](/assets/img/2012/03/WP_000427.jpg)

Microsoft explained us how easy it is to develop for the WinRT, using our already existing knowledge of our special languages. It is really easy to understand: Basic of the whole system is the kernel. On top of the kernel you have the WinRT APIs (for example communications, sensors). And with all supported languages you can call these APIs. You can develop Metro styled apps in C/C++, C# or VB (+ XAML), as well as in HTML/CSS or JavaScript.

Microsoft calls the WinRT APIs also the Metro style application APIs. These APIs are easy to understand.

[![WP_000429](/assets/img/2012/03/WP_000429.jpg "WP_000429")](/assets/img/2012/03/WP_000429.jpg)

- First we have the fundamental APIs: Application Services, Threading/Timers, Memory Management, Authentication, Cryptography and Globalization
- On top of that are APIs for Media, Devices as well as Communications &amp; Data
- Finally on top of that all we have the UI

**Desktop apps**

In two sentences: You can develop apps or programs also for the desktop, but without the advantages of the WinRT. You will have to decide if you want a desktop app or a Metro style app.

**Async development!**

Only async. full stop. No, seriously, Microsoft declared they only want async apps. The UI has to be “fast and fluid” at every time the user is in the app. This is relatively easy if you are an .NET/C# developer. You know it already. In Windows 8 we have a simple keyword for it: “await”. Example: calling the FilePicker to hand over an image to the photo app, you will call it with the simple word await in front.

**Process states**

[![WP_000434](/assets/img/2012/03/WP_000434.jpg "WP_000434")](/assets/img/2012/03/WP_000434.jpg)

Similar to Windows Phone, the used memory is controlled by the OS. This means when you app goes to background, you have to save the state of your app. Your apps has 5 seconds to handle all savings while going to suspended mode. While in suspending, your app runs no code.

You can create background tasks, but they need to be handled also for resuming. Imagine a download, you go to another app, return to your app. The background task has to resume while getting the right values from the download.

Another thing similar to Windows Phone: If the system runs to low memory, suspended apps will be terminated without any warning/notification. In this case all unsaved date is lost. It is up to us developers to save all data correctly within the 5 seconds until our app is suspended.

**Q &amp; A**

At the end of the short event Microsoft took some questions. Two of them were interesting:

- Q: Is there a way to share data between a desktop app and a Metro styled app?
- A: Not locally. You have to use the cloud to do so.

- Q: Will some of the features be available also for Windows Phone?
- A: No comment.

**What´s next?**

Microsoft gave out feedback questions. Within this feedback questions we were asked whether we plan to develop Metro styled apps. If so, we should describe what app we are planning to. Microsoft will contact us developers before events which are suitable for our app ideas.

I tried to keep this post as short as possible. If you have any questions, do not hesitate to post a comment below.