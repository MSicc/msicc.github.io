---
id: 6504
title: 'Visual Studio Extensions that will make your life easier in 2020'
date: '2020-01-05T18:23:41+01:00'
author: 'Marco Siccardi'
excerpt: 'We finally arrived in 2020 - time to kick off my blogging year as well. With this post, I will show you some of the Visual Studio Extensions I am using frequently and that are making my life as a developer a bit easier.'
layout: post
permalink: /visual-studio-extensions-that-will-make-your-life-easier-in-2020/
image: /assets/img/2020/01/vs-extension-post-2020-title.png
categories:
    - 'Dev Stories'
tags:
    - code
    - extensions
    - helpers
    - icons
    - markdown
    - msbuild
    - output
    - readability
    - regions
    - tools
    - 'Visual Studio'
    - vs2019
    - XAML
---

### Xaml Styler

Download Link: <https://marketplace.visualstudio.com/items?itemName=TeamXavalon.XAMLStyler>

Project Site: <https://github.com/Xavalon/XamlStyler>

In Visual Studio 2019, writing XAML normally ends up in very long lines because it does not automatically break into new lines. In May 2019, I discovered the Xaml Styler extension by watching a video on [Channel9](https://channel9.msdn.com/Shows/XamarinShow/Pretty-XAML-With-XAML-Styler).

Xaml Styler, even in its default configuration (which I did never change, tbh), breaks your XAML code into new lines once the count of properties of an element exceeds a certain amount. The extension as tons of options to configure, but for me, the default config always worked well.

 Here is a sample:

[![](/assets/img/2020/01/XamlStyler_Sample_Xaml.png)
](/assets/img/2020/01/XamlStyler_Sample_Xaml.png)
As you can see, the XAML file is a by far more readable now. The different colors on the namespace declarations and closing tags are coming from the next extension on my list.

### Viasfora

Download Link: <https://marketplace.visualstudio.com/items?itemName=TomasRestrepo.Viasfora>

Project Site: <https://viasfora.com/>

Viasfora aims to make your code more readable. To achieve that goal, it is coloring the code inside Visual Studio. The coloring makes it easy to see the scopes of each line, method, and even classes. As I discovered the extensions only a few weeks ago, I decided to not change any of the default settings, but it is already making my code a whole lot more readable.

Here is a sample from one of my recent projects:

[![](/assets/img/2020/01/Viasfora_colored_method_sample.png)
](/assets/img/2020/01/Viasfora_colored_method_sample.png)
### VSColorOutput

Download Link: <https://marketplace.visualstudio.com/items?itemName=MikeWard-AnnArbor.VSColorOutput>

Project Site: <https://mike-ward.net/vscoloroutput/>

Ever tried to search the Visual Studio output for warnings, build errors or exceptions? Of course, you did. VSColorOutput makes this search a whole lot easier, as it colors errors/exceptions in red, warning in yellow, and build success messages in green. You see everything at a glance, which can save you a lot of time. See yourself:

[![](/assets/img/2020/01/vscoloroutput_sample.png)
](/assets/img/2020/01/vscoloroutput_sample.png)
</figure>### CodeMaid

Download Link: <https://marketplace.visualstudio.com/items?itemName=SteveCadwallader.CodeMaid>

Project Site: <https://www.codemaid.net/>

CodeMaid is a very powerful extension. I am pretty sure I am not even using half of its functions, to be honest. I mainly use it to let it automatically sort my code files to my gusto and let it generate regions around it. This way, my code stays always organized in the same way and I don’t even have to think about it. I will explore more of the other functions moving onwards.

Here is a sample of a code file after CodeMaid cleaned it up (collapsed regions):

[![](/assets/img/2020/01/codemaid_sample_class_after_cleanup.png)
](/assets/img/2020/01/codemaid_sample_class_after_cleanup.png)
### Material icons generator

Download Link: <https://marketplace.visualstudio.com/items?itemName=nikainteristi.Materialiconsgenerator>

Project Site: <https://github.com/interisti/vs-material-icons-generator>

If you are developing mobile applications, chances are high you will need one or another icon within an app. Material icons generator tries to replicate the popular function of Android Studio, with the added bonus of making it also available for iOS and UWP apps. The extension sadly does not create Image Asset folders and .json files for iOS correctly, but it creates the images so one can create an Image Asset from it in just a minute. The project does seem to be actively developed according to Github – I put this extension on the list because I did not find a better alternative until now.

[![](/assets/img/2020/01/material_icons_generator_sample.png)
](/assets/img/2020/01/material_icons_generator_sample.png)
### Markdown Editor

Download Link: <https://marketplace.visualstudio.com/items?itemName=MadsKristensen.MarkdownEditor>

Project Site: <https://github.com/madskristensen/MarkdownEditor>

If you have to deal with Markdown files, there are floating quite a few helper apps around the web. I prefer to work on them in Visual Studio, as I use Markdown most of the time on Github when writing readme or similar files. Markdown Editor makes it easy to do so, and I almost immediately see the result in the preview window, which is quite helpful from time to time.

The extension is written by the creator of [Markdig](https://github.com/lunet-io/markdig), which is by far your best option if you have to deal with Markdown in your C# app. Here is a sample:

[![](/assets/img/2020/01/markdown_editor_vs_sample.png)
](/assets/img/2020/01/markdown_editor_vs_sample.png)
### Regex Editor

Download Link: <https://marketplace.visualstudio.com/items?itemName=GeorgyLosenkov.RegexEditorLite>

Project Site: N/A

If you ever had to deal with validations, chances are high you solved it with Regex. The extension provides a bunch of options that will help you to write Regex-patterns. You can test it in the same Window with some sample data to verify it will do what it is supposed to do. As a bonus, it is able to create a sample method of how to use it if you want to. Here is what it looks like with an email verification pattern:

[![](/assets/img/2020/01/RegexEditor_Test.png)
](/assets/img/2020/01/RegexEditor_Test.png)
### Conclusion

To kick off my blogging year 2020, I showed you some of the most helpful Visual Studio extensions I installed on my machine. I hope some of you will find the one or another extension as helpful as I do. If you have an extension that is not on my list and you consider it useful, feel free to sound off in the comments or via social media.

#### Until the next post, happy coding!