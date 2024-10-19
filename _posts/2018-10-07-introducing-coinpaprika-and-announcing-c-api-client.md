---
id: 5778
title: 'Introducing Coinpaprika and announcing C# API Client'
date: '2018-10-07T11:27:17+02:00'
author: 'Marco Siccardi'
excerpt: 'Some of you may have noticed that I am recently diving into the world of cryptocurrencies and blockchains. Recently, I stumbled on Coinpaprika. In this post, I tell you what it is and why I made a C# API client for their API.'
layout: post
permalink: /introducing-coinpaprika-and-announcing-c-api-client/
image: /assets/img/2018/10/coinpaprika_client.png
categories:
    - Archive
tags:
    - API
    - coinpaprika
    - coins
    - crypto
    - cryptocurrencies
    - data
    - library
    - OpenSource
    - OSS
    - prices
    - projects
    - research
    - trading
---

As I am diving deeper and deeper into the world of cryptocurrencies, I am exploring quite some interesting products. One of them is [Coinpaprika](https://coinpaprika.com/), a market research site with some extensive information on every coin they have listed.

## What is Coinpaprika?

In the world of cryptocurrencies, there are several things one needs to discover before investing. Starting with information on the project and its digital currencies, the persons behind a project as well as their current value and its price, there is a lot of data to walk through before investing. Several sites out there are providing aggregated information, and even provide APIs for us developers. However, most of them are

- extremely rate limited
- freemium with a complex pricing model
- slow

## Why Coinpaprika?

A lot of services that provide aggregated data rely on data of the big players like CoinMarketCap. Coinpaprika, however, has a different strategy. They are pulling their data from a whopping number of 176 exchanges into their own databases, without any proxy. They have their own valuation system and a very fast refreshing rate (16 000 price updates per minute). If you have some time and want to compare how prices match up with their competition, Coinpaprika even implemented a [metrics page](https://coinpaprika.com/metrics/) for you. In my personal experience, their data is more reliable average to those values I see on those exchanges I deal with (Binance, Coinbase, BitPanda, Changelly, Shapeshift).

## Coinpaprika API and Clients

Early last week, I discovered Coinpaprika on Steemit. They [announced their API is now available to the general public](https://steemit.com/cryptocurrency/@coinpaprika/coinpaprika-release-api-clients) along with clients for PHP, GO, Swift and NodeJS. Coinpaprika has also a [WordPress plugin](https://wordpress.org/plugins/coinpaprika/) and an embeddable widget (on a coin’s detail page) that allows you to easily show price information on your website. After discovering their site, I got in contact with them to discuss a possible C# implementation for several reasons:

- very generous rate limits (25 920 00 requests per month, others are around 6 000 to 10 000), which enables very different implementation scenarios
- their API is fast like hell
- their independence from third parties besides exchanges
- their very catchy name (just being honest)

A few days later, I was able to discuss the publication of the C# API client implementation I wrote with them. I am happy to announce that you can now download the C# API Client [from Nuget](https://www.nuget.org/packages/CoinpaprikaAPI/) or fork it from my [Github repository](https://github.com/MSiccDev/CoinpaprikaAPI). They will also link to it from their official [API repository](https://github.com/coinpaprika/api) soon. The [readme-file on Github](https://github.com/MSiccDev/CoinpaprikaAPI/blob/master/README.md) serves as documentation as well and shows how to easily integrate their data into your .NET apps. The library itself is written in .NET Standard 2.0. If there is the need to target lower versions, feel free to open a pull request on Github. The Github repo contains also a console tester application.

## Conclusion

If you need reliable market data and information on the different projects behind all those cryptocurrencies, you should evaluate Coinpaprika. They aggregate their data without any third party involved and provide an easy to use and blazing fast API. I hope my contribution in form of the C# API client will be helpful for some of you out there.

If you like their product as much as I do, follow them:

- Twitter: https://twitter.com/coinpaprika
- Facebook: https://www.facebook.com/coinpaprika/
- Steemit: https://steemit.com/@coinpaprika
- Medium: https://medium.com/coinpaprika
- Telegram: https://t.me/Coinpaprika

### Happy coding, everyone!

*Disclaimer: I am contributing to this project with code written by me under the MIT License. Future contributions may contain their own (and different) disclaimer. I am not getting paid for my contributions to the project.*

*Please note that none of my crypto related posts is an investment or financial advice. As crypto currencies are volatile and risky, you should only invest as much as you can afford to lose. Always do your own research!*