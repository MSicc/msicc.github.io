---
id: 3470
title: 'Application ID&#8217;s of built in Windows Phone 8 apps'
date: '2013-01-30T12:41:16+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /application-ids-of-built-in-windows-phone-8-apps/
categories:
    - Archive
tags:
    - AppId
    - 'built in'
    - launchApp
    - NFC
    - 'Windows Phone'
    - 'Windows Phone 8'
    - WP8
    - wp8dev
    - wpdev
---

As you may have noticed, I am currently working on an NFC app. Development goes pretty well at the moment, thanks to the absolutely awesome and easy to use [NDEF library](http://ndef.codeplex.com/) by Andreas Jakl.

If you want to open apps from your app or from an NFC tag, you will need to use the AppId of the desired app. If you have an installed app from the Windows Phone Store, this is pretty easy. You can go to the application list on your phone, long tap and hit “send”. If you now choose mail or SMS, you can obtain the AppId very easy, as it is the last part behind “appId=” on the web address.

With the built in apps, it is a bit more difficult. Luckily, the app [NFC interactor for Windows Phone 8](http://www.windowsphone.com/s?appid=4e1598fe-4885-4e2b-9c69-8d3f882c545b), which is aimed at developers, has a solution. The app is written also by Andreas Jakl, who provides a huge tool with this app to support you on developing your own app and it is worth every cent.

I made it through all records for built in apps and extracted the following list, which might come handy for some of you:

- Alarms AppId 5B04B775-356B-4AA0-AAF8-6491FFEA560A
- Bing Scan AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5682
- Calculator AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5603
- Calendar AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5612
- Camera AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5631
- Data Sense AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5646
- Games AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5634
- Help+Tips AppId E05410F1-753B-47BC-B101-226E5802B9E1
- IE AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5660
- Maps AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5661
- Messaging AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5610
- Music+Videos AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5630
- Office AppId 5B04B775-356B-4AA0-AAF8-6491FFEA561E
- OneNote AppId 5B04B775-356B-4AA0-AAF8-6491FFEA561B
- People AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5615
- Phone AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5611
- Photos AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5632
- Rooms AppId 5B04B775-356B-4AA0-AAF8-6491FFEA562D
- SIM Applications AppId 5B04B775-356B-4AA0-AAF8-6491FFEA562C
- Start (Home Screen) AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5602
- Store AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5633
- Wallet AppId 5B04B775-356B-4AA0-AAF8-6491FFEA5683

All credits for this App IDs goes to Andreas Jakl, I only put them together as a list to find them more easily.