---
id: 3452
title: 'Dev Story Series (Part 4 of many): How to open links from a WebBrowser/WebView in Internet Explorer'
date: '2013-01-29T21:57:41+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /dev-story-series-part-4-of-many-how-to-open-links-from-a-webbrowserwebview-in-internet-explorer/
categories:
    - Archive
tags:
    - webbrowser
    - webview
    - win8dev
    - 'Windows 8'
    - 'Windows Phone'
    - WordPress
    - wpdev
---

[![XAMLWebView](/assets/img/2013/01/Screenshot-87-300x174.png)](/assets/img/2013/01/Screenshot-87.png)

Today I will share my solution of how to open links from a WebBrowser or WebView on Windows Phone and Windows 8 (as I did in my app for msicc.net).

Generally links are opened within the same WebBrowser or WebView element. On Windows Phone you can solve this problem pretty easy with only one simple method:

``` csharp
        public void WebBrowser_Navigating(object sender, NavigatingEventArgs e)
        {
            e.Cancel = true;
            WebBrowserTask wbt = new WebBrowserTask();
            wbt.Uri = e.Uri;
            wbt.Show();
        }
```
 

In Windows 8 this gets a bit more complex. There is no Navigating method, and this is why we have to combine different languages together. We will use the ScriptNotify event to pass the link via a small java script and launch IE with the new link.

First, you need to add the function to your HTML string. I used a RegEx method to add all the pattern that is needed to all links. As I know that a lot of us (especially junior devs like me) have their problems with RegEx, here is my method to add them:

``` csharp
        public static string AddScriptToLink(string text)
        {
            const string hrefScript = " onclick="return OnLinkClick('{0}')" ";
            const string pattern = @"href=""(.*?)""";

            var result = text;

            var matches = Regex.Matches(text, pattern);
            var sortedMatches = matches.Cast<Match>().OrderByDescending(x => x.Index);
            foreach (var match in sortedMatches)
            {
                var replacement = string.Format(hrefScript, match.Groups[1].Value);
                result = result.Insert(match.Index, replacement);
            }

            return result;
        }
```
 
After you did that, you will be able to use the following script:

``` js
<script type='text/javascript'>
	function OnLinkClick(a) 
		{       
         	 window.external.notify(a);       
		 return false;    
		}
</script>
```
 

I pass this script together with the HTML string (HTML methods need to be first!) to the WebView. I recommend to save the script as a static string, so you have do insert only the name of the string.

If you add the ScriptNotify Event to your WebView, you will be able to use LaunchUriAsync with the value of e, which is the link.

``` csharp
        private async void WebView_ScriptNotify(object sender, NotifyEventArgs e)
        {
            await Windows.System.Launcher.LaunchUriAsync(new Uri(e.Value));
        }
```
 

I am not sure if there is a better way to do this, but that is my way. It works like it should and does not hurt the experience of my app.

As always I hope this is helpful for someone out there. If you have any way to improve that, feel free to leave a comment below.