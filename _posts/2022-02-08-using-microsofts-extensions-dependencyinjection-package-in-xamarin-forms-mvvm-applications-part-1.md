---
id: 6988
title: 'Using Microsoft&#8217;s Extensions.DependencyInjection package in (Xamarin.Forms) MVVM applications (Part 1) [Updated]'
date: '2022-02-08T20:24:08+01:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I am showing you how I am using Microsoft''s Extensions.DependencyInjection package and how I implemented my own IServiceProvider for it.'
layout: post
permalink: /using-microsofts-extensions-dependencyinjection-package-in-xamarin-forms-mvvm-applications-part-1/
image: /assets/img/2022/02/MS_DI_Title_Image.png
categories:
    - 'Dev Stories'
    - Xamarin
tags:
    - 'Dependency Injection'
    - Ioc
    - mvvm
    - mvvmlight
    - toolkit
    - xamarin
    - 'xamarin forms'
---

As some of you might remember, I was always a big fan of the `MVVMLight` toolkit. As the later one is now deprecated and MAUI around, I took a look at the [`CommunityToolkit.Mvvm`](https://github.com/CommunityToolkit/dotnet/tree/main/CommunityToolkit.Mvvm), which is officially the successor to `MVVMLight`.

As stated in the documentation of the new Toolkit, one could now use the Microsoft’s `Extensions.DependencyInjection` package for anything related to Inversion of Control (which used to be handled by the `SimpleIoc` implementation of `MVVMLight` before). Because this is also the built-in way for .NET 6 and web applications, I decided to adapt it already now for my `Xamarin.Forms` apps (especially my new one I am currently working on).

### \[Update\] Nuget packages

Please note that while the toolkit’s source is now separated from the `Windows CommunityToolkit`, the documentation isn’t. This can be confusing (as it was for me). On top of that, there are now two Toolkit MVVM packages:

![](/assets/img/2022/02/Toolkit-MVVM-Nugets.png)


I thought I got it right when writing this blog post initially. After [Brandon Minnick](https://twitter.com/thecodetraveler) from Microsoft pointed me to [the right package](https://www.nuget.org/packages/CommunityToolkit.Mvvm/), I realized I was not. Up on further research, I found also [this discussion in the GitHub repo](https://github.com/CommunityToolkit/dotnet/discussions/52), stating the one and only will be the `CommunityToolkit` package. Please use only this one if you are following my tutorials here. I updated all mentions of the Toolkit in this post accordingly.

### Default IServiceProvider implementation

The toolkit has a default implementation for the `IServiceProvider` provided by the `Extension.DependencyInjection` package. You can read about it [here in the documentation](https://docs.microsoft.com/en-us/windows/communitytoolkit/mvvm/ioc) and see the source [here on GitHub](https://github.com/CommunityToolkit/dotnet/blob/main/CommunityToolkit.Mvvm/DependencyInjection/Ioc.cs). It focuses heavily on thread safety, its usage is pretty strict, and it does not allow adding ViewModels dynamically. If you do not need stuff like this in your app, you’re probably fine using the `Ioc.Default` implementation of the toolkit.

### Custom IServiceprovider implementation

In TwistReader, the application I am currently working on, I had my requirements easily resolved by the `SimpleIoc` implementation of the `MVVMLight` toolkit. With the `Extensions.DependencyInjection` package, I had to move on with a custom implementation, on which we will have a deeper look in this post. Before you move on reading, make sure you have read the [documentation](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection).

### IIocManagerBase interface

Of course, I wanted my custom implementation to be reusable. So I extended my existing base interface that my applications need to implement:

``` csharp
 public interface IIocManagerBase 
{
    IServiceProvider? Services { get; }
    ServiceCollection? ServiceDescriptors { get; }

    IServiceProvider? BuildServiceProvider(bool resetExisiting = false);

    void Initialize(bool useDefaultNavigationService = true);

    void RegisterServices(bool useDefaultNavigationService);

    void RegisterViewModels();
}
```
 
I added the `ServiceDescriptors` property as well as an `IServiceProvider` property including a method to (re)build the `ServiceProvider` if needed. Let’s continue by having a look at the `Xamarin.Forms` base implementation.

### FormsIocManagerBase base class

Building up on the interface before, I created a base implementation for my `Xamarin.Forms` apps. Let’s go a bit into the details.

In the `Initialize` method, I am just calling the `RegisterServices` and the `RegisterViewModels` methods. One important thing to notice is that I am instantiating the `ServiceDescriptors` property in the `RegisterViewModels` method. I also add my default services already to collection there. The `RegisterViewModels` method remains empty in the base implementation.

``` csharp
 public virtual void Initialize(bool useDefaultNavigationService = true)
{
    RegisterServices(useDefaultNavigationService);
    RegisterViewModels();
}

public virtual void RegisterServices(bool useDefaultNavigationService)
{
    this.ServiceDescriptors = new ServiceCollection();

    this.ServiceDescriptors.TryAddSingleton<IDialogService>(DependencyService.Get<IDialogService>());
    this.ServiceDescriptors.TryAddSingleton<IActionSheetService>(new ActionSheetService());

    if (useDefaultNavigationService)
        this.ServiceDescriptors.TryAddSingleton<INavigationService>(new NavigationService());
    else
        System.Diagnostics.Debug.WriteLine("***** DON'T FORGET TO REGISTER YOUR INavigationService INSTANCE(S)!  *****");
}

public virtual void RegisterViewModels()
{
}
```
 
Until I switched to `CommunityToolkit.Mvvm`, this was all I had in there (using `SimpleIoc` for service registrations). Now that I am using the `Extensions.DependencyInjection` package, there is some more work to do:

``` csharp
 public ServiceCollection? ServiceDescriptors { get; private set; }

private IServiceProvider? _services;

public IServiceProvider? Services => _services ??= BuildServiceProvider();

public IServiceProvider? BuildServiceProvider(bool resetExisiting = false)
{
    if (this.ServiceDescriptors == null)
        throw new ArgumentNullException($"Please register your Services and ViewModels first with the {nameof(RegisterServices)} and {nameof(RegisterViewModels)} methods.");

    if (resetExisiting)
        _services = null;

    if (_services == null)
        _services = ServiceDescriptors.BuildServiceProvider();

    return _services;
}
```
 
The code is not that complex, but helps with the `IServiceProvider` instance handling. The `BuildServiceProvider` method has a reset flag that allows me to rebuild the provider at runtime. One scenario where we can use this one is for adding ViewModel registrations dynamically during the runtime of our app, but.

### IocManager in-app implementation

The next code block shows a typical in-app implementation of my `IocManager`. You may have noticed I am using the `TryAdd{Lifetime}` methods already before when adding items to the `ServiceCollection`. This makes sure that I have always just one registration and does not throw an exception if I try to add it again. If you prefer the exception, just switch to the `Add{Lifetime}` version.

``` csharp
 public class IocManager : FormsIocManagerBase
{
    private static IocManager _instance;

    public static IocManager Current => _instance ??= new IocManager();


    public override void Initialize(bool useDefaultNavigationService = true)
    {
        base.Initialize(useDefaultNavigationService);
    }

    public override void RegisterServices(bool useDefaultNavigationService)
    {
        base.RegisterServices(useDefaultNavigationService);

        this.ServiceDescriptors.TryAddSingleton<ITestService, TestService1>();
        this.ServiceDescriptors.TryAddScoped<ITestService, TestedTestService>();
    }

    public override void RegisterViewModels()
    {
        this.ServiceDescriptors.TryAddSingleton<MainViewModel>();
        this.ServiceDescriptors.TryAddSingleton<SecondaryViewModel>();
    }

    public MainViewModel MainVm => this.Services.GetRequiredService<MainViewModel>();
    public SecondaryViewModel SecondaryVm => this.Services.GetRequiredService<SecondaryViewModel>();
}
```
 
For my` Xamarin.Forms` applications, I always use the `IocManager` implementation as a singleton. This makes it pretty easy with the different lifetimes on all platforms. As you can see, there is nothing complicated in the registration process, I just add both my services and my ViewModels to the `ServiceCollection`.

I also have some convenience properties for the most important ViewModels that make `Binding` easier *(as I tend to keep code behind files as clean as possible*). If you need a service in another place in your app, and you are not using constructor injection (which gets automatically resolved by the `Microsoft.Extensions.DepedencyInjection` package), you can get the instance in the same way as I do with the ViewModel instances above.

### Conclusion

Creating a custom IServiceProvider implementation is not that hard. The custom implementation allows one to recreate the `IServiceProvider` (*handle with care!*) if needed. In the next post, I will show you how to deal with keyed ViewModel instances when using the `Extensions.DependencyInjection` package.

Have you already used the `Microsoft.Extensions.DependencyInjection` package with `Xamarin.Forms` or other platforms (not web)? What are your experiences? If so, leave a comment or chat with me on [Twitter](https://twitter.com/msicc)!

As always, I hope this post is helpful for some of you.

#### Until the next post – happy coding, everyone!