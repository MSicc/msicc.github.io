---
id: 3663
title: 'Getting productive with WAMS: respect time zone offset for every single user'
date: '2013-07-14T08:53:49+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /getting-productive-with-wams-respect-time-zone-offset-for-every-single-user/
categories:
    - Azure
    - 'Dev Stories'
tags:
    - AzureDev
    - 'C#'
    - JavaScript
    - offset
    - time
    - 'time zones'
    - timezone
    - utc
    - WAMS
    - wpdev
---

![time_Azure](/assets/img/2013/07/time_Azure.png "time_Azure")

In my second post about WAMS I will show you how to respect the time zone of every user.

If you have users that are from all over the world, they have all different time zones. Your Mobile Service script runs always at UTC time, and every user gets the same date &amp; time if you send them push notifications or update a live tile for example. Users don’t want to calculate the time zone differences, so we need to handle that for them.

UTC time is the time since 01/01/1970 00:00 in milliseconds. If we know this, it is somewhat easy to show users their local date and time.

### Let’s have a look at the Windows Phone code.

To get the local time zone in our Windows Phone app, we only need three lines of code:

``` csharp 
TimeZoneInfo localZone = TimeZoneInfo.Local;
DateTime localTime = DateTime.Now;
TimeSpan offsetToUTC = localZone.GetUtcOffset(localTime);
```
 
As you can see, we are getting the local time zone first. This is essential as this one is UTC based. Then we are creating a TimeSpan on our actual DateTime object to get the offset. To make this TimeSpan working on our Azure Mobile Service, which uses JavaScript, we need to convert it to milliseconds. That is the value that has an equal value on all programming languages.

``` csharp
useritemLookUp.TimezoneOffset = offsetToUTC.TotalMilliseconds;
```
 
This is the final line of code, which is used to update our user’s item in our SQL table row (for example).

### Let’s have a look at the Azure code.

The code is similar to our Windows Phone part. First, we need to fetch the time zone offset from our SQL table:

``` csharp
var sql = "select * from users";

mssql.query(sql, {
success: function (results) {
    if (results.length > 0) {
        for (var i = 0; i < results.length; i++) {
            userResult = 
            {
                TimeZoneOffset: results[i].TimezoneOffset,
            }
        }
    }
}

TimeZoneOffset = userResult.TimeZoneOffset;
```
 
This way, we can run through our whole table on an Azure Script and calculate the correct time, which is pretty easy to achieve:

``` csharp
var d = new Date();
var locald = new Date(d.getTime() + TimeZoneOffset);
```
 
These two lines generate the local time in Milliseconds for the specific user entry. You don’t have to worry whether a user is before or after UTC, it will always calculate the correct time.

If you want to use it for example to show the updated time to your users, you can format the time [like I described earlier in this post]({% post_url 2013-05-04-how-to-format-date-and-time-on-windows-azure %}).

That’s all about respecting local time for your users with Windows Azure and Windows Phone.

Happy coding everyone!