---
id: 2416
title: '[WPDev] Add Multiline Text to a Resource File in Windows Phone'
date: '2012-05-04T21:59:18+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /wpdev-add-multiline-text-to-a-resource-file-in-windows-phone/
categories:
    - Archive
tags:
    - 'Windows Phone'
---

Today I had to solve the problem that I needed to add multiline text into my resource file. I did not find a Windows Phone specific solution, but enough hints to solve the problem.

Here is a short guide for you:

First I added a TextBlock control and formatted it in XAML with Linebreaks:

[![TextBlockLineBreaks](/assets/img/2012/05/TextBlockLineBreaks.png "TextBlockLineBreaks")](/assets/img/2012/05/TextBlockLineBreaks.png)

The result is a nicely formatted TextBlock:

[![TextBlockResultCorrect](/assets/img/2012/05/TextBlockResultCorrect.png "TextBlockResultCorrect")](/assets/img/2012/05/TextBlockResultCorrect.png)

Now I tried of course to copy and paste the string in my resource file, and got an ugly result:

[![ResourceCopyPasteTextBlockResult](/assets/img/2012/05/ResourceCopyPasteTextBlockResult.png "ResourceCopyPasteTextBlockResult")](/assets/img/2012/05/ResourceCopyPasteTextBlockResult.png)

As mentioned above, I found out enough hints to solve this problem. There are two ways to solve it:

- The simple way is to add your text, and for each new line press “Shift+Enter”. The result looks like this in your resource file: [![cleanResource](/assets/img/2012/05/cleanResource.png "cleanResource")](/assets/img/2012/05/cleanResource.png)
    
    Please note that you have to click inside of the value field to display the whole content.
- The second way is to edit the XML-File directly. To open the resource file in XML-mode, right click your resource file and choose “Open with..”. In the following menu choose XML (Text)-Editor. Locate your resource string. Also if it looks not nice in code, you have to format your entry like this: [![xmlResourceLinebreak](/assets/img/2012/05/xmlResourceLinebreak.png "xmlResourceLinebreak")](/assets/img/2012/05/xmlResourceLinebreak.png)

Both methods will result in a nicely formatted multiline TextBlock after building your app again.

I hope this short guide is helpful.