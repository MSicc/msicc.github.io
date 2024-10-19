---
id: 1317
title: 'WPDEV on W8CP: How to install the Windows Phone SDK'
date: '2012-03-03T00:04:01+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /wpdev-on-w8cp-how-to-install-the-windows-phone-sdk/
bitly_url:
    - 'http://bit.ly/2qECM4P'
bitly_hash:
    - 2qECM4P
bitly_long_url:
    - 'https://msicc.net/wpdev-on-w8cp-how-to-install-the-windows-phone-sdk/'
categories:
    - Archive
tags:
    - SDK
    - w8cp
    - 'Windows Phone'
    - windows8
    - wpdev
---

Yesterday I was all day playing around with the Windows 8 Consumer Preview (W8CP) as many of us.

[![Screenshot](/assets/img/2012/03/Screenshot1.png "Screenshot")](/assets/img/2012/03/Screenshot1.png)

Today I wanted to get a bit more serious and tried to install the Windows Phone SDK.

Once downloaded from App Hub, I clicked “Install”, and waited for the SDK to complete . Install went trough, and I was waiting for opening my current project.

But then I was shocked by numerous error messages, all of the regarding XNA plugins and updates. So the dream was over. The SDK does not support W8CP.

[![screen](/assets/img/2012/03/screen.png "screen")](/assets/img/2012/03/screen.png)

I am a very Metro-addicted man, so I was searching for a solution. On Twitter I was tipped by [@nikovrdoljak](https://twitter.com/#!/nikovrdoljak) to one part of the solution.

**The errors are caused by Games for Windows – LIVE Redistributable**

> XNA Game Studio installs a version of the Games for Windows – LIVE Redistributable behind the scenes. Some older versions of the Games for Windows – LIVE Redistributable attempt to install and use a file that is being installed by Windows 8, and the older versions of the redistributable are not compatible with the newer version of the file that is installed by Windows 8. Newer versions of the Games for Windows – LIVE Redistributable are compatible with Windows 8, and if you pre-install the new redistributable before installing XNA Game Studio, setup will recognize that it is already there and use the new version instead of trying to install the old version.
> 
> The reason this issue also impacts the Windows Phone SDK 7.1 is that this SDK installs XNA Game Studio behind the scenes, which in turn installs the Games for Windows – LIVE Redistributable behind the scenes.

So where to download the actual version? Let´s check this later. I followed the steps shown on [Aaron Stebner’s WebLog](http://blogs.msdn.com/astebner/default.aspx). But that was not all.

I think more WPDev are willing to try, so here is a checklist:

- uninstall all parts of the Windows Phone SDK – Note: there are some bits left after you use the automatic install, so you have to uninstall remaining parts of the SDK manually
- do a reboot (that thing you do not want to do once you started playing around with W8CP). Don´t worry, it is faster as on Windows 7 (on my 2 year old ASUS under one minute).
- go to the Games for Windows download page, which you can find on this page: [http://www.xbox.com/de-DE/LIVE/PC/DownloadClient](http://www.xbox.com/de-DE/LIVE/PC/DownloadClient "http://www.xbox.com/de-DE/LIVE/PC/DownloadClient"). Replace the “de-DE”-part with your regional language code, for the US p. e. “en-US” to download the correct version of the redistributable.
- start installation
- in the middle of the installation, the UAC asks you to allow the installation. Now click on “change, when this message appears” (the original wording might be slightly different, I translated it from German).
- pull down the switch to set Windows to not ask you anymore (yes, even if this is not recommended)
- now let the installation of Games for Windows –LIVE Redistributable finish.
- finally, start installation of the Windows Phone SDK. This time, it takes a little longer than before (probably due to the now working install parts)
- if you want to, turn the UAC on again (control panel/more settings/users/settings for user access control)

**Congrats, you installed the Windows Phone SDK, but…**

you still have to deal with some points:

- the SDK currently works only with Visual Studio 2010
- the emulator does not work, so you have to debug on device
- the updated Windows Phone SDK 7.1.1 (which is only to have a look, not the final version), might afford additional steps

For me it only worked with above mentioned steps. If you had another steps to do or additional information, leave a comment to discuss.

**happy coding on Windows 8, everyone!**