---
id: 5539
title: '#XfQaD: Limit maximum lines of Label and indicate text truncation'
date: '2018-03-21T16:30:51+01:00'
author: 'Marco Siccardi'
excerpt: 'If we want to display some text in our Xamarin.Forms app, chances are high we are going to use the Label control. But what if we want to limit the maximum visible lines of such a Label? This is what this #XfQaD is about.'
layout: post
permalink: /xfqad-limit-maximum-lines-of-label-and-indicate-text-truncation/
image: /assets/img/2018/03/limit_truncated_label.png
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - Android
    - iOS
    - label
    - limit
    - 'limit text'
    - 'max lines'
    - renderer
    - 'text trimming'
    - 'text truncation'
    - uwp
    - xamarin
    - 'xamarin forms'
    - XfQaD
---

## The problem

`Xamarin.Forms.Label`has a common set of properties we can use to configure how our text is shown. However, it does miss a property to limit the maximum of text lines and a proper indication of eventually truncated text. Knowing that UWP, Android and iOS have working and easy-to-use implementations on their platform controls used for the `Xamarin.Forms.Label`, there is only one solution to the problem: exposing a custom control and its platform renderers. That’s what we are going to do in this [\#XfQaD]({{ site.baseurl }}{/tags/xfqad/})

### XfMaxLines Label implementation

Let’s have a look at the `Xamarin.Forms` implementation first. I am just adding a `BindableProperty`to a derived class implementation to define the maximum of lines I want to see:

``` csharp
 public class XfMaxLinesLabel : Label
{
    public XfMaxLinesLabel(){ }

    public static BindableProperty MaxLinesProperty = BindableProperty.Create("MaxLines", typeof(int), typeof(XfMaxLinesLabel), int.MaxValue, BindingMode.Default);

    public int MaxLines
    {
        get => (int)GetValue(MaxLinesProperty);
        set => SetValue(MaxLinesProperty, value);
    }
}
```
 
### UWP

The UWP renderer uses the [TextBlock](https://docs.microsoft.com/en-us/uwp/api/Windows.UI.Xaml.Controls.TextBlock) properties `MaxLines`to limit the amount of shown lines, while the `TextTrimming`property is set to ellipsize the last word before reaching the limit. The implementation is pretty straight forward:

``` csharp
 [assembly: ExportRenderer(typeof(XfMaxLinesLabel), typeof(XfMaxLinesLabelRenderer))]
namespace MaxLinesLabel.UWP
{
    public class XfMaxLinesLabelRenderer : LabelRenderer
    {
        protected override void OnElementChanged(ElementChangedEventArgs<Label> e)
        {
            base.OnElementChanged(e);

            if (((XfMaxLinesLabel)e.NewElement).MaxLines == -1 || ((XfMaxLinesLabel)e.NewElement).MaxLines == int.MaxValue)
                return;

            this.Control.MaxLines = ((XfMaxLinesLabel)e.NewElement).MaxLines;
            this.Control.TextTrimming = Windows.UI.Xaml.TextTrimming.WordEllipsis;
        }

        protected override void OnElementPropertyChanged(object sender, PropertyChangedEventArgs e)
        {
            base.OnElementPropertyChanged(sender, e);

            if (e.PropertyName == XfMaxLinesLabel.MaxLinesProperty.PropertyName)
            {
                if (((XfMaxLinesLabel)this.Element).MaxLines == -1 || ((XfMaxLinesLabel)this.Element).MaxLines == int.MaxValue)
                    return;

                this.Control.MaxLines = ((XfMaxLinesLabel)this.Element).MaxLines;
                this.Control.TextTrimming = Windows.UI.Xaml.TextTrimming.WordEllipsis;
            }
        }
    }
}
```
 
## Android

The Android implementation uses the `MaxLines`property of Android’s [TextView](https://developer.android.com/reference/android/widget/TextView.html) control to limit the maximum visible lines. The `Ellipsize`property is used to show the three dots for truncation at the end of the last visible line. Once again, pretty straight forward.

``` csharp
 [assembly: ExportRenderer(typeof(XfMaxLinesLabel), typeof(XfMaxLinesLabelRenderer))]
namespace MaxLinesLabel.Droid
{
    class XfMaxLinesLabelRenderer : LabelRenderer
    {
        public XfMaxLinesLabelRenderer(Context context) : base(context)
        {
        }


        protected override void OnElementChanged(ElementChangedEventArgs<Label> e)
        {
            base.OnElementChanged(e);

            if (((XfMaxLinesLabel)e.NewElement).MaxLines == -1 || ((XfMaxLinesLabel)e.NewElement).MaxLines == int.MaxValue)
                return;
            this.Control.SetMaxLines(((XfMaxLinesLabel)e.NewElement).MaxLines);
            this.Control.Ellipsize = TextUtils.TruncateAt.End;
        }


        protected override void OnElementPropertyChanged(object sender, PropertyChangedEventArgs e)
        {
            base.OnElementPropertyChanged(sender, e);

            if (e.PropertyName == XfMaxLinesLabel.MaxLinesProperty.PropertyName)
            {
                if (((XfMaxLinesLabel)this.Element).MaxLines == -1 || ((XfMaxLinesLabel)this.Element).MaxLines == int.MaxValue)
                    return;
                this.Control.SetMaxLines(((XfMaxLinesLabel)this.Element).MaxLines);
                this.Control.Ellipsize = TextUtils.TruncateAt.End;
            }
        }
    }
}
```
 
## iOS

Like Android and Windows, also the [UILabel](https://developer.apple.com/documentation/uikit/uilabel) control on iOS has a `MaxLines`property. You’re right, we’ll use this one to limit the count of visible lines. Using the `LineBreakMode`property, we can automate the text truncation indicator equally easy as on Android and UWP:

``` csharp
 [assembly: ExportRenderer(typeof(XfMaxLinesLabel), typeof(XfMaxLinesLabelRenderer))]
namespace MaxLinesLabel.iOS
{
    public class XfMaxLinesLabelRenderer : LabelRenderer
    {
        protected override void OnElementChanged(ElementChangedEventArgs<Label> e)
        {
            base.OnElementChanged(e);

            if (((XfMaxLinesLabel)e.NewElement).MaxLines == -1 || ((XfMaxLinesLabel)e.NewElement).MaxLines == int.MaxValue)
                return;

            this.Control.Lines = ((XfMaxLinesLabel)e.NewElement).MaxLines;
            this.Control.LineBreakMode = UILineBreakMode.TailTruncation;
        }

        protected override void OnElementPropertyChanged(object sender, PropertyChangedEventArgs e)
        {
            base.OnElementPropertyChanged(sender, e);

            if (e.PropertyName == XfMaxLinesLabel.MaxLinesProperty.PropertyName)
            {
                if (((XfMaxLinesLabel)this.Element).MaxLines == -1 || ((XfMaxLinesLabel)this.Element).MaxLines == int.MaxValue)
                    return;

                this.Control.Lines = ((XfMaxLinesLabel)this.Element).MaxLines;
                this.Control.LineBreakMode = UILineBreakMode.TailTruncation;
            }
        }
    }
}
```
 
## Conclusion

As you can see, it is pretty easy to create a line limited, truncation indicating custom Label for your Xamarin.Forms app. The implementation is done in a few minutes, but it makes writing your cross platform app a bit easier. I don’t know why this is not (yet) implemented in current Xamarin.Forms iterations, but I do hope they’ll do so to further reduce the number of needed custom renderers.

In the meantime, feel free to check the [sample code on GitHub](https://github.com/MSiccDev/XfQADs/tree/master/MaxLines/MaxLinesLabel) and use it in your apps. As always, I hope this post is helpful for some of you.

### Happy coding, everyone!