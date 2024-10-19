---
id: 4063
title: 'How to detect all urls in a string to match Twitter&rsquo;s requirements (Windows 8(.1) and Windows Phone 8(.1))'
date: '2014-04-22T11:55:02+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-convert-all-urls-in-a-string-to-match-twitters-requirements-windows-8-1-and-windows-phone-8-1/
categories:
    - Archive
tags:
    - 'multiple urls'
    - oAuth
    - Regex
    - t.co
    - twitter
    - 'url handling'
    - 'url shortener'
    - 'Windows Phone Runtime'
    - WINPRT
    - WP8.1
    - wpdev
---

![url](/assets/img/2014/04/url.jpg "url")

As you might guess, I am still working on that app I mentioned in my last blog post. As I was diving deeper into the functions I want, I recognized that Twitter does a very well url handling server side.

Like the official documentation says, every url will be shortened with a link that has 22 characters (23 for https urls).

I was trying to write a RegEx expression to detect all the links that can be:

- http
- https
- just plain domain names like “msicc.net”

This is not as easy as it sounds, and so I was a bit struggling. I then talked with [@\_MadMatt (follow him!)](https://twitter.com/MadMatt) who has a lot of experience with twitter. My first attempt was a bit confusing as I did first only select http and https, then the plain domain names.

I found the names by their domain ending, but had some problems to get their length (which is essential). After the very helpful talk with Matthieu, I finally found a very good working RegEx expression [here on GitHub](https://gist.github.com/gruber/8891611).

I tested it with tons of links, and I got the desired results and it is now also very easy for me to get their length.

Recovering the amount of time I needed for this, I decided to share my solution with you. Here is the method I wrote:

``` csharp
         public int CalculateTweetCountWithLinks(int currentCount, string text)
        {
            int resultCount = 0;

            if (text != string.Empty)
            {
                //detailed explanation: https://gist.github.com/gruber/8891611
                string pattern = @"(?i)\b((?:https?:(?:/{1,3}|[a-z0-9%])|[a-z0-9.\-]+[.](?:com|net|org|edu|gov|mil|aero|asia|biz|cat|coop|info|int|jobs|mobi|museum|name|post|pro|tel|travel|xxx|ac|ad|ae|af|ag|ai|al|am|an|ao|aq|ar|as|at|au|aw|ax|az|ba|bb|bd|be|bf|bg|bh|bi|bj|bm|bn|bo|br|bs|bt|bv|bw|by|bz|ca|cc|cd|cf|cg|ch|ci|ck|cl|cm|cn|co|cr|cs|cu|cv|cx|cy|cz|dd|de|dj|dk|dm|do|dz|ec|ee|eg|eh|er|es|et|eu|fi|fj|fk|fm|fo|fr|ga|gb|gd|ge|gf|gg|gh|gi|gl|gm|gn|gp|gq|gr|gs|gt|gu|gw|gy|hk|hm|hn|hr|ht|hu|id|ie|il|im|in|io|iq|ir|is|it|je|jm|jo|jp|ke|kg|kh|ki|km|kn|kp|kr|kw|ky|kz|la|lb|lc|li|lk|lr|ls|lt|lu|lv|ly|ma|mc|md|me|mg|mh|mk|ml|mm|mn|mo|mp|mq|mr|ms|mt|mu|mv|mw|mx|my|mz|na|nc|ne|nf|ng|ni|nl|no|np|nr|nu|nz|om|pa|pe|pf|pg|ph|pk|pl|pm|pn|pr|ps|pt|pw|py|qa|re|ro|rs|ru|rw|sa|sb|sc|sd|se|sg|sh|si|sj|Ja|sk|sl|sm|sn|so|sr|ss|st|su|sv|sx|sy|sz|tc|td|tf|tg|th|tj|tk|tl|tm|tn|to|tp|tr|tt|tv|tw|tz|ua|ug|uk|us|uy|uz|va|vc|ve|vg|vi|vn|vu|wf|ws|ye|yt|yu|za|zm|zw)/)(?:[^\s()<>{}\[\]]+|\([^\s()]*?\([^\s()]+\)[^\s()]*?\)|\([^\s]+?\))+(?:\([^\s()]*?\([^\s()]+\)[^\s()]*?\)|\([^\s]+?\)|[^\s`!()\[\]{};:'.,<>?«»“”‘’])|(?:(?<!@)[a-z0-9]+(?:[.\-][a-z0-9]+)*[.](?:com|net|org|edu|gov|mil|aero|asia|biz|cat|coop|info|int|jobs|mobi|museum|name|post|pro|tel|travel|xxx|ac|ad|ae|af|ag|ai|al|am|an|ao|aq|ar|as|at|au|aw|ax|az|ba|bb|bd|be|bf|bg|bh|bi|bj|bm|bn|bo|br|bs|bt|bv|bw|by|bz|ca|cc|cd|cf|cg|ch|ci|ck|cl|cm|cn|co|cr|cs|cu|cv|cx|cy|cz|dd|de|dj|dk|dm|do|dz|ec|ee|eg|eh|er|es|et|eu|fi|fj|fk|fm|fo|fr|ga|gb|gd|ge|gf|gg|gh|gi|gl|gm|gn|gp|gq|gr|gs|gt|gu|gw|gy|hk|hm|hn|hr|ht|hu|id|ie|il|im|in|io|iq|ir|is|it|je|jm|jo|jp|ke|kg|kh|ki|km|kn|kp|kr|kw|ky|kz|la|lb|lc|li|lk|lr|ls|lt|lu|lv|ly|ma|mc|md|me|mg|mh|mk|ml|mm|mn|mo|mp|mq|mr|ms|mt|mu|mv|mw|mx|my|mz|na|nc|ne|nf|ng|ni|nl|no|np|nr|nu|nz|om|pa|pe|pf|pg|ph|pk|pl|pm|pn|pr|ps|pt|pw|py|qa|re|ro|rs|ru|rw|sa|sb|sc|sd|se|sg|sh|si|sj|Ja|sk|sl|sm|sn|so|sr|ss|st|su|sv|sx|sy|sz|tc|td|tf|tg|th|tj|tk|tl|tm|tn|to|tp|tr|tt|tv|tw|tz|ua|ug|uk|us|uy|uz|va|vc|ve|vg|vi|vn|vu|wf|ws|ye|yt|yu|za|zm|zw)\b/?(?!@)))";

                //generating a MatchCollection
                MatchCollection linksInText = Regex.Matches(text, pattern, RegexOptions.Multiline);

                //going forward only when links where found
                if (linksInText.Count != 0)
                {
                    //important to set them to 0 to get the correct count
                    int linkValueLength = 0;
                    int httpOrNoneCount = 0;
                    int httpsCount = 0;

                    foreach (Match m in linksInText)
                    {
                        //https urls need 23 characters, http and others 22
                        if (m.Value.Contains("https://"))
                        {
                            httpsCount = httpsCount + 1;
                        }
                        else
                        {
                            httpOrNoneCount = httpOrNoneCount + 1;
                        }

                        linkValueLength = linkValueLength + m.Value.Length;
                    }

                    //generating summaries of character counts
                    int httpOrNoneReplacedValueLength = httpOrNoneCount * 22;
                    int httpsReplacedValueLength = httpsCount * 23;

                    //calculating final count
                    resultCount = (currentCount - linkValueLength) + (httpOrNoneReplacedValueLength + httpsReplacedValueLength);                    
                }
                else
                {
                    resultCount = currentCount;
                }
            }
            return resultCount;
        }
```
 
First, we are detecting links in the string using the above mentioned RegEx expression and collect them in a [MatchCollection](http://msdn.microsoft.com/en-us/library/system.text.regularexpressions.matchcollection(v=vs.110).aspx).

As https urls have a 23 character length on t.co (Twitter’s url shortener), I am generating two new counts – one for https, one for all other urls.

The last step is to substract the the length of all Match values and add the newly calculated replaced link values lengths.

Add this little method to your TextChanged event, and you will be able to detect the character count on the fly.

As always, I hope this is helpful for some of you.

Happy coding, everyone!