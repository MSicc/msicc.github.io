---
id: 4982
title: 'Xamarin Forms, the MVVMLight Toolkit and I: EventToCommandBehavior'
date: '2017-07-16T08:00:31+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post I am showing you how to implement an EventToCommandBehavior, which makes it very easy to get View events via Binding into a ViewModel.'
layout: post
permalink: /xamarin-forms-the-mvvmlight-toolkit-and-i-eventtocommandbehavior/
image: /assets/img/2017/07/eventtocommand-behavior-xf-mvvmlight.png
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - Android
    - behavior
    - behaviors
    - 'bindable properties'
    - binding
    - command
    - Events
    - iOS
    - uwp
    - View
    - ViewModel
    - VisualElement
    - 'xamarin forms'
    - XAML
---

Often, we want to/need to know when views throw certain events. However, due to using the MVVM pattern, our application logic is separated from the view. There are several ways to get those events into our ViewModel while keeping it separated from the views. One of those is using an interface, which I showed you already in [my blog post about navigation in Xamarin.Forms with MVVMLight](http://bit.ly/2sA1dof).

Another way is the good old `EventToCommand `approach. Some of you might have used this approach already in WPF and other .NET applications. `Xamarin.Forms` has them too, this post will show you how to implement it.

## Xamarin.Forms Behaviors

In Windows applications like WPF or UWP, we normally use the Interactivity namespace to use behaviors. `Xamarin.Forms` however has its own implementation, so we need to use the [Behavior](https://developer.xamarin.com/api/type/Xamarin.Forms.Behavior/) and [Behavior&lt;T&gt;](https://developer.xamarin.com/api/type/Xamarin.Forms.Behavior%3CT%3E/) classes. All controls that derive from [View](https://developer.xamarin.com/api/type/Xamarin.Forms.View/) are providing this [BindableProperty](https://developer.xamarin.com/api/type/Xamarin.Forms.BindableProperty/), so we can use Behaviors in a lot of scenarios. Until the new [XAML Standard is finally defined](https://blogs.windows.com/buildingapps/2017/05/19/introducing-xaml-standard-net-standard-2-0/#KRg5LPVcCEAEQwOm.97), we have to deal with this.

## EventToCommandBehavior

Xamarin provides a nearly ready-to-use `EventToCommandBehavior` implementation and [an quite detailed explanation](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/behaviors/reusable/event-to-command-behavior/) (which is why I won’t go into details on that). The implementation has two part – the `BehaviorBase<T>`implementation and the `EventToCommandBehavior` implementation itself.

While we are able to use the [BehaviorBase&lt;T&gt; implementation](https://github.com/xamarin/xamarin-forms-samples/blob/master/Behaviors/EventToCommandBehavior/EventToCommandBehavior/Behaviors/BehaviorBase.cs) as is, we have to do some minor changes to the `EventToCommandBehavior` to enable a few more usage scenarios.

The first change we need to make is to derive [Xamarin’s EventToCommandBehavior sample](https://github.com/xamarin/xamarin-forms-samples/blob/master/Behaviors/EventToCommandBehavior/EventToCommandBehavior/Behaviors/EventToCommandBehavior.cs) from [VisualElement](https://developer.xamarin.com/api/type/Xamarin.Forms.VisualElement/) instead of `View`. This way, we can also use the behavior on controls that do not derive from `View`, especially in [Pages](https://developer.xamarin.com/api/type/Xamarin.Forms.Page/). Pages do not derive from `View`, but they do from `VisualElement `(like `View`does, too). You need to change the Type also on the parameter of the `OnAttachedTo` and `OnDetachingFrom `methods in this case (which are the other two changes we need to do).

The rest of the implementation is basically the same like in the Xamarin sample and works quite well.

To show you a simple sample in Action, we are using the `Appearing` and `Disappearing` events to attach them via the behavior into our `ModalPageViewModel`on the `ModalPage `we integrated before. This way, you won’t need the `IViewEventBrokerService `I showed you in [my post on navigation and modal pages](http://bit.ly/2sA1dof). It is up to you to choose the way you want to go along, both ways are fully respecting the MVVM pattern.

## Implementation

The implementation has two parts. As we want to handle the events in a `Command`, the first step to take is to implement two Commands in the corresponding ViewModel. I am using a base implementation (in my apps and also in this sample), so I am going to implement the Commands there. This way, every derived ViewModel can bind to this `Command`. Additionally, I am using a `Execute...Command` method and a `CanExecute` boolean method, which can both be overriden in derived ViewModels to implement the code to execute. Let’s have a look at the code:

``` csharp
 public RelayCommand ViewAppearingCommand => _viewAppearingCommand ?? (_viewAppearingCommand = new RelayCommand(ExecuteViewAppearingCommand, CanExecuteViewAppearingCommand));

public virtual void ExecuteViewAppearingCommand()
{

}

public virtual bool CanExecuteViewAppearingCommand()
{
    return true;
}

public RelayCommand ViewDisappearingCommand => _viewDisappearingCommand ?? (_viewDisappearingCommand = new RelayCommand(ExecuteViewDisappearingCommand, CanExecuteViewDisappearingCommand));

public virtual void ExecuteViewDisappearingCommand()
{

}

public virtual bool CanExecuteViewDisappearingCommand()
{
    return true;
}
```
 
The second part is the XAML part, which includes the Binding to the Command properties we just created. The implementation is as easy as these four lines for both events:

``` xml
     <baseCtrl:XfNavContentPage.Behaviors>
        <behaviors:EventToCommandBehavior EventName="Appearing" Command="{Binding ViewAppearingCommand}"></behaviors:EventToCommandBehavior>
        <behaviors:EventToCommandBehavior EventName="Disappearing" Command="{Binding ViewDisappearingCommand}"></behaviors:EventToCommandBehavior>
    </baseCtrl:XfNavContentPage.Behaviors>
```
 
That’s it, if you want to attach the behavior only for individual Pages. If you have a base page implementation like I do however, you can automatically attach the event already there to have it attached to all pages:

``` csharp
 private void XfNavContentPage_BindingContextChanged(object sender, EventArgs e)
{
    if (this.BindingContext is XfNavViewModelBase)
    {
        this.Behaviors.Add(new EventToCommandBehavior()
        {
            EventName = "Appearing",
            Command = ((XfNavViewModelBase)this.BindingContext).ViewAppearingCommand
        });
 
        this.Behaviors.Add(new EventToCommandBehavior()
        {
            EventName = "Disappearing",
            Command = ((XfNavViewModelBase)this.BindingContext).ViewDisappearingCommand
        });
    }
}
```
 
I am attaching the behaviors only if the `BindingContext`does derive from my `XfNavViewModelBase`. The `Command `can be set directly in this case, without the need to use the `SetBinding `method.

These few lines are connecting the `Event `to the `Command`, the only thing we need to do is to override the base implementations of the “Execute…Command” methods:

``` csharp
 public override async void ExecuteViewAppearingCommand()
{
    base.ExecuteViewAppearingCommand();
    await _dialogService.ShowMessageAsync(this.CorrespondingViewKey, $"from overriden {nameof(ExecuteViewAppearingCommand)}");
}
 
public override async void ExecuteViewDisappearingCommand()
{
    base.ExecuteViewDisappearingCommand();
    await _dialogService.ShowMessageAsync(this.CorrespondingViewKey, $"from overriden {nameof(ExecuteViewDisappearingCommand)}");
}
```
 
The above overrides are using the IDialogService you will find in the sample application to show a simple message from which overriden `Execute...Command` method they are created from.

## Converting EventArgs to specific types

`Xamarin.Forms` has only a few events that have usefull `EventArgs`. At the time of writing this post, I tried to find valid scenarios where we want to get some things of the events to attach also an `IValueConverter`implementation to get this data out of them. Fact is, the only one I ever used is the one from the Xamarin sample – which is a converter that gets the selected Item for a `ListView`. Because `Xamarin.Forms` Views already provide most of the properties I ever needed, I was able to solve everything else via Binding. To make this post complete, you can [have a look into Xamarin’s sample implementation here](https://github.com/xamarin/xamarin-forms-samples/blob/master/Behaviors/EventToCommandBehavior/EventToCommandBehavior/Converters/SelectedItemEventArgsToSelectedItemConverter.cs).

## Conclusion

Hooking into events on the view side of our applications can be done in several ways. It is up to you to choose the route you want to go. With this post, I showed you a second way to achieve this.

If you have some more valid scenarios for using the `EventToCommandBehavior`with a Converter that cannot be solved via Binding directly, I would love to hear them. Feel free to leave a comment here or via social networks. Of course, [I updated the sample on Github](https://github.com/MSiccDev/XfMvvmLight) with the code from this post.

As always, I hope this post is helpful for some of you. Until the next post, happy coding!