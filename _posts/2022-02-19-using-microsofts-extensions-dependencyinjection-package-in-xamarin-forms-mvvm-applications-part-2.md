---
id: 7017
title: 'Using Microsoft’s Extensions.DependencyInjection package in (Xamarin.Forms) MVVM applications (Part 2)'
date: '2022-02-19T09:57:04+01:00'
author: 'Marco Siccardi'
excerpt: 'In the last post, I showed you how to use the IServiceProvider interface in general for dependency injection in your Xamarin.Forms app. In this post, I will show you how to add multiple registrations of the same ViewModel type and make them accessible with a key.'
layout: post
permalink: /using-microsofts-extensions-dependencyinjection-package-in-xamarin-forms-mvvm-applications-part-2/
image: /assets/img/2022/02/MS_DI_P2_Title_Image.png
categories:
    - 'Dev Stories'
    - Xamarin
tags:
    - 'Dependency Injection'
    - Ioc
    - key
    - keyed
    - keys
    - mvvm
    - mvvmlight
    - toolkit
    - ViewModel
    - xamarin
    - 'xamarin forms'
---

### The Key

Our goal is to add keyed registrations to the `IServiceCollection`, so we need a common denominator to build upon. As I was able to use a string with the `SimpleIoc` implementation of `MVVMLight` for years now, I decided to move on with that and created the following, very complex interface:

``` csharp
 public interface IViewModelKey
{
    string Key { get; set; }
}
```
 
Every `ViewModel` that should be registered by Key needs to implement that interface from now on in my MVVM environment.

### The Resolver

Back in the `MVVMLight` times, I was able to query the `SimpleIoc` registrations with the key I was searching for. In the `Microsoft.Extensions.DependencyInjection` world, things get a bit more complex. While there are different ways to solve the problem (there are some libraries extending the IServiceProvider with additional methods out there, for example), I decided to use the `IServiceProvider` itself and go down the *resolver interface/implementation* road.

Let’s have a look at the interface first:

``` csharp
 public interface IViewModelByKeyResolver<T> where T : IViewModelKey
{
    public T GetViewModelByKey(string key);
}
```
 
Nothing too special here, just a generic implementation of the resolver interface with the requirement of the `IViewModelKey` implementation from above. This makes the usage pretty straight forward. The more important part here is the implementation, though. Let’s have a look at mine:

``` csharp
 public class ViewModelByKeyResolver<T> : IViewModelByKeyResolver<T> where T : IViewModelKey
{
    private readonly IServiceProvider _serviceProvider;

    public ViewModelByKeyResolver(IServiceProvider serviceProvider)
        => _serviceProvider = serviceProvider;

    public T GetViewModelByKey(string key)
        => _serviceProvider.GetServices<T>().SingleOrDefault(vm => vm.Key == key);
}
```
 
The registration of the implementation will automatically inject the `IServiceProvider` instance at runtime for me here. The `GetViewModelByKey` method searches all registrations of the given type for the key and returns the desired instance.

### Registering the Resolver and keyed ViewModels

The registration of the resolver is done like all the other registrations:

``` csharp
 this.ServiceDescriptors.TryAddSingleton<IViewModelByKeyResolver<KeyedViewModel>, ViewModelByKeyResolver<KeyedViewModel>>();
```
 
Replace `KeyedViewModel` with your individual type that implements your key interface. That’s it.

For the registration of the `KeyedViewModel` instances, there is one thing to pay attention to, though. You cannot use the `TryAdd{Lifetime}` methods here for registration. Instead, just use the `Add{Lifetime}` method to register them. Here is a sample:

``` csharp
 this.ServiceDescriptors.AddSingleton<KeyedViewModel>(new KeyedViewModel("Key1"));
this.ServiceDescriptors.AddSingleton<KeyedViewModel>(new KeyedViewModel("Key2"));
this.ServiceDescriptors.AddSingleton<KeyedViewModel>(new KeyedViewModel("Key3"));
this.ServiceDescriptors.AddSingleton<KeyedViewModel>(new KeyedViewModel("Key4"));
this.ServiceDescriptors.AddSingleton<KeyedViewModel>(new KeyedViewModel("Key5"));
```
 
If you know the keyed ViewModels already at the time of your app startup, you can add them right away and create the `IServiceProvider` instance as shown [in my first post](https://msicc.net/using-microsofts-extensions-dependencyinjection-package-in-xamarin-forms-mvvm-applications-part-1/). In most cases, however, you will know the information of the keyed instances only at runtime. Luckily, my `Xamarin.Forms` implementation already has the solution built in. Here is a short reminder:

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
 
The `BuildServiceProvider` method has an additional parameter that allows to reset the existing `IServiceProvider`. This way, I can keep my existing registrations and just add the new keyed ones dynamically. *Please note that you may need to reinitialize your already registered and used ViewModels under certain circumstances after performing the reset.*

### Accessing a keyed ViewModel

Last but not least, I need to show you how to access a ViewModel by its key. Luckily, this is not that hard:

``` csharp
 KeyedViewModel vm4 = IocManager.Current.Services.GetService<IViewModelByKeyResolver<KeyedViewModel>>().GetViewModelByKey("Key4");
KeyedViewModel vm2 = IocManager.Current.Services.GetService<IViewModelByKeyResolver<KeyedViewModel>>().GetViewModelByKey("Key2");
```
 
### Conclusion

By switching to the `CommunityToolkit.MVVM` package and utilizing Microsoft’s `Extension.DependencyInjection` package together with it, my MVVM environment is ready for upcoming challenges like .NET MAUI. I will be able to use it on all .NET platforms and just need to adapt my `Xamarin.Forms` implementation to others (which I have done already for one of our internal tools at work in WPF). Even keyed ViewModel instance can be used similarly as before, as I showed you in this post.

As always, I hope this post will be helpful for some of you.

#### Until the next post, happy coding!