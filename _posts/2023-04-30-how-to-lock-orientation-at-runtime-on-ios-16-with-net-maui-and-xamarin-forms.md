---
id: 7306
title: 'How to lock orientation at runtime on iOS 16 with .NET MAUI and Xamarin.Forms'
date: '2023-04-30T08:29:08+02:00'
author: 'Marco Siccardi'
excerpt: 'With iOS 16, Apple made some old APIs non-functional. This includes also the established way of locking the orientation. In this post, I am going to show you how you can lock orientation on iOS 16 while the app is running with both .NET MAUI and Xamarin.Forms. '
layout: post
permalink: /how-to-lock-orientation-at-runtime-on-ios-16-with-net-maui-and-xamarin-forms/
image: /assets/img/2023/04/ios16_orientation_lock_title_bing_create.jpeg
categories:
    - Xamarin
    - 'Dev Stories'
    - iOS
    - MAUI
tags:
    - '.NET MAUI'
    - AppDelegate
    - iOS
    - landscape
    - MAUI
    - mvvm
    - 'orientaion lock'
    - orientation
    - override
    - portrait
    - Scene
    - ViewController
    - xamarin
    - 'xamarin forms'
---

### The old way

Before iOS 16, it was pretty easy to lock a `Page` into a certain orientation. It was basically just one line of code (if you don’t count the `DependencyService` boilerplate code in):

``` csharp
 UIDevice.CurrentDevice.SetValueForKey(new NSNumber((int)UIInterfaceOrientation.Portrait), new NSString("orientation"));
```
 
By calling this method whenever the size of a page was allocated, we were able to lock the orientation at runtime with Xamarin.Forms. With iOS 16, this does no longer work – even on native iOS applications.

### The new way

To understand the why of the new way, you have to understand the `SceneDelegate` architecture Apple introduced with iOS 13. Before continuing, you should read this blog post by [Donny Wals](https://www.linkedin.com/in/donny-wals-33660014/), which explains it very detailed: [Understanding the iOS 13 Scene Delegate – Donny Wals](https://www.donnywals.com/understanding-the-ios-13-scene-delegate/).

Now that we know that the `SceneDelegate` is, we can move on with our implementation.

### Page implementation

Both Xamarin.Forms and .NET MAUI implement the `SceneDelegate` architecture. That’s why we can update our code similarly to what native iOS implementations look like:

``` csharp
 var rootWindowScene = (UIApplication.SharedApplication.ConnectedScenes.ToArray()?.FirstOrDefault()) as UIWindowScene;

if (rootWindowScene == null)
    return;

rootWindowScene.RequestGeometryUpdate(new UIWindowSceneGeometryPreferencesIOS(UIInterfaceOrientationMask.Portrait),
error =>
{
    Debug.WriteLine("Error while attempting to lock orientation: {Error}", error.LocalizedDescription);
});
```
 
On top, we have to tell the underlying `ViewController`s to update their orientation as well:

``` csharp
 var rootViewController = UIApplication.SharedApplication.KeyWindow?.RootViewController;

if (rootViewController == null)
    return;

rootViewController.SetNeedsUpdateOfSupportedInterfaceOrientations();
rootViewController.NavigationController?.SetNeedsUpdateOfSupportedInterfaceOrientations();
```
 
The `ViewController` can be informed via the `SetNeedsUpdateOfSupportedInterfaceOrientations` method that it needs to redraw its view. If we put this all together, we can have a reusable implementation for our `DeviceOrientationService` implementation:

``` csharp
 private void SetOrientation(UIInterfaceOrientationMask uiInterfaceOrientationMask)
{
    var rootWindowScene = (UIApplication.SharedApplication.ConnectedScenes.ToArray()?.FirstOrDefault()) as UIWindowScene;
    
    if (rootWindowScene == null)
        return;
    
    var rootViewController = UIApplication.SharedApplication.KeyWindow?.RootViewController;

    if (rootViewController == null)
        return;
    
    rootWindowScene.RequestGeometryUpdate(new UIWindowSceneGeometryPreferencesIOS(uiInterfaceOrientationMask),
    error =>
    {
        Debug.WriteLine("Error while attempting to lock orientation: {Error}", error.LocalizedDescription);
    });
    
    rootViewController.SetNeedsUpdateOfSupportedInterfaceOrientations();
    rootViewController.NavigationController?.SetNeedsUpdateOfSupportedInterfaceOrientations();
}
```
 
To keep our existing code for older iOS versions working as well. We now just check if we are on iOS 16 and call our new method, below we still can use our traditional way:

``` csharp
 public void LockPortrait()
{
    if (UIDevice.CurrentDevice.CheckSystemVersion(16, 0))
    {
        _applicationDelegate.CurrentLockedOrientation = UIInterfaceOrientationMask.Portrait;
    
        SetOrientation(UIInterfaceOrientationMask.Portrait);
    }
    else
    {
        UIDevice.CurrentDevice.SetValueForKey(new NSNumber((int)UIInterfaceOrientation.Portrait), new NSString("orientation"));
    }
}
```
 
This will, however, do nothing without the last, very important step. You may have noticed the `CurrentLockedOrientation` property on the application delegate member above.

Every time the application has to decide whether to rotate or not, the `application:supportedInterfaceOrientationsForWindow:` gets called to ask for the supported orientations. Only if the application and the `ViewController` agree on the supported orientations, the action will be executed.

### Extending our `AppDelegate`

Just as on native iOS, we need to implement the method above in our `AppDelegate`. Xamarin and .NET MAUI do this via the `Export` attribute, which tells the compiler to override the eventually existing native implementation.

For my solution, I created a derived version of the `FormsApplicationDelegate` / `MauiUIApplicationDelegate` classes, passing the current desired `UIInterfaceOrientationMask` value to the `CurrentLockedOrientation` property. Finally, I implement the `GetSupportedInterfaceOrientationsForWindow` method and just return the value of the `CurrentLockedOrientation` property:

``` csharp
 public abstract class AppDelegateEx : MauiUIApplicationDelegate
{
    public virtual UIInterfaceOrientationMask CurrentLockedOrientation { get; set; }

    //according to the Apple docs, Application and ViewController have to agree on the supported orientation, this forces it
    //https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623107-application?language=objc
    [Foundation.Export("application:supportedInterfaceOrientationsForWindow:")]
    public virtual UIInterfaceOrientationMask GetSupportedInterfaceOrientationsForWindow(UIApplication application, UIWindow forWindow)
        => this.CurrentLockedOrientation;
}
```
 
Now we just make the `AppDelegate` derive from `AppDelegateEx` (or whatever you call it) to finish the implementation for the orientation lock. Finally, locking the orientation works also on iOS 16.

### Samples

I created two samples – one for Xamarin.Forms and one for .NET MAUI. The sample work similar on both platforms, and I wrote the code in a reusable way. You can find the samples [in the corresponding GitHub repo](https://github.com/MSicc/Ios16OrientationLockSample).

## Conclusion

It took me some time to figure out why the traditional way of locking the orientation doesn’t work any longer. After some research and some trial-and-error coding, I was able to come up with a clean and easy-to use solution, which is also reusable. I also learned some new things, like how MAUI implements `Scenes` and `ViewControllers` and got a better understanding of the iOS application structure and lifecycle on newer OS versions.

As always, I hope this post will be helpful for some of you as well.

#### Until the next post, happy coding!

---

Helpful links:

- [Understanding the iOS 13 Scene Delegate – Donny Wals](https://www.donnywals.com/understanding-the-ios-13-scene-delegate/)
- [swift – How to resolve: ‘keyWindow’ was deprecated in iOS 13.0 – Stack Overflow](https://stackoverflow.com/questions/57134259/how-to-resolve-keywindow-was-deprecated-in-ios-13-0)
- [requestGeometryUpdateWithPreferences:errorHandler: | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiwindowscene/3975944-requestgeometryupdatewithprefere?language=objc)
- [application:supportedInterfaceOrientationsForWindow: | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623107-application?language=objc)
- [(UIInterfaceOrientationMask)applic… | Apple Developer Forums](https://developer.apple.com/forums/thread/715052#715052021)
- [Forcing specific MAUI view to Landscape Orientation using MultiTargeting Feature working for Android but not iOS – Stack Overflow](https://stackoverflow.com/questions/74838009/forcing-specific-maui-view-to-landscape-orientation-using-multitargeting-feature)

---

[Title Image created via Bing Create with AI](https://ios16_orientation_lock_title_bing_create)