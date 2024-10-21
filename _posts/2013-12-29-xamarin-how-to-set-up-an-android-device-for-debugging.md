---
id: 3874
title: 'Xamarin: how to set up an Android device for Debugging'
date: '2013-12-29T07:08:30+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /xamarin-how-to-set-up-an-android-device-for-debugging/
categories:
    - Android
    - 'Dev Stories'
    - Xamarin
tags:
    - about
    - Android
    - 'debug app'
    - debugging
    - 'developer options'
    - device
    - 'device setup'
    - 'OS version'
    - settings
    - 'software version'
    - 'USB debugging'
    - xamarin
---

Xamarin comes with some emulators for development of an Android app. While these emulators are working and are also available in all supported screen sizes, the performance of them is really bad.

That’s why I chose debugging via device, which is a lot faster than the emulators once you have the required Xamarin extensions installed.

Here is how to set up an Android device for debugging. First thing you need to know is which OS version is running. On most Android devices, you will find this information under Settings/About (or Info)/Software information. If you don’t have the result there, you should check the manual where to find this information on your device.

![Screenshot_2013-12-29-07-03-31](/assets/img/2013/12/Screenshot_2013-12-29-07-03-31.png "Screenshot_2013-12-29-07-03-31")

### Activate Debugging until Android 3.2

- go to Settings from your Application Menu
- choose Applications
- open the Development item
- you will see now the option to activate USB debugging:

![Android_debug_until_32](/assets/img/2013/12/Android_debug_until_32.png "Android_debug_until_32")

### Activate Debugging on Android 4.0, 4.1

- go to the Settings Menu
- choose Developer options and activate USB debugging:

![Android_debug_until_32](/assets/img/2013/12/developer_options-40_41.png "Android_debug_until_32")

### Activate Debugging on Android 4.2 and higher

- go to the Settings Menu
- select About
- tap the Build Number 7 times (after the 4th tap a notification will appear that counts down the required taps)

![Screenshot_2013-12-29-07-04-36](/assets/img/2013/12/Screenshot_2013-12-29-07-04-36.png "Screenshot_2013-12-29-07-04-36")

### Installing drivers

Now our devices are ready to be connected to our PC, where we need to install the corresponding USB drivers. Normally you just need to plug in your Android device, Windows will search for the right driver and install it for you. If you want to install the driver manually, here is a list of OEM driver packages: [https://developer.android.com/tools/extras/oem-usb.html#Drivers](https://developer.android.com/tools/extras/oem-usb.html#Drivers "https://developer.android.com/tools/extras/oem-usb.html#Drivers")

### Deploying an App for debugging

After the drivers have been installed, your device should now be recognized by Windows and also by Xamarin Studio.

Using our gettingstarted project from my first blog post, you should have now two options to deploy the application to your device:

#### Run With…

![Screenshot (273)](/assets/img/2013/12/Screenshot-273.png "Screenshot (273)")

#### Just hit ‘Debug’…

![Screenshot (275)](/assets/img/2013/12/Screenshot-275.png "Screenshot (275)")

Xamarin Studio installs now the required packages for debugging, and after that, you will be able to debug you application.

### Speeding up re-deployments

Xamarin will install all developing packages including your app from scratch every time you deploy the app. To speed things up, there is an option to speed things a little up.

To activate/check if this option is already activated, click on the ‘options’ button of your project and select ‘Options’ in the context menu:

![Screenshot (276)](/assets/img/2013/12/Screenshot-276.png "Screenshot (276)")

Select ‘Android Build’ and activate these two options:

![Screenshot (276)](/assets/img/2013/12/Screenshot-2761.png "Screenshot (276)")

This way, Xamarin will deploy only changes that have been made to your code, while keeping the required packages on your device. Xamarin Studio will detect if an update to those packages is required.

However, sometimes you will need to clean your solution and do a complete re-deploy (like you are doing with a Windows Phone project sometimes, too).

In very seldom cases (happened 3 times to me by now), you will need to uninstall your app and all of the Xamarin packages from your device. To do so, go to Settings/Apps on your device. In the application list, you will find some apps that start with ‘Mono’:

![Screenshot_2013-12-29-07-57-13](/assets/img/2013/12/Screenshot_2013-12-29-07-57-13.png "Screenshot_2013-12-29-07-57-13")

To uninstall them, tap the items and select ‘Uninstall’ in the App info screen:

![Screenshot_2013-12-29-07-58-43](/assets/img/2013/12/Screenshot_2013-12-29-07-58-43.png "Screenshot_2013-12-29-07-58-43")

Xamarin Studio will now re-deploy all required packages to your device, and your app should run again for debugging.

Like always, I hope this post is helpful for some of you.

Happy coding!