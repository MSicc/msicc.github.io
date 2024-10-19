---
id: 3151
title: '[WPDEV] How a typo helped me to force SystemTray.Foreground to be White'
date: '2012-12-14T18:06:30+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /wpdev-how-a-typo-helped-me-to-force-systemtray-foreground-to-be-white/
image: /assets/img/2012/12/SystemTrayLightwithRGBerror.jpg
categories:
    - Archive
tags:
    - SystemTray
    - 'Windows Phone'
    - wpdev
---

Today was a sad day. My app was not going through Windows Phone Store certification because of logo issues., so I needed to change that all. That is bad on one side, because it really gets difficult to get the app approved now before the holidays.

But on the other side it was good, as I found some theming issues which I was able to iron out. Well, at least nearly all of them.

One theming problem was remaining, no matter what I tried.

My app has its own theme, the ForegroundColor is White. As I use the SystemTray to display the ProgressBar while loading data from the internet, I want it to be white, too (of course).

![](/assets/img/2012/12/SystemTrayLightwithRGBerror-300x212.jpg "SystemTrayLightwithRGBerror")

All was running good until I switched the phone’s background to light. The SystemTray was always black.

I tried several things, for example set in XAML:

```
  shell:SystemTray.ForegroundColor="White"
```
 

Next I tried to add it in code behind:

```
 SystemTray.SetForegroundColor(this, Colors.White);
```
 

But my app seemed to ignore the word “White” completely. No error showed up, it just did not work.

Finally, I tried to set it via the RGB code for white (255,255,255), but I made a typo in there:

```
  SystemTray.SetForegroundColor(this, Color.FromArgb(255, 254, 255, 255));
```
 

And the most funny thing is, it just worked!

![](/assets/img/2012/12/SystemTrayLightwithRGBcorrect-300x211.jpg "SystemTrayLightwithRGBcorrect")

Summary: Windows Phone seems to not like “White” in SystemTray. If you want to force it, just use the code above, and it will work!

I hope my typo will be helpful for some of you.

Happy coding!