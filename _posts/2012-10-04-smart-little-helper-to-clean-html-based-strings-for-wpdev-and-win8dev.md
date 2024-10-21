---
id: 2875
title: 'Smart little helper to clean HTML based strings for [WPDEV] and [WIN8DEV]'
date: '2012-10-04T05:14:25+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /smart-little-helper-to-clean-html-based-strings-for-wpdev-and-win8dev/
categories:
    - Archive
tags:
    - win8dev
    - wpdev
---

I thought I would share this small little class I created to clean out HTML based strings into readable strings. The class is both usable for your Windows Phone app as well as your Windows 8 app.

If you use cloud based or internet data, often strings contain “&amp;ndash;” or “&amp;#8211;” instead of their intended letter, like you can see on this image:

![htmlstringuncleaned](/assets/img/2012/10/Screenshot-161.png "htmlstringuncleaned")

The method uses Regex to replace all the wrong signs and clean out all of the unwanted signs. You can also add more to that if you need to replace wrong letters.

``` csharp
 class CleanHTML
    {      
        public static string RemoveEncoding(string text)
        {
            try
            {
                string temp="";

                temp = 
                    Regex.Replace
                    (text.
                    Replace("&ndash;", "-").
                    Replace("&nbsp;", " ").
                    Replace("&rsquo;", "'").
                    Replace("&amp;", "&").
                    Replace("&#038;", "&").
                    Replace("&quot;", """).
                    Replace("&#039;", "'").
                    Replace("&#8230;", "...").
                    Replace("&#8212;", "—").
                    Replace("&#8211;", "-").
                    Replace("&#8220;", "“").
                    Replace("&#8221;", "”").
                    Replace("&#8217;", "'").
                    Replace("&#160;", " ").
                    Replace("&gt;", ">").
                    Replace("&rdquo;", """).
                    Replace("&ldquo;", """).
                    Replace("&lt;", "<").
                    Replace("&#215;", "×").
                    Replace("&#8242;", "′").
                    Replace("&#8243;", "″").
                    Replace("&#8216;", "'"),
                    "<[^<>]+>", "");

                return temp;
            }
            catch
            {
                return "";
            }
        }

        }
```
 

As we created a class for this little helper, you can call it from everywhere within you app to clean out the string. Here is one example I am using:

``` csharp
  item.title = CleanHTML.RemoveEncoding(item.title);
```
 

After calling this method, your string is plain text:

![Screenshot (17)](/assets/img/2012/10/Screenshot-171.png "Screenshot (17)")

I hope this post will be helpful for some of you.

Happy coding!