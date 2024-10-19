---
id: 3842
title: '[Updated] NFC Toolkit uri schemes'
date: '2014-01-09T03:00:37+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /nfc-toolkit-uri-schemes/
categories:
    - Archive
tags:
    - 'add your app'
    - Dev
    - launchApp
    - launchUriAsync
    - NFC
    - 'nfc toolkit'
    - profiles
    - 'uri association'
    - 'uri scheme'
    - WinPhanDev
    - wpdev
---

![app2app_nfctoolkit](/assets/img/2013/12/app2app_nfctoolkit.png "app2app_nfctoolkit")

If you are following me, you might have noticed that uri schemes actually are playing a big role in my Windows Phone apps. ([read more](http://msicc.net/?p=3834))

NFC Toolkit, one of my main projects, offers now also the possibility to interact with your apps.

One of the unique features of NFC Toolkit is profiles. Profiles are designed to open up to 5 settings pages in a row (as directly setting them is not possible because of OS restrictions). At the end of a profile, an extra from within the app or another app can be launched.

In Phase I, NFC Toolkit enables other developers to add their custom uri association/scheme to the list of launchable apps in profiles.

Here is how the uri needs to be formatted to work:

``` csharp
 "nfctoolkit:addmyapp?appname=<yourappnamehere>&urischeme=<yoururischemehere>"
```
 
I will add more uri associations with the next updates and update this post with them.

**\[Update 2\]**

Starting with version 0.9.8.1 of NFC Toolkit, I added new uri schemes to enable you to write data on NFC tags. Here is the list of possible writing records:

- website record:

``` csharp
 "nfctoolkit:writetag?type=smartposter&url=<url>&title=<title of the website>&languagecode=<two letter lang code (standard: en)>"
```
 
- text record:

``` csharp
 "nfctoolkit:writetag?type=text&content=<your text here>&languagecode=<two letter lang code (standard: en)>"
```
 
- send mail record:

``` csharp
 "nfctoolkit:writetag?type=mail&mailaddress=<mailaddress>&mailsubject=<mailsubject>&mailbody=<mailbody>"
```
 
- settings page:

``` csharp
 //supported pages: flight mode, cellular, WiFi, Bluetooth, location, lock screen
"nfctoolkit:writetag?type=settings&page=<settings page name as listed above>"
```
 
- launch system app:

``` csharp
 //supported pages: 
//"Alarm", "background tasks", "brightness settings", "Bing Vision", "Calculator", "Calendar", "call history", 
//"Camera", "Data Sense", "date + time settings", "ease of access", "find my phone", "Games", "Help+Tips", 
//"Internet Explorer", "internet sharing", "keyboard settings", "kid's corner settings", "language + region settings", 
//"Maps",  "Messaging", "Music+Videos", "Office", "OneNote", "People", "Phone", "phone information", "Photos", 
//"Rooms", "SIM Applications", "Start", "Store", "theme settings", "Wallet"
"nfctoolkit:writetag?type=sysapp&name=<system app name as listed above>"
```
 
- launch any app by name or publisher name:

``` csharp
 //launches the store page search in NFC Toolkit and searches for the app/publisher
"nfctoolkit:writetag?type=launchapp&appname=<app name or publisher name>"
```
 
On top of that, I added an uri scheme that allows a plain launch of NFC Toolkit:

``` csharp
 "nfctoolkit:home"
```
 
I updated my custom uri scheme test app as well for you. Download it here: [CustomUriSchemeTestApp](/assets/img/2013/12/CustomUriSchemeTestApp1.zip)

Happy coding everyone!