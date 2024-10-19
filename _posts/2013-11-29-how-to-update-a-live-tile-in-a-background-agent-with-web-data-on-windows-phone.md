---
id: 3822
title: 'How to update a live tile in a background agent with web data on Windows Phone'
date: '2013-11-29T22:32:23+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-update-a-live-tile-in-a-background-agent-with-web-data-on-windows-phone/
categories:
    - Archive
tags:
    - 'Background Agent'
    - blog
    - 'live tiles'
    - PCL
    - 'portable class'
    - 'portable class library'
    - 'portable code'
    - telerik
    - 'web service'
    - 'Windows Phone'
    - wpdev
---

![scheduledtaskWP](/assets/img/2013/11/scheduledtaskWP.png "scheduledtaskWP")

In one of my recent projects, I needed to update the live tile of the app via a background agent with data from a WordPress based web site. I did what I always do when implementing a feature I never used before: researching.

I found tons of example on how to update the live tile periodically, but non of them told me how I can use the data from the web site or where I need to put it in.

In the end, it was [Jay Bennet](https://twitter.com/JayTBennett), developer of the fantatsic [WPcentral](http://wpcentral.com) Windows Phone app, who gave me the last hint I needed – where do I start the request for the data I need. Thanks to him for that!

Ok, but let’s start in the beginning. When it comes to web based services, you first need a class (or ViewModel) that can hold your data you receive from your service. I explained that already pretty well [here](http://msicc.net/?p=3398). No matter if you are running your request within your app project or in a PCL, it pretty much always works like this.

After stripping of our class out of the JSON string, we are now able to create our request as well as our Background Agent.

The first thing you need to do is to add a Windows Phone Scheduled Task Agent. Create a new project within your app and choose the project type mentioned before. Then, in your main project, add it as a reference (right click on References/Add Reference/Solution =&gt; select your background agent project there. That’s it.

No go to your ‘ScheduledAgent.cs’ file and open it. You will find this code in there:

``` csharp 
protected override void OnInvoke(ScheduledTask task)
{
  //TODO: Add code to perform your task in background

  NotifyComplete();
}
```
 
And thanks to Jay, I know that this is where all the action (not only updating the tile like in all samples I found) happens. This may sound very trivial for those of you who have experience with that, but if you’re new to it, it may hold you back a bit. However, there are a few points you’ll need to take care of:

- you only have 25 seconds to perform all action here
- your task will run every 30 minutes, no way to change that
- scheduled tasks need to be restarted within 14 days after the current start
- Battery saver and the OS/the user can deactivate the agent
- there is a long list of what you are not able to do in here ([check MSDN](http://msdn.microsoft.com/en-us/library/windowsphone/develop/hh202962(v=vs.105).aspx))
- there are memory limits ([check MSDN](http://msdn.microsoft.com/en-us/library/windowsphone/develop/hh202942(v=vs.105).aspx#BKMK_ConstraintsforallScheduledTaskTypes))


all action needs to be finished before NotifyComplete() is called

Based on this information, I created my task as following:

``` csharp
        protected async override void OnInvoke(ScheduledTask task)
        {
        //performing an async request to get the JSON data string        
            PostsFetcher postfetcher = new PostsFetcher();
            Constants.latestPostsForLiveTileFromWordPressDotComResultString = await postfetcher.GetLatestPostFromWordPressDotComForLiveTileOnPhone();

        //deserialize all data
            var lifetileBaseData = JsonConvert.DeserializeObject<json_data_class_Posts.Posts>(Constants.latestPostsForLiveTileFromWordPressDotComResultString);

            Deployment.Current.Dispatcher.BeginInvoke(() =>
            {
        //using Telerik's LiveTileHelper here
                Uri launchUri = new Uri("Mainpage.xaml", UriKind.Relative);
                RadFlipTileData fliptileData = new RadFlipTileData();
                fliptileData.BackContent = lifetileBaseData.posts[0].title;
                fliptileData.WideBackContent = lifetileBaseData.posts[0].title;
                fliptileData.BackTitle = string.Format("{0} {1}", DateTime.Now.ToShortDateString(), DateTime.Now.ToShortTimeString());
                fliptileData.Title = "LiveTileTitle";               

                if (lifetileBaseData.posts[0].featured_image != string.Empty)
                {
                    fliptileData.BackgroundImage = new Uri(lifetileBaseData.posts[0].featured_image, UriKind.RelativeOrAbsolute);
                    fliptileData.WideBackgroundImage = new Uri(lifetileBaseData.posts[0].featured_image, UriKind.RelativeOrAbsolute);
                    fliptileData.BackBackgroundImage = new Uri("Images/BackBackground.png", UriKind.RelativeOrAbsolute);
                    fliptileData.WideBackBackgroundImage = new Uri("Images/WideBackBackground.png", UriKind.RelativeOrAbsolute);
                }
                else
                {
                    fliptileData.BackgroundImage = new Uri("Images/PlaceholderBackgroundImage.png", UriKind.RelativeOrAbsolute);
                    fliptileData.WideBackgroundImage = new Uri("Images/PlaceholderWideBackgroundImage.png", UriKind.RelativeOrAbsolute);
                    fliptileData.BackBackgroundImage = new Uri("Images/BackBackground.png", UriKind.RelativeOrAbsolute);
                    fliptileData.WideBackBackgroundImage = new Uri("Images/WideBackBackground.png", UriKind.RelativeOrAbsolute);
                }

                foreach (ShellTile tile in ShellTile.ActiveTiles)
                {
                    LiveTileHelper.UpdateTile(tile, fliptileData);
                }
            });

            NotifyComplete();
        }
```
 

As I fetch different data from the site, I create a class that holds all request methods. In those methods, I just created an HttpClient that downloads the desired Json string into my app. I take only the first post in the case above, to make the live tile updating also on slow internet connections within the 25 seconds and to not reach any memory limit. In the end, I use Telerik’s LiveTileHelper to create a FlipTile with image and text from the site (the image will be downloaded automatically).

That’s all we need to to in the Scheduled Task Agent. Now we need to implement the agent into our app project.

First thing we should implement is a switch where the user can turn our agent on and off. I used a ToggleSwitch for that, saving the isChecked state as a Boolean to the IsolatedStorage of my app.

Knowing I need to restart the background agent after some time, I implemented the agent handling in App.xaml.cs. This way, I need to write less code as I tend to use separate settings pages. The only thing you need to think of is to set up all objects as static and public.

First, we need to declare a PeriodicTask as well as a name for it:

``` csharp
public static PeriodicTask LiveTileUpdaterPeriodicTask; 
public static string LiveTileUpdaterPeriodicTaskNameString = "LiveTileUpdaterPeriodicTaskAgent";
```
 
Now we need to generate a method that handles everything for our background task:

``` csharp
       public static void StartLiveTileUpdaterPeriodicTaskAgent()
        {
        //declare the task and find the already running agent
            LiveTileUpdaterPeriodicTask = ScheduledActionService.Find(LiveTileUpdaterPeriodicTaskNameString) as PeriodicTask;

            if (LiveTileUpdaterPeriodicTask != null)
            {
        //separate method, because we need to stop the agent when the user switches the Toggle to 'Off'
                StopLiveTileUpdaterPeriodicTaskAgent();
        //contains:
        //try
                //{
                //  ScheduledActionService.Remove(App.LiveTileUpdaterPeriodicTaskNameString);
                //}
                //catch { }
            }

        //generate a new background task 
            LiveTileUpdaterPeriodicTask = new PeriodicTask(LiveTileUpdaterPeriodicTaskNameString);

        //provide a description. if not, your agent and your app may crash without even noticing you while debugging
            LiveTileUpdaterPeriodicTask.Description = "This background agent checks every 30 minutes if there is a new blog post.";

        //start the agent and error handling
            try
            {
                ScheduledActionService.Add(LiveTileUpdaterPeriodicTask);
            }
        catch (InvalidOperationException exception)
            {
        //user deactivated or blocked the agent in phone settings/background tasks. Ask him to re-activate or unblock it
                if (exception.Message.Contains("BNS Error: The action is disabled"))
                {
                    RadMessageBox.ShowAsync("it seems you deactivated our Background Agent for the Live Tiles. Please go to settings/background tasks to activate our app again.", "Whoops!", MessageBoxButtons.OK);
                }

        //the maximum of running background agents is reached. No further notification to the user required, as this is handled by the OS
                if (exception.Message.Contains("BNS Error: The maximum number of ScheduledActions of this type have already been added."))
                {
                    //changing the Boolean to false, because the OS does not allow any new taks
                    isLiveTileActivated = false;
                }
            }
            catch (SchedulerServiceException)
            {
                //if there is a problem with the service, changing the Boolean to false. 
        //feel free to inform the user about the exception and provide additional info
                isLiveTileActivated = false;
            }
        }
```
 
In Application.Launching() and in ToggleSwitch\_Checked event, call this method. In ToggleSwitch\_UnChecked event, call ‘StopLiveTileUpdaterPeriodicTaskAgent()’ instead to stop the agent.

Before we are now able to debug our agent, there are two things left. The first thing is to declare

```
 chsarp
#define DEBUG_AGENT
```
 
in the very first line of ‘App.xaml.cs’ as well as in ‘ScheduledAgent.cs’ before all using statements.

The second thing is adding an if-Debug command to our ‘StartLiveTileUpdaterPeriodicTaskAgent()’ and also to our ‘OnInvoke(ScheduledTask task)’ methods:

``` csharp
#if(DEBUG_AGENT)
ScheduledActionService.LaunchForTest(LiveTileUpdaterPeriodicTaskNameString, TimeSpan.FromSeconds(60));
#endif
```
 
Add this after adding the task to the SchedulerService in our agent starter and before ‘NotifyComplete()’ in ‘ScheduledAgent.cs’.

If you run the project now in Debug mode, your agent will be debugged and forced to run after 60 seconds. If you run the project in Release mode, Visual Studio will throw an ‘Access Denied’ error, so make sure you set it up correctly.

If you follow all steps above, you will be able to add a Live Tile updated via background agent very easily to your app.

As always, I hope this real world scenario will help some of you.

Until then, happy coding!