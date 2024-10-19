---
id: 4459
title: '[Updated] How to create a pure icon button for your UWP app'
date: '2016-08-10T06:12:02+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-create-a-pure-icon-button-for-your-uwp-app/
categories:
    - 'Dev Stories'
    - Windows
tags:
    - Button
    - icon
    - style
    - 'user control'
    - uwp
    - wpdev
    - XAML
---

\[Updated: the first version of this blog post was using a UserControl. Thanks to a discussion on Twitter, I realized that wrapping it into a UserControl is overblown (yes, sometimes I tend to write more code than necessary). This is the updated version using only a Style for the button.\]

When writing apps, you often come along a point where you want to style controls differently than the original style. Today, I‘ll show you a pure icon button that does not show the surrounding shape and borders. Instead, it highlights the icon of the button when hovering over it, while it changes the color to the user’s accent color when it gets pressed. Here is a little animation of what I am talking about:![cromeless button demo](/assets/img/2016/08/cromeless-button-demo-1.gif "cromeless button demo")

It all begins with the default style of the Button control, which you can see [here](https://msdn.microsoft.com/en-us/library/windows/apps/mt299109.aspx). We are going to modify this Style until it matches the above animation. The first thing we change is the BackgroundBrush – set it to ‘Transparent’ to get rid of the grey shape that the button control comes with when hovering over it or pressing it:

``` xml
 <Setter Property="Background" Value="Transparent"/>
```
 
As we want an icon button, we need to choose a common source for the icons as well. I am using a Path Shape as the icon source, as it allows modifications to be done in XAML. So the next step is to add a [Path](https://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.shapes.path.aspx) shape to the Style:

``` xml
                             <Path x:Name="PathIcon"
                                  Data="{TemplateBinding Content}"
                                  Stretch="Uniform"
                                  Fill="{TemplateBinding Foreground}"
                                  Stroke="Transparent"
                                  StrokeThickness="1"
                                  Margin="4"
                                  RenderTransformOrigin="0.5,0.5">
                                <Path.RenderTransform>
                                    <TransformGroup>
                                        <TransformGroup.Children>
                                            <RotateTransform Angle="0" />
                                            <ScaleTransform ScaleX="1" ScaleY="1" />
                                        </TransformGroup.Children>
                                    </TransformGroup>
                                </Path.RenderTransform>
                            </Path>
```
 
In this case, as we just want to use the icon within our button, we can safely remove the ‘ContentPresenter’ part in the Style. We have made quite some progress already, but that all does not make the control behaving like in the animation yet.

Now it is the time to modify the CommonStates of the Button’s style. Our Button uses only an icon, so we need to add the color states for the Path’s ‘Fill (=Foreground)’ to the states. Here are the modifications:

‘PointerOver’ state:

``` xml
                                             <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Fill" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource SystemControlHighlightBaseHighBrush}"/>
                                            </ObjectAnimationUsingKeyFrames>
```
 
‘Pressed’ state:

``` xml
                                             <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Fill" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource SystemControlForegroundAccentBrush}"/>
                                            </ObjectAnimationUsingKeyFrames>
```
 
‘Disabled’ state:

``` xml
                                             <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Fill" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource SystemControlDisabledBaseMediumLowBrush}"/>
                                            </ObjectAnimationUsingKeyFrames>
```
 
To get the icon’s outline highlighting, we are going to use the Path’s ‘Stroke (=Border)’ property. Add these modifications to the Style in XAML:

‘PointerOver’ state:

``` xml
                                             <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Stroke" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource SystemControlHighlightAccentBrush}"/>
                                            </ObjectAnimationUsingKeyFrames>
```
 
‘Pressed’ state:

``` xml
                                             <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Stroke" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="Transparent"/>
                                            </ObjectAnimationUsingKeyFrames>
```
 
‘Disabled’ state:

``` xml
                                             <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Stroke" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="Transparent"/>
                                            </ObjectAnimationUsingKeyFrames>
```
 
All that is left is to use the Style on any desired button:

``` xml
     <Button x:Name="BaseButton" Style="{StaticResource TransparentButtonStyle}"></Button>
```
 
If you now use this one in an application, you will get the same result as in the initial animation.

For easier use, here is the complete code:

``` xml
         <Style x:Key="TransparentPathIconButtonStyle" TargetType="Button">
            <Setter Property="Background" Value="Transparent"/>
            <Setter Property="Foreground" Value="{ThemeResource SystemControlForegroundBaseHighBrush}"/>
            <Setter Property="BorderBrush" Value="{ThemeResource SystemControlForegroundTransparentBrush}"/>
            <Setter Property="BorderThickness" Value="{ThemeResource ButtonBorderThemeThickness}"/>
            <Setter Property="Padding" Value="8,4,8,4"/>
            <Setter Property="HorizontalAlignment" Value="Stretch"/>
            <Setter Property="VerticalAlignment" Value="Stretch"/>
            <Setter Property="FontFamily" Value="{ThemeResource ContentControlThemeFontFamily}"/>
            <Setter Property="FontWeight" Value="Normal"/>
            <Setter Property="FontSize" Value="{ThemeResource ControlContentThemeFontSize}"/>
            <Setter Property="UseSystemFocusVisuals" Value="True"/>

            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="Button">
                        <Grid x:Name="RootGrid" Background="{TemplateBinding Background}">
                            <VisualStateManager.VisualStateGroups>
                                <VisualStateGroup x:Name="CommonStates">
                                    <VisualState x:Name="Normal">
                                        <Storyboard>
                                            <PointerUpThemeAnimation Storyboard.TargetName="RootGrid"/>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="PointerOver">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Stroke" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource SystemControlHighlightAccentBrush}"/>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Fill" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource SystemControlHighlightBaseHighBrush}"/>
                                            </ObjectAnimationUsingKeyFrames>
                                            <PointerUpThemeAnimation Storyboard.TargetName="RootGrid"/>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="Pressed">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Fill" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource SystemControlForegroundAccentBrush}"/>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Stroke" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="Transparent"/>
                                            </ObjectAnimationUsingKeyFrames>
                                            <PointerDownThemeAnimation Storyboard.TargetName="RootGrid"/>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="Disabled">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Fill" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource SystemControlDisabledBaseMediumLowBrush}"/>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Stroke" Storyboard.TargetName="PathIcon">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="Transparent"/>
                                            </ObjectAnimationUsingKeyFrames>
                                        </Storyboard>
                                    </VisualState>
                                </VisualStateGroup>
                            </VisualStateManager.VisualStateGroups>

                            <Path x:Name="PathIcon"
                                  Data="{TemplateBinding Content}"
                                  Stretch="Uniform"
                                  Fill="{TemplateBinding Foreground}"
                                  Stroke="Transparent"
                                  StrokeThickness="1"
                                  Margin="4"
                                  RenderTransformOrigin="0.5,0.5">
                                <Path.RenderTransform>
                                    <TransformGroup>
                                        <TransformGroup.Children>
                                            <RotateTransform Angle="0" />
                                            <ScaleTransform ScaleX="1" ScaleY="1" />
                                        </TransformGroup.Children>
                                    </TransformGroup>
                                </Path.RenderTransform>
                            </Path>
                        </Grid>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
```
 
As always, I hope this post is helpful for some of you. If you have questions/ideas for improvements or just want to talk about the control, feel free to leave a comment below.

Happy Coding, everyone!