---
id: 4158
title: 'Prevent accidentally exit of your Windows Phone SL app (with Telerik RadMessageBox or Toolkit CustomMessageBox)'
date: '2014-09-27T09:43:22+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /prevent-accidentally-exit-of-your-windows-phone-sl-app-with-telerik-radmessagebox-or-toolkit-custommessagebox/
categories:
    - Archive
tags:
    - 'accidentally exit'
    - 'back button'
    - 'exit app'
    - prevent
    - 'quick tip'
    - radmessagebox
    - telerik
    - terminate
    - tip
    - windev
    - wpdev
---

![preventExit](/assets/img/2014/09/preventExit.png "preventExit")

After finally being able to use my dev center account again after three long weeks, I am back into [UniShare](https://www.windowsphone.com/s?appid=ee42cb1d-8a68-41c6-9c0c-d3e3fc61d6ea) development. One of the most requested fixes was to not exit the app when users are not at the compose pivot.

Of course I am listening and I found a solution that should fit nearly all possible scenarios. Here is what I did:

``` csharp
         //handle back key press to prevent accidentally exit of the app
        protected async override void OnBackKeyPress(CancelEventArgs e)
        {
            //get the base handler
            base.OnBackKeyPress(e);

            //go back to the first pivot on all other pivots
            if (MainPivot.SelectedIndex != 0)
            {
                MainPivot.SelectedIndex = 0;
                e.Cancel = true;
            }
            //handle app exit
            else if (MainPivot.SelectedIndex == 0)
            {
                //cancel the back button press
                e.Cancel = true;

                //check if bool isExitQuestionCheckBoxChecked is false and the message needs to be displayed
                if (isExitQuestionCheckBoxChecked == false)
                {
                    //show the message box
                    MessageBoxClosedEventArgs args = await RadMessageBox.ShowAsync("Do you really want to exit the app?", MessageBoxButtons.YesNo, null, "don't ask me again, just exit next time", false, true);
                    if (args.ClickedButton == null)
                    {
                        //save check box value (use IsolatedStorage on earlier versions of Windows Phone)
                        App.SettingsStore.isExitQuestionCheckBoxChecked = isExitQuestionCheckBoxChecked;
                        //go back to the app
                        return;
                        
                    }
                    if (args.ButtonIndex == 0)
                    {
                        //set the check box value
                        isExitQuestionCheckBoxChecked = args.IsCheckBoxChecked;
                        //save check box value (use IsolatedStorage on earlier versions of Windows Phone)
                        App.SettingsStore.isExitQuestionCheckBoxChecked = isExitQuestionCheckBoxChecked;
                        //exit the app
                        Application.Current.Terminate();
                        
                    }
                    if (args.ButtonIndex == 1)
                    {
                        //set the check box value
                        isExitQuestionCheckBoxChecked = args.IsCheckBoxChecked;
                        //save check box value (use IsolatedStorage on earlier versions of Windows Phone)
                        App.SettingsStore.isExitQuestionCheckBoxChecked = isExitQuestionCheckBoxChecked;
                        //go back to the app
                        return;
                    }
                }
                //check if bool isExitQuestionCheckBoxChecked is false and the app should be exited
                else if (isExitQuestionCheckBoxChecked == true)
                {
                    Application.Current.Terminate();
                }
            }
        }
```
 
Let me explain. The first thing I am checking is if the current PivotItem is the one I want to display the message. If not, I am moving the Pivot to it.

The next step is to show the MessageBox and handle the buttons and the CheckBox. If no button is pressed, the codes saves the value false and goes back to the app.

If the Yes-button is pressed, the code saved the value of the CheckBox and exits. Same happens for the No-button.

If the user presses the back button the next time, the isExitQuestionCheckBoxChecked boolean gets checked – if the user does not want the message and checked it, the app exits as expected.

The above snippet uses the RadMessageBox.

When we use Toolkit’s CustomMessageBox, we need a slightly different approach. First, we do not override the OnBackKeyPress event – instead, we declare a new BackKeyPress event in the page’s constructor:

``` csharp
 public MainPage()
{
    InitializeComponent();     
    BackKeyPress += MainPage_BackKeyPress;
}
```
 
Then, add this code to the newly generated event handler:

``` csharp
         //handle back key press to prevent accidentally exit of the app
        void MainPage_BackKeyPress(object sender, System.ComponentModel.CancelEventArgs e)
        {
            //go back to the first pivot on all other pivots
            if (MainPivot.SelectedIndex != 0)
            {
                MainPivot.SelectedIndex = 0;
                e.Cancel = true;
            }
            //handle app exit
            else if (MainPivot.SelectedIndex == 0)
            {   
                ////check if bool isExitQuestionCheckBoxChecked is false and the message needs to be displayed
                if (isExitQuestionCheckBoxChecked == false)
                {
                //generate a CheckBox as Content
                CheckBox chkbox = new CheckBox()
                {
                    Content = "don't ask me again, just exit next time",
                    Margin = new Thickness(0, 14, 0, -2)
                };

                //generate msg and handle the result
                CustomMessageBox msg = new CustomMessageBox()
                {
                    Title = "Attention",
                    Message = "Do you really want to exit the app?",
                    Content = chkbox,
                    LeftButtonContent = "yes",
                    RightButtonContent = "no"
                    
                };

                msg.Dismissed += (s1, e1) =>
                    {
                        switch (e1.Result)
                        {
                            case CustomMessageBoxResult.LeftButton:
                                //save check box value
                                isExitQuestionCheckBoxChecked = (bool)chkbox.IsChecked;

                                //save the check box value
                                if (IsolatedStorageSettings.ApplicationSettings.Contains("isExitQuestionCheckBoxChecked"))
                                {
                                    IsolatedStorageSettings.ApplicationSettings.Remove("isExitQuestionCheckBoxChecked");
                                }
                                IsolatedStorageSettings.ApplicationSettings.Add("isExitQuestionCheckBoxChecked", isExitQuestionCheckBoxChecked);

                                IsolatedStorageSettings.ApplicationSettings.Save();

                                //exit the app
                                Application.Current.Terminate();
                                break;
                            case CustomMessageBoxResult.RightButton:
                                //save check box value
                                isExitQuestionCheckBoxChecked = (bool)chkbox.IsChecked;

                                //save the check box value
                                if (IsolatedStorageSettings.ApplicationSettings.Contains("isExitQuestionCheckBoxChecked"))
                                {
                                    IsolatedStorageSettings.ApplicationSettings.Remove("isExitQuestionCheckBoxChecked");
                                }
                                IsolatedStorageSettings.ApplicationSettings.Add("isExitQuestionCheckBoxChecked", isExitQuestionCheckBoxChecked);

                                IsolatedStorageSettings.ApplicationSettings.Save();

                                //go back to the app
                                break;
                            case CustomMessageBoxResult.None:
                                break;
                            default:
                                break;
                        }
                    };
                    //show message
                    msg.Show();

                    //cancel all BackKey events
                    e.Cancel = true;

                }
                //check if bool isExitQuestionCheckBoxChecked is false and the app should be exited
                else if (isExitQuestionCheckBoxChecked == true)
                {
                    Application.Current.Terminate();
                }
            }
        }
```
 
This code is taken from a Windows Phone 8 project and does the same as the first snippet by using the Windows Phone Toolkit.

As always, I hope this is helpful for some of you.

Happy coding, everyone!