---
id: 4357
title: 'How to add Microsoft Application Insights v1 to your Windows 8.1 Universal app'
date: '2015-06-30T03:53:40+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-add-microsoft-application-insights-v1-to-your-windows-8-1-universal-app/
categories:
    - Archive
tags:
    - AI
    - 'Application Insights'
    - 'Exception Logging'
    - Telemetry
    - TelemetryClient
    - v1
    - 'Windows 8.1'
    - 'Windows Phone'
    - 'Windows Phone Runtime'
    - 'Windows Universal apps'
    - WindowsAppInitializer
---

Late in 2014, Microsoft finally started Application Insights (AI), their own telemetry service for all kind of apps. For Windows (Phone) 8.1 apps, the service was a long running beta. This month, Microsoft finally released [version 1.0 of Application Insights for Windows apps](http://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsApps/1.0.0).

[![Screenshot (15)](/assets/img/2015/06/Screenshot-15.png)](/assets/img/2015/06/Screenshot-15.png)

However, if you are upgrading from a previous version, you will see that AI will no longer send any data to Azure. This has a very simple reason. Microsoft moved the configuration from the former ApplicationInsights.config file to a class called WindowsAppInitializer. I only found this out because I commented at the corresponding documentation site, which causes Microsoft to send me an email with a [link to the solution in the forums](https://social.msdn.microsoft.com/Forums/vstudio/en-US/c3dec5f2-0434-4747-b044-07f14c771fe1/ai-crashing-my-app-upon-restart?forum=ApplicationInsights). You will not find these info in the documentation as of writing this blog post. Microsoft also [updated the docs tonight.](https://azure.microsoft.com/en-us/documentation/articles/app-insights-release-notes-windows/#version-100)

I strongly recommend you to remove all old references in favor of just updating to avoid any glitches with the new API.

I played around with the new WindowsAppInitializer class. If you want to collect all data with the automatic WindowsCollectors, all you have to add to your code is one line in your App.xaml.cs constructor (Microsoft recommends to add it before all other code):

``` csharp
 WindowsAppInitializer.InitializeAsync("your instrumentation key”);
```
 
That’s it. Really. Here’s a screen shot of the test app in Visual Studio I created to play around with the new WindowsAppInitializer class:

![Screenshot (24)](/assets/img/2015/06/Screenshot-24_thumb.png "Screenshot (24)")

As you can see, telemetry data gets written to the debug output, and with that, it will be submitted to your Azure account. If you do not want to use all Collectors, just add those you want to use after your InstrumentationKey, separated with ‘|’ in the IninitalizeAsync Method.

Adding custom telemetry data collection is still possible. Quick sample:

``` csharp
 var tc = new TelemetryClient();
tc.TrackEvent("MainPage loaded... [WP TestEvent]");
```
 
This will send this string as a custom event to your Azure account. For more info about custom telemetry data, [check this page](https://azure.microsoft.com/en-us/documentation/articles/app-insights-windows-get-started/).

As always, I hope this blog post is helpful for some of you.

Happy coding!