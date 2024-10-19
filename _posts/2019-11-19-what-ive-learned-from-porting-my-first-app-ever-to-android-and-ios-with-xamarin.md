---
id: 6459
title: 'What I&#8217;ve learned from porting my first app ever to Android and iOS with Xamarin'
date: '2019-11-19T17:00:47+01:00'
author: 'Marco Siccardi'
excerpt: 'I finally finished to port my first app ever from Windows Phone to Android and iOS - Fishing Knots + is finally published in the Google Play Store as well as on the Apple Appstore. This post summarizes what I''ve learned during the process of porting.'
layout: post
permalink: /what-ive-learned-from-porting-my-first-app-ever-to-android-and-ios-with-xamarin/
image: /assets/img/2019/11/porting-fishingkonts-xf-featured.jpg
categories:
    - 'Dev Stories'
    - Xamarin
tags:
    - Android
    - animations
    - app
    - behavior
    - code
    - docs
    - documentation
    - effect
    - iOS
    - native
    - porting
    - renderer
    - search
    - triggers
    - WP7
    - xamarin
    - 'xamarin forms'
---

### What’s the app about?

The app is about fishing knots. It sounds boring for most people, but for me, this app made me becoming a developer. So I have a somewhat emotional connection to it. It was back in time when Windows Phone 7 was new and hot. A new shiny OS from Microsoft, but clearly lacking the loads of apps that were available on iOS and Android. At that time, I also managed to get my fishing license in Germany.

As I had a hard time to remember how to tie fishing knots, I searched the store and found… nothing. I got very angry over that fact, partly because it meant I had to use one of the static websites back then, but more about the fact that there was this damn app gap (WP7 users will remember). So I finally decided to learn how to write code for Windows Phone and wrote my first app ever after some heavy self-studying months.

### Why porting it?

Writing code should soon become my favorite spare-time activity, effectively replacing fishing. And so the years went on, I made some more apps (most of them for Windows Phone) and also managed to become employed as a developer. After some time, S. Nadella became the CEO of Microsoft, and Windows for mobile phones was dead. So I had created all my “babies”, and they were now set to die as Windows Phone/Mobile did. Not accepting this, I started to create a plan to port my apps over to the remaining mobile platforms. After Facebook effectively killed my most successful app (UniShare – that’s another story, though), I stopped porting that one and started with Fishing Knots +.

### Reading your own old code (may) hurt

When I was starting to analyze which parts of the code I could reuse, I was kind of shocked. Of course, I knew that there was this code that I wrote when I didn’t know it better, but I refused to have a look into it and refactor it (for obvious reasons from today’s perspective). I violated a lot of best practices back then, the most prominent ones

- No MVVM
- Repeating myself over and over again
- Monster methods with more than 100 lines within

<span style="text-decoration: underline;">In the end, I did the only right thing and did not reuse any line of my old code.</span>

### Reusing the concept without reusing old code

After I took the right decision to not use my old codebase, I needed to abstract the concept from the old app and translate it into what I know <span style="text-decoration: underline;">*now*</span> about best practices and MVVM. I did not immediately start with the implementation, however.

The first thing I did was drawing the concept on a piece of paper. I used a no-code language in that sketch and asked my family if they understand the idea behind the app (you could also ask your non-tech friends). This approach helped me to identify the top 3 features of the app:

- Controllable animation of each knot
- Easy-to-follow 3-step instructions for each knot
- Read-Aloud function of the instructions

Having defined the so-called “*Minimum Viable Product*“, I was ready to think about the implementation(s).

### The new implementation

Finding the right implementation isn’t always straight forward. The first thing I wrote was the custom control that powers the controllable animation behind the scenes. I wrote it out of the context in a separated solution as I packed it into a NuGet package after finishing. It turned out to be also the most complex part of the whole app. It uses a common API in Xamarin.Forms, and custom renderers for Android and iOS. I had to go that route because of performance reasons – which is one of the learnings I took away from the porting.

It was also clear that I will use the MVVM pattern in the new version. So I was setting up some basic things using my own Nuget packages that I wrote during working on other Xamarin based projects.

When it came to the overall structure of my app, I thought a Master/Detail implementation would be fine. However, somehow this never felt right, and so I turned to Shell (which was pretty new, so I tried to give it a shot). In the end, I went with a more custom approach. The app uses a TabbedPage with 3 tabs, one being for the animation, the second for the 3-Step tutorial, and last but not least the Settings/About page. The first two pages share a custom top-down menu implementation, bound to the same ViewModel for its items and selection.

### More Xamarin.Forms features I learned (to love)

Xamarin and Xamarin.Forms itself are already powerful and have matured a lot since the time I used it to write my first Xamarin app for Telefonicá Germany. Here is a (high level) list of features I started to use:

- **Xamarin.Essentials** – the one library that kickstarts your application – seriously!
- **Xamarin Forms Animations** – polish the appearance of your app with some nice looking visual activity within the UI
- **Xamarin Forms Effects** – easily modify/enhance existing controls without creating a full-blown custom renderer
- **Xamarin Forms VisualStateManager** – makes it (sometimes) a whole lot easier to change the UI based on property changes
- **Xamarin.Forms Triggers** – alternative approach to modify the UI based on property changes (but not limited to that)

### The three musketeers

Because of Xamarin and Xamarin.Forms are such powerful tools, you may run into the situation of needing help/more information. My three musketeers to get missing information, implementation help or solution ideas:

- **Microsoft Xamarin Docs** – the docs for Xamarin are pretty extensive and by reading them (even again), I often had one of these “gotcha!”- moments
- **Github** – if the docs don’t help, Github may. Be it in the issues of Xamarin(.Forms) or studying the renderers, Github has been as helpful as the docs to me.
- **Web Search** – chances are high that someone had similar problems/ideas solved and wrote a blog about it. I don’t blindly copy those solutions. <span style="text-decoration: underline;">*First I read them, then I try to understand them and finally, I implement my own abstraction of them*</span>. This way, I am in a steady learning process.

### Learn to understand native implementations

I guarantee you will run into a situation where the musketeers do not help when focusing solely on Xamarin. Accept the situation that Xamarin is sitting on top of the native code of others and does the heavy conversion for us. Learn to read Objective-C, Swift, Java and Kotlin code and translate it into C# code. Once you found possible solutions in one of the native samples, blog posts or docs, you will see that most of them are *easy* to translate into Xamarin code. Do not avoid this part of Xamarin development, it will help you in future, trust me.

### Conclusion

Porting over my first app ever to Android and iOS has provided me not only a lot of fun but also huge learnings/practicing. Some of them are of behavioral nature, some of them are code implementations. This post is about the behavioral part – I will write about some of the implementations in my upcoming blog posts.

I hope you enjoyed reading this post. If you have questions or have similar experiences and want to discuss, feel free to leave a comment on this post or connect to me via social media.

#### Until the next post, happy coding!

### Helpful links:

- <https://docs.microsoft.com/en-us/xamarin/>
- <https://github.com/xamarin>
- <https://docs.microsoft.com/en-us/xamarin/essentials/>
- <https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/effects/>
- <https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/animation/>
- <https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/visual-state-manager>
- <https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/triggers>
- <https://developer.android.com/docs/>
- <https://developer.apple.com/documentation>