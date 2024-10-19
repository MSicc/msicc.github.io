---
id: 2743
title: 'WPDEV: 2 tools that make localization of your Windows Phone apps easy'
date: '2012-08-10T10:36:51+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /wpdev-2-tools-thatmake-localization-of-your-windows-phone-apps-easy/
image: /assets/img/2012/08/AppTitleLocalizer_thumb-640x372.png
categories:
    - Archive
tags:
    - app
    - localization
    - 'Windows Phone'
    - wpdev
---

Today I want to show you two small tools that will help you to make localizing your apps easier.

**Localizing in-app strings**

The first tool I want you to know about is a tool called “AppTranslator”, made by Xda-Member [singularity0821](http://forum.xda-developers.com/member.php?u=3916780 "singularity0821").

To provide the language settings, you need to name the strings in your app. I recommend to use format “x:name”, as sometimes without the “x:” your strings will not be accessible. Then you have to create a .resx file for your “neutral” language. You can find a good tutorial here at [MSDN](http://msdn.microsoft.com/en-us/library/ff637520(v=vs.92)).

Once you have done this, you have to translate all strings and put them in a separate .resx file. Doing this manually in Visual Studio can really be an awful job. This is where the “AppTranslator” comes into the game.

![apptranslator_1](/assets/img/2012/08/apptranslator_1_thumb.png "apptranslator_1")

As you can see it has a very clean UI. Everything you now have to do is to load your .resx file into the app and start to translate your app really fast:

![apptranslator_2](/assets/img/2012/08/apptranslator_2_thumb.png "apptranslator_2")

You can save your work at every point, in the end you will get a ready to paste in .resx file.

You can download the tool here at [xda](http://forum.xda-developers.com/showthread.php?t=1374036).

**Localizing app title**

Ok, now you have localized your app content. But the app title will remain the one that you set in you “neutral language”.

To localize your app title, you have to generate resource-only DLLs. This could be very difficult if you are developing your apps with the VS 2010 Express. You simply can´t do it. An overview of how to do it manually and an explanation can be found here at [MSDN](http://msdn.microsoft.com/en-us/library/ff967550(v=vs.92)).

No need to scream now, as also for this exists a tool from Patrick Getzmann, a German MVP.

![AppTitleLocalizer](/assets/img/2012/08/AppTitleLocalizer_thumb.png "AppTitleLocalizer")

The tools uses Bing to translate your app title. You can edit all the strings also manually, if you want. Once you´re done, just hit “Save DLLs” and you are ready to integrate them into your app.

The tool can be downloaded [here](http://patrickgetzmann.wordpress.com/wp7-localize/).

I hope you will enjoy the tools as much as I do.