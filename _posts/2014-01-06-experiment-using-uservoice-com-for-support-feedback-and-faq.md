---
id: 3916
title: 'Experiment: using uservoice.com for Support, Feedback and FAQ'
date: '2014-01-06T19:19:18+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /experiment-using-uservoice-com-for-support-feedback-and-faq/
categories:
    - Archive
tags:
    - customer
    - FAQ
    - feedback
    - 'knowledge base'
    - satisfaction
    - support
    - tickets
    - 'Windows Phone'
    - WinPhanDev
    - WP8
    - wpdev
---

On my daily job, I am working as a Mobile Hardware Support Agent of a German carrier. Because of that, I know how important a good support for customers (=users) is – and also, a little knowledge base where users can find most probably an answer for their questions.

I want my customers to be satisfied. As an indie developer, I have a little problem: I don’t have a support team that catches all the support questions for me. I need to do this by myself. Until now, I am doing this by using a send-me-a-mail button in my apps. However, due to the fact that I have a 9to5 job and a family besides developing apps, time is a bit limited for me. I searched for some solutions to improve the way I am doing my customer service, and [uservoice](https://uservoice.com/) seems like a good tool to improve my customer service a lot.

As you can see at the feature comparison list of uservoice, the free account already allows you to do a lot of things: [https://www.uservoice.com/plans/?ref=top\_pricing](https://www.uservoice.com/plans/?ref=top_pricing "https://www.uservoice.com/plans/?ref=top_pricing"). It is free for one support agent – perfect for an indie developer like me.

### What can you do with uservoice?

uservoice has a lot of features:

- support ticket system
- knowledge base
- feedback system/forum
- and many more, if you are going for a paid plan

Let’s have a look at the first three features:

## Support ticket system

Adding the ticketing system is very easy. You just need a EMailComposeTask pointing to your ticket system. Your standard mail address for that will be: “tickets@&lt;yourname&gt;.uservoice.com”. This is how a received ticket looks like:

![Screenshot (289)](/assets/img/2014/01/Screenshot-289.png "Screenshot (289)")

Answering to a user is as easy as writing an email – plus you have a ticket history at a glance.

On top of that, you are able to insert a so called ‘canned response’ based on your knowledge base:

![Screenshot (290)](/assets/img/2014/01/Screenshot-290.png "Screenshot (290)")

## Knowledge base

You can build up a knowledge base for your apps.

![Screenshot (286)](/assets/img/2014/01/Screenshot-286.png "Screenshot (286)")

In the left menu at the bottom at your admin console, you will find the ‘ARTICLES’ section. This is where all the knowledge base articles are collected.

You can create also topics within that KB. I did so for all of my apps:

![Screenshot (286)](/assets/img/2014/01/Screenshot-2861.png "Screenshot (286)")

This way, you can use your free account to support all of your apps with FAQ/KB articles – pretty easy.

Here is how to set up a new article: after clicking on ‘All articles’, there is a ‘New Article’ button on the right side of your screen. Click on that, you will see this on the right hand side:

![Screenshot (287)](/assets/img/2014/01/Screenshot-287.png "Screenshot (287)")

You can set up the article name, the text and the topic as well as the position you want to show it up in your knowledge base.

You can take a look at my knowledge base here: [https://msiccdev.uservoice.com/knowledgebase](https://msiccdev.uservoice.com/knowledgebase "https://msiccdev.uservoice.com/knowledgebase")

How can you use this in your apps? I plan to create an API wrapper for their C# SDK, which is not compatible with Windows Phone at the moment. I will write another blog post for that. Luckily, uservoice as a very nice mobile interface, so you can start off with a simple WebBrowserTask to open your knowledge base:

![wp_ss_20140106_0001](/assets/img/2014/01/wp_ss_20140106_0001.png "wp_ss_20140106_0001")

Technically, it would be possible to use a WebBrowser control, too. But you would need to code more to get a proper browsing handling.

## Feedback system/forum

Some of you might be already familiar with the feedback or idea forum of uservoice (even Microsoft uses this). This time, we look at the other side of the forum, from our admin console:

![Screenshot (292)](/assets/img/2014/01/Screenshot-292.png "Screenshot (292)")

You have a similar view to what users see, but there are some more parts to control your forum. As a free user, you have only one forum, so it might be useful if you add the app name in front of the title of an idea. You can respond, take notes, and change the status of the ideas:

![Screenshot (293)](/assets/img/2014/01/Screenshot-293.png "Screenshot (293)")

To integrate the feedback function in your app, you can again use a WebBrowserTask to open the mobile page of your idea forum:

![wp_ss_20140106_0002](/assets/img/2014/01/wp_ss_20140106_0002.png "wp_ss_20140106_0002")

These are the first simple steps to create your own customer support system with uservoice. Like I said before, I will write a Windows Phone wrapper to get this into an app as native, and will share it here with you all.

Until the next post, happy coding!