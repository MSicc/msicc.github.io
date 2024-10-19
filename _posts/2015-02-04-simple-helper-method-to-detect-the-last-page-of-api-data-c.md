---
id: 4286
title: 'Simple helper method to detect the last page of API data (C#)'
date: '2015-02-04T05:39:25+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /simple-helper-method-to-detect-the-last-page-of-api-data-c/
categories:
    - Archive
tags:
    - API
    - 'C#'
    - calculate
    - data
    - helper
    - indicator
    - 'last page'
    - universal
    - 'Windows 8.1'
    - 'Windows Phone'
    - wpdev
---

When you are working with APIs from web services, you probably ran already into the same problem that I did recently: how to detect if we are on the last page of possible API results.

Some APIs (like WordPress) use tokens to be sent as parameter with your request, and if the token is null or empty you know that you have reached the last page. However, not all APIs are working that way (for example UserVoice).

As I am rewriting Voices Admin to be a Universal app, I came up with a simple but effective helper method that allows me to easily detect if I am on the last page. Here is what I did:

``` csharp
 	public static bool IsLastPage(int total, int countperpage, int current)
        {
            bool value = false;

            if (current < Convert.ToInt32(Math.Ceiling(Convert.ToDouble(total)/countperpage)))
            {
                value = false;
            }

            if (current == Convert.ToInt32(Math.Ceiling(Convert.ToDouble(total)/countperpage)))
                value = true;

            return value;
        }
```
 
As you can see, I need the number of total records that can be fetched (which is returned by the API) and the property for the number per page (which is one of the optional parameters of the API). On top, I need the current page number to calculate where I am (which is also an optional parameter of the API and returned by the API result).

Now I simply need to divide the total records by the result count per page to get how many pages are used. Using the [Math.Ceiling()](https://msdn.microsoft.com/en-us/library/zx4t0t48(v=vs.110).aspx) method, I always get the correct number of pages back. What does the Math.Ceiling() method do? It just jumps up to the next absolute number, also known as “rounding toward positive infinity”.

Example: if you have 51 total records and a per page count of 10, the division will return 5.1 (which means there is only one result on the 6th page). However, we need an absolute number. Rounding the result would return 5 pages, which is wrong in this case. The Math.Ceiling() method however returns the correct value of 6.

Setting the method up as a static Boolean makes it easy to change the CanExecute property of a button for example, which will be automatically disabled if we just have loaded the last page (page 6 in my case).

As always, I hope this is helpful for some of you.

Happy coding, everyone!