---
id: 4310
title: 'Helper class to easily display local toast notifications (Windows Universal app)'
date: '2015-02-26T04:23:09+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /helper-class-to-easily-display-local-toast-notifications-windows-universal-app/
categories:
    - Archive
tags:
    - 'document object model'
    - DOM
    - formatting
    - helper
    - 'local toasts'
    - mvvm
    - notifications
    - PCL
    - 'shared project'
    - 'universal app'
    - ViewModel
    - 'Windows 8'
    - 'Windows Phone'
    - xml
---

Often , we need to display a confirmation that some action in our app has been finished (like some data has been updated etc.). There are several ways of doing this, like displaying a MessageBox or MessageDialog. This however breaks the user interaction, and a lot of users will start complaining on that if your app keeps doing so. There needs to be a better way.

With the [Coding4fun Toolkit](https://github.com/Coding4FunProjects/Coding4FunToolkit) floating around, you can mimic a toast notification – sadly only on Windows Phone (at least for the moment, but [Dave](https://twitter.com/hermitdave) told me he will work on implementing it for Windows, too). Also, [Toastinet](http://toastinet.codeplex.com/) library is floating around, which is also able to mimic the toast notification behavior (although for Windows Universal app, the implementation is not that intuitive as for Windows Phone). Both are fantastic libraries that I used in the past, but I wanted a solution that is implemented easily and works with my Universal app. So I did some searching in the Web and the MSDN docs, and found out that is pretty easy to use the system toast notifications on both platforms locally.

There are 8 possible ways to format toast notifications (as you can see here in the [toast template catalog](https://msdn.microsoft.com/en-us/library/windows/apps/hh761494.aspx)). This gives us pretty much options on how a notification can be styled. However, most options just work on Windows 8.1, while Windows Phone 8.1 apps will only show the notification in the way “app logo” “bold text” “normal text”. However, the notification system takes care of that, so you can specify some other type on Windows 8.1, while knowing that it gets converted on Windows Phone automatically. This allows us to write a helper class that implements all possible options without any headache.

#### the code parts for the notification

Let’s have a look at the code parts for the notification. First, you need to add two Namespaces to the class:

``` csharp
 using Windows.Data.Xml.Dom;
using Windows.UI.Notifications;
```
 
After that, we can start writing our code. Toast notifications are formatted using Xml. Because of this, we need to get a reference to the underlying Xml template for the system toast notification:

``` csharp
 ToastTemplateType xmlForToast= ToastTemplateType.ToastImageAndText01; 
XmlDocument toastXml = ToastNotificationManager.GetTemplateContent(xmlForToast);
```
 
System toast notifications can hold Text (and an Image on Windows 8.1). So we need to declare the elements of the toast notification. We are using the Xml methods of the DOM namespace to get the text elements of the chosen template first:

``` csharp 
xmlNodeList toastTextElements = xmlForToast.GetElementsByTagName("text");
toastTextElements[0].AppendChild(xmlForToast.CreateTextNode("text1"));
//additional texts, depending on the template:
//toastTextElements[1].AppendChild(xmlForToast.CreateTextNode("text2"));
//toastTextElements[2].AppendChild(xmlForToast.CreateTextNode("text3"));
```
 
This is how the image element is implemented:

``` csharp
xmlNodeList toastImageElement = xmlForToast.GetElementsByTagName("image");
//setting the image source uri:
if (toastImageElement != null) ((XmlElement) toastImageElement[0]).SetAttribute("src", imageSourceUri);
//setting optional alternative text for the image
if (toastImageElement != null)  ((XmlElement) toastImageElement[0]).SetAttribute("alt", imageSourceAlternativeText);
```
 
You can attach local or remote images to the toast notification, but remember this works only on Windows, not on Windows Phone.

The next part we are able to set is the duration. The duration options are long (25 seconds) and short (7 seconds). The default is short, which should be ok for most scenarios. Microsoft recommends to use long only when a personal interaction of the user is needed (like in a chat). This is how we do it:

``` csharp
 IXmlNode toastRoot = xmlForToast.SelectSingleNode("/toast");
((XmlElement) toastRoot).SetAttribute("duration", "short");
```
 
What we are doing here is to get the root element of the template’s Xml and add a new element for the duration. Now that we finally have set all options, we are able to create our toast notification and display it to the user:

``` csharp
 ToastNotification notification = new ToastNotification(xmlForToast);
ToastNotificationManager.CreateToastNotifier().Show(notification);
```
 
#### the helper class

That’s all we need to do for our local notification. You might see that always rewriting the same code just makes a lot of work. Because the code for the toast notification can be called nearly everywhere in an app (it does not matter if you are calling it from a ViewModel or code behind), I wrote this helper class that makes it even more easy to use the system toast notification locally:

``` csharp
     public class LocalToastHelper
    {
        public void ShowLocalToast(ToastTemplateType templateType, string toastText01, string toastText02 = null, string toastText03 = null, string imageSourceUri = null, string imageSourceAlternativeText = null, ToastDuration duration = ToastDuration.Short)
        {
            XmlDocument xmlForToast = ToastNotificationManager.GetTemplateContent(templateType);
            XmlNodeList toastTextElements = xmlForToast.GetElementsByTagName("text");

            switch (templateType)
            {
                case ToastTemplateType.ToastText01:
                case ToastTemplateType.ToastImageAndText01:
                    toastTextElements[0].AppendChild(xmlForToast.CreateTextNode(toastText01));
                    break;
                case ToastTemplateType.ToastText02:
                case ToastTemplateType.ToastImageAndText02:
                    toastTextElements[0].AppendChild(xmlForToast.CreateTextNode(toastText01));
                    if (toastText02 != null)
                    {
                        toastTextElements[1].AppendChild(xmlForToast.CreateTextNode(toastText02));
                    }
                    else
                    {
                        throw new ArgumentNullException("toastText02 must not be null when using this template type");
                        
                    }
                    ;
                    break;
                case ToastTemplateType.ToastText03:
                case ToastTemplateType.ToastImageAndText03:
                    toastTextElements[0].AppendChild(xmlForToast.CreateTextNode(toastText01));
                    if (toastText02 != null)
                    {
                        toastTextElements[1].AppendChild(xmlForToast.CreateTextNode(toastText02));
                    }
                    else
                    {
                        throw new ArgumentNullException("toastText02 must not be null when using this template type");
                    }
                    ;
                    break;
                case ToastTemplateType.ToastText04:
                case ToastTemplateType.ToastImageAndText04:
                    toastTextElements[0].AppendChild(xmlForToast.CreateTextNode(toastText01));
                    if (toastText02 != null)
                    {
                        toastTextElements[1].AppendChild(xmlForToast.CreateTextNode(toastText02));
                    }
                    else
                    {
                        throw new ArgumentNullException("toastText02 must not be null when using this template type");
                    }
                    ;
                    if (toastText03 != null)
                    {
                        toastTextElements[2].AppendChild(xmlForToast.CreateTextNode(toastText03));
                    }
                    else
                    {
                        throw new ArgumentNullException("toastText03 must not be null when using this template type");
                    }
                    ;
                    break;
            }

            switch (templateType)
            {
                case ToastTemplateType.ToastImageAndText01:
                case ToastTemplateType.ToastImageAndText02:
                case ToastTemplateType.ToastImageAndText03:
                case ToastTemplateType.ToastImageAndText04:
                    if (!string.IsNullOrEmpty(imageSourceUri))
                    {
                        XmlNodeList toastImageElement = xmlForToast.GetElementsByTagName("image");
                        if (toastImageElement != null)
                            ((XmlElement) toastImageElement[0]).SetAttribute("src", imageSourceUri);
                    }
                    else
                    {
                        throw new ArgumentNullException(
                            "imageSourceUri must not be null when using this template type");
                    }
                    if (!string.IsNullOrEmpty(imageSourceUri) && !string.IsNullOrEmpty(imageSourceAlternativeText))
                    {
                        XmlNodeList toastImageElement = xmlForToast.GetElementsByTagName("image");
                        if (toastImageElement != null)
                            ((XmlElement) toastImageElement[0]).SetAttribute("alt", imageSourceAlternativeText);
                    }
                    break;
                default:
                    break;
            }

            IXmlNode toastRoot = xmlForToast.SelectSingleNode("/toast");
            ((XmlElement) toastRoot).SetAttribute("duration", duration.ToString().ToLowerInvariant());

            ToastNotification notification = new ToastNotification(xmlForToast);
            ToastNotificationManager.CreateToastNotifier().Show(notification);
        }

        public enum ToastDuration
        {
            Short,
            Long
        }
    }
```
 
As you can see, you just need to provide the wanted parameters to the ShowLocalToast method, which will do the rest of the work for you.

One word to the second switch statement I am using. The image element needs to be set only when we are using the ToastImageAndTextXX templates. There are three ways to implement the integration: using an if with 4 “or” options, the switch statement I am using or a string comparison with String.Contains. The switch statement is the cleanest option for me, so I decided to go this way. Feel free to use any of the other ways in your implementation.

In my implementation, I added also some possible ArgumentNullExceptions to make it easy to find any usage errors.

For your convenience, I attached the source file. Just swap out the namespace with yours. [Download](/assets/img/2015/02/LocalToastHelper.zip)

The usage of the class is pretty simple:

``` csharp
 var _toastHelper = new LocalToastHelper();
_toastHelper.ShowLocalToast(ToastTemplateType.ToastText02, "This is text 1", "This is text 2");
```
 
#### audio options

The system toasts have another option that can be set: the toast audio. This way, you can customize the appearance of the toast a bit more. I did not implement it yet, because there are some more options and things to remind, and I haven’t checked them out all together. Once I did, I will add a second post to this one with the new information.

As always, I hope this is helpful for some of you.

Happy coding!