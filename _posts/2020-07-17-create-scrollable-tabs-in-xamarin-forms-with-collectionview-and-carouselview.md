---
id: 6747
title: 'Create scrollable tabs in Xamarin.Forms with CollectionView and CarouselView'
date: '2020-07-17T09:00:00+02:00'
author: 'Marco Siccardi'
excerpt: 'This post will show you how to create a scrollable tabs list that gets synchronized with items in a CarouselView in Xamarin.Forms.'
layout: post
permalink: /create-scrollable-tabs-in-xamarin-forms-with-collectionview-and-carouselview/
image: /assets/img/2020/07/ScrollableTabsXF_Title.png
categories:
    - 'Dev Stories'
    - Xamarin
tags:
    - Android
    - carouselview
    - collectionview
    - iOS
    - mvvm
    - tabs
    - template
    - View
    - ViewModel
    - 'xamarin forms'
---

When it comes to navigation patterns in mobile apps, the tabbed interface is one of the most popular options. While `Xamarin.Forms` has the `TabbedPage` (and `Shell`) to fulfill that need, it lacks one essential feature: scrollable tabs. After studying some of the samples floating around the web and some of the packages that provide such functionality, I tried find an easier solution.

### The View

Let’s have a look at the View first. Like you may have guessed from the title, we are using a `<a href="https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/collectionview/" rel="noreferrer noopener nofollow" target="_blank">CollectionView</a>` for the tabs and a `<a href="https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/carouselview/" rel="noreferrer noopener nofollow" target="_blank">CarouselView</a>` for the Content. This combination makes it quite easy to implement tabs that cover a whole page size or smaller ones within a page.

Here’s the XAML:

``` csharp
 <Grid x:DataType="{x:Null}" RowSpacing="0">
    <Grid.RowDefinitions>
        <RowDefinition Height="45" />
        <RowDefinition Height="45" />
        <RowDefinition Height="*" />
    </Grid.RowDefinitions>

    <CollectionView
        x:Name="CustomTabsView"
        Grid.Row="1"
        HorizontalScrollBarVisibility="Never"
        ItemSizingStrategy="MeasureAllItems"
        ItemsSource="{Binding TabVms}"
        ItemsUpdatingScrollMode="KeepItemsInView"
        SelectedItem="{Binding CurrentTabVm, Mode=TwoWay}"
        SelectionMode="Single"
        VerticalScrollBarVisibility="Never">
        <CollectionView.ItemsLayout>
            <LinearItemsLayout Orientation="Horizontal" />
        </CollectionView.ItemsLayout>
        <CollectionView.ItemTemplate>
            <DataTemplate x:DataType="local:TabViewModel">
                <Grid RowSpacing="0">
                    <Grid.RowDefinitions>
                        <RowDefinition Height="*" />
                        <RowDefinition Height="3" />
                    </Grid.RowDefinitions>
                    <Label
                        x:Name="TitleLabel"
                        Grid.Row="0"
                        Padding="15,0"
                        FontAttributes="Bold"
                        FontSize="Small"
                        HeightRequest="50"
                        HorizontalTextAlignment="Center"
                        Text="{Binding Title}"
                        TextColor="White"
                        VerticalTextAlignment="Center" />
                    <BoxView
                        x:Name="ActiveIndicator"
                        Grid.Row="1"
                        BackgroundColor="Red"
                        IsVisible="{Binding IsSelected, Mode=TwoWay}" />
                </Grid>
            </DataTemplate>
        </CollectionView.ItemTemplate>
    </CollectionView>

    <CarouselView
        Grid.Row="2"
        CurrentItem="{Binding CurrentTabVm, Mode=TwoWay}"
        CurrentItemChanged="CarouselView_CurrentItemChanged"
        HorizontalScrollBarVisibility="Never"
        IsScrollAnimated="True"
        IsSwipeEnabled="True"
        ItemsSource="{Binding TabVms}"
        VerticalScrollBarVisibility="Never">
        <CarouselView.ItemTemplate>
            <DataTemplate x:DataType="local:TabViewModel">
                <Grid Margin="10">
                    <Grid.RowDefinitions>
                        <RowDefinition Height="*" />
                    </Grid.RowDefinitions>
                    <Label
                        Grid.Row="0"
                        Margin="10"
                        LineBreakMode="WordWrap"
                        Text="{Binding Content}"
                        VerticalTextAlignment="Center" />
                </Grid>
            </DataTemplate>
        </CarouselView.ItemTemplate>
    </CarouselView>
</Grid>
```
 
Let me break that piece down. First, I wrapped everything in a `Grid` for this sample. The `CollectionView` of course should be horizontally scrolling but should not show any scroll bar. The tab item template is not a complex one – it is just a `Label` and a `BoxView` below it to help with indication of the selection. You are free to make the tab looking whatever you want because of the `CollectionView`, however.

Below that, we put a `CarouselView`. For this sample, I just made a simple one with a Lorem Ipsum `Label` in it on every item.

### The ViewModels

Most of you know that I absolutely love the MVVM pattern. And this sample proves me right once again. We need just need two `ViewModel`s to handle scrolling and synchronizing.

The first `ViewModel` is the `TabViewModel`:

Snippet

``` csharp
 public class TabViewModel : ObservableObject
{
    private string _title;
    private string _content;
    private bool _isSelected;

    public TabViewModel(string title)
    {
        this.Title = title;
        this.Content = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Tempor id eu nisl nunc mi ipsum faucibus vitae aliquet. Turpis egestas integer eget aliquet nibh praesent tristique magna. In fermentum posuere urna nec tincidunt. Vitae congue eu consequat ac felis donec et odio pellentesque. Augue lacus viverra vitae congue. Viverra vitae congue eu consequat. Orci nulla pellentesque dignissim enim sit amet venenatis urna. Et ultrices neque ornare aenean euismod elementum nisi. Id consectetur purus ut faucibus pulvinar. In cursus turpis massa tincidunt. Egestas pretium aenean pharetra magna. Et pharetra pharetra massa massa ultricies mi quis. Nunc sed blandit libero volutpat. Purus viverra accumsan in nisl nisi scelerisque eu ultrices vitae.";
    }

    public string Title { get => _title; set => Set(ref _title, value); }

    public string Content { get => _content; set => Set(ref _content, value); }

    public bool IsSelected { get => _isSelected; set =>Set(ref _isSelected, value); }
}
```
 
The `TabViewModel` in this sample just has the bare minimum, namely the `Title`, the `Content` and the `IsSelectedFlag` to control the Visibility of the Indicator-`BoxView`. Nothing dramatic so far.

The `MainViewModel` glues everything together, so let’s have a look:

``` csharp
 public class MainViewModel : ObservableObject
{
    private TabViewModel _currentTabVm;

    public MainViewModel()
    {

        this.TabVms = new ObservableCollection<TabViewModel>();
        this.TabVms.Add(new TabViewModel("Short Title"));
        this.TabVms.Add(new TabViewModel("A Little Longer Title"));
        this.TabVms.Add(new TabViewModel("An Even Longer Title Than Before"));
        this.TabVms.Add(new TabViewModel("Again Short Title"));
        this.TabVms.Add(new TabViewModel("Mini Title"));
        this.TabVms.Add(new TabViewModel("Different Title"));

        this.CurrentTabVm = this.TabVms.FirstOrDefault();
    }


    public ObservableCollection<TabViewModel> TabVms { get; set; } 

    public TabViewModel CurrentTabVm 
    { 
        get => _currentTabVm;
        set
        {
            Set(ref _currentTabVm, value);
            SetSelection();
        }
    }

    private void SetSelection()
    {
        this.TabVms.ForEach(vm => vm.IsSelected = false);
        this.CurrentTabVm.IsSelected = true;
    }
}
```
 
Once again, there is nothing complex in it. We are mocking a collection of `TabViewModel` and handle the tab selection via Binding. After the current item got selected, we are setting the `IsSelected` property on it to true to show the Indicator in the `CollectionView`.

For this sample, I didn’t use a fully blown MVVM framework, so I am setting the `BindingContext` in the `MainPage`‘s constructor. The Binding engine in `Xamarin.Forms` already does almost everything to make this work.

The only thing left is to handle the positioning of the tabs if we are swiping the `CarouselView`. As this is purely View related, I am using the `CurrentItemChanged` event in code behind to center the `CollectionView`‘s selected item:

``` csharp
 private void CarouselView_CurrentItemChanged(object sender, CurrentItemChangedEventArgs e)
{
    this.CustomTabsView.ScrollTo(e.CurrentItem, null, ScrollToPosition.Center, true);
}
```
 
The result of this setup looks like this:

<div class="wp-block-image"><figure class="aligncenter size-full">![](https://msicc.net/assets/img/2020/07/ScrollableTabsXFiOS.gif)</figure></div>### Conclusion

`Xamarin.Forms` provides a lot of solutions out of the box. Sometimes, however, these are not enough. Luckily, we can combine some of the solutions the framework provides to create fresh solutions within our apps. This post showed one of these. The additional bonus you get with this implementation is the ability to style the tabs in whatever way you want. As always, I hope this post will be helpful for some of you.

Of course, there is also sample for this post on [Github](https://github.com/MSicc/CustomTabViewSample).

##### Until the next post, happy coding, everyone!