---
id: 6712
title: 'Scroll to any item in your Xamarin.Forms CollectionView from your ViewModel'
date: '2020-06-19T21:35:51+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I am going to show you how we can easily extend CollectionView to implement ScrollTo-functionality through DataBinding from a ViewModel (for both grouped and non-grouped data sources).'
layout: post
permalink: /scroll-to-any-item-in-your-xamarin-forms-collectionview-from-your-viewmodel/
image: /assets/img/2020/06/CollectionViewEx-ScrollTo-Title.jpg
categories:
    - 'Dev Stories'
    - Xamarin
tags:
    - binding
    - collectionview
    - github
    - mvvm
    - sample
    - scroll
    - scrollto
    - ViewModel
    - xamarin
    - 'xamarin forms'
---

If you are working with collections in your app, chances are high you are going to want (or need) to scroll to a specific item at some point. `CollectionView` has the `ScrollTo` method that allows you to do so. If you are using MVVM in your app however, there is no built-in support to call this method.

#### My solution

My solution for this challenge consists of following parts:

- a `BindableProperty `in an extended `CollectionView` class to bind the item we want to scroll to
- a configuration class to control the scrolling behavior
- a base interface with the configuration and two variants derived from it (one for ungrouped items, one for grouped ones)

Let’s have a look at the `ScrollConfiguration` class:

``` csharp
 public class ScrollToConfiguration
{
    public bool Animated { get; set; } = true;

    public ScrollToPosition ScrollToPosition { get; set; } = ScrollToPosition.Center;
}
```
 
These two properties are used to tell our extended` CollectionView` how the scrolling to the item will behave. The above default values are my preferred ones, feel free to change them in your implementation.

Next, let us have a look at the base interface:

``` csharp
 public interface IConfigurableScrollItem
{
    ScrollToConfiguration Config { get; set; }
}
```
 
Then we will define two additional interfaces which we are going to use later in our `ViewModel`:

``` csharp
     public interface IScrollItem : IConfigurableScrollItem
    {
    }

    public interface IGroupScrollItem : IConfigurableScrollItem
    {
        object GroupValue { get; set; }
    }
```
 
For a non-grouped `CollectionView`, we just need to implement `IScrollItem`. If we have groups, we’ll use `IGroupScrollItem` to add an object that identifies the group (following the `Xamarin.Forms` API here).

#### Extending CollectionView

Let’s connect the dots and implement an extended version of the `CollectionView` – to do so, create a new class and derive from it. I named mine `CollectionViewEx` (ingenious, right?).

To wrap things up, we now add a `BindableProperty` with a `PropertyChanged` handler to our `CollectionViewEx` that we can bind against, and which is, most importantly, calling the `ScrollTo` method of `CollectionView`.

Here is the full class:

``` csharp
 public class CollectionViewEx : CollectionView
{
    public static BindableProperty ScrollToItemWithConfigProperty = BindableProperty.Create(nameof(ScrollToItemWithConfig), typeof(IConfigurableScrollItem), typeof(CollectionViewEx), default(IConfigurableScrollItem), BindingMode.Default, propertyChanged: OnScrollToItemWithConfigPropertyChanged);

    public IConfigurableScrollItem ScrollToItemWithConfig
    {
        get => (IConfigurableScrollItem)GetValue(ScrollToItemWithConfigProperty);
        set => SetValue(ScrollToItemWithConfigProperty, value);
    }

    private static void OnScrollToItemWithConfigPropertyChanged(BindableObject bindable, object oldValue, object newValue)
    {
        if (newValue == null)
            return;

        if (bindable is CollectionViewEx current)
        {
            if (newValue is IGroupScrollItem scrollToItemWithGroup)
            {
                if (scrollToItemWithGroup.Config == null)
                    scrollToItemWithGroup.Config = new ScrollToConfiguration();

                    current.ScrollTo(scrollToItemWithGroup, scrollToItemWithGroup.GroupValue, scrollToItemWithGroup.Config.ScrollToPosition, scrollToItemWithGroup.Config.Animated);

            }
            else if (newValue is IScrollItem scrollToItem)
            {
                if (scrollToItem.Config == null)
                    scrollToItem.Config = new ScrollToConfiguration();

                    current.ScrollTo(scrollToItem, null, scrollToItem.Config.ScrollToPosition, scrollToItem.Config.Animated);
            }
        }
    }
}
```
 
Let’s go through the code. The `BindableProperty` implementation should be common to most of us ([if not, read up the docs](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/bindable-properties)). The most important part happens in the `PropertyChanged` handler.

By allowing the value of the `BindableProperty` to be null, we can reset the item and scroll to the same item again if necessary. Because `IScrollItem` as well as `IGroupScrollItem` derive from `IConfigurableScrollItem`, we can handle them both in one method. To make sure there is a default `ScrollToConfiguration`, I am checking the `Config` property for null – in case it is (because I forgot it), there is at least the default. In the end, I am scrolling to the Item in the `CollectionView` using the `ScrollTo` method.

#### The ViewModel(s) and Binding

Here is one of the (simple) `ViewModels` from the sample application for this post:

``` csharp
 public class ItemViewModel : ViewModelBase, IScrollItem
{
    public ItemViewModel()
    {
        this.Config = new ScrollToConfiguration();
    }

    public string Text { get; set; }

    public int Number { get; set; }

    public ScrollToConfiguration Config { get; set; }
}
```
 
Now in the parent `ViewModel`, we just add another property that we can use to bind against the `CollectionViewEx`‘s `ScrollToItemWithConfig` property. The Binding is straight forward:

``` xml
 <controls:CollectionViewEx
    Grid.Row="3"
    Margin="6"
    ItemsSource="{Binding ScrollableItems}"
    ScrollToItemWithConfig="{Binding ScrollToVm}"
    SelectedItem="{Binding SelectedItemVm, Mode=TwoWay}"
    SelectionMode="Single">
    <controls:CollectionViewEx.ItemTemplate>
        <DataTemplate>
            <Grid>
                <Label Margin="5,10" Text="{Binding Text}" />
            </Grid>
        </DataTemplate>
    </controls:CollectionViewEx.ItemTemplate>
</controls:CollectionViewEx>
```
 
The result of this whole exercise looks like this:

[](/assets/img/2020/06/ScrollToItem_iOS.gif)

#### Conclusion

Even if the `CollectionView` control in `Xamarin.Forms` provides a whole bunch of optimized functionalities over `ListView`, there are some scenarios that require additional work. Luckily, it isn’t that hard to extend the `CollectionView`. Scrolling to a precise `ViewModel` is easy with the code above. Of course, I created also a [sample Github repo](https://github.com/MSiccDev/CollectionViewEx).

As always, I hope this post will be helpful for some of you.

###### Until the next post, happy coding, everyone!