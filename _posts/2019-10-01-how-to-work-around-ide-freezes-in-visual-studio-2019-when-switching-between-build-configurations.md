---
id: 6434
title: 'How to work around IDE freezes in Visual Studio 2019 when switching between Build configurations'
date: '2019-10-01T11:17:58+02:00'
author: 'Marco Siccardi'
excerpt: 'After updating to the latest Visual Studio version (16.3., followed by 16.3.1), two of our solutions at work with custom Build configurations all of sudden began to hang when switching between them.'
layout: post
permalink: /how-to-work-around-ide-freezes-in-visual-studio-2019-when-switching-between-build-configurations/
categories:
    - 'Dev Stories'
tags:
    - build
    - 'build configuration'
    - freeze
    - vs2019
    - workaround
---

After a lot of project unloading, deleting our custom configurations back and forth and project file modifications, we finally found a workaround to continue our work by browsing [other/similar issues to ours in the VS feedback forums](https://developercommunity.visualstudio.com/content/problem/607020/vs2019-will-randomly-hand-while-unloading-projects.html). The freeze of the solution is caused by a feature that is meant to fasten up solution loading: **parallel project initialization**.

*It seems to be an issue that some others already had with 16.2.x versions, so it may be a regression (as it is flagged as fixed).*

### The workaround

The workaround is pretty easy, just follow these simple steps:

- Open Tools / Options in Visual Studio 2019 and find the Projects and Solutions node
- **Unselect ‘Allow parallel project initialization’**
- Click ‘OK’ and close the solution

<div class="wp-block-image"><figure class="aligncenter size-large">![](https://msicc.net/assets/img/2019/10/paralell-project-init-setting.png)</figure></div>After that, we need to **delete the *.vs* folder** of Visual Studio 2019 within the local solution folder. This folder contains a SQLite DB that corresponds with some behind the scenes stuff for the parallel project initialization (and more).

Now open the solution – switching between Build configurations should now work again.

As always, I hope this post is helpful for some of you.

#### Until the next, happy coding!