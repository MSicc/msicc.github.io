---
id: 4305
title: 'How to easily check the pressed keyboard button with a converted event using MVVM (Windows Universal)'
date: '2015-02-16T19:05:26+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-easily-check-the-pressed-keyboard-button-with-a-converted-event-using-mvvm-windows-universal/
categories:
    - Archive
tags:
    - 'behaviors sdk'
    - binding
    - 'convert events'
    - focus
    - 'invoke commands'
    - keyboard
    - mvvm
    - 'mvvm light'
    - textbox
    - windev
    - 'Windows 8.1'
    - 'Windows Phone 8.1'
    - wpdev
---

In case you missed it, I lately am deeply diving into MVVM. Earlier today, I wanted to implement the well loved feature that a search is performed by pressing the Enter button. Of course, this would be very easy to achieve in code behind using the KeyUpEvent (or the KeyDownEvent, if you prefer).

However, in MVVM, especially in a Universal app, this is a bit trickier. We need to route the event manually to our matching command. There are surely more ways to achieve it, but I decided to use the [Behaviors SDK](https://msdn.microsoft.com/en-us/library/windows/apps/dn457340.aspx) to achieve my goal. The first step is of course downloading the matching extension (if you haven’t done so before). To do so, click on TOOLS/Extensions and Updates in Visual Studio and install the Behaviors SDK from the list:![image](/assets/img/2015/02/image.png "image")

The next step we need to do is to add a new Converter (I added it to the common folder, you may change this to your preferred place). As we are hooking up the KeyUpEventArgs, I called it KeyUpEventArgsConverter. After you created the class, implement the [IValueConverter](https://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.data.ivalueconverter.aspx) interface. You should now have a Convert and a ConvertBack method. We are just adding two lines of code to the Convert method:

``` csharp
             var args = (KeyRoutedEventArgs)value;
            return args;
```
 
That’s it for the Converter. Save the class and build the project. For the next step, we need to go to our View where the Converter should do its work. Before we can use it, we need to give our Converter a key to be identified by the Binding engine. You can do this app wide in App.xaml, or in your page:

``` xml
 <common:KeyUpEventArgsConverter x:Key="KeyUpEventArgsConverter"/>
```
 
Also, we need to add two more references to our View (besides the Common folder that holds our converter, that is):

``` xml
     xmlns:i="using:Microsoft.Xaml.Interactivity" 
    xmlns:core="using:Microsoft.Xaml.Interactions.Core"
```
 
The next step is to implement the Behavior to our input control (a TextBox in my case):

``` xml
 <TextBox  Header="enter search terms" PlaceholderText="search terms" Text="{Binding KnowledgeBaseSearchTerms, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" >
    <i:Interaction.Behaviors>
         <core:EventTriggerBehavior EventName="KeyUp">
             <core:InvokeCommandAction
                   Command="{Binding SearchTermKeyEventArgsCommand}"
                   InputConverter="{StaticResource KeyUpEventArgsConverter}">
             </core:InvokeCommandAction>
          </core:EventTriggerBehavior>
     </i:Interaction.Behaviors>
</TextBox>
```
 
With the EventTriggerBehavior, we are able to hook into the desired event of a control. We then only need to bind to a Command in our ViewModel and tell the Behaviors SDK that it should route the “KeyUp” event using our Converter.

Let’s have a final look at the command that handles the event:

``` csharp
         public RelayCommand<KeyRoutedEventArgs> SearchTermKeyEventArgsCommand
        {
            get
            {
                return _searchTermKeyEventArgsCommand
                    ?? (_searchTermKeyEventArgsCommand = new RelayCommand<KeyRoutedEventArgs>(
                    p =>
                    {
                        if (p.Key == Windows.System.VirtualKey.Enter)
                        {
                            //your code here
                        }
                    }));
            }
        }
```
 
As you can see, we are using a Command that is able to take a [Generic](https://msdn.microsoft.com/en-us/library/512aeb7t.aspx) (in my case it comes from the MVVM Light Toolkit, but there are several other version floating around). Because of this, we are finally getting the KeyRoutedEventArgs into our ViewModel and are able to use its data and properties.

The VirtualKey Enumeration holds a reference to a lot of (if not all) keys and works for both hardware and virtual keyboards. This makes this code safe to use in an Universal app.

As I am quite new to MVVM, I am not entirely sure if this way is the “best” way, but it works as expected with little efforts. I hope this will be useful for some of you.

Useful links that helped me on my way to find this solution:

[http://blog.galasoft.ch/posts/2014/01/using-the-eventargsconverter-in-mvvm-light-and-why-is-there-no-eventtocommand-in-the-windows-8-1-version/](http://blog.galasoft.ch/posts/2014/01/using-the-eventargsconverter-in-mvvm-light-and-why-is-there-no-eventtocommand-in-the-windows-8-1-version/ "http://blog.galasoft.ch/posts/2014/01/using-the-eventargsconverter-in-mvvm-light-and-why-is-there-no-eventtocommand-in-the-windows-8-1-version/")

[https://msdn.microsoft.com/en-us/library/windows/apps/xaml/hh868246.aspx](https://msdn.microsoft.com/en-us/library/windows/apps/xaml/hh868246.aspx "https://msdn.microsoft.com/en-us/library/windows/apps/xaml/hh868246.aspx")

Happy coding, everyone!