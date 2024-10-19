---
id: 3991
title: 'Detect and remove Emojis from Text on Windows Phone'
date: '2014-03-13T20:15:38+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /detect-and-remove-emoticons-from-text-on-windows-phone/
categories:
    - Archive
tags:
    - API
    - Emojis
    - Regex
    - remove
    - REST
    - 'string maniputilation'
    - strings
    - 'unsupporterd characters'
    - 'unwanted characters'
    - 'web service'
---

![noemoticonskeyboard](/assets/img/2014/03/noemoticonskeyboard.png "noemoticonskeyboard")

In my recent project, I work with a lot of text that get’s its value from user input. To enable the best possible experience for my users, I choose the InputScope Text on all TextBoxes, because it provides the word suggestions while writing.

The Text will be submitted to a webserver via a REST API. And now the problem starts. The emojis that are part of the Windows Phone OS are not supported by the API and the webserver.

Of course, I was immediately looking for a way to get around this. I thought this might be helpful for some of you, so I am sharing two little helper methods for detecting and removing the emojis .

After publishing the first version of this post, I got some feedback that made me investigating a bit more on this topic. The emojis are so called unicode symbols, and thanks to the Unicode behind it, they are compatible with all platforms that have the matching Unicode list implemented.

Windows Phone has a subset of all available Unicode characters in the OS keyboard, coming from different ranges in the Unicode characters charts. Like we have to do sometimes, we have to maintain our own list to make sure that all emojis are covered by our app in this case and update our code if needed.

If you want to learn more about unicode charts, they are officially available here: [http://www.unicode.org/charts/](http://www.unicode.org/charts/ "http://www.unicode.org/charts/")

Update 2: I am using this methods in a real world app. Although the underlying Unicode can be used, often normal text will be read as emoji. That’s why I reverted back to my initial version with the emojis in it. I never had any problems with that.

Now let’s have a look at the code I am using. First is my detecting method, that returns a bool after checking the text (input):

``` csharp
              public static bool HasUnsoppertedCharacter(string text)
             {
            string pattern = @"[{allemojisshere}]";

            Regex RegexEmojisKeyboard = new Regex(pattern);

            bool booleanreturnvalue = false;

            if (RegexEmojisKeyboard.IsMatch(text))
            {
                booleanreturnvalue = true;
            }
            else if (!RegexEmojisKeyboard.IsMatch(text))
            {
                booleanreturnvalue = false;
            }
            return booleanreturnvalue;
            }
```
 
As you can see, I declared a character range with all emojis. If one or more emojis is found, the bool will always return true. This can be used to display a MessageBox for example while the user is typing.

The second method removes the emojis from any text that is passed as input string.

``` csharp
              public static string RemovedUnSoppertedCharacterString(string text)
             {
            string result = string.Empty;
            string cleanedResult = string.Empty;

            string pattern = @"[{allemojishere}]";

            MatchCollection matches = Regex.Matches(text, pattern);

            foreach (Match match in matches)
            {
                result = Regex.Replace(text, pattern, string.Empty);
                cleanedResult = Regex.Replace(result, "  ", " ");
            }
            return cleanedResult;
             }
```
 
Also here I am using the character range with all emojis . The method writes all occurrences of emojis into a MatchCollection for Regex. I iterate trough this collection to remove all of them. The Method also checks the string for double spaces in the text and makes it a single space, as this happens while removing the emojis .

User Experience hint:

use this method with care, as it could be seen as a data loss from your users. I am using the first method to display a MessageBox to the user that emojis are not supported and that they will be removed, which I am doing with the second method. This way, my users are informed and they don’t need to do anything to correct that.

You might have noticed that there is a placeholder “{allemojishere}” in the code above. WordPress or the code plugin I use aren’t supporting the Emoticons in code, that’s why I attached [ my helper class](/assets/img/2014/03/Emoji-Helper.zip).

As always, I hope this will be helpful for some of you.

Happy coding!