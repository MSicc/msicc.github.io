---
id: 6891
title: 'Fix &#8216;Xcode is not currently installed or could not be found&#8217; error in Visual Studio 2019 for Mac'
date: '2021-06-05T09:33:59+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post I show you how to easily fix the Xcode install location in VS 2019 for Mac after installing the Xcode CLI tools.'
layout: post
permalink: /fix-xcode-is-not-currently-installed-or-could-not-be-found-error-in-visual-studio-2019-for-mac/
image: /assets/img/2021/06/Screenshot-2021-06-05-at-08.40.56.png
categories:
    - 'Dev Stories'
    - iOS
    - Xamarin
tags:
    - iOS
    - Mac
    - macOS
    - 'Visual Studio'
    - xamarin
    - Xcode
---

Every now and then, our IDE’s get some updates. This week, Visual Studio for MacOS got updated once again. After that, there was a separated Download initiated for the Xcode Command Line Tools. Two days later, Visual Studio started to greet me with this little message:

![Xcode missing message VS Mac](/assets/img/2021/06/Screenshot-2021-06-05-at-08.39.05.png)
Of course, I checked first that my installed version of Xcode is still working – it stopped already for me some time ago and I had to reinstall it. As you can see, that was not the case:

![Xcode 12.5 about window](/assets/img/2021/06/Screenshot-2021-06-05-at-08.40.26.png)
After doing some research on the web, [others had similar issues](https://docs.microsoft.com/en-us/answers/questions/296951/visual-studio-for-mac-no-sdk-found-at-specified-lo.html). The problem was that the installation of the Xcode CLI tools has overridden the location of Xcode in Preferences – as you can see in the

The fix is easy, just paste `/Applications/Xcode.app/` into the location field. Please note that the trailing slash is also important:

![](/assets/img/2021/06/Screenshot-2021-06-05-at-08.42.07.png)
The dialog will immediately verify the existence of Xcode (at least in version 8.9.10). Just hit that restart button and you are once again good to go.

As always, I hope this short post will be helpful for some of you.

#### Until the next post – happy coding, everyone. 