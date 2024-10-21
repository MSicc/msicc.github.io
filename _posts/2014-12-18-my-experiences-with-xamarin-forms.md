---
id: 4269
title: 'My experiences with Xamarin.Forms'
date: '2014-12-18T05:04:23+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /my-experiences-with-xamarin-forms/
categories:
    - 'Dev Stories'
    - Xamarin
tags:
    - 'faster build'
    - forms
    - gotchas
    - UI
    - unified
    - xamarin
    - 'xamarin forms'
    - XAML
---

[![Xamarin-logo-hexagon-blue](/assets/img/2014/12/Xamarin-logo-hexagon-blue.png)
](/assets/img/2014/12/Xamarin-logo-hexagon-blue.png)


As I have finished my first iOS app with Xamarin.Forms, I want to share my experience that I made during writing it.

It sounds great. Build the code once, run it on Android, iOS and Windows Phone (8). Xamarin is using the well known PCL to achieve this goal, or a shared asset project.

As I am familiar with the PCL structure, I decided to go with this one. [The application I wrote for Telefónica](https://apps.msicc.net/friends-you-telefonica-germany/) had already their Windows Phone and Android counterpart. My thought was to bring together all three after finishing the iOS app into the Xamarin.Forms project to make it easier to maintain them (that was before it was clear that I would leave, but that’s another story). In the end, I focused on the iOS platform and implementation, leaving the other two out.

It was far easier to start a new iOS app with Xamarin.Forms than in the traditional way. Although there are some XAML gotchas ([like Nicolò wrote already on his blog](https://blog.tpcware.com/2014/09/xamarin-xaml-vs-microsoft-xaml-the-devil-is-in-the-details/)), it is pretty easy to get started with it.

The number one tip I can give you is to wrap everything in a principal Grid and set you ColumnWidth (also if you have only one single Column). This will help you to better position your controls on the page.

One really annoying thing is the missing IntelliSense support when you’re writing your XAML code. What does that mean? It means your will spend a lot of time with trial and error as well as reading the [documentation](https://iosapi.xamarin.com/?link=N%3aXamarin.Forms) in the beginning.

One thing that is solved in a good way is the access to native functions that are not implemented in the Forms project. Connecting through interfaces and Xamarin’s DependencyService, you can write the implementation you need in the native project and call the function from the Forms PCL. I will cover this in another blog post.

Often, you want/need your app to be designed in a different way (like I had to for Telefónica). Some basic modifications are possible from the XAML part. But the most effective way to achieve this goal for the whole app is to use Custom Renderer. This will be another post’s topic in the coming days.

Overall, Xamarin.Forms is already impressive. But you need to know that you will work with some workarounds when you start. If you are willing to do this, you might be able write a cross platform app in little time.

If you do not want to dig into the documentation or use the techniques I wrote about, Xamarin.Forms might not yet be your starting point for your cross platform app.

One last tip: To make it easier for you, there is the [Xamarin.Forms Lab](https://github.com/MSiccDev/Xamarin-Forms-Labs) project. This community project has already extended Xamarin.Forms, and is worth a look and a second thought if you truly want to do a cross platform app with Xamarin.

Happy coding, everyone!