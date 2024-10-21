---
id: 6950
title: 'Use the iOS system colors in Xamarin.Forms (Updated)'
date: '2021-12-11T08:32:35+01:00'
author: 'Marco Siccardi'
excerpt: 'Apple has defined some best practices for coloring our iOS apps. To make it easier, Apple created system colors that are (partly) adaptive to the system''s coloring. In this post, I show you how to use these colors in Xamarin.Forms.'
layout: post
permalink: /use-the-ios-system-colors-in-xamarin-forms/
image: /assets/img/2021/12/iosSystemColorsInXFTitle.jpg
categories:
    - 'Dev Stories'
    - iOS
    - Xamarin
tags:
    - color
    - colors
    - iOS
    - ResourceDictionary
    - resources
    - system
    - 'system colors'
    - xamarin
    - 'xamarin forms'
    - XAML
---

### Update

After publishing this post, [Gerald Versluis](https://twitter.com/jfversluis) from Microsoft responded on Twitter with an interesting information on how to get the system colors into our `ResourceDictionary` without using the `DependencyService`:

<figure class="wp-block-embed aligncenter is-type-rich is-provider-twitter wp-block-embed-twitter"><div class="wp-block-embed__wrapper">> Cool post! You shouldn’t have to implement all the code to reach those iOS colors though. You can use Device.GetNamedColor() and use the identifiers here: <https://t.co/TACV0c7mk8>
> 
> — Gerald Versluis (@jfversluis) [December 11, 2021](https://twitter.com/jfversluis/status/1469657966479560710?ref_src=twsrc%5Etfw)

<script async="" charset="utf-8" src="https://platform.twitter.com/widgets.js"></script></div></figure>I had a quick look at the `NamedPlatformColor` class, but noticed that the implementation in `Xamarin.Forms` is incomplete. Gerald will try to update them. Once that is done, I will update the library [on Github](https://github.com/MSiccDev/SystemColorsiOS) and this post again.

## *Original version below:*

---

### Overview

Let me give you a short overview first. To achieve our goal to use the iOS system colors, we need just a few easy steps:

1. `Xamarin.Forms` interface that defines the colors
2. `Xamarin.iOS` implementation of that interface
3. `ResourceDictionary` to make the colors available in XAML
4. Merging this dictionary with the application’s resource
5. Handling of the `OnRequestedThemeChanged` event

Now that the plan is clear, let’s go into details.

### `ISystemColors` interface

We will use the `Xamarin.Forms DependencyService` to get the colors from iOS to `Xamarin.Forms`. Let’s create our common interface:

``` csharp
 using Xamarin.Forms;

namespace [YOURNAMESPACEHERE]
{
    public interface ISystemColors
    {
        Color SystemRed { get; }
        Color SystemOrange { get; }
        Color SystemYellow { get; }
        Color SystemGreen { get; }
        Color SystemMint { get; }
        Color SystemTeal { get; }
        Color SystemCyan { get; }
        Color SystemBlue { get; }
        Color SystemIndigo { get; }
        Color SystemPurple { get; }
        Color SystemPink { get; }
        Color SystemBrown { get; }
        Color SystemGray { get; }
        Color SystemGray2 { get; }
        Color SystemGray3 { get; }
        Color SystemGray4 { get; }
        Color SystemGray5 { get; }
        Color SystemGray6 { get; }
        Color SystemLabel { get; }
        Color SecondaryLabel { get; }
        Color TertiaryLabel { get; }
        Color QuaternaryLabel { get; }
        Color Placeholder { get; }
        Color Separator { get; }
        Color OpaqueSeparator { get; }
        Color LinkColor { get; }
        Color FillColor { get; }
        Color SecondaryFillColor { get; }
        Color TertiaryFillColor { get; }
        Color QuaternaryFillColor { get; }
        Color SystemBackgroundColor { get; }
        Color SecondarySystemBackgroundColor { get; }
        Color TertiarySystemBackgroundColor { get; }
        Color SystemGroupedBackgroundColor { get; }
        Color SecondarySystemGroupedBackgroundColor { get; }
        Color TertiarySystemGroupedBackgroundColor { get; }
        Color DarkTextColor { get; }
        Color LightTextColor { get; }
    }
}
```
 
As we are not able to change any of the system colors, we are just defining getters in the interface.

### The `Xamarin.iOS` platform implementation

The implementation is straight forward. We are implementing the interface and just get the values for each system color. The list is based on Apple’s documentation for [human interface](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/color/) and [UI element colors](https://developer.apple.com/documentation/uikit/uicolor/ui_element_colors).

``` csharp
 using [YOURNAMESPACEHERE];

using UIKit;

using Xamarin.Forms;
using Xamarin.Forms.Platform.iOS;

[assembly: Dependency(typeof(SystemColors))]
namespace [YOURNAMESPACEHERE]
{
    //https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/color/
    //https://developer.apple.com/documentation/uikit/uicolor/ui_element_colors

    public class SystemColors : ISystemColors
    {
        #region System Colors
        public Color SystemRed => UIColor.SystemRedColor.ToColor();
        public Color SystemOrange => UIColor.SystemOrangeColor.ToColor();
        public Color SystemYellow => UIColor.SystemYellowColor.ToColor();
        public Color SystemGreen => UIColor.SystemGreenColor.ToColor();
        public Color SystemMint => UIColor.SystemMintColor.ToColor();
        public Color SystemTeal => UIColor.SystemTealColor.ToColor();
        public Color SystemCyan => UIColor.SystemCyanColor.ToColor();
        public Color SystemBlue => UIColor.SystemBlueColor.ToColor();
        public Color SystemIndigo => UIColor.SystemIndigoColor.ToColor();
        public Color SystemPurple => UIColor.SystemPurpleColor.ToColor();
        public Color SystemPink => UIColor.SystemPinkColor.ToColor();
        public Color SystemBrown => UIColor.SystemBrownColor.ToColor();


        public Color SystemGray => UIColor.SystemGrayColor.ToColor();
        public Color SystemGray2 => UIColor.SystemGray2Color.ToColor();
        public Color SystemGray3 => UIColor.SystemGray3Color.ToColor();
        public Color SystemGray4 => UIColor.SystemGray4Color.ToColor();
        public Color SystemGray5 => UIColor.SystemGray5Color.ToColor();
        public Color SystemGray6 => UIColor.SystemGray6Color.ToColor();
        #endregion

        #region UI Element Colors
        public Color SystemLabel => UIColor.LabelColor.ToColor();
        public Color SecondaryLabel => UIColor.SecondaryLabelColor.ToColor();
        public Color TertiaryLabel => UIColor.TertiaryLabelColor.ToColor();
        public Color QuaternaryLabel => UIColor.QuaternaryLabelColor.ToColor();
        public Color Placeholder => UIColor.PlaceholderTextColor.ToColor();
        public Color Separator => UIColor.SeparatorColor.ToColor();
        public Color OpaqueSeparator => UIColor.SeparatorColor.ToColor();
        public Color LinkColor => UIColor.SeparatorColor.ToColor();

        public Color FillColor => UIColor.SystemFillColor.ToColor();
        public Color SecondaryFillColor => UIColor.SecondarySystemFillColor.ToColor();
        public Color TertiaryFillColor => UIColor.TertiarySystemFillColor.ToColor();
        public Color QuaternaryFillColor => UIColor.QuaternarySystemFillColor.ToColor();

        public Color SystemBackgroundColor => UIColor.SystemBackgroundColor.ToColor();
        public Color SecondarySystemBackgroundColor => UIColor.SecondarySystemBackgroundColor.ToColor();
        public Color TertiarySystemBackgroundColor => UIColor.TertiarySystemBackgroundColor.ToColor();

        public Color SystemGroupedBackgroundColor => UIColor.SystemGroupedBackgroundColor.ToColor();
        public Color SecondarySystemGroupedBackgroundColor => UIColor.SecondarySystemGroupedBackgroundColor.ToColor();
        public Color TertiarySystemGroupedBackgroundColor => UIColor.TertiarySystemGroupedBackgroundColor.ToColor();

        public Color DarkTextColor => UIColor.DarkTextColor.ToColor();
        public Color LightTextColor => UIColor.LightTextColor.ToColor();

        #endregion
    }
}
```
 
 Do not forget to add the `Dependency` attribute on top of the implementation, otherwise it won’t work.

### The `ResourceDictionary`

As I prefer defining my UI in XAML in `Xamarin.Forms,` I naturally want those colors to be available there as well. This can be done by loading the colors into a `ResourceDictionary`. [As you might remember]({% post_url 2021-11-30-xfqad-compile-xaml-without-code-behind-in-xamarin-forms  %}), I prefer codeless `ResourceDictionary` implementations. This time, however, we need the code-behind file to make the `ResourceDictionary` work for us.

First, add a new `ResourceDictionary`:

![Add_ResourceDictionary_XAML](/assets/img/2021/11/Add_ResourceDictionary_XAML.png)
Then, in the code-behind file, we are using the `DependencyService` of `Xamarin.Forms` to add the colors to the `ResourceDictionary`:

``` csharp
 using Xamarin.Forms;
using Xamarin.Forms.Xaml;

[assembly: XamlCompilation(XamlCompilationOptions.Compile)]
namespace [YOURNAMESPACEHERE]
{
    public partial class SystemColorsIosResourceDictionary
    {
        public SystemColorsIosResourceDictionary()
        {
            InitializeComponent();

            this.Add(nameof(ISystemColors.SystemRed), DependencyService.Get<ISystemColors>().SystemRed);
            this.Add(nameof(ISystemColors.SystemOrange), DependencyService.Get<ISystemColors>().SystemOrange);
            this.Add(nameof(ISystemColors.SystemYellow), DependencyService.Get<ISystemColors>().SystemYellow);
            this.Add(nameof(ISystemColors.SystemGreen), DependencyService.Get<ISystemColors>().SystemGreen);
            this.Add(nameof(ISystemColors.SystemMint), DependencyService.Get<ISystemColors>().SystemMint);
            this.Add(nameof(ISystemColors.SystemTeal), DependencyService.Get<ISystemColors>().SystemTeal);
            this.Add(nameof(ISystemColors.SystemCyan), DependencyService.Get<ISystemColors>().SystemCyan);
            this.Add(nameof(ISystemColors.SystemBlue), DependencyService.Get<ISystemColors>().SystemBlue);
            this.Add(nameof(ISystemColors.SystemIndigo), DependencyService.Get<ISystemColors>().SystemIndigo);
            this.Add(nameof(ISystemColors.SystemPurple), DependencyService.Get<ISystemColors>().SystemPurple);
            this.Add(nameof(ISystemColors.SystemPink), DependencyService.Get<ISystemColors>().SystemPink);
            this.Add(nameof(ISystemColors.SystemBrown), DependencyService.Get<ISystemColors>().SystemBrown);


            this.Add(nameof(ISystemColors.SystemGray), DependencyService.Get<ISystemColors>().SystemGray);
            this.Add(nameof(ISystemColors.SystemGray2), DependencyService.Get<ISystemColors>().SystemGray2);
            this.Add(nameof(ISystemColors.SystemGray3), DependencyService.Get<ISystemColors>().SystemGray3);
            this.Add(nameof(ISystemColors.SystemGray4), DependencyService.Get<ISystemColors>().SystemGray4);
            this.Add(nameof(ISystemColors.SystemGray5), DependencyService.Get<ISystemColors>().SystemGray5);
            this.Add(nameof(ISystemColors.SystemGray6), DependencyService.Get<ISystemColors>().SystemGray6);

            this.Add(nameof(ISystemColors.SystemLabel), DependencyService.Get<ISystemColors>().SystemLabel);
            this.Add(nameof(ISystemColors.SecondaryLabel), DependencyService.Get<ISystemColors>().SecondaryLabel);
            this.Add(nameof(ISystemColors.TertiaryLabel), DependencyService.Get<ISystemColors>().TertiaryLabel);
            this.Add(nameof(ISystemColors.QuaternaryLabel), DependencyService.Get<ISystemColors>().QuaternaryLabel);

            this.Add(nameof(ISystemColors.Placeholder), DependencyService.Get<ISystemColors>().Placeholder);
            this.Add(nameof(ISystemColors.Separator), DependencyService.Get<ISystemColors>().Separator);
            this.Add(nameof(ISystemColors.OpaqueSeparator), DependencyService.Get<ISystemColors>().OpaqueSeparator);
            this.Add(nameof(ISystemColors.LinkColor), DependencyService.Get<ISystemColors>().LinkColor);

            this.Add(nameof(ISystemColors.FillColor), DependencyService.Get<ISystemColors>().FillColor);
            this.Add(nameof(ISystemColors.SecondaryFillColor), DependencyService.Get<ISystemColors>().SecondaryFillColor);
            this.Add(nameof(ISystemColors.TertiaryFillColor), DependencyService.Get<ISystemColors>().TertiaryFillColor);
            this.Add(nameof(ISystemColors.QuaternaryFillColor), DependencyService.Get<ISystemColors>().QuaternaryFillColor);

            this.Add(nameof(ISystemColors.SystemBackgroundColor), DependencyService.Get<ISystemColors>().SystemBackgroundColor);
            this.Add(nameof(ISystemColors.SecondarySystemBackgroundColor), DependencyService.Get<ISystemColors>().SecondarySystemBackgroundColor);
            this.Add(nameof(ISystemColors.TertiarySystemBackgroundColor), DependencyService.Get<ISystemColors>().TertiarySystemBackgroundColor);

            this.Add(nameof(ISystemColors.SystemGroupedBackgroundColor), DependencyService.Get<ISystemColors>().SystemGroupedBackgroundColor);
            this.Add(nameof(ISystemColors.SecondarySystemGroupedBackgroundColor), DependencyService.Get<ISystemColors>().SecondarySystemGroupedBackgroundColor);
            this.Add(nameof(ISystemColors.TertiarySystemGroupedBackgroundColor), DependencyService.Get<ISystemColors>().TertiarySystemGroupedBackgroundColor);

            this.Add(nameof(ISystemColors.DarkTextColor), DependencyService.Get<ISystemColors>().DarkTextColor);
            this.Add(nameof(ISystemColors.LightTextColor), DependencyService.Get<ISystemColors>().LightTextColor);

        }
    }
}
```
 
That’s all for the implementation. Now let’s start having a look at how to use the whole code we wrote until now.

### Merging the `ResourceDictionary`

In `Xamarin.Forms`, we are able to merge `ResourceDictionary` classes to make them available for the whole app or on view/page level only. I consider our above created dictionary as an app-level dictionary. On top, to make it reusable, I put all these classes in a separate multi-platform library, which you can find [here on Github](https://github.com/MSiccDev/SystemColorsiOS).

Please note that [the syntax](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/resource-dictionaries#merged-resource-dictionaries) will be a little different if you implement the `ResourceDictionary` directly in your app. Using the library approach, you will merge the dictionary in this way in `App.xaml`:

``` xml
 <?xml version="1.0" encoding="utf-8" ?>
<Application
    x:Class="SystemColorsTest.App"
    xmlns="http://xamarin.com/schemas/2014/forms"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:systemcolors="clr-namespace:MSiccDev.Libs.iOS.SystemColors;assembly=MSiccDev.Libs.iOS.SystemColors">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <systemcolors:SystemColorsIosResourceDictionary />
                <!--  more dictionaries here  -->
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```
 
### Responding to system theme changes

Even if I personally only change the system theme at runtime for testing themes in my apps, your users may do so frequently. Luckily, it is just a matter of handling an event to handle this scenario. In your `App.xaml.cs` file, register for the `RequestedThemeChanged` event within the constructor:

``` csharp
         public App()
        {
            InitializeComponent();

            Application.Current.RequestedThemeChanged += OnRequestedThemeChanged;

            this.MainVm = new MainViewModel();
            MainPage mainPage = new MainPage()
            {
                BindingContext = this.MainVm
            };

            MainPage = mainPage;
        }
```
 
As the system colors respond to the system theme change, we need to reload them to get these changes.

Within the `OnRequestedThemeChanged` method, we are first getting the actual merged `ResourceDictionary` instance. Then, we will remove this instance and register a new instance of the `ResourceDictionary`. This will lead to a full reload of the system colors from iOS into the app. Here is the code:

``` csharp
 private void OnRequestedThemeChanged(object sender, AppThemeChangedEventArgs e)
{
    ResourceDictionary iosResourceDict = App.Current.Resources.MergedDictionaries.SingleOrDefault(dict => dict.GetType() == typeof(SystemColorsIosResourceDictionary));

    if (iosResourceDict != null)
    {
        App.Current.Resources.MergedDictionaries.Remove(iosResourceDict);
        App.Current.Resources.MergedDictionaries.Add(new SystemColorsIosResourceDictionary());
    }
}
```
 
That’s it, we are now ready to use the colors in XAML and our app adapts to system theme changes. Here is a sample XAML which I wrote to test the colors:

``` xml
 <?xml version="1.0" encoding="utf-8" ?>
<ContentPage
    x:Class="SystemColorsTest.MainPage"
    xmlns="http://xamarin.com/schemas/2014/forms"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:local="clr-namespace:SystemColorsTest"
    x:DataType="local:MainViewModel"
    BackgroundColor="{DynamicResource SystemBackgroundColor}">

    <StackLayout>
        <Frame
            Padding="12,42,24,12"
            BackgroundColor="{DynamicResource SystemGray3}"
            CornerRadius="0">
            <Label
                FontSize="36"
                HorizontalTextAlignment="Center"
                Text="iOS SystemColors in XF"
                TextColor="{AppThemeBinding Dark={DynamicResource LightTextColor},
                                            Light={DynamicResource DarkTextColor}}" />
        </Frame>

        <ScrollView>
            <StackLayout BindableLayout.ItemsSource="{Binding SystemColors}">
                <BindableLayout.ItemTemplate>
                    <DataTemplate>
                        <Frame
                            Margin="6,3"
                            x:DataType="local:SystemColorViewModel"
                            BackgroundColor="{Binding Value}">
                            <Label Text="{Binding Name}" />
                        </Frame>
                    </DataTemplate>
                </BindableLayout.ItemTemplate>
            </StackLayout>
        </ScrollView>
    </StackLayout>
</ContentPage>
```
 
Please note that I use `DynamicResource` instead of `StaticResource`, even if some colors are static. Using `DynamicResource` forces the app to reload the colors, and there are some that change (like the `SystemGray` color palette).

### Conclusion

Using the iOS system colors in `Xamarin.Forms` isn’t that complicated with this implementation. If you have more platforms, you could implement the same technique for the other platforms. As I am focusing on iOS for the moment, I just wrote that part. But who knows, maybe this will be extended in the future.

As always, I hope this post will be helpful for some of you.

#### Until the next post, happy coding, everyone!