---
id: 3579
title: 'How to format Date and Time on Windows Azure'
date: '2013-05-04T13:14:00+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-format-date-and-time-on-windows-azure/
categories:
    - Azure
    - 'Dev Stories'
tags:
    - AzureDev
    - Cloud
    - JavaScript
    - 'Windows Azure'
---

![time_Azure](/assets/img/2013/05/time_Azure.png "time_Azure")

Phew, my first post about my journey on starting development on Windows Azure. I started a few weeks ago using the Mobile Services from Windows Azure, and I did learn a lot about it.

This post is about formatting Date and Time strings, because Azure uses a different format than my Windows Phone app.

If we upload a DateTime String to Windows Azure from a Windows Phone app, it looks like this: *2013-05-04T06:45:12.042+00:00*

If we translate this, you have “YYYY-MM-DD” for the date. The letter “T” declares that the time string is starting now, formatted “HH:MM:ss.msmsms”. The part “+00:00” is the timezone offset.

So far, probably nothing new for you.

Now let’s get to Azure. Azure by standard uses the GMT time for Date strings (DateTime() in JavaScript = Date()). I have written a scheduler which fetches data from another web service and puts it into my table. Naturally, I wanted to know when the data were last checked, so I added a column for it.

Then I did what everyone that is new to JavaScript has done and added a variable with a new Date(). And now the trouble begins. The output of new Date() is a totally different string: *Sat, 04 May 2013 07:02:51 GMT***.**

Sure, we can parse and convert it within our app, but that would need (although not much) additional resources. So I decided to let to Azure the conversion to a Windows Phone readable string.

### How do we manipulate the Date()-string?

I binged a bit and finally found a very helpful page, that explains all about the JavaScript Date() object: [http://www.elated.com/articles/working-with-dates/](http://www.elated.com/articles/working-with-dates/ "http://www.elated.com/articles/working-with-dates/")

I then started off with the following code:

``` csharp var d = new Date();
var formattedDate = d.getFullYear() + "-" + d.getMonth() + "-" + d.getDate();
var formattedTime  = d.getHours() + ':' d.getMinutes() + ':' + d.getSeconds();
var checkedDateTime = formattedDate + "T" + t;
```
 
Those of you that are familiar with JavaScript will immediately see what I did wrong. Let me explain for the newbies:

First thing, date().getMonth is zerobased. So we will always get a result that is one month behind. We have to get it this way for the correct month:

``` csharp
d.getMonth()+1
```
 
But that is not all. If you will use the code above, your result will look like this: *2013-5-4T7:2:51*

JavaScript does not use leading zeros. If you want to insert it into a date formatted column, you will get the following error from Azure:

``` shell
Error occurred executing query: Error: [Microsoft][SQL Server Native Client 10.0][SQL Server]Conversion failed when converting date and/or time from character string.
```
 
So we need to add the leading zero before inserting it. Luckily we are able to that very easy. Here is my implementation:

```
 chsarp
 var d = new Date();
var formattedDate = d.getFullYear() + "-" + ('0' + (d.getMonth()+1)).slice(-2) + "-" + ('0' + d.getDate()).slice(-2);
var formattedTime  = ('0' + d.getHours()).slice(-2) + ':' + ('0' + d.getMinutes()).slice(-2) + ':' + ('0' + d.getSeconds()).slice(-2);
var checkedDateTime = formattedDate + "T" + t;
```
 

### What have we done here?

We are adding the leading 0 to each object string. The slice(-2) is for only picking the last two numbers. To make it more clear: if we have 9 as hour, adding the zero in front results in 09. Picking only the last two numbers by .slice(-2) results in still in 09. If we have 10 as hour, adding the leading zero results in 010. But the .slice(-2) operation will cut it back to 10. Easy enough, right?

If we run the code above to get the Date and Time, the result will look like this: *2013-05-04T7:02:51*

The timezone offset is automatically added to the date when we update the table. If we now send the data to our Windows Phone or Windows 8 app, no conversion is needed as we already have a correctly formatted string.

I hope this is helpful for some of you and will save you some time.

Happy coding everyone!