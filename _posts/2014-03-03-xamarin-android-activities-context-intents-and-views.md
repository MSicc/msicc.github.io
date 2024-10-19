---
id: 3978
title: 'Xamarin: Android Activities, Context, Intents and Views'
date: '2014-03-03T10:57:34+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /xamarin-android-activities-context-intents-and-views/
categories:
    - Android
    - 'Dev Stories'
    - Xamarin
tags:
    - Activities
    - Android
    - class
    - classes
    - differences
    - lifecycle
    - 'need to know'
    - Views
    - xamarin
---

![xamandroid](/assets/img/2014/03/xamandroid.png "xamandroid")

If you are coming from C#/Silverlight, Android can be a little diffusing. Android does not have the same structure than a Windows or Windows Phone app.

### Activities

Android has Activities. Activities are the key classes of Android were all actions take place. Unlike other apps or programs, you do not have a “Main” program that is your starting point when launched.

In Android, the starting point is an Activity. The Activity needs to be declared as the starting point. When you start a new project in Xamarin, an Activity called “MainActivity” gets created automatically.

This Activity has some attributes:

``` csharp
 [Activity (Label = "gettingstarted", MainLauncher = true)]
```
 
The ‘Label’ attribute is what you will see in the Title bar when launching the app. The attribute ‘MainLauncher=true’ tells the application to start from here. Think of this as your MainPage.xaml.cs in a Windows Phone app.

Every Activity has its own OnCreate event, where you can put all your starting, button handlers, stylings etc. in.

But an Activity still has more. It has events like OnStart(), OnPause(), OnRresume() and OnStop() and OnDestroy() and OnRestart(). I won’t get into deep this time, as the Xamarin documentation has already a very good overview of those events: [http://docs.xamarin.com/guides/android/application\_fundamentals/activity\_lifecycle/](http://docs.xamarin.com/guides/android/application_fundamentals/activity_lifecycle/ "http://docs.xamarin.com/guides/android/application_fundamentals/activity_lifecycle/").

Those events are important to understand for a lot of your application logic, like:

- saving data that has to be persistent
- starting and pausing animations
- register and unregister external events
- fetch data that are passed from other Activities
- and more (we will cover some of them in my further posts)

I absolutely recommend to go to the link above to read and understand those events.

### Context

The Context is often needed in an Android application and allows your code to be run within an Activity.

The Context allows you for example:

- to access Android services,
- to access Application resources like images or styles,
- to create views
- to assign gestures to an Activity

Without Context, often code does not get accepted by the IDE or makes your code getting ignored while running the app. Luckily, all methods that I used so far have been telling me if they want a Context, so you need only learn to find out if you need Context for the Activity resources or Application resources.

### Intents

But what if we want to have another page (like a separate about page for example)? How can we navigate between pages or call external functions?

This is what Intents are for. Intents are sending messages through the application if another function needs to be started, like the launch of a second page. Within these intents, we have declare the actions that the app needs to do, and if we need a callback, this will also be passed with an Intent.

Let’s hold this high level, here are some lines of code that navigate to a second page (Activity):

``` csharp
 var second = new Intent(this, typeof(SecondActivity)); StartActivity(second);
```
 
With this code, we are creating an new Intent with Context to our current running Activity to launch the SecondActivity.

To send data between Activities, we use the PutExtra() method of Intents:

``` csharp
 var second = new Intent(this, typeof(SecondActivity)); 

second.PutExtra("FirstPage", "Data from First Page"); 

StartActivity(second);
```
 
Of course, we need also some code to read the passed data on our second page:

``` csharp
 Intent.GetStringExtra("FirstPage") ?? “Data not available”;
```
 
We could now use this data on our second page by assigning it to a control or function.

Passing data between activities works similar to passing QueryStrings in NavigationsService.Navigate() method on WindowsPhone, so you should get familiar with it very fast.

### Views

The last part that is very important are Views. In a View you declare how the Activity looks like. Think of it as your MainPage.xaml page in a Windows Phone app.

Views can be very different, too. Let’s start with the simple View. In our getting started project, we also have a Layout file that holds our first View:

``` xml
 <?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
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
 
The View has a main LinearLayout which is acts as ContentPanel. All other controls like Buttons, TextViews etc. are going into this. There are a lot of properties that can be set, we’ll leave it this easy for now. If you want to know more about those properties, you can have a look at the Android documentation here: [http://developer.android.com/guide/topics/ui/declaring-layout.html](http://developer.android.com/guide/topics/ui/declaring-layout.html "http://developer.android.com/guide/topics/ui/declaring-layout.html")

This part is totally the same as with the one matching part in the Android SDK, so you will need to get familiar with the Android documentation as well.

To make a View visible, we need to assign it to an Activity. In our sample app, we are doing this with this line of code in our OnCreate event:

``` csharp
 SetContentView (Resource.Layout.Main);
```
 
This is the easy way for Views. But there are more complex Views in Android, too.

Views are also used in ListViews, where you declare the look of the items and the list in a Layout file, or also if you use the ActionBar with a tab navigation, and also in the menu of an ActionBar. I will cover all of these in my further blog posts, as we also need Adapters to get data binded to Views.

I hope you this post helps you to understand the high level structure of an Android application.

Until the next time, happy coding!