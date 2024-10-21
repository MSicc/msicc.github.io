---
id: 1817
title: 'How to install the Windows Phone SDK 7.1.1 on W8CP (WPDev)'
date: '2012-03-31T12:17:25+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-install-the-windows-phone-sdk-7-1-1-on-w8cp-wpdev/
image: /assets/img/2012/03/Screenshot-86.png
categories:
    - Archive
tags:
    - w8cp
    - 'Windows Phone'
    - wpdev
---

[![Screenshot (86)](/assets/img/2012/03/Screenshot-86.png "Screenshot (86)")](/assets/img/2012/03/Screenshot-86.png)


This week Microsoft released the Windows Phone SDK 7.1.1 to us developers. If you are on Windows 7, you can hit the [download](https://www.microsoft.com/download/en/details.aspx?id=29233) button and install the SDK, everything will go fine.

If you have used my [tutorial]({% post_url 2012-03-03-wpdev-on-w8cp-how-to-install-the-windows-phone-sdk %}) on how to install the SDK on W8CP, things are a little different. You might face the following error message:

[![Screenshot (74)](/assets/img/2012/03/Screenshot-74.png "Screenshot (74)")](/assets/img/2012/03/Screenshot-74.png)


The error says that the update is not applicable or is blocked by another condition on your computer (you can read more about that [here](https://support.microsoft.com/kb/2600847)). The Link to the KB-site does not offer anything helpful, so I played around to find a workaround.

Here is a my tutorial for installing the SDK in this case:

- perform a “repair install” of the Windows Phone SDK 7.1.1
- [download](https://www.microsoft.com/download/en/details.aspx?id=29233) the update
- install the update, if no error shows up, everything will go fine
- if you face the message again, abort the installation
- download the update again
- after second download, everything should go fine

I know this is a little bit confusing. In fact it was the only way for me to get the update installed. I tried several ways, and if a way did not work I restored the point before the repair install to get this tutorial done.

After the installation I also tried to open up my blog app on the 256 MB emulator. Of course I was curios whether my will start or not, luckily it did:

[![Screenshot (85)](/assets/img/2012/03/Screenshot-85.png "Screenshot (85)")](/assets/img/2012/03/Screenshot-85.png)


So I hope with my little tutorial I was able to help you to get developing started with Windows 8 again. For my part I am happy, as I often code while I am at train, and in battery mode debugging on device does not work on W8CP.

**Happy coding everyone!**