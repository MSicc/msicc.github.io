---
id: 4254
title: 'How to use WordPress to display remote notifications in your Windows Phone app'
date: '2014-11-27T05:10:36+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-use-wordpress-to-display-remote-notifications-in-your-windows-phone-app/
categories:
    - Archive
tags:
    - 'notification service'
    - remote
    - 'Windows Phone'
    - WordPress
    - 'WordPress Universal library'
    - WP8
    - WP8.1
---

Sometimes, we need to display information to our users on demand. The easiest way is to do this in our app via a remote notification service.

If you have a WordPress blog, my solution may also be helpful for you.

I am using a page that is not linked anywhere in my blog to display the message. To add a new page, go to your WordPress Dashboard and hover over “Pages” and click on “Add New”.

![image](/assets/img/2014/11/image.png "image")

Fill in the title and your notification and publish the page. Before closing your browser, save/remember the id of the page, we will need it later.

![image](/assets/img/2014/11/image1.png "image")

The next step is to download my WordPress Universal library, which can be downloaded [right here from my Github](https://github.com/MSiccDev/WordPressUniversal/zipball/master). You can add the project directly to your solution or build it and then reference the library you will find in the bin folder of the WordPress Universal project folder. If you want to learn more about the library, visit <http://bit.ly/WordPressUniversal>.

Now that we have everything in place, let’s add the code that does the magic for us:

``` csharp
         public async void ShowNotification()
        {
            //initialize the client and load the list of pages from your blog
            wpClient = new WordPressClient();
            var pages = await wpClient.GetPostList("msicc.net", WordPressUniversal.Models.PostType.page, WordPressUniversal.Models.PostStatus.publish, 20, 0);

            //select the notification page
            var notificationContentPage = from p in pages.posts_list where p.id == 4248 select p;

            //check if has content
            if (!String.IsNullOrEmpty(notificationContentPage.FirstOrDefault().content))
            {
                //convert parapgraphs into NewLines
                //you might have more HTML content in there which needs to be converted
                string content = notificationContentPage.FirstOrDefault().content.Replace("<p>", string.Empty).Replace("</p>", "\n\n");

                //App.SettingsStore = ApplicationData.Current.LocalSettings
                //change this to your appropriate storage like IsolatedStorage etc.
                //this displays the message only once to our users, but keeps the door open for an easy update mechanism
                if (App.SettingsStore.LastNotificationContent != content)
                {
                    MessageBoxResult result = MessageBox.Show(content, notificationContentPage.FirstOrDefault().title, MessageBoxButton.OK);
                    switch (result)
                    {
                        //the button click saves the actual message
                        case MessageBoxResult.OK:
                            App.SettingsStore.LastNotificationContent = content;
                            break;
                        //BackButtonPress does this as well
                        case MessageBoxResult.None:
                            App.SettingsStore.LastNotificationContent = content;
                            break;
                    }
                }
            }
        }
```
 
What does this code do? First, it fetches all pages from our WordPress blog. Then, we are selecting the page we created via its id. If your WordPress blog does not show you the id in the url of the page, set a BreakPoint at the “var notificationContentPage = …” line. you will then easily be able to get the id:

![image](/assets/img/2014/11/image2.png "image")

Naturally, the returned content is HTML formatted. To remove the paragraph tags and but respect their function, we are using a simple String.Replace pattern. You may have more HTML tags in your message that needs to be removed or converted.

To generate an easy way to display the message only once but keep it open for updates, we are saving the converted message locally. In this case, I used the LocalSettings of my Windows Phone 8.1 app. I am using the MessageBoxResult to make the method saving the message either at the point of the OK click as well as on BackButtonPress.

This is how the above generated WordPress Page looks as a Notification:

![wp_ss_20141127_0001](/assets/img/2014/11/wp_ss_20141127_0001.png "wp_ss_20141127_0001")

As my WordPress Universal library works cross platform for C# apps, you should be able to adapt this for your Windows 8.1 or Xamarin apps as well.

As always, I hope this is helpful for some of you.

Happy coding!