---
id: 2929
title: 'Dev Story Series (Part 1 of many): Creating a data class for both Windows 8 and Windows Phone app'
date: '2012-10-18T05:17:32+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /dev-story-series-part-1-of-many-creating-a-data-class-for-both-windows-8-and-windows-phone-app/
image: /assets/img/2012/10/json_string_unserialized.png
categories:
    - Archive
tags:
    - codeproject
    - Dev
    - json
    - win8dev
    - Windows
    - 'Windows Phone'
    - wpdev
---

As I promised earlier on my application for the [Intel App Innovation Contest on codeproject.com,](https://www.codeproject.com/Articles/475654/Simultan-development-for-Windows-8-and-Windows-Pho) I will do a series of blog posts for my application MSicc´s Blog for Windows 8 and Windows Phone.

This is the first article in my new Dev Story Series, where I describe the development process of the app. I am starting with the very first steps that you have to do if you plan such an application. To make the application useful, we first have to decide how we want to get the data from our blog/website into our application.

One hint that will make some of you crying out loud: At the moment I am not using the MVVM pattern. I am aware of the fact that most devs for Silverlight/C# swear on it, but I still decided to go without it – as well on Windows Phone as on Windows 8. I will continue to learn it in one of my future project, but for the moment I just want to get things done – also if that means that I have to create some – well, let us call it “compromises”.

**Which way to choose?**

If you consider to create a “blog reader”, you have to think about how you want to get the data from your blog into your app. There are different ways:

- via a RSS/Atom feed (that was in the old version of my Windows Phone app for msicc.net)
- via XMLRPC (if your blog supports that)
- via JSON

I decided to go with JSON for the new version. There were several reasons to do so:

- there is an app err… an plugin for that on WordPress
- JSON is fast
- JSON is set to be the new standard for data consumption
- With JSON.NET you can deserialize your data with only one line(!) of code

**How do we get the data in our app?**

Of course, we will download it. But if you only download your JSON data, the only thing you will get is a very weird looking string that looks like this:

![json_string_unserialized](/assets/img/2012/10/json_string_unserialized.png "json_string_unserialized")

That´s pretty ugly, right? Of course we could work us through all arrays and objects there. That would take hours and hours until we would have covered all data we need for our apps. Luckily, there are two tools that make the whole thing a lot easier.

**Two handy tools to make a dev´s life easier**

The first tool I want to show you is “Beautify JSON”. It is a web based tool that makes strings like the one above readable for a human. Find it [here](https://archive.dojotoolkit.org/nightly/dojotoolkit/dojox/gfx/demos/beautify.html).

Just paste the string of your JSON API in there, and you will get a readable version of your JSON string:

![beautify_json](/assets/img/2012/10/beautify_json.png "beautify_json")

If you want to use only some of the data that your API provides, you can now easily search the string for the objects and arrays that you need.

If you quickly want to create a data class for all data that is provided, you can do this by using [json2charp](https://json2csharp.com/ "https://json2csharp.com/"). It provides a ready-to-use code that can easily be copied and pasted into your class file in Visual Studio:

![json2charp](/assets/img/2012/10/json2charp.png "json2charp")

Now let´s have a look to the class itself:

``` csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.Serialization;
using System.Text;
using System.Threading.Tasks;

namespace WordPressDataClass
{
 public class Posts
    {
        [DataMember]
        public string status { get; set; }
        [DataMember]
        public int count { get; set; }
        [DataMember]
        public int count_total { get; set; }
        [DataMember]
        public int pages { get; set; }
        [DataMember]
        public List<Post> posts { get; set; }
    }
}
```
 

As you can see, I created a DataContract and DataMember based class by using System.Runtime.Serialization. If you want to know more about DataContracts and DataMembers I recommend you to read MSDN: [using Data Contracts](https://msdn.microsoft.com/en-us/library/ms733127.aspx).

I am not posting the whole class, only the key DataContract Posts, which will get us our List of posts. You can download the whole class at the end of the post. It will be very helpful if you want to create an app for your WordPress based blog.

I use the class in both my Windows 8 and my Windows Phone app without any differences. As you can see, If you are familiar with Windows Phone Development, there is only a small step to Windows 8 development.

I hope this article is helpful for some of you. In my next post I will show you how easily you can use the provided JSON Data in your app/s.
