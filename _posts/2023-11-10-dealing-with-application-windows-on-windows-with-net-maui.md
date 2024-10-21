---
id: 7811
title: 'Dealing with application windows on Windows with .NET MAUI'
date: '2023-11-10T17:00:00+01:00'
author: 'Marco Siccardi'
excerpt: 'Following up my blog post about application windows on macOS, this post will show you how to deal with application windows on Windows with .NET MAUI.'
layout: post
permalink: /dealing-with-application-windows-on-windows-with-net-maui/
image: /assets/img/2023/11/dealing-with-windows-winui-maui-title.png
categories:
    - 'Dev Stories'
    - MAUI
    - Windows
tags:
    - '.NET MAUI'
    - application
    - Desktop
    - MAUI
    - mvvm
    - window
    - Windows
    - WinUI
---

As we have done the work for macOS, we can turn our attention now to the Windows operating system. Spoiler: this post will be far shorter than the last one.

### Recap

[Dealing with application windows on macOS with .NET MAUI]({% post_url 2023-11-03-dealing-with-application-windows-on-macos-with-net-maui %})

In my last post, I showed you how to support multiple windows and handle their size. I introduced you the `IWindowService` to handle windows in an MVVM friendly way and told you some gotchas on macOS.

Let’s dive into the Windows implementation.

### Supporting multiple application windows

WinUI, the underlying technology for Windows apps with MAUI, supports multiple application windows by default. There is no additional setup necessary.

To determine between the main application window and secondary ones, we still keep the `PrimaryWindow` and the `SecondaryWindow` classes around.

### Window Sizing

We are lucky again. Both the way of setting the fixed size and also dynamic sizes work also on Windows. The only difference here is that we could also resize the window manually by setting just the height and width of it. To keep the API constant, we are going to reuse the MacCatalyst implementation, however.

So once again, we can rush ahead to the next step and take a look into the MVVM part of the whole setup.

### Handling application windows with MVVM

Let’s try to remember the components of my implementation:

- Windows resizing page base class
- Secondary window base page class and its base ViewModel
- our `PrimaryWindow` and `SecondaryWindow`
- last but not least, the `IWindowService`

#### ResizeableDesktopBasePage

Named `ResizeableMacBasePage` before, the implementation created in the last post is working fine with WinUI, so I renamed it to `ResizableDesktopBasePage`. That’s all I changed, the rest of the code is still the same.

The secondary window base page now derives from the newly named class, that’s all we need to change for now. This means we can fast-forward to the `IWindowService`!

#### IWindowService

Well, I know this gets boring, but once again, we do not need to change anything. Just use the `IWindowService` in the same way as already outlined in my last post.

There is one difference in the whole construct, however.

### Sizing the main window after restart

Unlike macOS, the Windows OS does not choose the best size for our main application window once it got resized. It just goes with the default window size, except if we tell it to use another one.

Because of this behavior, we can resize the main application window when the application is starting. To achieve this goal, I added a little helper method in App.xaml.cs:

``` csharp
 private void TryToSizeAsOnLastRun(PrimaryWindow window)
{
    var lastknownMainWindowHeight = Preferences.Default.Get(Constants.SettingsLastKnownPrimaryWindowHeight, double.PositiveInfinity);
    var lastknwonMainWindowWidth =
        Preferences.Default.Get(Constants.SettingsLastKnownPrimaryWindowWidth, double.PositiveInfinity);

    window.Height = lastknownMainWindowHeight;
    window.Width = lastknwonMainWindowWidth;
}
```
 
Once we have created the `PrimaryWindow` instance in the `CreateWindow` method, we call this method and our Window will be sized correctly once the main window starts up.

### Conclusion

As we did the “hard” work already when we wrote the MacCatalyst implementation, we went pretty fast through the WinUI implementation. We took advantage of the MAUI framework that allows us to handle the application windows in a common way. This blog post is a more documentary one, but I still hope it is helpful for some of you.

You can find the updated sample [here on GitHub](https://github.com/MSicc/MAUIMultiWindowTest).

#### Until the next post, happy coding, everyone!