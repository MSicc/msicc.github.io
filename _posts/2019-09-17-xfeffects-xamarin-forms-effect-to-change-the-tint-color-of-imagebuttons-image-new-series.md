---
id: 6228
title: '[Updated] #XfEffects: Xamarin.Forms Effect to change the TintColor of ImageButton&#8217;s image &#8211; (new series)'
date: '2019-09-17T18:00:00+02:00'
author: 'Marco Siccardi'
excerpt: 'Every now and then, we need to customize our Xamarin.Forms apps to match a theme or other requirements. The Forms API has already a lot of options, but sometimes they aren''t enough. A common practice is to write custom renderers, but with the Effects API, there is another cool kid in town.'
layout: post
permalink: /xfeffects-xamarin-forms-effect-to-change-the-tint-color-of-imagebuttons-image-new-series/
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Xamarin
tags:
    - Android
    - AppCompatImageButton
    - 'bindable properties'
    - color
    - ColorStateList
    - effects
    - Imagebutton
    - iOS
    - PlatformEffect
    - renderer
    - ResolutionGroup
    - RoutingEffect
    - Tint
    - UIButton
    - UIImageView
    - xamarin
    - 'xamarin forms'
    - XfEffects
---

[The documentation](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/effects/introduction) recommends using Effects when we just want to change properties on the underlying native control. I have begun to love Effects as they make my code more readable. With Effects, I always know that there is a platform-specific implementation attached, while that is not obvious when using a custom renderer. Nowadays, I always try to implement an Effect before a Renderer.

---

 Update: I updated the sample project to also respect changes in the `ImageButton`‘s source. I recently ran into the situation to change the `Source` (Play/Pause) via my ViewModel based on its state and realized that the effect needs to be reapplied in this case. Please be aware.

---

#### The basics

Effects work in a similar way to Renderers. You implement the definition in the `Xamarin.Forms` project, which attaches it to the control that needs the change. The `PlatformEffect` implementation needs to be exported to be compiled into the application. Like in a Renderer, the platform implementation also supports property changes. In this new series [\#XfEffects](https://msicc.net/tag/XfEffects/), I am going to show you some `Effects` that have been useful for me.

#### Effect declaration

Let’s turn over to this post’s `Effect`. We will change the `TintColor `of an `ImageButton` to our needs. Let’s start with creating the class for our `Effect`:

``` csharp
     public class ImageButtonTintEffect : RoutingEffect
    {
        public ImageButtonTintEffect() : base($"XfEffects.{nameof(ImageButtonTintEffect)}")
        {
        }
    }
```
 
All `Xamarin.Forms` `Effect` implementations need to derive from the `RoutingEffect` class and pass the `Effect`‘s name to the base class’ constructor. That’s pretty much everything we have to do for the `Effect` class itself.

#### Effect extension for passing parameters

The easiest way for us to get our desired `TintColor` to the platform implementation is an attached `BindableProperty`. To be able to attach the `BindableProperty`, we need a static class that provides the container for the attached property:

``` csharp
 public static class ImageButtonTintEffectExtensions
{
    public static readonly BindableProperty TintColorProperty = BindableProperty.CreateAttached("TintColor", typeof(Color), typeof(ImageButtonTintEffectExtensions), default, propertyChanged: OnTintColorPropertyChanged);
    public static Color GetTintColor(BindableObject bindable)
    {
        return (Color)bindable.GetValue(TintColorProperty);
    }
    public static void SetTintColor(BindableObject bindable, Color value)
    {
        bindable.SetValue(TintColorProperty, value);
    }
    private static void OnTintColorPropertyChanged(BindableObject bindable, object oldValue, object newValue)
    {
    }
}
```
 
Of course, we want to set the `TintColor` property as `Xamarin.Forms.Color` as this will make it pretty easy to control the color from a `Style` or even a ViewModel.

We want our effect to only being invoked if we change the default `TintColor` value. This makes sure we are running only code that is necessary for our application:

``` csharp
 private static void OnTintColorPropertyChanged(BindableObject bindable, object oldValue, object newValue)
{
    if (bindable is ImageButton current)
    {
        if ((Color)newValue != default)
        {
            if (!current.Effects.Any(e => e is ImageButtonTintEffect))
                current.Effects.Add(Effect.Resolve(nameof(ImageButtonTintEffect)));
        }
        else
        {
            if (current.Effects.Any(e => e is ImageButtonTintEffect))
            {
                var existingEffect = current.Effects.FirstOrDefault(e => e is ImageButtonTintEffect);
                current.Effects.Remove(existingEffect);
            }
        }
    }
}
```
 
Last but not least in our `Xamarin.Forms` application, we want to use the new `Effect`. This is pretty easy:

``` xml
 <!--import the namespace-->
xmlns:effects="clr-namespace:XfEffects.Effects"
<!--then use it like this-->
<ImageButton Margin="12,0,12,12"
    effects:ImageButtonTintEffectExtensions.TintColor="Red"
    BackgroundColor="Transparent"
    HeightRequest="48"
    HorizontalOptions="CenterAndExpand"
    Source="ic_refresh_white_24dp.png"
    WidthRequest="48">
    <ImageButton.Effects>
        <effects:ImageButtonTintEffect />
    </ImageButton.Effects>
</ImageButton>
```
 
We are using the attached property we created above to provide the `TintColor` to the `ImageButtonTintEffect`, which we need to add to the `ImageButton`‘s `Effects` List.

#### Android implementation

Let’s have a look at the Android-specific implementation. First, let’s add the platform class and decorate it with the proper attributes to export our effect:

``` csharp
 using Android.Content.Res;
using Android.Graphics;
using System;
using System.ComponentModel;
using Xamarin.Forms;
using Xamarin.Forms.Platform.Android;
using AWImageButton = Android.Support.V7.Widget.AppCompatImageButton;
[assembly: ResolutionGroupName("XfEffects")]
[assembly: ExportEffect(typeof(XfEffects.Droid.Effects.ImageButtonTintEffect), nameof(XfEffects.Effects.ImageButtonTintEffect))]
namespace XfEffects.Droid.Effects
{
    public class ImageButtonTintEffect : PlatformEffect
    {
        protected override void OnAttached()
        {
            
        }
        protected override void OnDetached()
        {
        }
        protected override void OnElementPropertyChanged(PropertyChangedEventArgs args)
        {
        }
    }
}
```
 
Remember: the `ResolutionGroupName` needs to be defined just once per app and should not change. Similar to a custom `Renderer`, we also need to export the definition of the platform implementation and the `Forms` implementation to make our `Effect` working.

Android controls like buttons have different states. Properties on Android controls like the color can be set based on their `State` attribute. `Xamarin.Android` implements these states in the `Android.Resource.Attribute` [class](https://docs.microsoft.com/en-us/dotnet/api/android.resource.attribute?f1url=https%3A%2F%2Fmsdn.microsoft.com%2Fquery%2Fdev16.query%3FappId%3DDev16IDEF1%26l%3DEN-US%26k%3Dk(Android.Resource.Attribute);k(TargetFrameworkMoniker-MonoAndroid,Version%3Dv9.0);k(DevLang-csharp)%26rd%3Dtrue&view=xamarin-android-sdk-9). We define our `ImageButton`‘s states like this:

``` csharp
 static readonly int[][] _colorStates =
{
    new[] { global::Android.Resource.Attribute.StateEnabled },
    new[] { -global::Android.Resource.Attribute.StateEnabled }, //disabled state
    new[] { global::Android.Resource.Attribute.StatePressed } //pressed state
};
```
 
Good to know: certain states like ‘*disabled*‘ are created by just adding a ‘-‘ in front of the matching state defined in the [OS states list](https://developer.android.com/reference/android/R$attr) (negating it). We need this declaration to create our own `ColorStateList`, which will override the color of the `ImageButton`‘s image. Add this method to the class created above:

``` csharp
 private void UpdateTintColor()
{
    try
    {
        if (this.Control is AWImageButton imageButton)
        {
            var androidColor = XfEffects.Effects.ImageButtonTintEffectExtensions.GetTintColor(this.Element).ToAndroid();
            var disabledColor = androidColor;
            disabledColor.A = 0x1C; //140
            var pressedColor = androidColor;
            pressedColor.A = 0x24; //180
            imageButton.ImageTintList = new ColorStateList(_colorStates, new[] { androidColor.ToArgb(), disabledColor.ToArgb(), pressedColor.ToArgb() });
            imageButton.ImageTintMode = PorterDuff.Mode.SrcIn;
        }
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine(
            $"An error occurred when setting the {typeof(XfEffects.Effects.ImageButtonTintEffect)} effect: {ex.Message}\n{ex.StackTrace}");
    }
}
```
 
This code works above the Android SDK 23, as only then the ability to modify the A-channel of the defined color was added. The `Xamarin.Forms` `ImageButton` translates into a `AppCompatImageButton` on Android. The `AppCompatImageButton` has the `ImageTintList `property. This property is of type [ColorStatesList](https://docs.microsoft.com/en-us/dotnet/api/android.content.res.colorstatelist.-ctor?f1url=https%3A%2F%2Fmsdn.microsoft.com%2Fquery%2Fdev16.query%3FappId%3DDev16IDEF1%26l%3DEN-US%26k%3Dk(Android.Content.Res.ColorStateList.%2523ctor);k(TargetFrameworkMoniker-MonoAndroid,Version%3Dv9.0);k(DevLang-csharp)%26rd%3Dtrue&view=xamarin-android-sdk-9), which uses the states we defined earlier and the matching colors for those states.

Last but not least, we need to set the composition mode. If you want to get a deeper understanding of that, a great starting point is [this StackOverflow question](https://stackoverflow.com/questions/8280027/what-does-porterduff-mode-mean-in-android-graphics-what-does-it-do). To make things not too complicated, we are infusing the color into the image. The final step is to call the method in the `OnAttached` override as well as in the `OnElementPropertyChanged` override.

The result based on the sample I created looks like this:

<div class="wp-block-image"><figure class="aligncenter size-large">![](https://msicc.net/assets/img/2019/09/tint-android.jpg)</figure></div>#### iOS implementation

Of course, also on iOS, we have to attribute the class, similar to the Android version:

``` csharp
 using System;
using System.ComponentModel;
using UIKit;
using Xamarin.Forms;
using Xamarin.Forms.Platform.iOS;
[assembly: ResolutionGroupName("XfEffects")]
[assembly: ExportEffect(typeof(XfEffects.iOS.Effects.ImageButtonTintEffect), nameof(XfEffects.Effects.ImageButtonTintEffect))]
namespace XfEffects.iOS.Effects
{
    public class ImageButtonTintEffect : PlatformEffect
    {
        protected override void OnAttached()
        {
        }
        protected override void OnDetached()
        {
        }
        protected override void OnElementPropertyChanged(PropertyChangedEventArgs args)
        {
        }
    }
}
```
 
The underlying control of the `Xamarin.Forms` `ImageButton` is a default `UIButton` on iOS. The `UIButton` control has an `UIImageView`, which can be changed with the `SetImage` method. Based on that knowledge, we are going to implement the `UpdateTintColor` method:

``` csharp
 private void UpdateTintColor()
{
    try
    {
        if (this.Control is UIButton imagebutton)
        {
            if (imagebutton.ImageView?.Image != null)
            {
                var templatedImg = imagebutton.CurrentImage.ImageWithRenderingMode(UIImageRenderingMode.AlwaysTemplate);
                //clear the image on the button
                imagebutton.SetImage(null, UIControlState.Normal);
                imagebutton.ImageView.TintColor = XfEffects.Effects.ImageButtonTintEffectExtensions.GetTintColor(this.Element).ToUIColor();
                imagebutton.TintColor = XfEffects.Effects.ImageButtonTintEffectExtensions.GetTintColor(this.Element).ToUIColor();
                imagebutton.SetImage(templatedImg, UIControlState.Normal);
            }
        }
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine($"An error occurred when setting the {typeof(ImageButtonTintEffect)} effect: {ex.Message}\n{ex.StackTrace}");
    }
}
```
 
Let’s review these code lines. The first step is to extract the already attached Image as a templated Image from the `UIButton`‘s `UIImageView`. The second step is to clear the Image from the Button, using the `SetImage` method and passing `null` as `UIImage` parameter. I tried to leave out this step, but it does not work if you do so.

The next step changes the `TintColor` for the `UIButton`‘s `UIImageView` as well as for the button itself. I was only able to get the `TintColor` changed once I changed both `TintColor` properties.

The final step is to re-attach the extracted image from step one to the `UIImageButton`‘s `UIImageView`, using once again the `SetImage` method – but this time, we are passing the extracted image as `UIImage` parameter.

Of course, also here we need to call this method in the `OnAttached` override as well as in `OnElementPropertyChanged`.

The result should look similar to this one:

<div class="wp-block-image"><figure class="aligncenter size-large">![](https://msicc.net/assets/img/2019/09/tint-ios.jpg)</figure></div>#### Conclusion

It is pretty easy to implement extended functionality with a Xamarin.Forms Effect. The process is similar to the one of creating a custom renderer. By using attached properties you can fine-tune the usage of this code and also pass property values to the platform implementations.

Please make sure to follow along for the other Effects I will post as part of this new series. I also created a sample app to demonstrate the Effects ([find it on Github](https://github.com/MSiccDev/XfEffects)). As always, I hope this post will be helpful for some of you.

##### Until the next post, happy coding, everyone!