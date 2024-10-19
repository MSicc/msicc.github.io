---
id: 7784
title: 'Dealing with application windows on macOS with .NET MAUI'
date: '2023-11-03T17:00:00+01:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I show you how to deal with application windows on macOS using .NET MAUI and the MVVM pattern.'
layout: post
permalink: /dealing-with-application-windows-on-macos-with-net-maui/
image: /assets/img/2023/11/dealing-with-windows-macos-maui-title.png
categories:
    - 'Dev Stories'
    - macOS
    - MAUI
tags:
    - '.NET MAU'
    - Desktop
    - dotNET
    - Mac
    - MacCatalyst
    - macOS
    - MAUI
    - mvvm
    - window
    - Windows
---

[As I posted already on my Mastodon feed](https://dotnet.social/@MSicc/111307246102636587), I recently had to deal with application windows for one of my side projects. It all started because I needed a secondary window of fixed size that can be opened with different items of the same type, providing the same options to all of them.

Let’s have a look what this post will cover:

- Supporting multiple application windows
- Handling the size of application windows
- Application windows and MVVM
- Gotchas
- Useful links

### Supporting multiple application windows

.NET Maui applications are single window applications by default. Developers can change that if needed. First, add a new class to the *MacCatalyst* folder:

``` csharp
 // ReSharper disable once RedundantUsingDirective
using Foundation;
using Microsoft.Maui;
using UIKit;

namespace MauiMultiWindowTest
{
    [Register("SceneDelegate")]
    public class SceneDelegate : MauiUISceneDelegate
    {
        
    }
}
```
 
After that, open the Info.plist file inside the *MacCatalyst* folder and add these lines just before the last `</dict>` tag:

``` xml
 <key>UIApplicationSceneManifest</key>
<dict>
	<key>UIApplicationSupportsMultipleScenes</key>
	<true/>
	<key>UISceneConfigurations</key>
	<dict>
		<key>UIWindowSceneSessionRoleApplication</key>
		<array>
			<dict>
				<key>UISceneDelegateClassName</key>
				<string>SceneDelegate</string>
				<key>UISceneConfigurationName</key>
				<string>__MAUI_DEFAULT_SCENE_CONFIGURATION__</string>
			</dict>
		</array>
	</dict>
</dict>
```
 
Now, let’s define two additional classes – `PrimaryWindow` and `SecondaryWindow`:

``` csharp
 public class PrimaryWindow : Window
{
    public PrimaryWindow(Page page) : base(page)
    {

    }
}

public class SecondaryWindow : Window
{
    public SecondaryWindow(Page page, string? title)
    {
        ArgumentNullException.ThrowIfNull(page);
        this.Title = string.IsNullOrWhiteSpace(title) ? throw new ArgumentException("Value cannot be null or whitespace.", nameof(title)) : title;
    }
}
```
 
These two classes make it easier to differentiate between the main application window, which is always a `PrimaryWindow` in my implementation, and the additional windows we are opening.

Head over to your App.xaml.cs file and find the `Application.Current.MainPage` assignment in there. Once you have it, delete it, so your constructor will only have the `InitializeComponent` call.

Next, override the `CreateWindow` method as like this:

``` csharp
 protected override Window CreateWindow(IActivationState activationState)
{    
    var window = new PrimaryWindow(new AppShell
    {
        BindingContext = new AppShellViewModel()
    });

    return window;
}
```
 
So what have done? With this approach, we have taken control over the Window creation process (at least as far as we are possible). We are creating the `PrimaryWindow` with the root page of our application. This would have been done by the MAUI framework otherwise and assigned to the `Application.Current.MainPage` property. Don’t worry about the assignment, the `AppShell` (or whatever your root page is) will still be assigned to `Application.Current.MainPage`.

Last but not least, we can open a secondary window with the following call:

``` csharp
 SecondaryWindow secondaryWindow = new SecondaryWindow(new SecondaryPage(), nameof(SecondaryWindow));
Application.Current?.OpenWindow(secondaryWindow);
```
 
### Window Sizing

In general, macOS does not provide a dedicated API to resize application windows programmatically.

You can specify the initial size of the window at the time of creation. The operating system will size a window within the bounds of minimum and maximum size. However, there is no guarantee that the size will be the one you set. If a user can resize the window (which is the default), also the operating system is able to do so (and it will).

I was only able to get the sizing to my expectations when I restarted the whole application and I saved the last window size, or when I fixed the window size. If you close a window during an application run (means not quitting it) and open it again, the OS decides the size of the window (if it is not of fixed size).

Now let’s take a look at how to handle the window sizes in code, finally!

#### Fixed size

Let’s take a look at the easy one first. Setting a fixed size on a window just requires you to set the minimum and maximum size to the same values. Every `ContentPage` has a reference to its parent window, so you can set the size from there.

``` csharp
 this.Window.MinimumWidth = 700;
this.Window.MaximumWidth = 700;
this.Window.MinimumHeight = 500;
this.Window.MaximumHeight = 500;
```
 
That’s it already for the fixed size windows.

#### Dynamic size

Setting a dynamic size is similar, you just would change the value to allow a range. Additionally, you also set the desired size, which must of course be in that range:

``` csharp
 this.Window.MinimumWidth = 300;
this.Window.MaximumWidth = 700;
this.Window.MinimumHeight = 250;
this.Window.MaximumHeight = 500;
this.Window.Width = 400;
this.Window.Height = 300;
```
 
In about 80 percent of all cases, the window will be sized in that way. Otherwise, the operating system decides the needed size based on the content of your window/page object.

#### Resizing a window programmatically

If you want to change the size of a window after it is created and visible, you can trigger a size change by setting the values of the minimum and maximum sizes, like in the fixed size scenario above. To (re-)enable resizing for the user after that, you need to specify the allowed range again after that, like in the dynamic size scenario above. You should dispatch the range setting, however:

``` csharp
 this.Dispatcher.Dispatch(() =>
{
this.Window.MinimumWidth = 300;
this.Window.MaximumWidth = 700;
this.Window.MinimumHeight = 250;
this.Window.MaximumHeight = 500;
});
```
 
Now that we have a common understanding of how window sizing works, we can look into a possible MVVM implementation for resizable and multiple application windows.

### Handling application windows with MVVM

If you are following my path already, you know that I apply the MVVM-pattern for almost any application I write. Handling application windows in an MVVM-friendly way isn’t that complicated, and I will explain the key aspects here. The full implementation is available on my GitHub account.

My implementation has the following components:

- Window resizing page base class
- Secondary window base page class and a corresponding base ViewModel
- `PrimaryWindow` and `SecondaryWindow` implementation
- `IWindowService` handling all the opening and closing stuff

#### ResizeableMacBasePage

To have a common base implementation for resizable pages, I have created the `ResizeableMacBasePage` class. The implementation defines several `BindableProperty` objects that I use to control the window size and its ability to resize. You can see the full implementation on [here on GitHub](https://github.com/MSicc/MAUIMultiWindowTest/blob/main/src/MauiMultiWindowTest/Pages/ResizeableMacBasePage.cs).

Pages that derive from this base class can now set the size of their parent window as following:

``` xml
 <?xml version="1.0" encoding="utf-8"?>

<pages:ResizeableMacBasePage
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:mauiMultiWindowTest="clr-namespace:MauiMultiWindowTest"
    xmlns:pages="clr-namespace:MauiMultiWindowTest.Pages"
    xmlns:viewModels="clr-namespace:MauiMultiWindowTest.ViewModels"
    x:Class="MauiMultiWindowTest.Pages.MainPage"
    ParentWindowHeight="400"
    ParentWindowWidth="600"
    ParentWindowMinHeight="300"
    ParentWindowMinWidth="500"
    ParentWindowMaxHeight="1000"
    ParentWindowMaxWidth="1400"
    ParentWindowAllowResize="True"
    x:DataType="viewModels:MainViewModel">    
  ...
</pages:ResizeableMacBasePage>
```
 
If you took a look into the file on GitHub, you may have noticed that I am saving the last used window size. Under normal circumstances, these values get reloaded after a fresh app start. If you are just closing the window and reopen it again, the application may set the window to the same size. Quite often, you may see different results in this case, though.

#### SecondaryWindowBasePage and SecondaryWindowPageViewModelBase

For secondary Windows, I created a base class that inherits from the ResizableMacBasePage. We’ll use the Pages that derive from this base class later when we use our iWindowService implementation. The class doesn’t do much else besides adding a BindableProperty for the ParentWindowKey and setting a Binding to the corresponding Property in the Viewmodel for it and also for the Title property as well:

``` csharp
 public class SecondaryWindowBasePage : ResizeableMacBasePage
{
    public static BindableProperty ParentWindowKeyProperty =>             BindableProperty.Create(nameof(SecondaryWindowBasePage.ParentWindowKey), typeof(string), typeof(SecondaryWindowBasePage));
    
    public SecondaryWindowBasePage()
    {
            
    }

    protected override void OnBindingContextChanged()
    {
        base.OnBindingContextChanged();

        if (this.BindingContext is not SecondaryWindowPageViewModelBase)
            return;

        SetBinding(TitleProperty, new Binding(nameof(SecondaryWindowPageViewModelBase.ParentWindowTitle), BindingMode.Default));
        SetBinding(SecondaryWindowBasePage.ParentWindowKeyProperty, new Binding(nameof(SecondaryWindowPageViewModelBase.ParentWindowKey), BindingMode.Default));
    }

    public string ParentWindowKey { get; set; }
}
```
 
Also, the SecondaryWindowPageViewModelBase is not that complicated (as you’d expect):

``` csharp
 public abstract class SecondaryWindowPageViewModelBase : ObservableObject
{
    private string? _parentWindowKey;
    private string? _parentWindowTitle;
    
    public string? ParentWindowKey   
    {
        get => _parentWindowKey;
        set => SetProperty(ref _parentWindowKey, value);
    }

    public string? ParentWindowTitle
    {
        get => _parentWindowTitle;
        set => SetProperty(ref _parentWindowTitle, value);
    }
}
```
 
#### SecondaryWindow

Let’s go back to the SecondaryWindow we already created above and update our code to include the window key:

``` csharp
 public class SecondaryWindow : Window
{
    public string? Key { get; private set; }

    public SecondaryWindow(Page page, string? title, string? key) : base(page)
    {
        ArgumentNullException.ThrowIfNull(page);
        
        this.Key = (string.IsNullOrWhiteSpace(key)) ?
            throw new ArgumentException("Value cannot be null or whitespace.", nameof(key)) : key;

        this.Title = string.IsNullOrWhiteSpace(title) ? throw new ArgumentException("Value cannot be null or whitespace.", nameof(title)) : title;
    }
    
}
```
 
As we intend to have only one PrimaryWindow instance, we don’t need to change anything there.

#### IWindowService

To be able to open a new Window from a Viewmodel, we need a service that abstracts the function away for us. Here is the definition of my IWindowService:

``` csharp
 public interface IWindowService
{
    Dictionary<string, SecondaryWindow> CurrentlyOpenedWindows { get; }

    void ShowWindowForPage<TPageType, TViewModelType>(TViewModelType vm)
        where TPageType : SecondaryWindowBasePage
        where TViewModelType : SecondaryWindowPageViewModelBase;
    
    void CloseWindow(string? windowKey);

    void CloseAllSecondaryWindows();

    SecondaryWindow? GetByKey(string? key);
}
```
 
Well the interface talks for itself. Let’s look at the implementation:

``` csharp
 public class WindowService : IWindowService
{
    public Dictionary<string, SecondaryWindow> CurrentlyOpenedWindows { get; } = new Dictionary<string, SecondaryWindow>();

    public void ShowWindowForPage<TPageType, TViewModelType>(TViewModelType vm) 
        where TPageType : SecondaryWindowBasePage 
        where TViewModelType : SecondaryWindowPageViewModelBase
    {
        //failing gracefully here
        if (this.CurrentlyOpenedWindows.ContainsKey(vm.ParentWindowKey))
            return;
        
        //failing hard here
        ArgumentNullException.ThrowIfNull(vm);
        ArgumentException.ThrowIfNullOrWhiteSpace(vm.ParentWindowKey);
        ArgumentException.ThrowIfNullOrWhiteSpace(vm.ParentWindowTitle);
            
        //this should be used with Transient registrations
        TPageType page = ServiceHelper.GetService<TPageType>();
        page.BindingContext = vm;

        SecondaryWindow windowToOpen = new SecondaryWindow(page, vm.ParentWindowTitle, vm.ParentWindowKey);
        
        windowToOpen.Created += (sender, args) => 
            this.CurrentlyOpenedWindows.Add(vm.ParentWindowKey, windowToOpen);

        windowToOpen.Destroying += (sender, args) =>
            this.CurrentlyOpenedWindows.Remove(vm.ParentWindowKey);
        
        Application.Current?.OpenWindow(windowToOpen);
    }

    public void CloseWindow(string? windowKey)
    {
        if (!this.CurrentlyOpenedWindows.TryGetValue(windowKey, out SecondaryWindow? value))
            return;
        
        Application.Current?.CloseWindow(value);
        this.CurrentlyOpenedWindows.Remove(windowKey);
    }

    public void CloseAllSecondaryWindows()
    {
        foreach (var key in this.CurrentlyOpenedWindows.Keys.ToList())
        {
            CloseWindow(key);
            this.CurrentlyOpenedWindows.Remove(key);
        }
    }

    public SecondaryWindow? GetByKey(string? key)
    {
        if (string.IsNullOrWhiteSpace(key))
            return null;

        return !this.CurrentlyOpenedWindows.TryGetValue(key, out SecondaryWindow? value) ? null : value;

    }
}
```
 
The implementation has a Dictionary of `SecondaryWindow` objects with their corresponding keys. When we open a new secondary window, we add the key to the dictionary. When a window gets closed, it will be removed from the dictionary. To take advantage of the type safety, we are constraining the page and Viewmodel instances to be deriving from our base classes defined above.

We are able to get instances by their key with the `GetByKey` method. Last but not least, we will be able to close one or all secondary windows of our application.

### How to use the IWindowService

You need to register the `IWindowService` and its implementation during app startup:

``` csharp
 serviceCollection.AddSingleton<IWindowService, WindowService>();
```
 
Your Viewmodel needs the service injected:

``` csharp
 public class MainViewModel : ObservableObject
{
    private readonly IWindowService _windowService;

    public MainViewModel(IWindowService windowService)
    {
        _windowService = windowService;
    }

    ...
}
```
 
Whenever you need a secondary window, it is just a few lines away:

``` csharp
 SecondaryPageViewModel vm = ServiceHelper.GetService<SecondaryPageViewModel>();
vm.ParentWindowTitle = "Single fixed window";
vm.ParentWindowKey = "SingleFixedWindow";
            
_windowService.ShowWindowForPage<SecondaryPage, SecondaryPageViewModel>(vm);
```
 
### Conclusion

Dealing with application windows is not that difficult once you understand how the windowing system works, even though every platform has its own quirks. I hope this blog post helps you to understand these and also enables you to implement a similar system in your application if needed.

In the near future, I will port the sample ([find it here on GitHub](https://github.com/MSicc/MAUIMultiWindowTest/tree/main)) over to the Windows operating system. Once I have done this, I will once again write a blog post similar to this one.

As always, I hope this blog post is helpful for some of you.

#### Until the next post, happy coding, everyone!

---

Helpful links:

- [App lifecycle – .NET MAUI | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/maui/fundamentals/app-lifecycle)
- [.NET MAUI windows – .NET MAUI | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/maui/fundamentals/windows#multi-window-support)
- [MacCatalyst app launches multiple windows if previous launch had multiple windows open · Issue #10939 · dotnet/maui](https://github.com/dotnet/maui/issues/10939)
- [Windows | Apple Developer Documentation](https://developer.apple.com/design/human-interface-guidelines/windows)
- [Positioning and sizing windows | Apple Developer Documentation](https://developer.apple.com/documentation/visionos/positioning-and-sizing-windows)