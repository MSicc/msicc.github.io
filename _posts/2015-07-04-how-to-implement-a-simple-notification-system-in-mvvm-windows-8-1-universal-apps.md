---
id: 4365
title: 'How to implement a simple notification system in MVVM Windows 8.1 Universal apps'
date: '2015-07-04T13:31:03+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-implement-a-simple-notification-system-in-mvvm-windows-8-1-universal-apps/
categories:
    - Archive
tags:
    - confirmation
    - mvvm
    - 'mvvm light'
    - notification
    - system
    - 'universal app'
    - 'Windows 8.1'
    - 'Windows Phone'
    - 'Windows Runtime'
---

[![screenshot_07042015_152814](/assets/img/2015/07/screenshot_07042015_152814_thumb.png "screenshot_07042015_152814")](/assets/img/2015/07/screenshot_07042015_152814.png)

If your app performs actions, you most probably want to add also some confirmation if an action has finished. There are some ways to do this, like using local toast notifications or MessageDialogs. While I was working on Voices Admin v2, which is a universal app, I came along with a [helper to simplify using local toast notifications](http://msicc.net/?p=4310). However, there came the point, where I got annoyed by the sound of these, and I looked into possible ways to replace them. My solution is a simple notification system, that uses the MVVM Light Messenger.

The first thing I did was adding a new property that broadcasts its PropertyChangedMessage to my ExtendedViewModelBase (which inherits from the MVVM Light ViewModelBase). This simplifies setting the notification text across multiple ViewModels as I don’t need to create a property in every ViewModel of my app:

``` csharp
 public class ExtendedViewModelBase : ViewModelBase
    {
        public ExtendedViewModelBase()
        {
            
        }


        /// <summary>
        /// The <see cref="NotificationText" /> property's name.
        /// </summary>
        public const string NotificationTextPropertyName = "NotificationText";

        private string _notificationText = string.Empty;

        /// <summary>
        /// Sets and gets the NotificationText property.
        /// Changes to that property's value raise the PropertyChanged event. 
        /// This property's value is broadcasted by the MessengerInstance when it changes.
        /// </summary>
        public string NotificationText
        {
            get
            {
                return  _notificationText = string.Empty;
            }
            set
            {
                Set(() => NotificationText, ref _notificationText, value, true);
            }
        }
   }
```
 
The second step is to create the possibility to bind this into my view. I am using a custom PageBase class to simplify this. For those binding purposes it is common to add a DependencyProperty, and this is exactly what I did:

``` csharp
 /// <summary>
        /// global property to bind the notification text against
        /// </summary>
        public static readonly DependencyProperty AppNotificationTextProperty = DependencyProperty.Register(
            "AppNotificationText", typeof (string), typeof (PageBase), new PropertyMetadata(string.Empty, (s, e) =>
            {
                var current = s as PageBase;

                if (current == null)
                {
                    return;
                }

                current.CheckifNotificationMessageIsNeeded(s);
            }));

        /// <summary>
        /// gets or sets the AppNotificationText
        /// </summary>
        public string AppNotificationText
        {
            get { return (string)GetValue(AppNotificationTextProperty); }
            set { SetValue(AppNotificationTextProperty, value); }}
```
 
You may have noticed that I hooked up into the PropertyChangedCallback of the DependecyProperty, which passes the execution to an separate method. Before we’ll have a look on that method, we need to add two private members to my PageBase: one for a StackPanel (mainly to set the Background color) and another one for Textblock. This is needed because this is the visible part of the notification. In the constructor of my PageBase class, I am filling them with live and connect them together:

``` csharp
             //instantiate and create StackPanel and TextBlock
            //you can put anything you want in the panel
            _panel = new StackPanel()
            {
                Background = new SolidColorBrush(Colors.Blue),
                Visibility = Visibility.Collapsed,
            };

            _textBlock = new TextBlock()
            {
                FontSize = 20,
                Margin = new Thickness(39, 10, 10, 10),
                TextAlignment = TextAlignment.Center
            };
            _panel.Children.Add(_textBlock);
```
 
The next thing we need to do is the FindChildren&lt;T&gt; helper method, which I took from the MSDN docs:

``` csharp
         /// <summary>
        /// Gets a list of DependencyObjects from the Visual Tree
        /// </summary>
        /// <typeparam name="T">the type of the desired object</typeparam>
        /// <param name="results">List of children</param>
        /// <param name="startNode">the DependencyObject to start the search with</param>
        public static void FindChildren<T>(List<T> results, DependencyObject startNode) where T : DependencyObject
        {
            int count = VisualTreeHelper.GetChildrenCount(startNode);
            for (int i = 0; i < count; i++)
            {
                var current = VisualTreeHelper.GetChild(startNode, i);
                if ((current.GetType()) == typeof(T) || (current.GetType().GetTypeInfo().IsSubclassOf(typeof(T))))
                {
                    T asType = (T)current;
                    results.Add(asType);
                }
                FindChildren<T>(results, current);
            }
         }
```
 
This helper enables us to find the top level grid, where we will add the StackPanel and control its visibilty and the TextBlock’s text. Which we are doing with the CheckifNotificationMessageIsNeeded() method:

``` csharp
         /// <summary>
        /// handles the visibility of the notification
        /// </summary>
        /// <param name="currentDependencyObject">the primary depenedency object to start with</param>
        private void CheckifNotificationMessageIsNeeded(DependencyObject currentDependencyObject)
        {
            if (currentDependencyObject == null) return;

            var children = new List<DependencyObject>();
            FindChildren(children, currentDependencyObject);
            if (children.Count == 0) return;

            var rootGrid = (Grid)children.FirstOrDefault(i => i.GetType() == typeof(Grid));

            if (rootGrid != null)

                if (!string.IsNullOrEmpty(AppNotificationText))
                {
                    if (!rootGrid.Children.Contains(_panel))
                    {
                        rootGrid.RowDefinitions.Add(new RowDefinition() {Height = new GridLength(_panel.ActualHeight, GridUnitType.Auto)});
                        _panel.SetValue(Grid.RowProperty, rootGrid.RowDefinitions.Count);

                        rootGrid.Children.Add(_panel);
                    }

                    _textBlock.Text = AppNotificationText;
                    _panel.Visibility = Visibility.Visible;
                }
                else if (string.IsNullOrEmpty(AppNotificationText))
                {
                    _textBlock.Text = string.Empty;
                    _panel.Visibility = Visibility.Collapsed;
                }
        }
```
 
Once we have the rootGrid on our Page, we are adding a new Row, set the StackPanel’s Grid.Row property to that and finally add the StackPanel to the Grid’s Children – but only if it does not exist already. No everytime the AppNotificationText property changes, the visibility of the StackPanel changes accordingly. Same counts for the TextBlock’s text. That’s all we need to do in the PageBase class.

The final bits of code we have to add are in the MainViewModel. I am using the MainViewModel as a kind of root ViewModel, which controls values and actions that are needed across multiple ViewModels. If you do not use it in the same way, you might need to write that code in all of your ViewModels where you want to use the notifications. The biggest advantage of my way is that the notification system (and other things) also works across pages.

The first thing we need is of course a property for the notification Text, which we will use to bind against on all pages where we want to use the notification system:

``` csharp
         /// <summary>
        /// The <see cref="GlobalNotificationText" /> property's name.
        /// </summary>
        public const string GlobalNotificationTextPropertyName = "GlobalNotificationText";

        private string _globalNotificationText = string.Empty;

        /// <summary>
        /// Sets and gets the GlobalNotificationText property.
        /// Changes to that property's value raise the PropertyChanged event. 
        /// </summary>
        public string GlobalNotificationText
        {
            get
            {
                return _globalNotificationText;
            }
            set
            {
                Set(() => GlobalNotificationText, ref _globalNotificationText, value);
            }
        }
```
 
Now we have this, we are hooking into the MVVM Messenger to catch the broadcasted NotificationText’s PropertyChangedMessage:

``` csharp
             Messenger.Default.Register<PropertyChangedMessage<string>>(this, message =>
            {
                if (message.PropertyName == ExtendedViewModelBase.NotificationTextPropertyName)
                {
                }
             });
```
 
If we would stop here, you would need to find a good point to set the NotificationText (and/or the GlobalNotificationText) property back to an empty string. This can be like the search a needle in the hay, believe me. That’s why I am giving every notification 5 seconds to be displayed, and the I am resetting the GlobalNotificationText property in my MainViewModel automatically. To achieve this goal, I am using a simple DispatcherTimer with an Interval of 1 second:

``` csharp
             _notificationTimer = new DispatcherTimer() { Interval = new TimeSpan(0, 0, 1) };
```
 
DispatcherTimer has a Tick event, which fires every time a Tick happened. In our case, it fires every second. Hooking up into this event is essential, so add this line of code and let Visual Studio create the handler for you:

``` csharp
 //in constructor:       
_notificationTimer.Tick += _notificationTimer_Tick;

//generated handler:
        private void _notificationTimer_Tick(object sender, object e)
        {
        }
```
 
Inside the Tick event handler, I am counting the ticks (using a private member in my MainViewModel). Once the timer passed 5 seconds, I am stopping the DispatcherTimer, reset the counter and finally set the GlobalNotificationText property back to empty, which causes the notification to disappear:

``` csharp
             _notificationTimerElapsedSeconds++;

            if (_notificationTimerElapsedSeconds > 5)
            {
                _notificationTimer.Stop();
                _notificationTimerElapsedSeconds = 0;
                GlobalNotificationText = string.Empty;
            }
```
 
Of course we also need to start the DispatcherTimer. The perfect time for this is within the handler of the received PropertyChangedMessage we added earlier:

``` csharp
             //register for the global NotificationText PropertyChangedMessage from all VMs that derive from ExtendenViewModelBase
            Messenger.Default.Register<PropertyChangedMessage<string>>(this, message =>
            {
                if (message.PropertyName == ExtendedViewModelBase.NotificationTextPropertyName)
                {
                    if (!_notificationTimer.IsEnabled)
                    {
                        _notificationTimer.Start();
                    }
                    else
                    {
                        _notificationTimerElapsedSeconds = 0;
                    }

                    GlobalNotificationText = message.NewValue;
                }
            });
```
 
I am just checking if the DispatcherTimer is not yet enabled (= running) and start the timer in this case. If it is already running, I am just resetting my counter property to make sure that the notification is visible for 5 seconds again.

That’s it. Your MVVM (Light) app has now a simple and not so annoying notification system. It also provides the same experience across both platforms. There are sure ways to improve this here and there, that’s why I [put up a sample to play around and contribute to on my Github account](https://github.com/MSiccDev/CustomNotificationSystem).

As always, I hope this post is helpful for some of you.

Happy coding!