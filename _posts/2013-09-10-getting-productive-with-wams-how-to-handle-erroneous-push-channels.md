---
id: 3767
title: 'Getting productive with WAMS: How to handle erroneous push channels'
date: '2013-09-10T06:21:12+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /getting-productive-with-wams-how-to-handle-erroneous-push-channels/
categories:
    - Azure
    - 'Dev Stories'
tags:
    - '404'
    - '412'
    - AzureDev
    - channel
    - error
    - 'error code'
    - 'Mobile Services'
    - mpns
    - push
    - WAMS
    - 'Windows Azure'
    - 'Windows Phone'
    - wpdev
---

![WAMS](/assets/img/2013/09/WAMS.png "WAMS")

As I wrote already in my former article [‘Getting productive with WAMS- about the mpns object (push data to the user’s Windows Phone)’]({% post_url 2013-08-14-getting-productive-with-wams-about-the-mpns-object-push-data-to-the-users-windows-phone %}), I needed to add some more detailed error handling to the mpns object on my Mobile Service.

There are two error codes that appear frequently: 404 (Not Found) and 412 (Precondition Failed).

### The 404 error code

happens when the push channel gets invalid. Reasons for that can be uninstall or reinstall of the app, or also a hard reset of the users device. In this case, we are following the recommendation of Microsoft to stop sending push notifications via this channel.

There might be several ways, but I prefer to work with SQL queries. This is how I delete those channels within error: function(error):

``` js
 if (error.statusCode === 404)
{
 var sqlDelInvalidChannel = "DELETE from pushChannel WHERE id = " + channel.id;
 mssql.query(sqlDelInvalidChannel, {
 success: function(){
	console.log("deleted invalid push channel with id: " + channel.id);
	},
	error: function(err)
	 {
           console.log("there was a problem deleting push channel with id: " + channel.id + ", " + err)
         }
});
```
 
As you can see, this is a very simple approach to get rid of those invalid channels.

### The 412 error code

needs some more advanced handling. Microsoft recommends to send the push notification as normal, but the code recommends a delay of 61 minutes to resend. Also, with some research on the web, I found out that often the 412 will turn into a 404 after those 61 minutes (at least I found a lot of developers stating this). This is why I went with a different approach. I am going to wait those 61 minutes, and do not send a push notification to those devices.

For that, I do a simple trick. I am saving the time the error shows up in my push channel table, as well as the time after those 61 minutes. On top, I use a Boolean to determine if the channel is in the delay phase.

Here is the simple code for that, again within error: function(error):

``` js
if (error.statusCode === 412)
{
	var t = new Date();
	var tnow = t.getTime();
	var tnow61 = tnow + 3660000;

	var sqlSave412TimeStamp = "UPDATE pushChannel SET Found412Time=" +  tnow + ", EndOf412Hour=" + tnow61 + ", IsPushDelayed = 'true' WHERE id=" + channel.id;

	mssql.query(sqlSave412TimeStamp);
}
```
 
Now if my script runs the next time over this push channel, I need to check if the push channel is within the delay phase. Here’s the code:

``` js
if (channel.IsPushDelayed === true)
{
	var t = new Date();
	var tnow = t.getTime();
	var EndofHour = channel.EndOf412Hour;
	var tdelay = EndofHour - tnow;

	if (tdelay > 0)
	{
	 console.log("push delivery on id: " + channel.id + " is delayed for: " + (Math.floor(tdelay/1000/60)) + " minutes");
	}
	else if (tdelay < 0)
	{
	 var sqlDelete412TimeStamp = "UPDATE pushChannel SET Found412Time=0 , EndOf412Hour= 0, IsPushDelayed = 'false' WHERE id=" + channel.id;
	 mssql.query(sqlDelete412TimeStamp);
	}
}
```
 
As you can see, I am checking my Boolean that I added before. If it is still true, I am writing a log entry. If the time has passed already, I am resetting those time values to 0 respective the Boolean to false.

The script will now check if the push channel is still a 412 or turned in to a 404, and so everything starts over again.

### Other error codes

There might be other error codes as well. I did not see any other than those two in my logs, but for the case there would be another, I simply added this code to report them:

```js
else
{
 console.error("error in Toast Push Channel: " + channel.twitterScreenName, channel.id, error)
}
```
 
This way, you can easily handle push channel errors in your Mobile Service.

If you have other error codes in your logs, check this list from Microsoft to determine what you should do: [https://msdn.microsoft.com/en-us/library/windowsphone/develop/ff941100(v=vs.105).aspx#BKMK\_PushNotificationServiceResponseCodes](https://msdn.microsoft.com/en-us/library/windowsphone/develop/ff941100(v=vs.105).aspx#BKMK_PushNotificationServiceResponseCodes "https://msdn.microsoft.com/en-us/library/windowsphone/develop/ff941100(v=vs.105).aspx#BKMK_PushNotificationServiceResponseCodes")

Note: there might be better ways to handle those errors. If you are using such a way, feel free to leave a comment with your approach.

Otherwise, I hope this post will be helpful for some of you.

Happy coding!