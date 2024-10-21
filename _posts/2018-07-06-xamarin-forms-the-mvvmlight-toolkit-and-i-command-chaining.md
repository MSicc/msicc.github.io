---
id: 5719
title: 'Xamarin Forms, the MVVMLight Toolkit and I: Command Chaining'
date: '2018-07-06T08:00:07+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I will show you how to invoke methods within custom or derived controls in MVVM driven Xamarin.Forms apps, using a technique called "Command Chaining".'
layout: post
permalink: /xamarin-forms-the-mvvmlight-toolkit-and-i-command-chaining/
image: /assets/img/2018/07/chains-919058_1280.jpg
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - command
    - 'command chaining'
    - ICommand
    - 'invoke commands'
    - mvvm
    - mvvmlight
    - RelayCommand
    - ViewModel
    - xamarin
    - 'xamarin forms'
---

### The problem

Sometimes, we want to invoke a method that is available via code for a control. Due to the abstraction of our MVVM application, the ViewModel has no access to all those methods that are available if we would access the control via code. There are several approaches to solve this problem. In one of my recent projects, I needed to invoke a method of a custom control, which should be routed into the platform renderers I wrote for such a custom control. I remembered that I have indeed read quite a few times about command chaining for such cases and tried to implement it. In the beginning, it may sound weird to do this, but the more often I see this technique, the more I like it.

### Simple Demo control

For demo purposes, I created this really simple Xamarin.Forms user control:

``` xml
 <?xml version="1.0" encoding="UTF-8"?>
<ContentView xmlns="https://xamarin.com/schemas/2014/forms" 
             xmlns:x="https://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="XfMvvmLight.Controls.CommandChainingDemoControl">
  <ContentView.Content>
      <StackLayout HorizontalOptions="FillAndExpand" VerticalOptions="FillAndExpand">
            <Label x:Name="LabelFilledFromBehind" Margin="12" FontSize="Large" />   
        </StackLayout>
  </ContentView.Content>
</ContentView>
```
 
As you can see, there is just a label without text. We will write the necessary code to fill this label with some text just by invoking a method in the code behind. To be able to do so, we need a BindableProperty (once again) to get our foot into the door of the control:

``` csharp
 public static BindableProperty DemoCommandProperty = BindableProperty.Create(nameof(DemoCommand), typeof(ICommand), typeof(CommandChainingDemoControl), null, BindingMode.OneWayToSource);

public ICommand DemoCommand
{
    get => (ICommand)GetValue(DemoCommandProperty);
    set => SetValue(DemoCommandProperty, value);
}
```
 
The implementation is pretty straightforward. We have done this already during this series, so you should be familiar if you were following. One thing, however, is different. For this BindableProperty, we are using [BindingMode.OneWayToSource](https://docs.microsoft.com/en-us/dotnet/api/xamarin.forms.bindingmode?view=xamarin-forms). By doing so, we are basically making it a read-only property, which sends its changes only down to the ViewModel (the source). If we would not do this, the ViewModel could change the property, which we do not want here.

Now we have the BindableProperty in place, we need to create an instance of the Command that will be sent down to the ViewModel. We are doing this as soon as the control is instantiated in the constructor:

``` csharp
 public CommandChainingDemoControl()
      {
          InitializeComponent();

          this.DemoCommand = new Command(() =>
          {
              FillFromBehind();
          });
      }

      private void FillFromBehind()
      {
          this.LabelFilledFromBehind.Text = "Text was empty, but we used command chaining to show this text inside a control.";
      }
```
 
That’s all we need to do in the code behind.

### ViewModel

For this demo, I created a new page and a corresponding ViewModel in the demo project. Here is the very basic ViewModel code:

``` csharp
 using System.Windows.Input;
using GalaSoft.MvvmLight.Command;

namespace XfMvvmLight.ViewModel
{
    public class CommandChainingDemoViewModel : XfNavViewModelBase
    {
        private ICommand _invokeDemoCommand;
        private RelayCommand _demo1Command;

        public CommandChainingDemoViewModel()
        {
        }

        public ICommand InvokeDemoCommand { get => _invokeDemoCommand; set => Set(ref _invokeDemoCommand, value); }

        public RelayCommand Demo1Command => _demo1Command ?? (_demo1Command = new RelayCommand(() =>
        {
            this.InvokeDemoCommand?.Execute(null);
        }));
    }
}
```
 
As you can see, the ViewModel includes two Commands. One is the pure ICommand implementation that gets its value from the OneWayToSource-Binding. We are not using MVVMLight’s RelayCommand here to avoid casting between types, which always led to an exception when I tested the implementation first. The second command is bound to a button in the `CommandChainingDemoPage` and will be the trigger to execute the `InvokeDemoCommand`.

### Final steps

The final steps are just a few simple ones. We need to connect the <span style="display: inline !important; float: none; background-color: transparent; color: #333333; cursor: text; font-family: Georgia,'Times New Roman','Bitstream Charter',Times,serif; font-size: 16px; font-style: normal; font-variant: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;"> </span>`InvokeDemoCommand` to the user control we created earlier, while we need to bind the `Demo1Command`to the corresponding button in the view. This is the page’s code after doing so:

``` xml
 <?xml version="1.0" encoding="utf-8" ?>
<baseCtrl:XfNavContentPage
    xmlns:baseCtrl="clr-namespace:XfMvvmLight.BaseControls" xmlns="https://xamarin.com/schemas/2014/forms"
             xmlns:x="https://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:ctrl="clr-namespace:XfMvvmLight.Controls;assembly=XfMvvmLight"
             x:Class="XfMvvmLight.View.CommandChainingDemoPage" RegisteredPageKey="{Binding CommandChainingDemoPageKey, Source=Locator}">
    <ContentPage.BindingContext>
        <Binding Path="CommandChainingDemoVm" Source="{StaticResource Locator}" />
    </ContentPage.BindingContext>

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <ctrl:CommandChainingDemoControl Grid.Row="0" DemoCommand="{Binding InvokeDemoCommand, Mode=OneWayToSource}" Margin="12"></ctrl:CommandChainingDemoControl>

        <Button Text="Execute Command Chaining" Command="{Binding Demo1Command}" Margin="12" Grid.Row="1" />

    </Grid>
</baseCtrl:XfNavContentPage>
```
 
One thing to point out is that we are also specifying the OneWayToSource binding here once again. It should work with normal binding, but it I recommend to do like I did, which makes the code easier to understand for others (and of course yourself). That’s all – we have now a working command chain that invokes a method inside the user control from our ViewModel.

## Conclusion

Command chaining can be a convenient way to invoke actions on controls that are otherwise not possible due to the abstraction of layers in MVVM. Once you got the concept, they are pretty easy to implement. This technique is also usable outside of Xamarin.Forms, so do not hesitate to use it out there. Just remember the needed steps:

- create a user control (or a derived one if you need to call a method on framework controls)
- add a `BindableProperty`/`DependecyProperty` and set its default binding mode to OneWayToSource
- instantiate the `BindableProperty`/`DependecyProperty`<span style="display: inline !important; float: none; background-color: transparent; color: #333333; cursor: text; font-family: Georgia,'Times New Roman','Bitstream Charter',Times,serif; font-size: 16px; font-style: normal; font-variant: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;"> inside the constructor of the user control</span>
- pass the method call/code into the `Action` part of the newly created `Command` instance
- create the commands in the ViewModel
- connect the Commands to your final view implementation

Like I wrote earlier, I came across this (again) when I was writing a custom Xamarin.Forms control with renderers, where I had to invoke methods inside the renderer from my ViewModel. Other techniques that I saw to solve this is using Messengers (be it the one from MVVMLight or the Xamarin.Forms Messenger implementation) or the good old Boolean switch implementation (uses also a `BindableProperty`/`DependecyProperty`). I decided to use the command chaining approach as it is pretty elegant in my eyes and not that complicated to implement.

The series’ sample project is updated and available [here on Github](https://github.com/MSiccDev/XfMvvmLight). Like always, I hope this post is useful for some of you.

#### Happy coding, everyone!

---

#TODO: all articles of this series

[title image credit](https://pixabay.com/en/chains-crane-industrial-heavy-load-919058/)