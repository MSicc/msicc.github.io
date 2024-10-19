---
id: 6693
title: 'Code Snippets for Xamarin.Forms BindableProperty'
date: '2020-06-05T15:00:46+02:00'
author: 'Marco Siccardi'
excerpt: 'This post is my contribution to Louis Matos'' Xamarin Month with the topic Code Snippets. '
layout: post
permalink: /code-snippets-for-xamarin-forms-bindableproperty/
image: /assets/img/2020/06/xf-bindp-snippets-title2.png
categories:
    - 'Dev Stories'
    - Xamarin
tags:
    - 'bindable properties'
    - binding
    - snippets
    - xamarin
    - 'xamarin forms'
    - xamarinmonth
---

If you haven’t heard about the [\#XamarinMonth](#XamarinMonth) before, you should either click on the hashtag or read Luis’ [blog post](https://luismts.com/code-snippetss-xamarin-month/ "Code Snippets Xamarin Month") where he introduces the topic. At the end of his post, you will also find a link to all the contributions.

### Why Code Snippets?

Code snippets make it easier to reuse code you write repeatedly during your dev life. For Xamarin Forms, there are no “built-in” snippets, so I decided to start creating my own snippets some time ago. If you want to get started, [have a look at the docs](https://docs.microsoft.com/en-us/visualstudio/ide/code-snippets?view=vs-2019).

### The Snippets

If you ever wrote a custom control or an Effect for Xamarin Forms, chances are high you also used the `BindableProperty` [functionality](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/bindable-properties). I got tired of remembering the syntax, so I decided to write some snippets for it:

- `BindableProperty `(*bindp*)
- `BindableProperty` with `PropertyChanged` handler (*bindppc*)
- Attached `BindableProperty` (*bindpa*)
- Attached `BindableProperty` with `PropertyChanged` handler (*bindpapc*)

You can download all of them from [my Github repo](https://github.com/MSiccDev/XfSnippets/tree/master/BindableProperty).

Once you insert (*write shortcut, Tab, Tab*) the snippet, you just need to define the name of the property as well as its type and hit enter – that’s it.

Tip: To make sure that the snippets show up with IntelliSense, go to **Tools/Options**, find the **Text Editor** section followed by the **C#** entry. Under **IntelliSense**, find the option for ‘**Snippet behavior**‘ and choose ‘*Always include snippets*‘.

### Conclusion

Snippets can save a lot of time. Luckily, the implementation is not that difficult (see docs link above). As always, I hope this post is helpful for some of you.

##### Until the next post, happy coding, everyone!