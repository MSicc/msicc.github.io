---
id: 4951
title: 'Xamarin Forms, the MVVMLight Toolkit and I: showing dialog messages'
date: '2017-06-29T13:00:41+02:00'
author: 'Marco Siccardi'
excerpt: 'This post will show you a simple way to show dialogs across platforms, utilizing Dependency Injection to route the Xamarin.Forms code to its native implementation.'
layout: post
permalink: /xamarin-forms-the-mvvmlight-toolkit-and-i-showing-dialog-messages/
image: /assets/img/2017/06/showing-message-dialog-xf-mvvmlight.png
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - Android
    - 'Dependency Injection'
    - dialog
    - interfaces
    - iOS
    - 'message dialog'
    - messages
    - mvvm
    - 'mvvm light'
    - uwp
    - xamarin
    - 'xamarin forms'
---

In this post, I will show you how to display dialog messages (also known as message box). This time, we will use again native implementations ([like in the second post about Dependency Injection](https://msicc.net/xamarin-forms-the-mvvmlight-toolkit-and-i-dependecy-injection/)) to get the job done. I could have used the Xamarin.Forms `Page.DisplayAlert`method, but that one does not allow a lot of customization, so I went down to implement it my way.

## Like always, the interface dictates functionality

Because of the Xamarin Forms code being a portable class library, we need an new interface that can be called from all three platforms. I am covering four scenarios which I use frequently in my apps:

``` csharp
 public interface IDialogService
{
    void CloseAllDialogs();

    Task ShowMessageAsync(string title, string message);

    Task ShowErrorAsync(string title, Exception error, string buttonText, Action<bool> closeAction, bool cancelableOnTouchOutside = false, bool cancelable = false);

    Task ShowMessageAsync(string title, string message, string buttonText, Action<bool> closeAction, bool cancelableOnTouchOutside = false, bool cancelable = false);

    Task ShowMessageAsync(string title, string message, string buttonConfirmText, string buttonCancelText, Action<bool> closeAction, bool cancelableOnTouchOutside = false, bool cancelable = false);
}
```
 
Of course, I want to display those message dialogs asynchronously, that’s why I wrap them in a Task. Let’s have a look into the implementation:

## Android

``` csharp
 List<AlertDialog> _openDialogs = new List<AlertDialog>();

public void CloseAllDialogs()
{
    foreach (var dialog in _openDialogs)
    {
        dialog.Dismiss();
    }
    _openDialogs.Clear();
}


public async Task ShowMessageAsync(string title, string message)
{
    await Task.Run(() => ShowAlert(title, message, "OK", null, null, false, false));
}


public async Task ShowErrorAsync(string title, Exception error, string buttonText, Action<bool> callback, bool cancelableOnTouchOutside = false, bool cancelable = false)
{
    await Task.Run(() => ShowAlert(title, error.ToString(), buttonText, null, callback, cancelableOnTouchOutside, cancelable));
}

public async Task ShowMessageAsync(string title, string message, string buttonText, Action<bool> callback, bool cancelableOnTouchOutside = false, bool cancelable = false)
{
    await Task.Run(() => ShowAlert(title, message, buttonText, null, callback, cancelableOnTouchOutside, cancelable));
}


public async Task ShowMessageAsync(string title, string message, string buttonConfirmText, string buttonCancelText, Action<bool> callback, bool cancelableOnTouchOutside = false, bool cancelable = false)
{
    await Task.Run(() => ShowAlert(title, message, buttonConfirmText, buttonCancelText, callback, cancelableOnTouchOutside, cancelable));
}


```
 
I am tunneling the defined Tasks of the interface via a call to `Task.Run()` into one single method call that is able to handle all scenarios I want to support. Let’s have a look at the `ShowAlert` method:

``` csharp
 internal void ShowAlert(string title, string content, string confirmButtonText = null, string cancelButtonText = null, Action<bool> callback = null, bool cancelableOnTouchOutside = false, bool cancelable = false)
{
    var alert = new AlertDialog.Builder(Forms.Context);
    alert.SetTitle(title);
    alert.SetMessage(content);

    if (!string.IsNullOrEmpty(confirmButtonText))
    {
        alert.SetPositiveButton(confirmButtonText, (sender, e) =>
        {
            callback?.Invoke(true);
            _openDialogs.Remove((AlertDialog)sender);
        });
    }

    if (!string.IsNullOrEmpty(cancelButtonText))
    {
        alert.SetNegativeButton(cancelButtonText, (sender, e) =>
        {
            callback?.Invoke(false);
            _openDialogs.Remove((AlertDialog)sender);
        });
    }

    Device.BeginInvokeOnMainThread(() =>
    {
        var dialog = alert.Show();
        _openDialogs.Add(dialog);
        dialog.SetCanceledOnTouchOutside(cancelableOnTouchOutside);
        dialog.SetCancelable(cancelable);

        if (cancelableOnTouchOutside || cancelable)
        {
            dialog.CancelEvent += (sender, e) =>
            {
                callback?.Invoke(false);
                _openDialogs.Remove((AlertDialog)sender);
            };
        }


    });
}
```
 
In the first three lines I am setting up a new `AlertDialog` , taking into account the actual `Xamarin.Forms.Context`, setting the title and the message content. If no other option is used, this shows just with the standard “OK”-Button to close the message.

Often we want to modify the button text, that’s where the `confirmButtonText` and `cancelButtonText`overloads are being used. I am also using a callback method that takes a Boolean to show which button on the message was pressed. Showing the Dialog needs to be done on the main UI Thread. Xamarin Forms provides the `Device.BeginInvokeOnMainThread` method to dispatch the code within the action into the right place.

I am showing the dialog while keeping a reference in the `_openDialogs` List. This reference is removed once the matching button or cancel action is executed. If the message is allowed to be dismissed via outside touches or other cancel methods, this happens in the `CancelEvent` Eventhandler delegate I am attaching.

The implementation also has a way to close all open dialogs, but it is a good practice to have only one open at a time, especially as other platforms do support only one open dialog at a time.

## UWP

Microsoft recommends to use the `ContentDialog` [class](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.contentdialog) to show messages and dialogs of all kind. So this is what we will use to show our dialog messages. Like on Android, we need a `Task.Run()`wrapper to make the implementation async.

``` csharp
 List<ContentDialog> _openDialogs = new List<ContentDialog>(); 
  
  
public void CloseAllDialogs() 
{ 
    foreach (var dialog in _openDialogs) 
    { 
        dialog.Hide(); 
    } 
    _openDialogs.Clear(); 
} 
  
  
public async Task ShowErrorAsync(string title, Exception error, string buttonText, Action<bool> closeAction, bool cancelableOnTouchOutside = false, bool cancelable = false) 
{ 
    await Task.Run(() => { ShowContentDialog(title, error.ToString(), buttonText, null, closeAction, cancelableOnTouchOutside, cancelable); }); 
} 
  
public async Task ShowMessageAsync(string title, string message) 
{ 
    await Task.Run(() => { ShowContentDialog(title, message, "OK", null, null, false, false); }); 
} 
  
public async Task ShowMessageAsync(string title, string message, string buttonText, Action<bool> closeAction, bool cancelableOnTouchOutside = false, bool cancelable = false) 
{ 
    await Task.Run(() => { ShowContentDialog(title, message, buttonText, null, closeAction, cancelableOnTouchOutside, cancelable); }); 
} 
  
public async Task ShowMessageAsync(string title, string message, string buttonConfirmText, string buttonCancelText, Action<bool> closeAction, bool cancelableOnTouchOutside = false, bool cancelable = false) 
{ 
    await Task.Run(() => { ShowContentDialog(title, message, buttonConfirmText, buttonCancelText, closeAction, cancelableOnTouchOutside, cancelable); }); 
} 
  

```
 
Even if UWP allows only one `ContentDialog` to be open, we are using the list of open dialogs to be able to close “them” for compatibility reasons. The next step to implement is the `ShowContentDialog` Task all methods above are using:

``` csharp
 internal void ShowContentDialog(string title, string content, string confirmButtonText = null, string cancelButtonText = null, Action<bool> callback = null, bool cancelableOnTouchOutside = false, bool cancelable = false) 
{ 
    Device.BeginInvokeOnMainThread(async () => 
    { 
        var messageDialog = new ContentDialog() 
        { 
            Title = title, 
            Content = content,                     
        }; 
  
  
    if (!string.IsNullOrEmpty(confirmButtonText)) 
    { 
  
        if (string.IsNullOrEmpty(cancelButtonText)) 
        { 
            messageDialog.CloseButtonText = confirmButtonText; 
  
            messageDialog.CloseButtonClick += (sender, e) => 
            { 
                callback?.Invoke(true); 
                _openDialogs.Remove((ContentDialog)sender); 
            }; 
        } 
        else 
        { 
            messageDialog.PrimaryButtonText = confirmButtonText; 
  
            messageDialog.PrimaryButtonClick += (sender, e) => 
            { 
                callback?.Invoke(true); 
                _openDialogs.Remove((ContentDialog)sender); 
            }; 
  
        } 
    } 
  
        if (!string.IsNullOrEmpty(cancelButtonText)) 
        { 
            messageDialog.CloseButtonText = cancelButtonText; 
  
            messageDialog.CloseButtonClick += (sender, e) => 
            { 
                callback?.Invoke(false); 
                _openDialogs.Remove((ContentDialog)sender); 
            }; 
        } 
  
  
  
        _openDialogs.Add(messageDialog); 
  
        await messageDialog.ShowAsync(); 
    }); 
} 
```
 
The setup of the dialog is kind of similar to android. First, we are creating a new dialog setting the title and the message.

The second part is a bit more complex here. Dialogs should hook into the `CloseButton `properties and events in all cases, at least after OS-Version 1703 (Creators Update). This way, the `CloseButtonClick`event is also raised when the user presses the ESC-Button, the system back button, the close button of the dialog as well as the B-Button on the Xbox-Controller.

When we have only one button to show, our `confirmButtonText` is directed to the `CloseButton`, otherwise to the `PrimaryButton`. In the second case, the `CloseButton`is connected with the `cancelButtonText`. We are using the same callback Action as on Android, where the bool parameter indicates which button was pressed.

In the UWP implementation the additional parameters `cancelableOnTouchOutside`and `cancelable`are not used.

## iOS

The base implementation on iOS is basically the same like on Android and UWP. In case of iOS, we are using the `UIAlertController` [class](https://developer.xamarin.com/api/type/MonoTouch.UIKit.UIAlertController/), which is mandatory since iOS 8.

``` csharp
 List<UIAlertController> _openDialogs = new List<UIAlertController>();

public void CloseAllDialogs()
{
    foreach (var dialog in _openDialogs)
    {
        Device.BeginInvokeOnMainThread(() =>
        {
            dialog.DismissViewController(true, null);
        });
    }
    _openDialogs.Clear();
}

public async Task ShowErrorAsync(string title, Exception error, string buttonText, Action<bool> closeAction, bool cancelableOnTouchOutside = false, bool cancelable = false)
{
    await Task.Run(() => { ShowAlert(title, error.ToString(), buttonText, null, closeAction, cancelableOnTouchOutside, cancelable); });
}

public async Task ShowMessageAsync(string title, string message)
{
    await Task.Run(() => { ShowAlert(title, message, "OK", null, null, false, false); });
}

public async Task ShowMessageAsync(string title, string message, string buttonText, Action<bool> closeAction, bool cancelableOnTouchOutside = false, bool cancelable = false)
{
    await Task.Run(() => { ShowAlert(title, message, buttonText, null, closeAction, cancelableOnTouchOutside, cancelable); });
}

public async Task ShowMessageAsync(string title, string message, string buttonConfirmText, string buttonCancelText, Action<bool> closeAction, bool cancelableOnTouchOutside = false, bool cancelable = false)
{
    await Task.Run(() => { ShowAlert(title, message, buttonConfirmText, buttonCancelText, closeAction, cancelableOnTouchOutside, cancelable); });
}
```
 
Now that we have the Task wrappers in place, we need to implement the `ShowAlert` method:

``` csharp
 internal void ShowAlert(string title, string content, string confirmButtonText = null, string cancelButtonText = null, Action<bool> callback = null, bool cancelableOnTouchOutside = false, bool cancelable = false)
{
    //all this code needs to be in here because UIKit demands the main UI Thread
    Device.BeginInvokeOnMainThread(() =>
    {
        var dialogAlert = UIAlertController.Create(title, content, UIAlertControllerStyle.Alert);

        var okAction = UIAlertAction.Create(!string.IsNullOrEmpty(confirmButtonText) ? confirmButtonText : "OK", UIAlertActionStyle.Default, _ =>
        {
            callback?.Invoke(true);
            _openDialogs.Remove(dialogAlert);
        });
        dialogAlert.AddAction(okAction);


        if (!string.IsNullOrEmpty(cancelButtonText))
        {
            var cancelAction = UIAlertAction.Create(cancelButtonText, UIAlertActionStyle.Cancel, _ =>
            {
                callback?.Invoke(false);
                _openDialogs.Remove(dialogAlert);
            });
            dialogAlert.AddAction(cancelAction);
        }
        _openDialogs.Add(dialogAlert);

        var rootController = UIApplication.SharedApplication.KeyWindow.RootViewController;

        rootController.PresentViewController(dialogAlert, true, null);
    });
}
```
 
What I am doing here is straight forward – I am setting up a new `UIAlertController` instance via its creation method, telling it to be styled as an alert. Then I need to create two `UIAlertAction` instances, one for the `confirmButtonText` and one for the `cancelButtonText`. Of course, I am hooking up the prior defined callback action, which will inform the `Xamarin.Forms` class about the result of the dialog.

To display the Alert, we need a reference to the `RootViewController` of the iOS application. In most `Xamarin.Forms` applications, the above code will do the job to present the `UIAlertController` via the `PresentViewController` [method](https://developer.xamarin.com/api/member/UIKit.UIViewController.PresentViewController/) provided by the OS. Like the UWP implementation, also the iOS implementation needs to be executed in the main UI thread, because the `UIKit` demands it. That’s why also here, the whole code is running inside the `Device.BeginInvokeOnMainThread` method’s action delegate.

The additional parameters `cancelableOnTouchOutside`and `cancelable`are not used on iOS.

## Updating our ViewModelLocator

If you have read [my post on Dependency Injection](https://msicc.net/xamarin-forms-the-mvvmlight-toolkit-and-i-dependecy-injection/), you might remember that we are able to combine the power of MVVMLight with `Xamarin.Forms` own `DependencyService`. This is what we will do once again in our `ViewModelLocator` in the `RegisterServices` method:

``` csharp
 var dialogService = DependencyService.Get<IDialogService>();
SimpleIoc.Default.Register<IDialogService>(() => dialogService);
```
 
And that’s already all we need to do here. Really.

## Finally: Showing message dialogs

I added three buttons to the sample’s main page. One shows a simple message, one shows the exception that I manually throw and the last one provides a two choice dialog. Let’s have a quick look at the commands bound to them:

``` csharp
  
private RelayCommand _showMessageCommand; 
  
public RelayCommand ShowMessageCommand => _showMessageCommand ?? (_showMessageCommand = new RelayCommand(async () => 
{ 
    await SimpleIoc.Default.GetInstance<IDialogService>().ShowMessageAsync("Cool... ?", "You really clicked this button!"); 
}));
```
 
This shows a simple message without using the additional parameters. I use this one primarily for confirmations.

``` csharp
 private RelayCommand _showErrorWithExceptionCommand;

public RelayCommand ShowErrorWithExceptionCommand => _showErrorWithExceptionCommand ?? (_showErrorWithExceptionCommand = new RelayCommand(async () =>
{
    try
    {
        throw new NotSupportedException("You tried to fool me, which is not supported!");
    }
    catch (Exception ex)
    {
        await SimpleIoc.Default.GetInstance<IDialogService>().ShowErrorAsync("Error", ex, "Sorry",
            returnValue =>
            {
                Debug.WriteLine($"{nameof(ShowErrorWithExceptionCommand)}'s dialog returns: {returnValue}");
            }, false, false);
    }
}));
```
 
This one takes an exception and shows it on the screen. I use them mainly for developing purposes.

``` csharp
 private RelayCommand _showSelectionCommand;

public RelayCommand ShowSelectionCommand => _showSelectionCommand ?? (_showSelectionCommand = new RelayCommand(async () =>
{

    await SimpleIoc.Default.GetInstance<IDialogService>().ShowMessageAsync("Question:",
        "Do you enjoy this blog series about MVVMLight and Xamarin Forms?", "yeah!", "nope", async returnvalue =>
        {
            if (returnvalue)
            {
                await SimpleIoc.Default.GetInstance<IDialogService>()
                    .ShowMessageAsync("Awesome!", "I am glad you like it");
            }
            else
            {
                await SimpleIoc.Default.GetInstance<IDialogService>()
                    .ShowMessageAsync("Oh no...", "Maybe you could send me some feedback on how to improve it?");
            }
        },
        false, false);

}));
```
 
This one provides a choice between two options. A good example where I use this is when there is no internet connection and I ask the user to open the WiFi settings or cancel. In the sample, I am also showing another simple message after one of the buttons has been pressed. The content of this simple message depends on which button was clicked.

## Conclusion

Using Xamarin.Forms’ `DependencyService` together with the SimpleIoc implementation of MVVMLight, we are once again easily able to connect platform specific code to our Xamarin.Forms project. Every platform implementation follows the dialog recommendations and is executed using the native implementations while keeping some options open to use different kind of message dialogs.

As always, I hope this post is helpful for some of you. Until the next post, happy coding!