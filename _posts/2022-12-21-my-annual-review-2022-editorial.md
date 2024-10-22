---
id: 7180
title: 'My annual review (2022) [Editorial]'
date: '2022-12-21T17:00:00+01:00'
author: 'Marco Siccardi'
excerpt: 'As I did in the last years, also in 2022, I am reviewing the year from my personal perspective and give a short outlook on what I expect from 2023.'
layout: post
permalink: /my-annual-review-2022-editorial/
image: /assets/img/2022/12/2022_yearinreview_title.png
categories:
    - Editorials
tags:
    - .NET
    - '.NET MAUI'
    - '#CASBAN6'
    - asp.net
    - Azure
    - Dev
    - dotNET
    - editorial
    - outlook
    - programming
    - Review
    - running
    - sport
    - web
    - xamarin
    - 'xamarin forms'
---

In the beginning of 2022, I would never have thought of such a turbulent year. After all, we went more or less over the pandemic, and there were at least some positive signs that were sparking some hope. Pretty fast, the year turned into a memorable one, but for very ugly reasons.

#### Ukraine war

Russia began a war in Ukraine. We all have to battle the consequences of this war. Because of the ongoing war, prices of living have raised in all areas, be it groceries, energy costs and also entertainment costs. No day passes by without getting notified about the horrible things Russian troops are doing in Ukraine. I still pray (and you should too, if you’re into that) for the war to be finally over soon.

#### Twitter take-over

The second impacting event was the take-over of Twitter by Elon Musk. While he made Tesla a profitable company by focusing on the product outcome and made space travel less expensive with SpaceX, he is currently about to destroy Twitter. While I still was somehow neutral back in April when the deal became more real, I lost hope for my favourite social network the day he fired half of the company’s staff.

At the time of the take-over, I was actively working on my app TwistReader, which was a reader app for Twitter lists. I had already a beta running on TestFlight when things began to turn bad on Twitter. After UniShare (which was in the process of being ported to Android and iOS when it died), I had to take the though decision to let go also this app. I cancelled the domain I bought for the app and shut down all Azure resources already. *If someone wants to continue the project, I am open to talk about it.*

![TwistReader promotional image](/assets/img/2022/12/TwistReader_header_temp.png)


This is now the second time I had to stop an app for social media. Ultimately, I decided I will not develop against any of the social networks from now on (even though I have several ideas to improve my social flow).

As we all know, things on Twitter aren’t becoming better. My presence on the bird site serves now solely as a guide to other social media I am active on. I decided to not delete my two main accounts, but to lock them for new followers, and stopped using the service. I am mostly active on [Mastodon](https://mastodon.social/@msicc), followed by [LinkedIn](https://www.linkedin.com/in/msicc/) (although the later one needs some more attention).

#### NASA is flying to the moon again

Besides all the negative stuff, there were also some good news for all of us space fans. [The NASA finally sent a space-ship to the moon again](https://blogs.nasa.gov/artemis/). They are playing the save game and did an unmanned launch, letting the capsule orbit the Moon and come back to Earth. They made some really awesome photos along the way, and the mission was a full success.

![orion space-ship with moon and earth ](/assets/img/2022/12/orion-moon-earth-nasa.jpg)
[Image by NASA](https://images.nasa.gov/details-art001e000678)

#### New blog series #CASBAN6

Besides working on TwistReader, I also started to port [my portfolio website](https://msiccdev.net) away from WordPress to a self written website in ASP.NET Core with Razor pages. The site itself is already published, with links to my apps in the stores, but the news section still needs a blog. I evaluated all the options, like existing CMS plugins and other blogging platforms.

In the end, I opted into learning something new by using some bits of what I already know – and I started my recent [\#CASBAN6]({% post_url 2022-09-05-casban6-creating-a-serverless-blog-on-azure-with-net-6-new-series %}) blog series about creating a serverless blog engine on Azure. This is now my main side project.

#### Other dev stuff

While I am focusing on the serverless blog engine, I also have some libraries I made and use internally for `Xamarin.Forms` that I need to port to `.NET MAUI`. Some parts can be easily removed and replaced with `Essentials` and `CommunityToolkit`. There is still plenty of code worth porting left, though.

At work, I broke up the internally used libraries to be more modular and finished implementing the service templates that use them. I also continued to push source control management within the team. Besides that, I wrote some interfaces for our customers that took advantage of these things, but needed additional items as well. Over all, I was able to use some of my learnings at work and vice versa.

I also decided to not cancel my Parallels subscription. I used it around 10 time throughout the year, which is not worth paying more than 100 bucks for the yearly licence.

Furthermore, I will use the freed budget to buy a [Jetbrains Ultimate](https://www.jetbrains.com/dotnet/) licence instead, which I started to use recently. The experience in writing code is far ahead of what Microsoft offers with Visual Studio on Mac, so I guess that’s a good investment.

### Sports

If you have been following along for some time, you may know that I only became a non-smoker again (after 25 years of chain-smoking) two years ago. In terms of sports, I took part in three challenges this year (Run4Fun 6,8km, 10km at Winterthur marathon and Kyburglauf 2022 10.3 km (including 425 stairs just at the end of km 10). If you want to follow my running adventures, [you can find me here on Strava](https://www.strava.com/athletes/msicc).

![Me running the 10 km at Winterthur Marathon 2022](/assets/img/2022/12/14371731_orig.jpg)

### Outlook into 2023

Next year, the roller coaster continues to ride. I will start a new role in March as a .NET mobile developer at Galliker Switzerland, which is one of the leading companies in logistics. They have a `Xamarin.Forms` code base and started the transition to `.NET MAUI`. There will be projects where I will have to do API and Web stuff as well, so this new position will help me to move towards my goal of becoming a full stack .NET developer as well. Another plus is that I am free to choose my preferred IDE – which will be most probably RIDER after my recent experiences with it.

Of course, I will continue to with my [\#CASBAN6]({{ site.url}}/tags/casban6/) project as well. As I stated in my last post in the series, the Azure functions part is coming up next. I will have some posts on that topic alone, but I will also keep developing it further until the final product is ready to be used in production.

Besides that, I will start to port my Fishing Knots app to `.NET MAUI`, which will help me to learn the upgrade process and make the app ready for the future.

In terms of sports, I will continue with running, starting up with a focus on improving my average pace to get permanently below 5 min/km. On top of that, I want to run a half-marathon at the end of the next season. I will give [runningCoach](https://runningcoach.me/) another try – hopefully they will be able to import my Strava results correctly this time.

### Conclusion

What was your 2022 like? What are you all looking forward in 2023? Feel free to get in contact via my social media accounts or the comments section below.

What’s left is to wish all of you a

### Merry Christmas and a Happy New Year!