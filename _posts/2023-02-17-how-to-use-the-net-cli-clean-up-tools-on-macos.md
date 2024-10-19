---
id: 7241
title: 'How to use the .NET CLI clean-up tools on macOS'
date: '2023-02-17T08:48:09+01:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I will show you how to use the .NET CLI tools on macOS to clean-up installed SDKs, runtimes, workloads, and NuGet caches.  '
layout: post
permalink: /how-to-use-the-net-cli-clean-up-tools-on-macos/
image: /assets/img/2023/02/macos-cleanup-net-cli-title.png
categories:
    - 'Dev Stories'
    - macOS
    - MAUI
    - Xamarin
tags:
    - .NET
    - '.NET MAUI'
    - CLI
    - 'CLI tools'
    - commands
    - dotNET
    - macOS
    - Nuget
    - 'Nuget package'
    - Runtime
    - SDK
    - Terminal
    - tools
    - workload
    - workloads
---

Earlier this week, the .NET SDK and Runtimes received some updates. Together with that, also Visual Studio for Mac was updated. Once I got past the installation of all updates, both Visual Studio and Rider were no longer restoring the required NuGet packages for my .NET MAUI project running on .NET 6.

I eventually fixed that issue by cleaning up all the .NET SDKs, Runtimes, workloads and NuGet caches on my MacBook Pro. Read on to learn about the tools I used.

### .NET uninstall tool

I have been using the .NET uninstall tool in the past. Unlike on Windows, you have to download the executable from GitHub in its zipped form.

While [the releases page](https://github.com/dotnet/cli-lab/releases) shows some terminal commands to unpack and run the tool, they never worked for me as stated there. While I was able to make the new directory with the `mkdir` command, the unpacking always shows an error. So I opened up Finder and unzipped it manually with the *Archive Utility* app that ships with macOS.

<div class="wp-block-image"><figure class="aligncenter size-full is-resized">![dotnet-core-uninstall unzipping](https://msicc.net/assets/img/2023/02/SCR-20230217-9ok.png)</figure></div>After switching to the folder in Terminal, the tool is supposed to show the help. Instead, I got an error showing me that I am not allowed to run this app for security reasons. The OS blocks the execution. If the same happens for you, right click on the extracted executable and select “*Open With*” followed by “*Terminal.app (default)*“. This will prompt you with this screen:

<div class="wp-block-image"><figure class="aligncenter size-full is-resized">![app downloaded from the internet message](https://msicc.net/assets/img/2023/02/SCR-20230217-9zv.png)</figure></div>Once you click on “*Open*“, a new Terminal window appears. Close this window, it is unusable as we are already in the exited state. Instead, open a new Terminal and change to the installation folder and call the help command:

``` shell
 cd ~/dotnet-core-uninstall
./dotnet-core-uninstall -h
```
 
<div class="wp-block-image"><figure class="aligncenter size-full is-resized">![Terminal with dotnet-core-uninstall help](https://msicc.net/assets/img/2023/02/SCR-20230217-a4t.png)</figure></div>Now that we are able to run the tool, let’s have a look what we have installed by running the `dotnet --list` command. We need to call the command twice, once for the installed `-sdks` and once for the installed `-runtimes`:

``` shell
 dotnet --list-sdks
dotnet --list-runtimes
```
 
You may be as surprised (I was, at least) how many versions you are accumulating over time. They never get removed by newer versions (it’s by design, according to Microsoft). To get rid of all versions except the latest, run the following commands with the uninstall tool (again once for — sdk, once for –runtime):

``` shell
 sudo ./dotnet-core-uninstall remove --all-but-latest --sdk
sudo ./dotnet-core-uninstall remove --all-but-latest --runtime
```
 
After uninstalling all previous versions, you may [have to reinstall the latest .NET 6 SDK again](https://learn.microsoft.com/en-us/dotnet/core/install/macos). You could also use the –`-all-but [Versions]` command to specify the versions explicitly. No matter which way you’re going, if you run the `dotnet --list` commands again, you should see something similar to this:

<div class="wp-block-image"><figure class="aligncenter size-full is-resized">![dotnet --list command](https://msicc.net/assets/img/2023/02/SCR-20230217-ahq.png)</figure></div>Download: [https://github.com/dotnet/cli-lab/releases ](https://github.com/dotnet/cli-lab/releases)

Documentation: [https://learn.microsoft.com/en-us/dotnet/core/additional-tools/uninstall-tool?tabs=macos#step-3—uninstall-net-sdks-and-runtimes](https://learn.microsoft.com/en-us/dotnet/core/additional-tools/uninstall-tool?tabs=macos#step-3---uninstall-net-sdks-and-runtimes)

### dotnet workload command

As I had problems getting the required NuGet packages for my MAUI app, I decided to uninstall all .NET MAUI workloads as well. First, I had a look what is installed with the `list` command:

``` shell
 dotnet workload list
```
 
<div class="wp-block-image"><figure class="aligncenter size-large is-resized">![dotnet workload list result](https://msicc.net/assets/img/2023/02/SCR-20230217-b0b-1024x355.png)</figure></div>Once you have that list, you need to call the `uninstall` command for every single installed workload:

``` shell
 sudo dotnet workload uninstall macos maui-maccatalyst maui-ios maui-android ios maccatalyst maui tvos android
```
 
Once they are uninstalled, I cleared the Terminal and installed them all again using the `install` command:

``` shell
 sudo dotnet workload install macos maui-maccatalyst maui-ios maui-android ios maccatalyst maui tvos android
```
 
<div class="wp-block-image"><figure class="aligncenter size-large is-resized">![dotnet workload install result in Terminal](https://msicc.net/assets/img/2023/02/SCR-20230217-b7p-1024x557.png)</figure></div>Now we have the latest .NET MAUI workload installed as well as the platform specific workloads as well.

Documentation: <https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-workload>

### dotnet nuget locals

The final clean-up step involves all NuGet caches on your machine. Yes, you read that right, multiple caches. To see them all, run the following command:

``` shell
 dotnet nuget locals all --list
```
 
This will get you something like this:

<div class="wp-block-image"><figure class="aligncenter size-large is-resized">![dotnet nuget locals cache results in terminal](https://msicc.net/assets/img/2023/02/SCR-20230217-bgo-1024x183.png)</figure></div>Now let’s get rid of all those old NuGet packages:

``` shell
 sudo dotnet nuget locals all --clear
```
 
If you’re lucky, you will see this message:

<div class="wp-block-image"><figure class="aligncenter size-large is-resized">![local nuget caches cleared in terminal](https://msicc.net/assets/img/2023/02/SCR-20230217-bkf-1024x199.png)</figure></div>My first attempt was not that successful. I needed to open the global packages’ folder in Finder and delete some remaining packages manually. Only after that, I was able to run the `clear` command with success.

### Conclusion

Neither Visual Studio nor the .NET installer perform clean-up tasks on macOS. Until Microsoft changes their mind here, we will have to clean-up old packages manually to keep our system smoothly running. Luckily, there are at least CLI tools around to help us with that job. As always, I hope this blog post will be helpful for some of you.

#### Until the next post, happy coding, everyone!