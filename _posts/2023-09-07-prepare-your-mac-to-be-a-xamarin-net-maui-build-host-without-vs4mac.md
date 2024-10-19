---
id: 7721
title: 'Prepare your Mac to be a Xamarin/.NET MAUI build host without VS4MAC [Updated]'
date: '2023-09-07T10:00:00+02:00'
author: 'Marco Siccardi'
excerpt: 'Microsoft will discontinue Visual Studio for Mac (VS4MAC) in 2024. To be able to compile for the App Store, we still need a Mac based build host. In this post, I''ll show you how to prepare a Mac without VS4MAC as such a host.'
layout: post
permalink: /prepare-your-mac-to-be-a-xamarin-net-maui-build-host-without-vs4mac/
image: /assets/img/2023/09/Connect-Mac-Build-Host-No-VS4MAC-Title.png
categories:
    - 'Dev Stories'
    - MAUI
    - Xamarin
tags:
    - '.NET MAUI'
    - IDE
    - iOS
    - Mac
    - macOS
    - MAUI
    - 'Visual Studio'
    - VS4Mac
    - Windows
    - xamarin
---

## Overview

You can connect to a Mac Build host from your Windows machine without Visual Studio for Mac installed. All you need is to install the right packages, which are (luckily and for the time being), available to be downloaded separately. You just need to select the right versions that match your Visual Studio installation on Windows.

## Install .NET 

Of course, the first step is to install the .NET SDK. Head over to the .NET website and download the latest Release and/or Preview version.

<https://dotnet.microsoft.com/en-us/download>

### Install Mono

As Xamarin and MAUI rely on the Mono framework, we also need to install it manually. I recommend the stable version. You’ll need to install it with right-click and selecting *Open* to unblock the installation file.

<https://www.mono-project.com/download/stable/>

## Install Xamarin packages

The next step involves installing the Xamarin packages needed for compilation. Without them installed, the connection attempt will always fail. Only with them installed, the connection process can install the needed additional packages.

### Xamarin.iOS and Xamarin.Mac

The Xamarin.iOS and Xamarin.Mac packages can be found on GitHub:

<https://github.com/xamarin/xamarin-macios/blob/main/DOWNLOADS.md>

Download and install the version that matches your Windows installation. In case you’re wondering about the Mac package – it seems to be a dependency for the build process. Only after installing it, I was able to connect from my VM.

### Xamarin.Android

If you want to use a [local Android emulator installed on your Mac, like I described here](https://msicc.net/how-to-use-the-android-emulator-on-a-macos-host-for-debugging-in-a-virtual-machine-with-windows/), you will need to install the Xamarin.Android package as well. The installer is on GitHub as well:

<https://aka.ms/xamarin-android-commercial-d17-5-macos>

You may need to download Android SDK tools as well:

<https://developer.android.com/tools>

## Install MAUI workloads

If you want to compile .NET MAUI apps for the App Store, you will need to install also the MAUI workloads on your Mac. The process is well described in the Microsoft documentation:

<https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-workload>

## Connect to the Mac

Once you have installed all the packages above, you should be able to connect with your Mac for compiling iOS applications:

<div class="wp-block-image"><figure class="aligncenter size-large">![VS2022 connected to a build host in the Pair to Mac dialog.](https://msicc.net/assets/img/2023/09/Mac-Build-Host-Connected-1024x446.jpg)</figure></div>## Conclusion

Even though Microsoft is killing VS4Mac, we still are able to connect from Windows machines to a Mac build host. The process is done in less than an hour and just involves some package installations. Of course, there is always the alternative of using other IDEs like Rider or VS Code (.NET MAUI support is still beta as of writing this post). I recommend keeping the downloaded packages in a save place (just in case). As always, I hope this post is helpful for some of you.

#### Until the next post, happy coding!

*Update 2023-09-29: Added Mono installation step*