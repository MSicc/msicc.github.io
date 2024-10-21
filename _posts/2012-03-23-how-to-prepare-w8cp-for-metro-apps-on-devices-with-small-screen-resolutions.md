---
id: 1383
title: '(Updated x2) How to prepare W8CP for Metro apps on devices with small screen resolutions'
date: '2012-03-23T20:00:17+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-prepare-w8cp-for-metro-apps-on-devices-with-small-screen-resolutions/
categories:
    - Archive
tags:
    - Metro
    - w8cp
    - Windows
    - windows8
---

[![cantopenmetro](/assets/img/2012/03/cantopenmetro.jpg "cantopenmetro")](/assets/img/2012/03/cantopenmetro.jpg)

Microsoft stated that Windows 8 will be able to an huge amount of devices, such as PCs, tablets as well as other devices. Well, a netbook is also a kind of a PC.

There are certain netbooks out there, which can be run in 800\*600 max. screen resolution by default. Currently drivers from vendors are missing for these devices, so there won´t be a possibility to change. A colleague of me has a Nokia 3G booklet. He could not get the Metro part of the W8CP to work. He is running into an error message like the one illustrated above.

**There is a solution.**

Paul Thurrott posted a solution to this on his [supersite for Windows](https://www.winsupersite.com). OK, it seems more than a hack, but it will lead to target.

Here is how to get the Metro part working through changing your screen resolution settings:

- run Regedit (Win + C, search, type in regedit (note: you have to type in complete, otherwise the app shows not up)
- search for “display1\_downscalingsupported” (CTRL+F)
- change its value from 0 to 1
- search all entries in registry by using the F3-key of your netbook, and change again the value from 0 to 1

Paul noted that the look of the desktop part maybe a little bit skewed or squished. But now you have additional screen resolution options.

Note: if you do not know what you’re doing, you should not follow these steps! I am not responsible for any errors or damage caused by changing your registry.

I will try to do this on my colleague´s netbook, and will update this post.

**Update 2: Nokia Booklet loves W8CP!**

So my colleague was playing around with his netbook and installed the device graphics driver. He had do install it with the Windows device manager. He simply downloaded the driver ([here](https://europe.nokia.com/support/product-support/booklet-3g/software "here")), and pointed the device manager to use this file. So he is now enjoying the Metro apps on his netbook.

Until then, feel free to use the steps above and leave some comments. Most important thing:

**Have fun using the Metro apps also on your netbook!**