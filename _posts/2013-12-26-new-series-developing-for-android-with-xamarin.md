---
id: 3858
title: 'New Series: developing for Android with Xamarin'
date: '2013-12-26T09:46:36+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /new-series-developing-for-android-with-xamarin/
categories:
    - Android
    - 'Dev Stories'
    - Xamarin
tags:
    - Activities
    - Android
    - 'cross platform'
    - install
    - Intents
    - methods
    - series
    - structures
    - Views
    - xamarin
---

### ![xamandroid](/assets/img/2013/12/xamandroid.png "xamandroid")

### Prolog

In the last few month I was developing not only my personal projects, but also on my daily 9to5 job. I am working on an internal app that will help to improve the customer experience amongst the customers of my employer Telefónica.

The project has a simple setup in phase I, with collecting data and sending them via mail. The challenge for me was how to get the Windows Phone app (yes, Windows Phone was first!) to Android.

I knew it would take some time to get started with Android development, and was searching for possible solutions to speed things up. I knew from Xamarin already, and was able to obtain a 1-year license for Android (maybe iOS will follow later).

Some of my fellow WinPhans and WinPhanDevs will scream out very loud at the moment. I understand that. But honestly, I could not let go away this huge opportunity for me – as it touches both my daily job and my private developer story. To calm you down: I will always love Windows Phone and Windows and develop for it first. I will always remain a WinPhanDev. But I need to go forward, and this is a big step.

Let’s go back to the topic.

With this blog post, I am kicking off a new blogging series for Android development with Xamarin. The huge advantage of using Xamarin is that I am able to use my C# knowledge to develop apps for other platforms – which makes it a little easier to get things done.

However, if you start with another platform, you still have to learn the platform structure. Without knowing or be willing to learn it, you will be lost very fast. Every OS has its own specific UI rules, save handling and so on. This is what I will blog about.

The Windows Phone app I ported to Android has a item Pivot, which I ported over to Android. It has a little different appearance, but it works in a similar way with tapping the header or a swiping gesture to move between the items/pages.

To get there, I had to learn a lot about Android – and I will share it all with you out there.

My series will cover the following topics:

- installing Xamarin and getting started (this post)
- [Setting up an Android device for debugging and deployment]
- Resources (Layouts, Strings, etc.)
- Activities and Views
- states of an Android app
- Fragments
- show and hide keyboard on Android
- Focus of controls
- using SQLite to save data temporarily and permanently
- using Intents to send Mail or save a Calendar entry
- getting contacts from address book
- Lists and Adapters
- create a SplashScreen
- and more… (I will update this list with links to the corresponding blog posts)

### Installing Xamarin

Installing Xamarin is pretty easy – but will take some time. Click on this [link](https://xamarin.com/studio) to download the Xamarin IDE. The IDE will ask you for which platform you want to develop for (let’s choose only Android for the moment) and then download a huge amount of additional SDKs like the Java and the Android SDK (Android apps are Java based, if you didn’t knew).

![Screenshot (267)](/assets/img/2013/12/Screenshot-267.png "Screenshot (267)")

On the screenshot above you can see the starting page of Xamarin Studio. On the left side you have the list of recent solutions, in the middle you see Xamarin news and finally the pre-built apps section.

Xamarin Studio has a lot together with Visual Studio, and you also can take a lot of settings to adjust the appearance for your needs. Let’s start a new project by clicking on the ‘New’ button

![Screenshot (270)](/assets/img/2013/12/Screenshot-270.png "Screenshot (270)")

Xamarin will set up a .sln file with everything we need for the moment. After creating the project has been created, you’ll see these lines of code:

``` csharp
using System;
using Android.App;
using Android.Content;
using Android.Runtime;
using Android.Views;
using Android.Widget;
using Android.OS;

namespace gettingstarted
{
	[Activity (Label = "gettingstarted", MainLauncher = true)]
	public class MainActivity : Activity
	{
		int count = 1;

		protected override void OnCreate (Bundle bundle)
		{
			base.OnCreate (bundle);

			// Set our view from the "main" layout resource
			SetContentView (Resource.Layout.Main);

			// Get our button from the layout resource,
			// and attach an event to it
			Button button = FindViewById<Button> (Resource.Id.myButton);

			button.Click += delegate {
				button.Text = string.Format ("{0} clicks!", count++);
			};
		}
	}
}
```
 
These few lines should help to get started with the basic structure of an Android App.

Android has Activities. Activities can be seen as the code behind files in a C# project. Here begins all the action of an Android app. Xamarin automatically ads the Label (project name) and in this case, also the MainLauncher property, which will run this code on app start up.

To get something displayed in our app, we need to load a Layout or create the view from the code. This is what the SetContentView() method is about. Without this, the app will compile but displays nothing. In this demo app, a basic layout with a button inside is created:

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="https://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
    <Button
        android:id="@+id/myButton"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/hello" />
</LinearLayout>
```
 
We already have the first difference to other C# projects here. In order to make the button visible for our code in the Activity, we need to search our resources for the button. This is done by the FindViewById&lt;T&gt;() method. As soon as it is visible to our code, we are able to create a delegate for our button Click event.

The button counts then the clicks on it and changes the text based on the number of clicks.

To understand more about the structure of an Android App, I highly recommend to read the documentation. For me, especially these links were helpful:

- [Hello, Android](https://docs.xamarin.com/guides/android/getting_started/hello,_world/)
- [Hello, Multiscreen Applications](https://docs.xamarin.com/guides/android/getting_started/hello,_multi-screen_applications/)
- [Activity Lifecycle](https://docs.xamarin.com/guides/android/application_fundamentals/activity_lifecycle/)
- [Android Resources](https://docs.xamarin.com/guides/android/application_fundamentals/resources_in_android/)

There are some more resources that are helpful, but for getting started those four links should be enough. I will cover some of the topics above also with my upcoming posts and link additional resources as well.

As with the Windows Phone development, also [StackOverflow](https://stackoverflow.com/search?q=xamarin) is a big help as well as the [Xamarin Android forums](https://forums.xamarin.com/categories/android).

Until the next post, happy coding!