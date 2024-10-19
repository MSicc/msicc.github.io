---
id: 3777
title: 'How to save a List or a Collection on a NFC tag'
date: '2013-10-20T17:12:34+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-save-a-list-or-a-collection-on-a-nfc-tag/
categories:
    - Archive
tags:
    - Collections
    - list
    - NFC
    - save
    - tag
    - WinPhanDev
    - wpdev
---

![WP8_NFC_PostLogo](/assets/img/2013/10/WP8_NFC_PostLogo.png "WP8_NFC_PostLogo")

As I am currently working again on my [NFC app](http://www.windowsphone.com/s?appid=2c33cb7d-c97b-4204-aa8b-1e8712718519), I needed to save an ObservableCollection to a tag. My first attempts resulted in a heavily overlong string that I wasn’t able to save.

Anyways, after a short convo on Twitter, I went for the right way – serialize to JSON.

The first thing you’ll need for that is the JSON.NET library, [which you can get here](http://james.newtonking.com/json) or via the NuGet package manager in Visual Studio.

After that, you will be able to save your List or ObservableCollection in a few easy steps.

One thing I need to recommend is to make the names in your class/viewmodel as short as possible. Here is my example:

``` csharp
    public class ListItems
    {
        //ListItems
        public string i { get; set; }
        //isChecked
        public bool iC { get; set; }
    }
```
 
You will need every space you can get on your tag, so you really should go for a similar way like I did above.

After you are done with that, you only need one line of code to convert your List/Collection for writing on your tag:

``` csharp 
var ListToSave = JsonConvert.SerializeObject(ItemsList);
```
 
Now you will be able to save it as a Text record or whatever record type you need to save it to.

Deserializing the JSON string works also with only one line of code:

``` csharp
ItemsList = JsonConvert.DeserializeObject<ObservableCollection<ListItems>>(StringFromYourTag);
```
 
Then you set the ItemsSource of your ListBox to that (or whatever else the List/Collection is for).

I my case, I was able to save a Collection of 25 items to my tag (writable size 716 bytes) with having still about 200 bytes left in this way.

As always, I hope this will be helpful for some of you.

Happy coding!