---
id: 7974
title: 'Handling lifecycle events on iOS and MacCatalyst  with .NET MAUI'
date: '2024-07-24T16:00:00+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I''ll show you how to handle the application lifecycle events on iOS and MacCatalyst with .NET MAUI.'
layout: post
permalink: /handling-lifecycle-events-on-ios-and-maccatalyst-with-net-maui/
image: /assets/img/2024/07/App-lifecycle-title-image.png
categories:
    - 'Dev Stories'
    - macOS
    - MAUI
tags:
    - '.NET MAUI'
    - app
    - appearance
    - Apple
    - application
    - endif
    - Events
    - if
    - iOS
    - iosdev
    - lifecycle
    - MacCatalyst
    - macdev
    - macOS
    - Scene
    - Traits
    - UIWindowSceneDelegate
    - WindowSceneDidUpdateCoordinateSpace
---

My current side project is running on iOS and MacCatalyst. Even if both of them are able to share a lot of code, there are some challenges that you should be aware of if you’re going down this route. One of these challenging areas is the handling of the lifecycle events.

### Read the docs first

Before you continue to read this post, I recommend reading [the MAUI documentation about the app lifecycle](https://learn.microsoft.com/en-us/dotnet/maui/fundamentals/app-lifecycle?view=net-maui-8.0) to get a basic understanding.

You’re back? Great choice!

### Handling lifecycle events following the docs

Let’s create a new .NET MAUI application. According to the documentation, we should be able to hook into the application’s platform lifecycle events like this:

``` csharp
 public static MauiApp CreateMauiApp()
{
    var builder = MauiApp.CreateBuilder();
    builder
        .UseMauiApp<App>()
        .ConfigureFonts(fonts =>
        {
            fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
            fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");

        })
        .ConfigureLifecycleEvents(events =>
        {
#if IOS || MACCATALYST
                    events.AddiOS(ios => ios
                        .OnActivated((app) => LogEvent(nameof(iOSLifecycle.OnActivated)))
                        .OnResignActivation((app) => LogEvent(nameof(iOSLifecycle.OnResignActivation)))
                        .DidEnterBackground((app) => LogEvent(nameof(iOSLifecycle.DidEnterBackground)))
                        .WillEnterForeground((app) => LogEvent(nameof(iOSLifecycle.WillEnterForeground)))
                        .WillTerminate((app) => LogEvent(nameof(iOSLifecycle.WillTerminate))));
#endif
            static bool LogEvent(string eventName, string type = null)
            {
                Console.WriteLine($"Lifecycle event: {eventName}{(type == null ? string.Empty : $" ({type})")}");
                return true;
            }
        });

#if DEBUG
    builder.Logging.AddDebug();
#endif

    return builder.Build();
}
``` 

#### Testing on iOS

Let’s test the code on iOS. I am using the following pattern:

- App start
- App to background
- App to foreground
- close App by swiping up

These are the results:

``` shell
 2024-07-23 17:33:07.037 ApplelifeCycleEventsTests[44614:19300605] Lifecycle event: OnActivated
2024-07-23 17:33:12.953 ApplelifeCycleEventsTests[44614:19300605] Lifecycle event: OnResignActivation
2024-07-23 17:33:16.628 ApplelifeCycleEventsTests[44614:19300605] Lifecycle event: DidEnterBackground
2024-07-23 17:33:20.371 ApplelifeCycleEventsTests[44614:19300605] Lifecycle event: WillEnterForeground
2024-07-23 17:33:20.677 ApplelifeCycleEventsTests[44614:19300605] Lifecycle event: OnActivated
2024-07-23 17:33:22.449 ApplelifeCycleEventsTests[44614:19300605] Lifecycle event: OnResignActivation
2024-07-23 17:33:24.527 ApplelifeCycleEventsTests[44614:19300605] Lifecycle event: DidEnterBackground
2024-07-23 17:33:24.543 ApplelifeCycleEventsTests[44614:19300605] Lifecycle event: WillTerminate
```
 
As you can see, we are hitting all delegate handlers we specified. Exactly what we wanted.

#### Testing on macOS

If we follow the same pattern on macOS (App start, minimize, bring back to foreground, close), these are the results:

``` shell
 2024-07-23 17:21:21.158 ApplelifeCycleEventsTests[33400:1531238] Lifecycle event: OnActivated
2024-07-23 17:21:34.679 ApplelifeCycleEventsTests[33400:1531238] Lifecycle event: OnResignActivation
2024-07-23 17:21:34.680 ApplelifeCycleEventsTests[33400:1531238] Lifecycle event: DidEnterBackground
2024-07-23 17:21:34.685 ApplelifeCycleEventsTests[33400:1531238] Lifecycle event: WillTerminate
```
 
Well, not what we expected. We are getting only the `OnActivated` event when the app starts, and the `OnResignActivation-DidEnterBackground-WillTerminate` combo when the app closes. This does not help to detect if the app is currently the top most window on the screen.

### `UIWindowSceneDelegate` to the rescue

Some years ago, Apple introduced `UIWindowScene` to handle everything related to the app UI ([Apple docs](https://developer.apple.com/documentation/uikit/uiwindowscene?language=objc)). In order to get the events from our MacCatalyst app, we need to update our MAUI app to use such a `Scene`.

First, create a new class that derives from `MauiUISceneDelegate` and register the class to the macOS system:

``` csharp
 [Register(nameof(MySceneDelegate))]
public class MySceneDelegate : MauiUISceneDelegate
{

}
```
 
Once you have that class in place, update your` Info.plist` file by adding these lines to the end of the file:

``` xml
 <key>UIApplicationSceneManifest</key>
<dict>
	<key>UIApplicationSupportsMultipleScenes</key>
	<false/>
	<key>UISceneConfigurations</key>
	<dict>
		<key>UIWindowSceneSessionRoleApplication</key>
		<array>
			<dict>
				<key>UISceneConfigurationName</key>				<string>__MAUI_DEFAULT_SCENE_CONFIGURATION__</string>
				<key>UISceneDelegateClassName</key>
				<string>MySceneDelegate</string>
			</dict>
		</array>
	</dict>
</dict>
```
 
Please note that setting `UIApplicationSupportsMultipleScenes` to true would enable multi-window support ([see more on my post about handling windows on MacCatalyst]({% post_url 2023-11-03-dealing-with-application-windows-on-macos-with-net-maui %})).

### Handle the Scene’s events

After setting the app up to use the `UIWindowSceneDelagate`, we will be able to handle the lifecycle events in our MacCatalyst app. Let’s update the code to handle the newly available events:

``` csharp
 public static MauiApp CreateMauiApp()
{
    var builder = MauiApp.CreateBuilder();
    builder
        .UseMauiApp<App>()
        .ConfigureFonts(fonts =>
        {
            fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
            fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");

        })
        .ConfigureLifecycleEvents(events =>
        {
#if IOS || MACCATALYST
            events.AddiOS(ios => ios
                        .OnActivated((app) => LogEvent(nameof(iOSLifecycle.OnActivated)))
                        .OnResignActivation((app) => LogEvent(nameof(iOSLifecycle.OnResignActivation)))
                        .DidEnterBackground((app) => LogEvent(nameof(iOSLifecycle.DidEnterBackground)))
                        .WillEnterForeground((app) => LogEvent(nameof(iOSLifecycle.WillEnterForeground)))
                        .SceneOnActivated(scene => LogEvent(nameof(iOSLifecycle.SceneOnActivated)))
                        .SceneOnResignActivation(scene => LogEvent(nameof(iOSLifecycle.SceneOnResignActivation)))
                        .SceneDidEnterBackground(scene => LogEvent(nameof(iOSLifecycle.SceneDidEnterBackground)))
                        .SceneWillEnterForeground(scene => LogEvent(nameof(iOSLifecycle.SceneWillEnterForeground)))
                        .WindowSceneDidUpdateCoordinateSpace(((scene, space, orientation, collection) => CheckApplicationActiveAppearance(scene, collection)))
                        .WillTerminate((app) => LogEvent(nameof(iOSLifecycle.WillTerminate))));
#endif
            static bool LogEvent(string eventName, string type = null)
            {
                Console.WriteLine($"Lifecycle event: {eventName}{(type == null ? string.Empty : $" ({type})")}");
                return true;
            }
            
            static void CheckApplicationActiveAppearance(UIWindowScene windowScene, UITraitCollection previousTraitCollection)
            {
                var newActiveAppearance = windowScene.TraitCollection.ActiveAppearance;
                LogEvent($"{nameof(iOSLifecycle.WindowSceneDidUpdateCoordinateSpace)}:{newActiveAppearance}");
        
                if (newActiveAppearance != previousTraitCollection.ActiveAppearance &&
                    newActiveAppearance == UIUserInterfaceActiveAppearance.Active)
                {
                    Console.WriteLine("Application is now the top most windw on screen");
                }
            }
        });

#if DEBUG
    builder.Logging.AddDebug();
#endif

    return builder.Build();
}
```
 
Most of the scene events should be familiar, they are practically the same on the `UIApplication` class. We also have one new event, however: `WindowSceneDidUpdateCoordinateSpace` ([Apple docs](https://developer.apple.com/documentation/uikit/uiwindowscenedelegate/3198094-windowscene?language=objc)). This event allows us to determine if our app is the top most app by querying the `ActiveAppearance` property of the current and previous traits. Please note that the event gets fired multiple times, and you should never forget to compare the new appearance with the previous one.

Following our test scenario (App start, minimize, bring back to foreground, close), we are getting now the following output:

``` shell
 2024-07-24 06:38:27.225 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: SceneWillEnterForeground
2024-07-24 06:38:27.226 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: SceneOnActivated
2024-07-24 06:38:27.411 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: WindowSceneDidUpdateCoordinateSpace:Active
2024-07-24 06:38:30.088 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: SceneOnResignActivation
2024-07-24 06:38:30.089 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: SceneDidEnterBackground
2024-07-24 06:38:30.093 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: WindowSceneDidUpdateCoordinateSpace:Inactive
2024-07-24 06:38:30.094 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: WindowSceneDidUpdateCoordinateSpace:Inactive
2024-07-24 06:38:33.345 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: SceneWillEnterForeground
2024-07-24 06:38:33.345 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: SceneOnActivated
2024-07-24 06:38:33.350 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: WindowSceneDidUpdateCoordinateSpace:Active
2024-07-24 06:38:33.350 ApplelifeCycleEventsTests[37848:1849326] Application is now the top most windw on screen
2024-07-24 06:38:33.351 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: WindowSceneDidUpdateCoordinateSpace:Active
2024-07-24 06:38:36.055 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: SceneOnResignActivation
2024-07-24 06:38:36.055 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: SceneDidEnterBackground
2024-07-24 06:38:36.058 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: WindowSceneDidUpdateCoordinateSpace:Inactive
2024-07-24 06:38:36.059 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: WindowSceneDidUpdateCoordinateSpace:Inactive
2024-07-24 06:38:36.072 ApplelifeCycleEventsTests[37848:1849326] Lifecycle event: WillTerminate
```
 
As you can see, we are getting way more information now and can respond to the lifecycle events of our application on macOS in the same way we can on iOS.

## Conclusion

Handling the lifecycle events can be essential to an app. With the code above, you should be able to handle all the different states of both your iOS and your MacCatalyst app. As always, I hope this post will be helpful for some of you.

#### Until the next post, happy coding!