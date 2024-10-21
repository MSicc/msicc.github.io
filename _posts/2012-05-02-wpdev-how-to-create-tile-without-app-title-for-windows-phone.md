---
id: 2376
title: '[WPDev] How-to create a tile without app title for Windows Phone'
date: '2012-05-02T21:06:07+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /wpdev-how-to-create-tile-without-app-title-for-windows-phone/
categories:
    - Archive
tags:
    - 'Windows Phone'
    - wpdev
---

Every App has always a title on its dedicated tile for the Windows Phone start screen. But what if your tile icon contains already the title of your app?

Normally, you are setting the appearance of your app within the properties:

[![properties](/assets/img/2012/05/twT2.jpg "properties")](/assets/img/2012/05/twT2.jpg)

If you have done this, your logo is displayed, but contains the title string. So how can we delete this?

In properties, you are not allowed to delete the name. You will face an error: “Title cannot be empty”.

You need to go to the Application Manifest and edit the XML manually. Go to solution explorer, and open the properties tree. With a double click on “WMAppManifest.xml” you will see the following Window:

[![wpappmanifest](/assets/img/2012/05/wpappmanifest.jpg "wpappmanifest")](/assets/img/2012/05/wpappmanifest.jpg)

To edit the title of the start screen, search for the <tokens> section. Within this section, you will find a Title section, which contains the title of your app (as you have set within the properties). Now simply delete the title string.</tokens>

[![TwTTile](/assets/img/2012/05/TwT3.png "TwTTile")](/assets/img/2012/05/TwT3.png)


If you now debug your application, you will see that your tile no longer displays the title.

I hope this short how-to will be helpful for you.