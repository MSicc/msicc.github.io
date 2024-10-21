---
id: 3974
title: 'How to build a custom Application Bar for your Windows Phone app (the easy way)'
date: '2014-02-20T04:44:52+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-build-a-custom-application-bar-for-your-windows-phone-app-the-easy-way/
categories:
    - Archive
tags:
    - 'application bar'
    - control
    - 'custom app bar'
    - customized
    - Events
    - RadImageButton
    - telerik
    - WinPhanDev
    - wpdev
---

![customappbarexpanded](/assets/img/2014/02/customappbarexpanded.png "customappbarexpanded")

In one of my recent projects, I was forced to use icons for the ApplicationBarButtons that didn’t fit into the circled template of the standard Windows Phone application bar.

The icons have special circles themselves, and I am not allowed to change anything on that icons (you’re right, it is for the corporate app I am working on). That’s why I needed to find another solution – and I started to write my own “ApplicationBar”.

As I am using Telerik’s Windows Phone Controls, I knew that the RadImageButton have exactly the same behavior than the buttons in the standard ApplicationBar. That point was already save, the only thing I needed to change was the ButtonShape of the RadImageButton from Rectangle to Ellipse – done.

This is the UserControl I created to achieve my goal:

``` xml
 <Border x:Name="customAppBarBorder"  Height="72" VerticalAlignment="Bottom">
    <Grid >
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="80"/>
            <ColumnDefinition Width="80"/>
            <ColumnDefinition Width="80"/>
            <ColumnDefinition Width="80"/>
            <ColumnDefinition Width="80"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"></RowDefinition>
            <RowDefinition Height="Auto"></RowDefinition>
        </Grid.RowDefinitions>

        <telerikPrimitives:RadImageButton x:Name="CustomAppBarRadImageButton1" Grid.Row="0" Grid.Column="1" HorizontalAlignment="Center" ButtonShape="Ellipse" RestStateImageSource="null" ></telerikPrimitives:RadImageButton>
        <telerikPrimitives:RadImageButton x:Name="CustomAppBarRadImageButton2" Grid.Row="0" Grid.Column="2" HorizontalAlignment="Center" RestStateImageSource="null" ButtonShape="Ellipse" ></telerikPrimitives:RadImageButton>
        <telerikPrimitives:RadImageButton x:Name="CustomAppBarRadImageButton3" Grid.Row="0" Grid.Column="3" HorizontalAlignment="Center" ButtonShape="Ellipse" RestStateImageSource="null"></telerikPrimitives:RadImageButton>
        <telerikPrimitives:RadImageButton x:Name="CustomAppBarRadImageButton4" Grid.Row="0" Grid.Column="4" HorizontalAlignment="Center" ButtonShape="Ellipse" RestStateImageSource="null"></telerikPrimitives:RadImageButton>

        <Image x:Name="overflowDots" Grid.Row="0" Grid.Column="5" Width="72" Source="/Assets/AppBar/overflowdots.png" VerticalAlignment="Top" HorizontalAlignment="Right" Tap="overflowDots_Tap"></Image>

        <TextBlock x:Name="CustomAppBarButtonItem1Text" Grid.Row="1" Grid.Column="1" Width="72" FontSize="14" VerticalAlignment="Center" HorizontalAlignment="Center" TextAlignment="Center" Margin="0,-4,0,0" />
        <TextBlock x:Name="CustomAppBarButtonItem2Text" Grid.Row="1" Grid.Column="2" Width="72" FontSize="14" VerticalAlignment="Center" HorizontalAlignment="Center" TextAlignment="Center" Margin="0,-4,0,0" />
        <TextBlock x:Name="CustomAppBarButtonItem3Text" Grid.Row="1" Grid.Column="3" Width="72" FontSize="14" VerticalAlignment="Center" HorizontalAlignment="Center" TextAlignment="Center" Margin="0,-4,0,0" />
        <TextBlock x:Name="CustomAppBarButtonItem4Text" Grid.Row="1" Grid.Column="4" Width="72" FontSize="14" VerticalAlignment="Center" HorizontalAlignment="Center" TextAlignment="Center" Margin="0,-4,0,0" />
    </Grid>
</Border>
```
 
It was a bit tricky to get all the size to get a similar appearance, but it does look like the original one.

The RadImageButton Control has also a Text that we can enter, but the text was to small or to close to the button, no matter what I did. That’s why there is another row in my Grid with the corresponding text.

You may have recognized the image with the Source “overflowdots.png” above. These are located in the Windows Phone icon folder (in Microsoft SDKs under program files). We need this icon to generate the transition the standard ApplicationBar has. It is done with two simple StoryBoards:

``` xml
 <UserControl.Resources>
    <Storyboard x:Name="FadeCustomAppBarButtonTextIn">
        <DoubleAnimation Storyboard.TargetName="customAppBarBorder"
                         Storyboard.TargetProperty="Height"
                         From="72" To="102" Duration="0:0:0.2"/>
    </Storyboard>

    <Storyboard x:Name="FadeCustomAppBarButtonTextOut">
        <DoubleAnimation Storyboard.TargetName="customAppBarBorder"
                         Storyboard.TargetProperty="Height"
                         From="102" To="72" Duration="0:0:0.2"/>
    </Storyboard>
</UserControl.Resources>
```
 
All we need now is a proper EventHandler – the TapEvent of the image inside the control is perfect for that:

``` csharp
 private void overflowDots_Tap(object sender, System.Windows.Input.GestureEventArgs e)
{
    if (customAppBarBorder.ActualHeight == 72)
    {
        FadeCustomAppBarButtonTextIn.Begin();
    }
    else if (customAppBarBorder.ActualHeight == 102)
    {
        FadeCustomAppBarButtonTextOut.Begin();
    }
}
```
 
I am using a Border for the animation because with a Grid it was not as fluent as I wanted. That’s the whole code of the control I created. Let’s have a look at the implementation:

First thing is an additional row in our LayoutRoot Grid, where we can add our custom app bar control to (set the Height to “Auto”). Add this code to add the app bar:

``` csharp
 //declare the control:

public CustomAppBarWP8 customappbar;

//add your data to the app bar:

customappbar = new CustomAppBarWP8();

customappbar.CustomAppBarBackground = new SolidColorBrush(Colors.Green);

customappbar.CustomAppBarButtonItem1Text.Text = "test 1";
customappbar.CustomAppBarButtonItem2Text.Text = "test 2";
customappbar.CustomAppBarButtonItem3Text.Text = "test 3";
customappbar.CustomAppBarButtonItem4Text.Text = "test 4";

customappbar.CustomAppBarRadImageButton1.RestStateImageSource = new BitmapImage(new Uri("Assets/AppBar/microphone.png", UriKind.RelativeOrAbsolute));
customappbar.CustomAppBarRadImageButton2.RestStateImageSource = new BitmapImage(new Uri("Assets/AppBar/save.png", UriKind.RelativeOrAbsolute));
customappbar.CustomAppBarRadImageButton3.RestStateImageSource = new BitmapImage(new Uri("Assets/AppBar/delete.png", UriKind.RelativeOrAbsolute));
customappbar.CustomAppBarRadImageButton4.RestStateImageSource = new BitmapImage(new Uri("Assets/AppBar/questionmark.png", UriKind.RelativeOrAbsolute));

//registering the tap events:

customappbar.CustomAppBarRadImageButton1.Tap += CustomAppBarRadImageButton1_Tap;
customappbar.CustomAppBarRadImageButton2.Tap += CustomAppBarRadImageButton2_Tap;
customappbar.CustomAppBarRadImageButton3.Tap += CustomAppBarRadImageButton3_Tap;
customappbar.CustomAppBarRadImageButton4.Tap += CustomAppBarRadImageButton4_Tap;

//adding the app bar to the dedicated Grid:
AppBarGrid.Children.Add(customappbar);
```
 
The standard Application Bar does fading out the text on a button tap, so we need to add this line in every tap event. Otherwise, it would remain open all the time.

``` csharp
 if (customappbar.ActualHeight == 102)
{
    customappbar.FadeCustomAppBarButtonTextOut.Begin();
}
```
 
To get the same result on tapping outside our custom app bar, add the same code to your main Grid’s MouseLeftButtonDown event. This way, you have the same behavior like in the original control.

> Additional note: I needed to find a quick way to achieve this, that’s why I may have not been using best practices. I also used the RadImageButton Control to speed things up. I will refine this control when I have more time available for it, as well as add a version without Telerik controls and adding the menu items.
{: .prompt-warning }

If you have any idea on how to improve this, feel free to left a comment below.

Anyways, you can download the source of the code above here: [https://github.com/MSiccDev/CustomAppBar\_WP8](https://github.com/MSiccDev/CustomAppBar_WP8 "https://github.com/MSiccDev/CustomAppBar_WP8")

As always, I hope this will be helpful for some of you.