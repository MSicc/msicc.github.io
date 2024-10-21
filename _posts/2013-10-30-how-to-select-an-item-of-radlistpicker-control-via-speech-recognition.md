---
id: 3795
title: 'How to select an item of (Rad)ListPicker control via speech recognition'
date: '2013-10-30T19:37:11+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-select-an-item-of-radlistpicker-control-via-speech-recognition/
categories:
    - Archive
tags:
    - ListPicker
    - RadListPicker
    - recognition
    - speech
    - SpeechRecognizer
    - voice
    - winphand
    - WP8
    - wp8dev
    - wpdev
---

This one is a short follow up to my [post from yesterday about speech recognition]({% post_url 2013-10-29-make-your-app-listening-to-the-users-voice %}) in Windows Phone 8 apps.

I made the first steps with speech recognition in the last few days with speech recognition, coming to the point where I had to select Items from a (Rad)ListPicker control (tried both, the Windows Phone Toolkit one as well as the Telerik one).

I was then realizing that calling the SelectedItem or the SelectedIndex does not work. By my app’s design the ListPicker has the IsExpanded property set to true, so there was one problem: in this state, it accepts only touch input.

Today I found a quick solution for this problem. If you rely on the input of the ListPicker control, just make sure that its IsExpanded state is false. You then will be able to set the SelectedItem or the SelectedIndex via code.

As always, I hope this will be helpful for some of you.

Happy coding!