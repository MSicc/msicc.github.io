---
id: 6535
title: '#XfEffects: Forms Effect to automatically scale FontSize on Label'
date: '2020-01-17T22:00:00+01:00'
author: 'Marco Siccardi'
excerpt: 'In this second post of my #XfEffects series, I am going to show you how to make the text of a Label grow and shrink within a defined range, based on the Xamarin.Forms.NamedSize enumeration.'
layout: post
permalink: /xfeffects-forms-effect-to-automatically-scale-fontsize-on-label/
image: /assets/img/2020/01/autofitfontsize_effect_title.jpg
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Xamarin
tags:
    - Android
    - autoscale
    - effects
    - font
    - fontsize
    - iOS
    - label
    - maxlines
    - text
    - xamarin
    - 'xamarin forms'
    - XfEffects
---

### Why do I need this?

When working with text, we often have to deal with some or all of the following:

- dynamic text with different length on every instance
- multiple devices with different screen resolutions
- limited number of lines

As the amount of places where I need to automatically scale the FontSize is steadily increasing within my apps, I had to come up with a solution – the AutoFitFontSizeEffect.

### The shared code

Of course, every `Effect` has a shared code part. Like in my first post, there are two classes for this – the `Effect` wrapper and the static parameter class on top of it. The wrapper is pretty straight forward:

``` csharp
     public class AutoFitFontSizeEffect : RoutingEffect
    {
        #region Protected Constructors

        public AutoFitFontSizeEffect() : base($"XfEffects.{nameof(AutoFitFontSizeEffect)}")
        {
        }

        #endregion Protected Constructors
    }
```
 
Like with all effects, we are deriving from `RoutingEffect` and initializing the class with the effect’s id. As I wanted the effect to be configurable with a minimum and a maximum size, I added a static class that takes these two parameters:

``` csharp
     public static class AutoFitFontSizeEffectParameters
    {
        #region Public Fields

        public static BindableProperty MaxFontSizeProperty = BindableProperty.CreateAttached("MaxFontSize", typeof(NamedSize), typeof(AutoFitFontSizeEffectParameters), NamedSize.Large, BindingMode.Default);
        public static BindableProperty MinFontSizeProperty = BindableProperty.CreateAttached("MinFontSize", typeof(NamedSize), typeof(AutoFitFontSizeEffectParameters), NamedSize.Default, BindingMode.Default);

        #endregion Public Fields

        #region Public Methods

        public static NamedSize GetMaxFontSize(BindableObject bindable)
        {
            return (NamedSize)bindable.GetValue(MaxFontSizeProperty);
        }

        public static NamedSize GetMinFontSize(BindableObject bindable)
        {
            return (NamedSize)bindable.GetValue(MinFontSizeProperty);
        }

        public static double MaxFontSizeNumeric(BindableObject bindable)
        {
            return Device.GetNamedSize(GetMaxFontSize(bindable), typeof(Label));
        }

        public static double MinFontSizeNumeric(BindableObject bindable)
        {
            return Device.GetNamedSize(GetMinFontSize(bindable), typeof(Label));
        }

        public static void SetMaxFontSize(BindableObject bindable, NamedSize value)
        {
            bindable.SetValue(MaxFontSizeProperty, value);
        }

        public static void SetMinFontSize(BindableObject bindable, NamedSize value)
        {
            bindable.SetValue(MinFontSizeProperty, value);
        }

        #endregion Public Methods
    }
```
 
Let’s break that class down. First, I created two attached `BindableProperty` objects of type `NamedSize`. The `NamedSize` enumeration makes it easy for us to determine the minimum and maximum sizes. If you want to know the values behind the enum entries, [check this table in the docs](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/text/fonts#named-font-sizes).

To get and set those values out of the `BindableProperty`, I implemented corresponding methods. As we will see later on, I implemented also two methods that get the numeric values, which will be used in our platform-specific implementations.

### Android implementation

Android has a built-in method on `TextView` to achieve the auto-scaling functionality we desire ([read more about it on the Android docs](https://developer.android.com/guide/topics/ui/look-and-feel/autosizing-textview)). This makes the implementation pretty straight forward:

``` csharp
     public class AutoFitFontSizeEffect : PlatformEffect
    {
        #region Protected Methods

        protected override void OnAttached()
        {
            if (this.Control is TextView textView)
            {
                if (AutoFitFontSizeEffectParameters.GetMinFontSize(this.Element) == NamedSize.Default &&
                    AutoFitFontSizeEffectParameters.GetMaxFontSize(this.Element) == NamedSize.Default)
                    return;

                var min = (int)AutoFitFontSizeEffectParameters.MinFontSizeNumeric(this.Element);
                var max = (int)AutoFitFontSizeEffectParameters.MaxFontSizeNumeric(this.Element);

                if (max <= min)
                    return;

                textView.SetAutoSizeTextTypeUniformWithConfiguration(min, max, 1, (int)ComplexUnitType.Sp);
            }
        }

        protected override void OnDetached()
        {
        }

        #endregion Protected Methods
    }
```
 
Before using the `SetAutoSizeTextTypeUniformWithConfiguration` method on the `TextView`, I am running two checks: one if both parameters are set to `NamedSize.Default`, and the other one if the minimum value is bigger than the maximum value. If we pass past these check, we are using the above mentioned method. That is already everything it needs to make the text scaling automatically within the bounds of the `TextView` on Android.

### iOS implementation

Like Android, also iOS has a pretty easy way to automatically scale the FontSize:

``` csharp
     public class AutoFitFontSizeEffect : PlatformEffect
    {
        #region Protected Methods

        protected override void OnAttached()
        {
            if (this.Control is UILabel label)
            {
                if (AutoFitFontSizeEffectParameters.GetMinFontSize(this.Element) == NamedSize.Default &&
                    AutoFitFontSizeEffectParameters.GetMaxFontSize(this.Element) == NamedSize.Default)
                    return;

                var min = (int)AutoFitFontSizeEffectParameters.MinFontSizeNumeric(this.Element);
                var max = (int)AutoFitFontSizeEffectParameters.MaxFontSizeNumeric(this.Element);

                if (max <= min)
                    return;

                label.AdjustsFontSizeToFitWidth = true;
                label.MinimumFontSize = (float)min;
                label.Font = label.Font.WithSize((float)max);
            }
        }

        protected override void OnDetached()
        {
        }

        #endregion Protected Methods
    }
```
 
We are running the same checks as on Android before we are effectively changing the properties on the UILabel that will make the text scale automatically. With setting AdjustsFontSizeToFitWidth to true and setting the MinimumFontSize to our min value as well as the max value as FontSize, we have already done everything it needs on iOS.

### Conclusion

The checks we run before using the codes are not random. It may happen that you only add the effect to your Xamarin.Forms.Label without setting the MinFontSize and MaxFontSize. In this case, I am just returning.

Besides mixing up the sizes, the main reason for the second check is that the platform-specific size values are different between platforms. Also in this case, I am just returning.

Besides that, we are able to use all other properties of the default `Xamarin.Forms.Label` implementation, with `MaxLines` and `LineBreakMode` being the two most important ones.

As always, I hope this post will be helpful for some of you. Of course, the sample project for this series is updated on [GitHub](https://github.com/MSiccDev/XfEffects).

#### Happy coding, everyone! 