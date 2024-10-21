---
id: 4840
title: '[Updated] Xamarin Forms, the MVVMLight Toolkit and I: navigation and modal pages'
date: '2017-06-14T13:45:31+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post of my ongoing series I will show you a simple but effective way to implement navigation and modal pages in Xamarin.Forms MVVMLight applications.'
layout: post
permalink: /xamarin-forms-the-mvvmlight-toolkit-and-i-navigation-and-modal-pages/
image: /assets/img/2017/06/showing-modal-page-xf-mvvmlight.png
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - 'bindable properties'
    - 'Dependency Injection'
    - DI
    - implementations
    - interfaces
    - 'modal pages'
    - mvvm
    - 'mvvm light'
    - navigation
    - ViewModel
    - xamarin
    - 'xamarin forms'
---

After showing you the basic [MVVMLight Setup](https://bit.ly/2qEcvnd) I am using as well as [how to combine Xamarin.Forms’ DependencyService with our own Dependecy Injection-mechanism](https://bit.ly/2s1BFjr), this article is all about Navigation and modal pages in Xamarin.Forms.

## Some preliminary words

I always liked how the MVVMLight Toolkit provided the NavigationService for my Windows &amp; Windows Phone applications. That said, I tried to keep the idea of its NavigationService and ported it over to my Xamarin.Forms structure. I also extended it to fit the needs of my XF apps, for example for pushing modal pages (we’ll see that later). So for the base idea, I have to say thanks to [Laurent Bugnion](https://twitter.com/LBugnion) for the Windows platform implementation in MVVMLight and for keeping it simple. The later fact allows one to easily extend and adapt the service.

Also, there are tons of really good and also working samples around the web to show navigation in Xamarin.Forms. However, most of them follow another strategy, so I came to the point where decided to write my own implementation. My way may work for most scenarios without making a difference between navigation and showing modal pages. It provides an easy implementation and usage while following the MVVM pattern as far as it can go with Xamarin.Forms.

## Components

My implementation has some key components, which I will describe briefly in this post:

- Interfaces for Navigation and view events
- Implementation of these interfaces
- using the implementations in views
- using the implementations in ViewModels

## Interfaces for the win!

If we want to follow the MVVM pattern, we need to keep our Views completely separated from our ViewModels. A common practice to achieve this goal is the usage of interfaces, and so do I. If you have read my previous blog post about Dependency Injection, you know already a bit about the things we have to do. First, we will create an interface for the navigation itself:

``` csharp
 public interface IXfNavigationService
{
    void Initialize(NavigationPage navigation);
    void Configure(string pageKey, Type pageType);
    (bool isRegistered, bool isModal) StackContainsNavKey(string pageKey);
    Task GoHomeAsync();
    Task GoBackAsync();
    Task GoBackModalAsync();
    int ModalStackCount { get; }
    string CurrentModalPageKey { get; }
    Task ShowModalPageAsync(string pageKey, bool animated = true);
    Task ShowModalPageAsync(string pageKey, object parameter, bool animated = true);
    int NavigationStackCount { get; }
    string CurrentPageKey { get; }
    Task NavigateToAsync(string pageKey, bool animated = true);
    Task NavigateToAsync(string pageKey, object parameter, bool animated = true);
}
```
 
As you can see, it is a pretty big interface that covers nearly all possible navigation scenarios of a Xamarin.Forms app (at least those that I had to deal with already). We are going to hook into a Xamarin.Forms `NavigationPage` and configure our page handling cache with the configure method. As all navigation methods in Xamarin Forms are async, all navigational tasks will be implemented async as well. Additionally, we will have some helpful properties that we can use in our application implementation.

In Windows applications, we have some additional events for pages like `Loaded` or `OnNavigatedTo`(and so on). Xamarin.Forms pages have something similar: `ViewAppearing` and `ViewDisappearing`. These two events correspond to `OnNavigatedTo` and `OnNavigatedFrom` in a Windows application. Sometimes, we need to know when those events happen in our ViewModels or other places behind the scenes to get stuff done. That’s why I created an interface for this purpose:

``` csharp
 public interface IViewEventBrokerService
{
    event EventHandler<ViewEventBrokerEventArgs> ViewAppearing;
    event EventHandler<ViewEventBrokerEventArgs> ViewDisAppearing;
    void RaiseViewAppearing(string pageKey, Type pageType, bool isModal);
    void RaiseViewDisAppearing(string pageKey, Type pageType, bool isModal);
}
```
 
I also created my own EventArgs to make it easier to pass all data around (as you can see above). Here is the class:

``` csharp
 public class ViewEventBrokerEventArgs : EventArgs
{
    public ViewEventBrokerEventArgs(string pageKey, Type pageType, bool isModal)
    {
        PageKey = pageKey;
        PageType = pageType;
        IsModal = isModal;
    }
    public string PageKey { get; private set; }
    public Type PageType { get; private set; }
    public bool IsModal { get; private set; }
}
```
 
Now that we have our interfaces defined, we are going the next step and have a look into their implementations.

## IXfNavigationService implementation

First it is important to understand what Xamarin.Forms does when we are using a `NavigationPage`. We are performing a hierarchical navigation here. [**A must read is this page from the Xamarin documentation.**](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/navigation/hierarchical/)

So you’re back from reading the documentation? Great, then lets have a look what our `XfNavigationService`implementation does. It builds on top of the Xamarin.Forms `NavigationPage`, which implements the `INavigation` [interface](https://developer.xamarin.com/api/type/Xamarin.Forms.INavigation/). Other solutions I saw are pulling down this interface and implement it in their implementation. I prefer to keep it simple and just use the already implemented interface in the Xamarin.Forms `NavigationPage`, which is mandatory for my implementation. We are pulling it into our implementation with the `Initialize(NavigationPage navigationPage)` method:

``` csharp
 public void Initialize(NavigationPage navigationPage)
{
    _navigationPage = navigationPage;
}
```
 
We could also use a constructor injection here. Most of the time it would work, but I had some problems to get the timing right under certain circumstances. That’s why I prefer to manually initialize it to overcome these timing problems – and it does not really hurt in real life applications to have it a bit more under control.

Before we are going to see the actual implementation, I need to break out of this scope and explain a threading issue.

### Locking the current executing thread

Often we are running a lot of code at the same time. This could bring up some issues with our`XfNavigationService`. If multiple threads try to change or execute code from our interface implementation, the result may not the one we desired. To prevent this, we would normally use the [lock statement](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/lock-statement) to allow only one thread modifying our code. Problem with the `lock` statement is, that we cannot run asynchronous code within.

As asynchronous code execution removes some of the headache in programming, we need a solution for this. And there is one, utilizing the [SemaphoreSlim](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphoreslim?view=netframework-4.7) class of the .NET framework. Basically, the `SemaphoreSlim` class provides a single application [Semaphore](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphore?view=netframework-4.7) implementation. Without going deeper into it, we are replacing the `lock` statement we would normally use with this. We just need to initialize the `SemaphoreSlim` class as a private static member:

``` csharp
 //using this instead of lock statement
private static SemaphoreSlim _lock = new SemaphoreSlim(1, 1);
```
 
This will only allow one thread to access the objects we are passing after it, and before releasing it (which is our responsibility). The code itself is recommended to run in a `try..finally` block (you’ll see that below). Pretty much the same thing the `lock` statement would do, but with the possibility to run asynchronous code.

### Back to the implementation

The next step is to allow registering pages with a name and their type from the `ViewModelLocator`. I am doing the same as in the standard MVVMLight implementation here:

``` csharp
 //declare private member for the registered pages cache
private readonly Dictionary<string, Type> _pagesByKey = new Dictionary<string, Type>();


public void Configure(string pageKey, Type pageType) 
{ 
    //synchronous lock 
    _lock.Wait(); 
    try 
    { 
        if (_pagesByKey.ContainsKey(pageKey)) 
        { 
            _pagesByKey[pageKey] = pageType; 
        } 
        else 
        { 
            _pagesByKey.Add(pageKey, pageType); 
        } 
    } 
    finally 
    { 
        _lock.Release(); 
    } 
} 

```
 
Once we have our pages configured (we’ll see that later in this post), we are easily able to identify our registered pages for navigation purposes. Sometimes we want to get the current Page, so we need to implement a property for that, as well as for the current page key and the current modal page key:

``` csharp
 public Page GetCurrentPage()
{
    return _navigationPage?.CurrentPage;
}


public string CurrentPageKey 
{ 
    get 
    { 
        _lock.Wait(); 
        try 
        { 
            if (_navigationPage?.CurrentPage == null) 
            { 
                return null; 
            } 
  
            var pageType = _navigationPage.CurrentPage.GetType(); 
  
            return _pagesByKey.ContainsValue(pageType) 
                ? _pagesByKey.First(p => p.Value == pageType).Key 
                : null; 
        } 
        finally 
        { 
            _lock.Release(); 
        } 
    } 
} 


public string CurrentModalPageKey 
{ 
    get 
    { 
        _lock.Wait(); 
  
        try 
        { 
            if (ModalStackCount == 1) 
            { 
                return null; 
            } 
  
            //only INavigation holds the ModalStack 
            var pageType = _navigationPage.Navigation.ModalStack.Last().GetType(); 
  
            return _pagesByKey.ContainsValue(pageType) ? _pagesByKey.FirstOrDefault(p => p.Value == pageType).Key 
                : null; 
        } 
        finally 
        { 
            _lock.Release(); 
        } 
    } 
} 

```
 
I found myself in situations where I needed to check if I am on a modal page, so lets add a method for that:

``` csharp
 public (bool isRegistered, bool isModal) StackContainsNavKey(string pageKey) 
{ 
  
    bool isUsedModal = false; 
    bool isRegistered = false; 
  
    _lock.Wait(); 
    try 
    { 
        isRegistered = _pagesByKey.ContainsKey(pageKey); 
  
  
        if (isRegistered) 
        { 
            var pageType = _pagesByKey.SingleOrDefault(p => p.Key == pageKey).Value; 
  
            var foundInNavStack = _navigationPage.Navigation.NavigationStack.Any(p => p.GetType() == pageType); 
            var foundInModalStack = _navigationPage.Navigation.ModalStack.Any(p => p.GetType() == pageType); 
  
            if (foundInNavStack && !foundInModalStack || !foundInNavStack && !foundInModalStack) 
            { 
                isUsedModal = false; 
            } 
            else if (foundInModalStack && !foundInNavStack) 
            { 
                isUsedModal = true; 
            } 
            else 
            { 
                throw new NotSupportedException("Pages should be used exclusively Modal or for Navigation"); 
            } 
        } 
        else 
        { 
            throw new ArgumentException($"No page with key: {pageKey}. Did you forget to call the Configure method?", nameof(pageKey)); 
        } 
    } 
    finally 
    { 
        _lock.Release(); 
    } 
  
    return (isRegistered, isUsedModal); 
} 
```
 
This method checks if a page is registered correctly with its key and if it is presented modal at the moment. It also makes sure that an exception is thrown if a page is used modal AND in a navigation flow, which I think is a good practice to avoid problems when using the service. Finally, let’s add a Task for navigation to another page:

``` csharp
 public async Task NavigateToAsync(string pageKey, object parameter, bool animated = true) 
{ 
    await _lock.WaitAsync(); 
  
    try 
    { 
  
        if (_pagesByKey.ContainsKey(pageKey)) 
        { 
            var type = _pagesByKey[pageKey]; 
            ConstructorInfo constructor = null; 
            object[] parameters = null; 
  
            if (parameter == null) 
            { 
                constructor = type.GetTypeInfo() 
                    .DeclaredConstructors 
                    .FirstOrDefault(c => !c.GetParameters().Any()); 
  
                parameters = new object[] 
                { 
                }; 
            } 
            else 
            { 
                constructor = type.GetTypeInfo() 
                    .DeclaredConstructors 
                    .FirstOrDefault( 
                        c => 
                        { 
                            return c.GetParameters().Count() == 1 
                                   && c.GetParameters()[0].ParameterType == parameter.GetType(); 
                        }); 
  
                parameters = new[] { parameter }; 
            } 
  
            if (constructor == null) 
            { 
                throw new InvalidOperationException("No constructor found for page " + pageKey); 
            } 
  
            var page = constructor.Invoke(parameters) as Page; 
            if (_navigationPage != null) 
            { 
                Device.BeginInvokeOnMainThread(async () => 
                { 
                    await _navigationPage.Navigation.PushAsync(page, animated); 
                }); 
            } 
            else 
            { 
                throw new NullReferenceException("there is no navigation page present, please check your page architecture and make sure you have called the Initialize Method before."); 
            } 
        } 
        else 
        { 
            throw new ArgumentException( 
                $"No page with key: {pageKey}. Did you forget to call the Configure method?", 
                nameof(pageKey)); 
        } 
    } 
    finally 
    { 
        _lock.Release(); 
    } 
} 
```
 
*\[Update:\] Until the latest Xamarin.Forms release, there was no need to use the* `Device.BeginInvokeOnMainThread`*method for the navigation itself. I only checked it recently that the Navigation needs now the call to it to dispatch the navigation to the main UI thread. Otherwise, the navigation will end in a dead thread on Android and iOS (but strangely not for UWP). I updated the code above and the source on Github as well.*

I try to avoid passing parameters to pages while navigating, that’s why I added another task without the `parameter` overload as well:

``` csharp
 public async Task NavigateToAsync(string pageKey, bool animated = true)
{
    await NavigateToAsync(pageKey, null, animated);
}
```
 
Of course we will run into situations where we want to go back manually, for example in a cancel function. So we have a method for that as well:

``` csharp
 public async Task GoBackAsync()
{
    await _navigationPage.Navigation.PopAsync(true);
}
```
 
That’s a lot of functionality already, but until now, we are only able to navigate to other pages and back. We want to support modal pages as well, so we’ll have some more methods to implement. They are following the same syntax as their navigation counterpart, so no surprise here:

``` csharp
 public async Task ShowModalPageAsync(string pageKey, object parameter, bool animated = true) 
{ 
    await _lock.WaitAsync(); 
  
    try 
    { 
  
        if (_pagesByKey.ContainsKey(pageKey)) 
        { 
            var type = _pagesByKey[pageKey]; 
            ConstructorInfo constructor = null; 
            object[] parameters = null; 
  
            if (parameter == null) 
            { 
                constructor = type.GetTypeInfo() 
                    .DeclaredConstructors 
                    .FirstOrDefault(c => !c.GetParameters().Any()); 
  
                parameters = new object[] 
                { 
                }; 
            } 
            else 
            { 
                constructor = type.GetTypeInfo() 
                    .DeclaredConstructors 
                    .FirstOrDefault( 
                        c => 
                        { 
                            return c.GetParameters().Count() == 1 
                                   && c.GetParameters()[0].ParameterType == parameter.GetType(); 
                        }); 
  
                parameters = new[] { parameter }; 
            } 
  
  
            var page = constructor.Invoke(parameters) as Page; 
  
            if (_navigationPage != null) 
            { 
                Device.BeginInvokeOnMainThread(async () => 
                { 
                    await _navigationPage.Navigation.PushModalAsync(page, animated); 
                }); 
            } 
            else 
            { 
                throw new NullReferenceException("there is no navigation page present, please check your page architecture and make sure you have called the Initialize Method before."); 
            } 
        } 
        else 
        { 
            throw new ArgumentException($"No page found with key: {pageKey}. Did you forget to call the Configure method?", nameof(pageKey)); 
        } 
    } 
    finally 
    { 
        _lock.Release(); 
    } 
} 

public async Task GoBackModalAsync()
{
    await _navigationPage.Navigation.PopModalAsync(true);
}
```
 
The last things that I wanted to note are properties that return the current counts of both the modal and the navigation stack of the navigation page as well as the `GoHomeAsync()` method to return to the root page. You will see these in the source code on Github.

## IViewEventBrokerService implementation

Like I wrote already before, sometimes we need to know when a view is appearing/disappearing. In that case, the IViewEventBrokerService comes in. The implementation is pretty straight forward:

``` csharp
 public class ViewEventBrokerService : IViewEventBrokerService
{
    public event EventHandler<ViewEventBrokerEventArgs> ViewAppearing;
    public event EventHandler<ViewEventBrokerEventArgs> ViewDisAppearing;


    public void RaiseViewAppearing(string pageKey, Type pageType, bool isModal)
    {
        ViewAppearing?.Invoke(this, new ViewEventBrokerEventArgs(pageKey, pageType, isModal));
    }

    public void RaiseViewDisAppearing(string pageKey, Type pageType, bool isModal)
    {
        ViewDisAppearing?.Invoke(this, new ViewEventBrokerEventArgs(pageKey, pageType, isModal));
    }
}
```
 
We have two events that can be raised with their corresponding Raise… methods. That’s all we have to do in here.

## Implementing the Interfaces in Views (Pages)

Well, we have written a lot of code until here, now it is time to actually use it. As I want all my pages to be raise their appearing and disappearing automatically, I created a new base class, derived from Xamarin.Forms’ `ContentPage`:

``` csharp
 public class XfNavContentPage : ContentPage
{
    private IViewEventBrokerService _viewEventBroker;
    private IXfNavigationService _navService;

    public XfNavContentPage()
    {
        _viewEventBroker = SimpleIoc.Default.GetInstance<IViewEventBrokerService>();
        _navService = SimpleIoc.Default.GetInstance<IXfNavigationService>();
    }

    public static BindableProperty RegisteredPageKeyProperty = BindableProperty.Create("RegisteredPageKey", typeof(string), typeof(XfNavContentPage), default(string), BindingMode.Default, propertyChanged: OnRegisteredPageKeyChanged);

    private static void OnRegisteredPageKeyChanged(BindableObject bindable, object oldValue, object newValue)
    {
        //todo, but not needed atm.
    }

    public string RegisteredPageKey
    {
        get { return (string)GetValue(RegisteredPageKeyProperty); }
        set { SetValue(RegisteredPageKeyProperty, value); }
    }

    private(bool isRegistered, bool isModal) _stackState { get; private set; }

    protected override void OnAppearing()
    {
        base.OnAppearing();

        if (!string.IsNullOrEmpty(RegisteredPageKey))
        {
            _stackState = _navService.StackContainsNavKey(RegisteredPageKey);

            if (_stackState.isRegistered)
            {
                _viewEventBroker?.RaiseViewAppearing(RegisteredPageKey, GetType(), _stackState.isModal);
            }
        }
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();

        if (!string.IsNullOrEmpty(RegisteredPageKey))
        {
            if (_stackState.isRegistered)
            {
                _viewEventBroker?.RaiseViewDisAppearing(RegisteredPageKey, GetType(), _stackState.isModal);
            }
        }
    }
}
```
 
Let me explain what this base class does. It provides a `BindableProperty`(which is the Xamarin.Forms version of a `DependencyProperty`) that takes the registered key to provide it as a parameter. You can bind the key from your ViewModel or set it in code behind/XAML directly, all these ways will work. The base page implements our interfaces of the `IXfNavigationService` and the `IViewBrokerService`interfaces as members and loads them in its constructor. The `_stackState`member is just an easy way to provide the needed data to the event-raising methods of our `IViewEventBroker`service. If used correctly, every page that derives from this base class will throw the corresponding events, which can be used in other parts of our application then.

## Implementing the Interfaces in ViewModels

Like I did for Views, I am also creating a new base class for ViewModels that implement our two interfaces. Here is how it looks like:

``` csharp
 public class XfNavViewModelBase : ViewModelBase
{
    protected readonly IXfNavigationService _navService;

    protected readonly IViewEventBrokerService _viewEventBroker;


    public XfNavViewModelBase()
    {
        _navService = SimpleIoc.Default.GetInstance<IXfNavigationService>();
        _viewEventBroker = SimpleIoc.Default.GetInstance<IViewEventBrokerService>();
        _viewEventBroker.ViewAppearing += OnCorrespondingViewAppearing;
        _viewEventBroker.ViewDisAppearing += OnCorrespondingViewDisappearing;
    }


    protected virtual void OnCorrespondingViewAppearing(object sender, ViewEventBrokerEventArgs e)
    {
    }

    protected virtual void OnCorrespondingViewDisappearing(object sender, ViewEventBrokerEventArgs e)
    {
    }

    private string _correspondingViewKey;

    public string CorrespondingViewKey
    {
        get => _correspondingViewKey; set => Set(ref _correspondingViewKey, value);
    }

    public bool IsBoundToModalView()
    {
        if (!string.IsNullOrEmpty(CorrespondingViewKey))
        {
            var checkIfIsModal = _navService.StackContainsNavKey(CorrespondingViewKey);

            if (checkIfIsModal.isRegistered)
            {
                return checkIfIsModal.isModal;
            }
        }
        return false;
    }
}
```
 
The `XfNavViewModelBase` implements both the `IXfNavigationService` and the `IViewEventBrokerService` interfaces. It also registers for the events thrown by `IViewEventBrokerService`. Making them virtual allows us to override them in derived ViewModels, if we need to. It has a string property to set the name of the View we want to use it in as well as a method that checks automatically if we are on a modal page if it gets called.

## Using the created navigation setup

Now our navigation setup is ready it is time to have a look into how to use it. First we are going to create two pages, one for being shown modal, and one for being navigated to.

![created-pages-snip](/assets/img/2017/06/created-pages-snip.png)


Then we’ll need to register and configure our services and pages in the `ViewModelLocator`. Remember the static RegisterServices() method we created in the first post? This is were we will throw the interface registrations in:

``` csharp
 private static void RegisterServices() 
{ 
    //this one gets the correct service implementation from platform implementation 
    var osService = DependencyService.Get<IOsVersionService>(); 
  
    // which can be used to register the service class with MVVMLight's Ioc 
    SimpleIoc.Default.Register<IOsVersionService>(() => osService); 
  
    SimpleIoc.Default.Register<IXfNavigationService>(GetPageInstances); 
    SimpleIoc.Default.Register<IViewEventBrokerService, ViewEventBrokerService>(); 
}
```
 
The method contains already our `IOsVersionService` from [my last blog post]({% post_url 2017-06-02-xamarin-forms-the-mvvmlight-toolkit-and-i-dependecy-injection %}). We are also using DI here for the `IXfNavigationService`, as we need to Configure our page keys as well. First, we are adding static strings as page keys to the ViewModelLocator:

``` csharp
 public static string ModalPageKey => nameof(ModalPage);
public static string NavigatedPageKey => nameof(NavigatedPage);
```
 
I am always using the [nameof()-operator](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/nameof) for this, because it makes life a bit easier. The second step is the missing piece in the service registration, the `GetPageInstances()` method that returns an instance of the `IXfNavigationService` interface implentation and registers our two pages via the `Configure()` method.

``` csharp
 public static XfNavigationService GetPageInstances()
{
    var nav = new XfNavigationService();

    nav.Configure(ModalPageKey, typeof(ModalPage));
    nav.Configure(NavigatedPageKey, typeof(NavigatedPage));

    return nav;
}
```
 
This all will never work if we are not using a `NavigationPage`. So we are going to load `MainPage.xaml` as a `NavigationPage`in `App.xaml.cs` and pass the navigation page to our `IXfNavigationService`:

``` csharp
 public App()
{
    InitializeComponent();

    var rootNavigation = new NavigationPage(new View.MainPage());
    MainPage = rootNavigation;
    SimpleIoc.Default.GetInstance<IXfNavigationService>().Initialize(rootNavigation);
}
```
 
### Modifying pages

Until now, the pages we created are standard ContentPages, so we need to make them derive from our earlier created base class. First we are adding the namespace of our base class to it:

``` xml
xmlns:baseCtrl="clr-namespace:XfMvvmLight.BaseControls;assembly=XfMvvmLight"
```
 
After that, we are able to use the base class to change `ContentPage` to `baseCtrl:XfNavContentPage`. Now open the code behind file for the page and change it to derive from `XfNavContentPage` as well:

``` csharp
 public partial class ModalPage : XfNavContentPage
```
 
The sample project has a modal page and a navigated page to demonstrate both ways.

### Setting up ViewModels

Normally, we also have a ViewModel for every additional page. That´s why I created a `ModalPageViewModel `as well as a `NavigatedPageViewModel` for the sample. They derive from the `XfNavViewModelBase`, so wie can easyily hook up the view events (if needed):

``` csharp
 public class ModalPageViewModel : XfNavViewModelBase
{ 
    public ModalPageViewModel()
    {
        CorrespondingViewKey = ViewModelLocator.ModalPageKey;
    }
 
    protected override void OnCorrespondingViewAppearing(object sender, ViewEventBrokerEventArgs e)
    {
        base.OnCorrespondingViewAppearing(sender, e);
    }
 
    protected override void OnCorrespondingViewDisappearing(object sender, ViewEventBrokerEventArgs e)
    {
        base.OnCorrespondingViewDisappearing(sender, e);
    }
 
 
    private RelayCommand _goBackCommand;
 
    public RelayCommand GoBackCommand => _goBackCommand ?? (_goBackCommand = new RelayCommand(async () =>
    {
        await _navService.GoBackModalAsync();
    })); 
}
```
 
Remember the page keys we added earlier to the `ViewModelLocator`? We are just setting it as `CorrespondingViewKey` in the constructor here, but you could choose to it the way you prefer as well and can easily be bound to the view now:

``` xml
 <baseCtrl:XfNavContentPage 
   xmlns="https://xamarin.com/schemas/2014/forms"
   xmlns:x="https://schemas.microsoft.com/winfx/2009/xaml"
   xmlns:baseCtrl="clr-namespace:XfMvvmLight.BaseControls;assembly=XfMvvmLight"
   x:Class="XfMvvmLight.View.ModalPage" 
   RegisteredPageKey="{Binding CorrespondingViewKey}">
<!--code omitted for readability-->
</baseCtrl:XfNavContentPage>
```
 
As the sample pages have a button with back function, we have also a `GoBackCommand` bound to it. Now that’s already all we need to, and if we set a breakpoint in the event handling overrides, it will be hit as soon as the view is about to appear. See the sample working here in action on Android:

![navigation-service-xf-mvvmlight](/assets/img/2017/06/navigation-service-xf-mvvmlight.gif)

No we have a fully working and MVVM compliant navigation solution in Xamarin.Forms using the MVVMLight Toolkit. I know there are several other toolkits and helpers floating around, but I like it to have less dependencies. Another advantage of going this route: I am mostly familiar with the setup as I am already used to it from my Windows and Windows Phone applications. Creating this setup for navigation does not take a whole lot of time (the initial setup took me around 3 hours including conception). One can also pack the interfaces and the implementations in a (portable) class library to have all this work only once.

If you have feedback for me, feel free to leave a comment below. Like always, I hope this article is helpful for some of you. The updated sample application can be found [here on my Github account](https://github.com/MSiccDev/XfMvvmLight).

Happy coding, everyone!