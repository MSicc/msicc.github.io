---
id: 7293
title: 'Deploy MAUI apps with Rider on your iOS device after these Xcode errors'
date: '2023-04-09T08:31:09+02:00'
author: 'Marco Siccardi'
excerpt: 'In this short post, I show you how to make Rider deploy .NET MAUI iOS apps after getting Xcode errors about missing watchOS and tvOS extensions.'
layout: post
permalink: /deploy-maui-apps-with-rider-on-your-ios-device-after-these-xcode-errors/
image: /assets/img/2023/04/Screenshot-2023-04-09-at-07.20.21.png
categories:
    - 'Dev Stories'
    - iOS
    - macOS
    - MAUI
tags:
    - '.NET MAUI'
    - Debug
    - Deploy
    - dotNET
    - IDE
    - iOS
    - Mac
    - MAUI
    - Rider
    - 'Visual Studio'
    - VS4Mac
    - Xcode
---

## The Error

I am working a lot with .NET MAUI lately (both at work and for my private projects) and I use JetBrains Rider as the primary IDE on macOS. If you try to deploy an iOS app, your first attempt will likely fail with a big red error message like in the picture above.

The details tell us that the error is coming from missing Xcode extensions:

``` shell
 Failed to install application on device MSicc‘s iPhone 13 Pro: 
2023-04-07 07:41:25.382 mlaunch [18882:2811826] Requested but did not find extension point with identifier Xcode.IDEDebugger.VariablesViewQuickLookProvider for extension Xcode.IDEDebugger.SpriteKitQuickLookProvider of plug-in com.apple.IDESpriteKitParticleEditor 
2023-04-07 07:41:25.384 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.IDEDebugger.VariablesViewQuickLookProvider for extension Xcode.SpriteKit.GKStateMachineQuickLookProvider of plug-in com.apple.IDESpriteKitParticleEditor 
2023-04-07 07:41:25.476 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.WatchOS.WatchApplication of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.476 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.DebuggerFoundation.DataSourceConnection for extension Xcode.DebuggerFoundation.watchOSSimulator.DataSourceConnectionTargetHub of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.476 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.WatchOS.Application of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.476 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.WatchOS.Tool of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.DebuggerFoundation.ViewDescriber for extension Xcode.DebuggerFoundation.watchOSSimulator.ViewDescriber of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.WatchOS.IntentsService-AppExtension of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.InfoEditorType for extension Xcode.Xcode3ProjectSupport.InfoEditorType.WatchOS.Bundle of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.WatchOS.WatchKit2-AppExtension of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.InfoEditorSlice for extension Xcode.Xcode3ProjectSupport.InfoEditorSlice.WatchOS.BundleInfo of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.DebuggerFoundation.ViewDescriber for extension Xcode.DebuggerFoundation.watchOS.ViewDescriber of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.WatchOS.Framework of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.WatchOS.ExtensionKitAppExtension of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.WatchOS.AppExtension of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.DebuggerFoundation.DataSourceConnection for extension Xcode.DebuggerFoundation.watchOS.DataSourceConnectionTargetHub of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.477 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.IDEiPhoneSupport.TargetEditor for extension Xcode.IDEiPhoneSupport.TargetEditor.WatchOS.Application of plug-in com.apple.dt.IDEWatchSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.IDEAppleTVSupportUIFramework of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.IDEAppleTVSupportUI.Application of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.IDEAppleTVSupportUI.AppExtension of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.DebuggerFoundation.DataSourceConnection for extension Xcode.DebuggerFoundation.tvOSSimulator.DataSourceConnectionTargetHub of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.IDEAppleTVSupportUI.ExtensionKitAppExtension of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.InfoEditorType for extension Xcode.Xcode3ProjectSupport.InfoEditorType.appletvos.Bundle of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.DebuggerFoundation.ViewDescriber for extension Xcode.DebuggerFoundation.ATVSimulator.ViewDescriber of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.DebuggerFoundation.DeviceIconProvider for extension Xcode.DebuggerFoundation.DeviceIconProvider.AppleTV of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.DebuggerFoundation.ViewDescriber for extension Xcode.DebuggerFoundation.ATV.ViewDescriber of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.DebuggerFoundation.DataSourceConnection for extension Xcode.DebuggerFoundation.tvOS.DataSourceConnectionTargetHub of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.TargetSummaryEditor for extension Xcode.Xcode3ProjectSupport.TargetSummaryEditor.IDEAppleTVSupportUI.XPC of plug-in com.apple.dt.IDEAppleTVSupportUI 
2023-04-07 07:41:25.479 mlaunch[18882:2811826] Requested but did not find extension point with identifier Xcode.Xcode3ProjectSupport.InfoEditorSlice for extension Xcode.Xcode3ProjectSupport.InfoEditorSlice.appletvos.BundleTargetInfo of plug-in com.apple.dt.IDEAppleTVSupportUI 
error MT1006: Could not install the application '/Users/msiccdev/Git/RunnersTools/src/MSiccDev.RunnersTools.Client/bin/Debug/net7.0-ios/iossimulator-x64/MSiccDev.RunnersTools.App.app' on the device 'MSicc‘s iPhone 13 Pro': AMDeviceSecureInstallApplicationBundle returned: 0xe800801c.
```
 
### What the internet says…

Searching the web for this kind of errors will likely lead you to some solutions/workarounds about `CommandLineTools`. They all basically recommend the following approach:

- uninstall existing Xcode `CommandLineTools` installations
- install them again (best via Terminal)
- set the `CommandLineTools` variable in Xcode and other IDEs to the separately installed tools

**Spoiler alert**: None of them work in this case.

## The solution

The solution for this problem is to modify the `.csproj` file and the `info.plist` files of your .NET MAUI app. Visual Studio for Mac does these changes implicitly for you, this is something the Rider team could do as well.

### Modifying the `.csproj` file

First, open the `.csproj` file in Rider by right-clicking on ‘*Edit*‘ and selecting ‘*Edit ‘YourApp,csproj*”. Add this `PropertyGroup`:

``` xml
 <PropertyGroup Condition="$(TargetFramework.Contains('-ios'))">
 <!--DEBUG ON DEVICE-->
<RuntimeIdentifier>ios-arm64</RuntimeIdentifier>
 <!--DEBUG ON SIMULATOR-->
<!--<RuntimeIdentifier>iossimulator-x64</RuntimeIdentifier>-->
</PropertyGroup>
```
 
This will be your manual switch between deploying to the simulator and deploying to your device. Just comment/uncomment the `RuntimeIdentifier` as you need.

### Modifying the `info.plist` files

The next step is to locate the `info.plist` files in your iOS (and Mac Catalyst) projects in the `Platforms` folder. Open **both of them** with by double click. At the bottom of the editor window, select ‘*Text*‘:

![Jetbrains Rider Plist Editor / Text Switch](/assets/img/2023/04/Screenshot-2023-04-09-at-08.00.13.png)
Add these lines to before the closing before the last closing `dict` tag:

``` xml
 <key>CFBundleIdentifier</key>
<string>com.companyname.appname</string>
<key>MinimumOSVersion</key>
<string>15.0</string>
```
 
You need to specify the `CFBundleIdentifier` (just copy the `ApplicationId` value from your .`csproj` file) explicitly. Same goes for the `MinimumOSVersion` – please note that you need to specify the value exactly as in the `SupportedOSPlatformVersion` property of your .`csproj` file (**for both iOS and Mac Catalyst**).

Save all of your modifications and close the solution. I recommend to manually delete the bin and object folders as well. Reopen the solution, let Rider load all the NuGet packages and recreate the bin and object stuff it needs.

When you hit the debug button, your Debug Console will still be full of red messages like those below, but you will be able to deploy (and debug) on your iOS device again.

![Jetbrains Rider Debug Console with a lot of errors.](/assets/img/2023/04/Screenshot-2023-04-09-at-08.12.44.png)


## Conclusion

Visual Studio for Mac does all these steps for you. If you try and debug the app with VS4Mac, you will notice the same error messages also there, but they are not blocking you from debugging. If you want to deploy and debug with Rider, you’ll have to perform these extra steps to get it working.

If you want to add yourself to the issues on YouTrack, you can do so on these two posts related to the issue:

<https://youtrack.jetbrains.com/issue/RIDER-76794/Cannot-deploy-MAUI-project-to-physical-iOS-device>

<https://youtrack.jetbrains.com/issue/RIDER-80950/Cant-deploy-a-.NET-MAUI-application-on-my-iPhone-SE>

As always, I hope this post will be helpful for some of you.

#### Until the next post, happy coding, everyone!