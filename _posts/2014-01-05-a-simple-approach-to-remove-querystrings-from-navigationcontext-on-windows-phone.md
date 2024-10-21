---
id: 3901
title: 'A simple approach to remove QueryStrings from NavigationContext on Windows Phone'
date: '2014-01-05T09:23:59+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /a-simple-approach-to-remove-querystrings-from-navigationcontext-on-windows-phone/
categories:
    - Archive
tags:
    - add
    - navigationcontext
    - querystring
    - remove
    - tombstoned
    - tombstoning
    - 'uri scheme'
    - 'Windows Phone 8'
    - WinPhanDev
    - wpdev
---

![QuerystringsRemove](/assets/img/2014/01/QuerystringsRemove.png "QuerystringsRemove")

Like most of you know, I am currently adding a lot of new features to my [NFC Toolkit](https://www.windowsphone.com/s?appid=2c33cb7d-c97b-4204-aa8b-1e8712718519) for Windows Phone.

Today, my coding session is all about uri schemes again. I successfully added some new uri schemes that you will be able to use soon.

However, if you start an app via an uri scheme, you will have QueryStrings in your NavigationContext on the desired ApplicationPage. If a user continues to use your app, navigates away from the launched page and back to it, the code based on your QueryStrings will be called again.

As I am a daily Windows Phone user by myself, I know this is somewhat annoying. So I searched for a solution. I first thought about removing entries in the BackStack, but that would need a lot of code if you are running an application that has more than two pages – you’ll need to add methods to check if you are coming from an uri scheme, clear the BackStack, pass the parameters to the next page, get back to your MainPage, check the parameters again. Totally unusable in my opinion.

The NavigationContext.QueryString property has a Remove() method as I found out while researching a bit more – that’s what I am using now to get the result I want – make the app running like it was launched normally without uri scheme after the action has taken place.

I found some examples that are using the method in the OnNavigatedTo() event. I don’t think that’s the right way, as I often happen to have some more code in this event, and it will probably break other functionality. I am using the OnNavigatedFrom() event.

Add this lines of code for every QueryString you want to remove (depending on your method structure, it is ok to remove only the QueryString you rely on in your further code, additional parameters can be still there ):

``` csharp
protected override void OnNavigatedFrom(NavigationEventArgs e)
{
   if (NavigationContext.QueryString.ContainsKey("key1"))
   {
      NavigationContext.QueryString.Remove("key1");                                
   }
   if (NavigationContext.QueryString.ContainsKey("key2"))
   {
       NavigationContext.QueryString.Remove("key2");
   }            
   base.OnNavigatedFrom(e);
}
```
 
This way, if you are navigating to another page in your app or even if the user uses the Windows Button, the corresponding QueryString will be removed – and your users are able to use the app as they would have launch it without an uri scheme.

However, that’s not all. When your app get’s Tombstoned, the UriMapper will map the uri scheme again on Activation – not what you might want. To get around this, I use a simple Boolean, set it to true when I am coming back from Tombstoning:

``` csharp
private void Application_Activated(object sender, ActivatedEventArgs e)
{
    if (!e.IsApplicationInstancePreserved)
    {
        wasTombstoned = true;  
    }
}
```
 
Now the only thing you will need to to is to add

``` csharp
 && App.wasTombstoned == false
```
 
as second argument to your QueryString clause – that’s it!

I am not sure if this is a good approach or not, but it works. It allows you to launch your uri scheme, and enables the user to use your app after that as it would have been launched normally. If you have other ways to get the same result, feel free to leave a comment below.

Otherwise, I hope this post is helpful for some of you.

Happy coding!