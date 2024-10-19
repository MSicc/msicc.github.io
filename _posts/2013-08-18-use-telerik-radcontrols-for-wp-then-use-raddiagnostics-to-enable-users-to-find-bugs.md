---
id: 3740
title: 'Use Telerik RadControls for WP? Then use RadDiagnostics to enable users to find bugs!'
date: '2013-08-18T11:04:06+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /use-telerik-radcontrols-for-wp-then-use-raddiagnostics-to-enable-users-to-find-bugs/
categories:
    - Archive
tags:
    - beta
    - 'beta testing'
    - RadControls
    - RadDiagnostics
    - telerik
    - 'Windows Phone'
    - wpdev
---

I know this is a little provocative title – that is my intention. And of course I will tell you why.

I am participating in several betas for Windows Phone apps. I know some of the developers from Twitter, and had several conversations about bug finding and solving methods. In the end, we as developers have all one goal: a buttery smooth running app without any bugs.

Some of the betas I use are also using Telerik RadControls. They enable us developers to easily create awesome apps without writing the controls form the ground up. On Top, we are able to customize them for our needs with a bit of XAML manipulation.

One of these controls is RadDiagnostics. RadDiagnostics is catching all unhandled exceptions from your application. And believe me, there will be a ton of it – even if you handle a lot of them already.

I didn’t believe that, too. But then I ran the beta of my TweeCoMinder app for Windows Phone. I tested the app a lot before publishing my beta, including several scenarios like no network connection available, low battery, and so on. At this time, I stumbled over the RadDiagnostics Control in the documentation.  
I immediately saw the advantages of using this control:

- advanced error logging, including a great bunch of information like OS Version, device, network type, and more
- the app won’t crash even on unhandled exceptions
- the user feels more integrated in the development process
- users tend to use the app in other ways than we developers think – and we can’t even get close to catch all those usage scenarios while testing alone
- there are a lot of users that are really searching for bugs – which is both a good and a bad thing

Let me hook into the last point a bit deeper – as we are able to get the most from it. If users find bugs, they will get either annoyed and stop using our apps or they will talk about it. Let us make them talk about the problems to us, the developer! Like I mentioned above, the RadDiagnostics control makes this very easy for the user.

A MessageBox shows up where the user is asked to send us the report, which is done via email. This has two additional advantages: First, we are able to collect all kind of errors. An even bigger advantage: We are able tor respond directly to the user as we have their mail address. I learned a lot during the first beta versions of TweeCoMinder, be it how users use my app as well as from the coding part – and I was able to continuously improve my app.

Some of you might argue that you don’t want the user to see your code. Really, that is stupid. An average user does not even know what all those lines in of the exception mean. Only developers are able to understand what is going on. Where is the problem? That they see you have an exception at point x? Chances are very high, that they had the same exception at another point and needed them to iron out.

Personally, I don’t have a problem with the fact that the exception is readable for some beta tester/users. I made the experience that users love to give feedback – if you make it very easy for them. RadDiagnostics is a very easy way as it needs only three taps (ok =&gt; choose mail account =&gt; send button).

As this is a dev post, here is how easy it is to integrate it into your app:

- Declare the RadDiagnostics in *public partial class App : Application
```
 chsarp
    public RadDiagnostics diagnostics;
```
 

- Initiate Telerik ApplicationUsageHelper in *private void Application\_Launching(object sender, LaunchingEventArgs e)*

``` csharp
ApplicationUsageHelper.Init("0.9.5");
```
 
- call RadDiagnostic in the *App()* constructor:

``` csharp
 diagnostics = new RadDiagnostics();
{
diagnostics.EmailTo = "yourmail@outlook.com";
diagnostics.EmailSubject = "Here is your email subject";
diagnostics.HandleUnhandledException = true;
diagnostics.Init();
 }
```
 

That’s all you need to get it working. A few line of codes that will give you so much input to improve your app, which will result in this message:

![ReadDiagnosticsMSG](/assets/img/2013/08/ReadDiagnosticsMSG.png "ReadDiagnosticsMSG")

Once again, if you use RadControls for Windows Phone, then use RadDiagnostics to let users help you finding bugs. You will not get better and more helpful feedback other than this!

You can find the complete documentation of RadDiagnostics here: [http://www.telerik.com/help/windows-phone/diagnostics-gettingstarted.html](http://www.telerik.com/help/windows-phone/diagnostics-gettingstarted.html "http://www.telerik.com/help/windows-phone/diagnostics-gettingstarted.html")

Another great article that helps you to deeply understand how RadDiagnostics work is over at [Kunal Chowdhury’s blog](http://www.kunal-chowdhury.com/2012/05/learn-about-raddiagnostics-for.html).

Feel free to discuss your sight below in comments. Until then, Happy coding!