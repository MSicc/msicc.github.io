---
id: 3834
title: '[Updated] Why Windows Phone apps deserve a custom uri scheme (and how to add a simple launch uri)'
date: '2013-12-08T17:00:07+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /why-windows-phone-apps-deserve-a-custom-uri-scheme-and-how-to-add-a-simple-launch-uri/
categories:
    - Archive
tags:
    - apps
    - 'custom uri association'
    - 'custom uri scheme'
    - launch
    - launcher
    - launchUriAsync
    - 'Windows Phone'
    - WinPhanDev
    - wpdev
---

![app2app_Windows_Phone](/assets/img/2013/12/app2app_Windows_Phone.png "app2app_Windows_Phone")

Microsoft introduced custom uri schemes or uri associations in Windows Phone 8 along with the ability to launch apps based on a file type.

Sadly, this feature is not used across a broad range of apps yet. Why am I so interested in that point? I am going to explain you.

First, there is one big issue: You can share any app via NFC, and launch any app from the Store via NFC tag that has a launch app record. But you cannot launch any apps from your app – just those that have implemented a custom uri scheme. On the other two big operating systems, sharing between apps is meanwhile a standard feature used by a lot of developers.

Most of the settings pages have one of those uri schemes, for example the Wi-Fi settings page can be launched with this simple line of code:

``` csharp
 await Windows.System.Launcher.LaunchUriAsync(new Uri("ms-settings-wifi:"));
```
 
I don’t know if other developers just don’t know about this feature or if they don’t think it is necessary. I talked to a lot of users of my app NFC Toolkit for Windows Phone 8, and all asked me to add the possibility to launch a desired app at the end of my profiles (if you don’t know, my profiles launch programmable sequence of settings pages).

Searching for a solution to satisfy my user, I spent a few days with researching of all possibilities and talking to other developers. Some of them wanted use cases for adding that feature, here are three:

- Every day I leave for work, I disable Wi-Fi, activate Bluetooth and switch the cellular connection from 2G to 3G/4G. I often use Nokia Music to listen to music while driving to work (about 45 minutes). This is one of the use cases.
- The second scenario should be also familiar to some of you: I switch off all data connections because otherwise, my phone would ring very often during the night. In the morning, I switch them back on and check for news with a feed reader app. This would fit perfectly into a profile that I save on a NFC tag or launch via secondary tile.
- Third one: there are a lot of blog reader apps out there. Sure, Windows Phone has social networks built in – but only Facebook (user only), Twitter and LinkedIn. What if I want to share to another network, for example [Geeklist](https://geekli.st/msicc)? Then I have to copy and paste instead of just hitting the share button.

There are more use cases, but I am leaving you with these three for the moment.

Back to the conversations I had with my users. They want and deserve a great user experience. This blog post is part of my efforts to provide users their deserved user experience.

Sadly, for a handful of developers it is not as easy to gain some attention for this. This is why I am trying to get DVLUP on board. By creating a challenge for this, more developers would join to earn the XP. The winners in this case are the users out there, because this way, their user experience will be improved. I know this should be done by Microsoft. In the meantime, it is up to us developers to improve. That’s why I ask you to vote for my idea case on DVLUP here: [http://www.dvlup.com/Feedback?query=custom+uri+scheme+to+all+apps%21#](http://www.dvlup.com/Feedback?query=custom+uri+scheme+to+all+apps%21# "http://www.dvlup.com/Feedback?query=custom+uri+scheme+to+all+apps%21#") (the first entry in the list is the one to vote).

There is one reason left why you should add a custom uri scheme to your app. This point is for you, the developer reading this article. You have a free possibility to promote your app across other apps. The most important point: it takes only 5 minutes to add a simple launch uri scheme.

Do you already have a custom uri scheme for your app? Great, then add it to this list: [URI Association Schemes List – Nokia Developer Wiki](http://developer.nokia.com/Community/Wiki/URI_Association_Schemes_List "URI Association Schemes List - Nokia Developer Wiki"). This way, other developers can use them to interact with your app and bring you new users, too.

To close this article, I want to show you how to add a simple custom uri launch scheme. I have done this with my app Mix Play &amp; Share recently (update submitted, will add it to the list above as soon as it certified).

### A simple launch uri – the code

First, open your WMAppManifest.xml by right-clicking on it and Open with… =&gt;XML (Text) Editor.

After the &lt;/Tokens&gt; Element, add the following code:

``` xml
 <Extensions>
     <Protocol Name="your-custom-uri-scheme-here" NavUriFragment="encodedLaunchUri=%s" TaskID="_default" />
</Extensions>
```
 
Save and close the Document.

Add a new class to your project. Add the following code in your class:

``` csharp
    class UriSchemeMapper : UriMapperBase
    {
        private string tempUri;

        public override Uri MapUri(Uri uri)
        {
            tempUri = System.Net.HttpUtility.UrlDecode(uri.ToString());

            // updated code begins here:
            if (tempUri.Contains("your-custom-uri-scheme-here"))
            {
                return new Uri("/MainPage.xaml", UriKind.Relative);
            }
            //updated code ends here
            return uri;
        }
    }
```
 
To make your app using this custom uri scheme, you just have to add another line of code after the declaration of your RootFrame in App.xaml.cs:

``` csharp
 //Handle custom uri scheme
RootFrame.UriMapper = new UriSchemeMapper();
```
 
That’s all, your app now is able to be launched by other apps!

You can also add more advanced custom uri schemes (up to 10 per app), to read more about it, check MSDN: [http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206987(v=vs.105).aspx#BKMK\_URIassociations](http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206987(v=vs.105).aspx#BKMK_URIassociations "http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206987(v=vs.105).aspx#BKMK_URIassociations")

I hope after reading this post, you understand why I think it is important to add a custom uri scheme. With the simple code above, you’re done in 5 minutes.

**Update 12/14/2013:**

I needed to update the UriMapper class above, because without handling a scheme without parameters we will have no guarantee that the app launches. In my tests it works sometimes and sometimes not, but to make sure your app launches, please handle this case properly.

Until then, happy coding!