---
id: 7069
title: '#CASBAN6: Creating A Serverless Blog on Azure with .NET 6 (new series)'
date: '2022-09-05T22:26:09+02:00'
author: 'Marco Siccardi'
excerpt: 'This is the first post of my new #CASBAN6 blog series, where I will document how I am creating a serverless blog on Azure as well as dedicated clients with .NET 6.'
layout: post
permalink: /casban6-creating-a-serverless-blog-on-azure-with-net-6-new-series/
image: /assets/img/2022/09/serverles-blog-init-tiitle-image.jpg
categories:
    - 'Dev Stories'
    - Android
    - Azure
    - iOS
    - MAUI
    - Web
tags:
    - .NET
    - .NET6
    - Android
    - asp.net
    - Azure
    - 'Azure Functions'
    - blog
    - EF
    - 'Entity Framework'
    - functions
    - github
    - iOS
    - macOS
    - MAUI
    - 'Open Source'
    - serverless
    - SQL
    - Windows
---

### Motivation

I was planning to run my blog without WordPress for quite some time. For one, because WordPress is really blown up as a platform. The second reason is more of a practical nature – this project gives me lots of stuff to improve my programming skills. I already started to move [my developer website](https://msiccdev.net) away from WordPress with ASP.NET CORE and Razor Pages. Eventually I arrived at the point where I needed to implement a blog engine for the news section. So, I have two websites (including this one here) that will take advantage of the outcome of this journey.

### High Level Architecture

Now that the ‘why’ is clear, let’s have a look at the ‘how’:

<div class="wp-block-image"><figure class="aligncenter size-large is-resized">![](https://msicc.net/assets/img/2022/09/ServerlessBlogAzure-1024x728.png)</figure></div>There are several layers in my concept. The data layer consists of a serverless MS SQL instance on Azure, on which I will work with the help of Entity Framework Core and Azure Functions for all the CRUD operations of the blog. I will use the powers of Azure API Management, which will allow me to provide a secure layer for the clients – of course, an ASP.NET CORE Website with RazorPages, flanked by a .NET MAUI admin client (no web administration). Once the former two are done, I will also add a mobile client for this blog. It will be the next major update for my existing blog reader that is already in the app stores.

For comments, I will use Disqus. This way, I have a proven comment system where anyone can use his/her favorite account to participate in discussions. They also have an API, so there is a good chance that I will be able to implement Disqus in the Desktop and Mobile clients.

Last but not least, there are (for now) two open points – performance measuring/logging and notifications. I haven’t decided yet how to implement these – but I guess there will be an Azure based implementation as well (until there are good reasons to use another service).

### Open Source

Most of the software I will write and blog about in this series will be available publicly on GitHub. You can [find the repository](https://github.com/MSiccDev/ServerlessBlog) already there, including stuff for the next two upcoming blog posts already in there.

### Index

I will update this blog post regularly with a link new entries of the series.

- \#CASBAN6: Creating A Serverless Blog on Azure with .NET 6 (new series) (this post)
- [\#CASBAN6: How to set up a local Microsoft SQL database on macOS](https://msicc.net/casban6-how-to-set-up-a-local-microsoft-sql-database-on-macos/)
- [\#CASBAN6: the data model explained](https://msicc.net/casban6-the-data-model-explained/)
- [\#CASBAN6: Implementing the data model using EntityFramework Core (separate libraries)](https://msicc.net/casban6-implementing-the-data-model-using-entityframework-core-separate-libraries/)
- [\#CASBAN6: the DTOs and mappings](https://msicc.net/casban6-the-dtos-and-mappings/)
- [\#CASBAN6: Setting up an Azure Functions project for the API](https://msicc.net/casban6-setting-up-an-azure-functions-project-for-the-api/)
- [\#CASBAN6: Function base class (and an update to the DTO models)](https://msicc.net/casban6-function-base-class-and-an-update-to-the-dto-models/)
- \#CASBAN6: tbd

*Additional note*

*Please note that I am working on this in my spare time. This may result in delays between the blog posts and the updates committed into the repository on GitHub.*

#### Until the next post – happy coding, everyone!

---

Title Image by [Roman](https://pixabay.com/users/akitada31-172067/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=6515064) from [Pixabay](https://pixabay.com//?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=6515064)