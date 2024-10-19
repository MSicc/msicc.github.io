---
id: 3830
title: 'How to update a live tile in a background task with web data on Windows 8.1'
date: '2013-12-02T20:18:11+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-update-a-live-tile-in-a-background-task-with-web-data-on-windows-8-1/
categories:
    - Archive
tags:
    - 'Background Agent'
    - IAsyncOperation
    - 'live tiles'
    - PCL
    - por
    - 'portable class'
    - win8dev
    - 'Windows Runtime'
---

![scheduledtaskWindows](/assets/img/2013/12/scheduledtaskWindows.png "scheduledtaskWindows")

Like I wrote in [my last post](http://msicc.net/?p=3822), I recently finished a PCL project with a Windows Phone and a Windows 8.1 app. Like the Windows Phone version, also the Windows 8.1 app has a live tile that fetches the same data from WordPress.

Taking advantage of our PCL solution structure, we are able to reuse the JSON data class from our PCL.

However, there are a few things that differ from the Windows Phone background agent.

First, we need to add a new project to our solution. In the new project dialog, select ‘Windows Runtime Component’ in the C# Windows Store section.

To make it a background task, implement the interface [IBackgroundTask](http://msdn.microsoft.com/en-us/library/windows/apps/windows.applicationmodel.background.ibackgroundtask.aspx) to the generated class in our Runtime component. In order to make it doing some work, we are going to add the ‘Run’-method that will start our background agent.

Here we are already with the first difference. The Windows Runtime does not support direct async Tasks of type string (I fetch all articles via HttpClient in an async task that returns the JSON string). That’s why we need to wrap it in an [IAsyncOperation](http://msdn.microsoft.com/en-us/library/windows/apps/br206598.aspx). To do this, add a new **sealed** class to your Windows Runtime background agent project.

Here is how the code for the IAyncOperation:

``` csharp
        public IAsyncOperation<string> GetLastPostJsonString(string url)
        {
            try
            {
                return AsyncInfo.Run((System.Threading.CancellationToken ct) => GetInternal(url));
            }
            catch (Exception ex)
            {
                Debug.WriteLine(ex.ToString());
            }
            return null;
        }

        private static async Task<string> GetInternal(string url)
        {
            try
            {
                HttpClient getJsonStringClient = new HttpClient();
                getJsonStringClient.DefaultRequestHeaders.IfModifiedSince = DateTime.Now;

                var response = await getJsonStringClient.GetAsync(url);
                var JsonString = await response.Content.ReadAsStringAsync();
                return JsonString;
            }
            catch (Exception ex)
            {
                Debug.WriteLine(ex.ToString());
            }
            return null;
        }
```
 
Let me explain the code. The IAsyncOperation starts and returns the string of our async Task&lt;string&gt;. To achieve this, we need to start another async [Run](http://msdn.microsoft.com/en-us/library/hh779740(v=vs.110).aspx) that delegates our Task&lt;string&gt; result to the main background task. Unlike a direct HttpClient call, we are not able to save the string directly to an string object. Instead, we need to read the stream.

I highly recommend to wrap this all in try/catch blocks to prevent any crashes of your app.

Let’s go back to our main background task class. Now we are able to fetch the JSON string from web service, we finally can start to implement the Run() method that will actually update our live tiles:

``` csharp
       public async void Run(IBackgroundTaskInstance taskInstance)
       {
           try
           {
               BackgroundTaskDeferral deferral = taskInstance.GetDeferral();

               PostfetcherForTileUpdate fetchJsonString = new PostfetcherForTileUpdate();

               Constants.latestPostsForLiveTileFromWordPressDotComResultString = await fetchJsonString.GetLastPostJsonString(Constants.GetLatestPostsForLiveTileFromWordPressDotComUriString);

               );

               UpdateTile();

               deferral.Complete();
           }
           catch (Exception ex)
           {
               Debug.WriteLine(ex.ToString());
           }
       }
```
 
The Run method needs to implement the Interface for [IBackgroundTaskInstance](http://msdn.microsoft.com/en-us/library/windows/apps/windows.applicationmodel.background.ibackgroundtaskinstance.aspx). To inform the OS that our background agent may do work after the Run has returned data, we need to use the GetDeferral() method. If we would do that, it may happen that the task will be closed before all action has taken place.

After getting the JSON String from the IAsyncOperation, we can now write it to an object, call our UpdateTile() method and tell the system that our deferral is completed.

Within the UpdateTile() method, we are going to deserialize the JSON String and update our live tile. There are a lot of considerations for the live tiles in Windows 8 and 8.1, make sure you have read the [guidelines for tiles and badges](http://msdn.microsoft.com/en-us/library/windows/apps/hh465403.aspx) on MSDN. Make also sure you check the [tile template catalogue](http://msdn.microsoft.com/en-us/library/windows/apps/Hh761491.aspx) to choose the right templates for your app.

Let’s have a look at the UpdateTile() code:

``` csharp
       private static void UpdateTile()
        {
            try
            {
                //create a new Tile updater and allow it to be added to the notification queue
                var updater = TileUpdateManager.CreateTileUpdaterForApplication();
                updater.EnableNotificationQueue(true);
                updater.Clear();

        //deserialize the JSON string
                var LatestPostFromWordpress = JsonConvert.DeserializeObject<json_data_class_Posts.Posts>(Constants.latestPostsForLiveTileFromWordPressDotComResultString);

        //fill in the Tile Templates
                foreach (var item in LatestPostFromWordpress.posts)
                {
            //supporting both Medium and Wide tiles
                    XmlDocument WidetileXML = TileUpdateManager.GetTemplateContent(TileTemplateType.TileWide310x150PeekImage04);
                    XmlDocument SquareTileXML = TileUpdateManager.GetTemplateContent(TileTemplateType.TileSquare150x150PeekImageAndText04);

            //setting the text on our tiles
                    var title = item.title;
                    WidetileXML.GetElementsByTagName("text")[0].InnerText = title;
                    SquareTileXML.GetElementsByTagName("text")[0].InnerText = title;

            //providing the remote image source 
            //if not empty:
                    if (item.featured_image != string.Empty)
                    {
                        XmlNodeList tileImageAttributes = SquareTileXML.GetElementsByTagName("image");
                        ((XmlElement)tileImageAttributes[0]).SetAttribute("src", item.featured_image);
                        ((XmlElement)tileImageAttributes[0]).SetAttribute("alt", "no image");

                        XmlNodeList tileWideImageAttributes = WidetileXML.GetElementsByTagName("image");
                        ((XmlElement)tileWideImageAttributes[0]).SetAttribute("src", item.featured_image);
                        ((XmlElement)tileWideImageAttributes[0]).SetAttribute("alt", "no image");
                    }
            //if empty:
                    else if (item.featured_image == string.Empty)
                    {
                        XmlNodeList tileImageAttributes = WidetileXML.GetElementsByTagName("image");
            //this image has to be in the main Windows 8.1 project, not in the background task!
                        ((XmlElement)tileImageAttributes[0]).SetAttribute("src", "ms-appx:///Images/NoImgPlaceholderMedium.png");
                        ((XmlElement)tileImageAttributes[0]).SetAttribute("alt", "no image");

                        XmlNodeList tileWideImageAttributes = WidetileXML.GetElementsByTagName("image");
            //this image has to be in the main Windows 8.1 project, not in the background task!
                        ((XmlElement)tileWideImageAttributes[0]).SetAttribute("src", "ms-appx:///Images/NoImgPlaceholderWide.png");
                        ((XmlElement)tileWideImageAttributes[0]).SetAttribute("alt", "no image");
                    }

            //perform the update of our live tiles
                    updater.Update(new TileNotification(WidetileXML));
                    updater.Update(new TileNotification(SquareTileXML));
                }
            }
            catch (Exception ex)
            {
                Debug.WriteLine(ex.ToString());
            }
        }
```
 
To make the background task updating our tile, we need to use the [TileUpdateManager class](http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.notifications.tileupdatemanager.aspx). As we are updating our main tile, we are using the CreateTileUpdaterForApplication() method. To make sure our tile is able to queue our update request, we are setting the [EnableNotificationQueue](http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.notifications.tileupdater.enablenotificationqueue.aspx) property to true.

After deserializing the JSON string, we fill the Template by using [DOM methods](http://msdn.microsoft.com/en-us/library/windows/apps/windows.data.xml.dom.aspx). I used the [GetElementsByTagName(string)](http://msdn.microsoft.com/en-us/library/windows/apps/windows.data.xml.dom.xmldocument.getelementsbytagname.aspx) method to find the title and the image ChildNodes in the template. This way, I don’t need to care for the Xml structure.

The title content can be set by using the InnerText property. For providing the images source, we need to go through the NodeList, searching for the image NodeChild and use the [SetAttribute()](http://msdn.microsoft.com/en-us/library/windows/apps/windows.data.xml.dom.xmlelement.setattribute.aspx) method of the XmlElement class.

With the code above, the tile gets a placeholder image, if the JSON string does not return an url for the featured image. Important: the placeholder image has to be in your main Windows 8 project, not in the background task project. Make also sure that the image(s) have a unique name and do not use the same file name like your base tile image.

That’s all we need to do in our background task project.

Let’s have a look at our main app project. First, we have to declare our background task to make our app registering for the periodic notifications. Open the Package.appxmanifest of your app and navigate to the ‘Declarations’ tab.

![Screenshot (259)](/assets/img/2013/12/Screenshot-259.png "Screenshot (259)")

Under ‘Available Declarations’, select ‘Background Tasks’ and ‘Add’.

![Screenshot (259)](/assets/img/2013/12/Screenshot-2591.png "Screenshot (259)")

Under ‘Description’ check the ‘Timer’ Option. Make sure you referenced your background task project and add it as ‘Entry Point’.

That’s all that wee need to set up in the first step. Now we are going to write our method to register the background task:

``` csharp
        //needs to be async because of the BackgroundExecutionManager
        public async void RegisterBackgroundTask()
        {
            try
            {
        // calling the BackgroundExecutionManager
        //this performs the message prompt to the user that allows the update of our tiles and the permissions entry
                var backgroundAccessStatus = await BackgroundExecutionManager.RequestAccessAsync();

        //checking if we have access to set up our live tile
        if (backgroundAccessStatus == BackgroundAccessStatus.AllowedMayUseActiveRealTimeConnectivity ||
                    backgroundAccessStatus == BackgroundAccessStatus.AllowedWithAlwaysOnRealTimeConnectivity)
                {
            //unregistering our old task, if there is one                   
            foreach (var task in BackgroundTaskRegistration.AllTasks)
                    {
                        if (task.Value.Name == taskName)
                        {
                            task.Value.Unregister(true);
                        }
                    }

            //building up our new task and registering it
                    BackgroundTaskBuilder taskBuilder = new BackgroundTaskBuilder();
                    taskBuilder.Name = taskName;
                    taskBuilder.TaskEntryPoint = taskEntryPoint;
                    taskBuilder.SetTrigger(new TimeTrigger(60, false));
                    var registration = taskBuilder.Register();
                }
            }
        //catching all exceptions that can happen
            catch (Exception ex)
            {
        //async method used, but wil be marked by VS to be executed synchronously
                var message = new MessageDialog("we were not able to activate the live tile. Please restart the application to try again.");
                message.Title = "Sorry,";
                message.ShowAsync();

            }

        }
```
 

We are calling the BackgroundExecutionManager of Windows 8.1. This adds the Permissions dialog on the app’s first run as well as the entry in the settings charm. Only if we are allowed by the user, our tile will be updated.

Unregistering already running tasks and re-adding them is a common practice, which I did also here.

After that, I build up a new background task with the BackgroundTaskBuilder class, referencing to the taskEntryPoint I set up in the Declarations tab before.

Unlike the Windows Phone background agent, we can change the time of the background task. I used 60 minutes for this app. You will need to specify a trigger (minimum of 15 minutes). I tried it without, which lead to an exception.

I am catching all exceptions and display a message to the user here. The method will be marked as executing synchronously, but it will work. The reason is that no async methods are allowed in the catch block.

Like always, I added this to my App.xaml.cs file, as this is set up application wide. To start the task, I just call it in the Application.Launched event. It would also work in the OnWindowCreated event, if you want your Launched event free.

Setting up a background task to update your live tile with data from web is not as easy as on Windows Phone, but with this article you will be able to get started on that.

As always, I hope that this post is helpful for some of you.

Happy coding, everyone!