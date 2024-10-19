---
id: 6932
title: '#XFQaD: Compile XAML without code behind in Xamarin.Forms'
date: '2021-11-30T15:00:00+01:00'
author: 'Marco Siccardi'
excerpt: 'Recently, I discovered the possibility to have XAML files without code behind in Xamarin.Forms. This #XFQaD shows you how to make them work.'
layout: post
permalink: /xfqad-compile-xaml-without-code-behind-in-xamarin-forms/
image: /assets/img/2021/11/Compile_XAML_Only_Title2.png
categories:
    - 'Dev Stories'
tags:
    - Compile
    - ResourceDictionary
    - resources
    - 'xamarin forms'
    - XAML
    - XfQaD
---

### How I discovered this [\#XFQaD](https://msicc.net/tag/xfqad/)

When I was reorganizing the application resources on my current side project, I decided to create some thematically divided ResourceDictionary files. This has led me to do a quick research on Xamarin.Forms resource dictionaries.

If you’re doing this research, you will stumble upon [this post from 2018 (!)](https://devblogs.microsoft.com/xamarin/better-resource-organization-xamarin-forms/), where the hint to the magic I’ll show you soon was hidden.

Until now, I only used this with ResourceDictionary files. Maybe it will be helpful also for other XAML resources like controls (I will try that in the future).

### How to make XAML only files compile

Add a new XAML file (sample is still ResourceDictionary) to your project:

<div class="wp-block-image is-style-default"><figure class="aligncenter size-full is-resized">![Add_ResourceDictionary](https://msicc.net/assets/img/2021/11/Add_ResourceDictionary_XAML.png)</figure></div>Delete the code behind file, and add your XAML code. Before hitting the Build button, add this line immediately after the .xml file header:

``` xml
 <?xaml-comp compile="true" ?>
```
 
This line is where the magic happens. It tells the compiler to include the XAML file in the Build, pretty much like it does with XAML compile attribute in the code behind file.

### Conclusion

The only place where I found this hint was the blog post mentioned above. Neither the docs on [XAML Compilation](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/xamlc) nor the docs on [resource dictionaries](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/resource-dictionaries) are mentioning this trick. It somehow went quietly from the nightly builds into production code.

Using this trick, we are able to have a more clean project structure. As always, I hope this post will be helpful for some of you.

#### Until the next post, happy coding!