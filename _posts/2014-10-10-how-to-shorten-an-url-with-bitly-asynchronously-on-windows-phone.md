---
id: 4174
title: 'How to shorten an url with Bitly asynchronously on Windows Phone'
date: '2014-10-10T03:18:57+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-shorten-an-url-with-bitly-asynchronously-on-windows-phone/
categories:
    - Archive
tags:
    - 'escaped url'
    - sharing
    - social
    - 'social networks'
    - 'text length limit'
    - 'text limit'
    - 'url shortener'
    - web
    - wpdev
---

![bitly_logo](/assets/img/2014/10/bitly_logo.png "bitly_logo")

It was only yesterday when I decided to shorten shared links in [UniShare](https://www.windowsphone.com/s?appid=ee42cb1d-8a68-41c6-9c0c-d3e3fc61d6ea). The reason is not Twitter as suggested by some of my users, but other networks like LinkedIn, that have text length limits as well.

After digging into the Bitly API, I created a little helper that returns the a shorten Bitly-url. In case this is not possible for whatever reason, it returns the long url that was tried to be shortened.

Before we can start to write some code, we need to get our generic access token for the Bitly API. Log in to your Bitly account, and click on ‘Settings’, and choose the ‘Advanced’ tab. If you already verified your mail address, you are nearly done. Enter your password and click on the ‘Generate Token’ Button:

![Screenshot (388)](/assets/img/2014/10/Screenshot-388.png "Screenshot (388)")

This is one of the constants we need soon. In your project, generate a new class called BitlyHelper. Declare the following constant strings:

``` csharp
 private const string bitlyGenericAccessToken = "your_generic_access_token";
private const string base_url = "https://api-ssl.bitly.com/v3/shorten?access_token={0}&longUrl={1}&format=txt";
```
 
The base\_url string contains the url we are calling, setting our generic access token as well as the long url we want to share. As we are only interested in getting the shortened url, we need to add the ‘format=txt’ part at the end.

The next step is to create an async Task that returns the HttpResponseMessage:

``` csharp
         /// <summary>
        /// Task that starts the async request for the HttpResponseMessage
        /// </summary>
        /// <param name="longUrl">the url to shorten</param>
        /// <returns>the HttpResponseMessage that contains the shortened url</returns>
        private async Task<HttpResponseMessage> getShortUrl(string longUrl)
        {
            //do not forget to UrlEncode the longUrl!
            string request_url = string.Format(base_url, bitlyGenericAccessToken, System.Web.HttpUtility.UrlEncode(longUrl));

            HttpClient client = new HttpClient();
            client.DefaultRequestHeaders.IfModifiedSince = DateTime.Now;

            //get the response from the Bitly API
            return await client.GetAsync(new Uri(request_url));
        }
```
 
The only important thing we think of here is to UrlEncode our long url, using the System.Web.HttpUtility.UrlEncode() method. If we would not do that, we would get an error on some links from the Bitly API. The rest is pretty straight forward to any other HttpClient usage.

To get our BitlyHelper to be helpful, we are creating another Task that detects the HttpStatusCode of our HttpResponseMessage and returns the shortened url:

``` csharp
         /// <summary>
        /// gets the shortened url out of the HttpResponseMessage
        /// </summary>
        /// <param name="longUrl">the url to shorten</param>
        /// <returns>the shortened url as string</returns>
        public async Task<string> GetShortenedUrl(string longUrl)
        {
            string short_url = string.Empty;

            //using try/catch to avoid Exceptions
            try
            {
                var response = await getShortUrl(longUrl);

                if (response.StatusCode == HttpStatusCode.Ok)
                {
                    short_url = await response.Content.ReadAsStringAsync();
                }
                //on error StatusCodes, just return the longUrl
                else
                {
                    short_url = longUrl;
                }
            }
            catch
            {
                short_url = longUrl;
            }

            return short_url;            
        }
```
 
In case the HttpStatusCode is not OK (200), we are simply returning the long url. To avoid any Exceptions, we are doing the same in case there is one. This way, we are keeping it as simple as possible while keeping our desired functionality.

The usage is very simple:

``` csharp
 BitlyHelper bitly = new BitlyHelper();
var shorturl = await bitly.GetShortenedUrl("https://yourlongurl.com");
```
 
If you want more advanced features, you can perform the whole oAuth dance with your users and get some more features into your app ([read more in the API docs](https://dev.bitly.com/get_started.html)). If you just need to shorten the url, this BitlyHelper is all you need.

As always, I hope this is helpful for some of you.

Happy coding!