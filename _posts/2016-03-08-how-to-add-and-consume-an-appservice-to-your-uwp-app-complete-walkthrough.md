---
id: 4425
title: 'How to add and consume an AppService to your UWP app (complete walkthrough)'
date: '2016-03-08T06:45:38+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-add-and-consume-an-appservice-to-your-uwp-app-complete-walkthrough/
categories:
    - Archive
tags:
    - App2App
    - AppService
    - AppServiceConnection
    - BackgroundTask
    - 'Class library'
    - communication
    - 'Nuget package'
    - uwp
    - 'windows 10'
---

One of the very cool and helpful new features Microsoft added for UWP apps are AppServices. AppServices allow your application to provide functionality to other applications without even being launched. This post shows how to add and consume an AppService.

#### Creating the AppService

An AppService is a background task that is hosted by your application. To add an AppService to your existing application, right click on the solution name in the Solution Explorer and select ‘add new project’. Under category Windows\\Universal, select ‘Windows Runtime Component’, give it a name and click on ‘OK’:

![image](/assets/img/2016/03/image.png "image")

This adds a new project to your solution. Rename or Replace Class1.cs to something that matches your need (in my sample, I just use ‘Handler’). The next step is to implement the IBackgroundTask interface:

![image](/assets/img/2016/03/image-1.png "image")

This will add the ‘Run(IBackgroundTaskInstance taskInstance)’ method to the class. Before we continue with to integrate our AppService further into the system, we need to declare two members in the Handler class:

``` csharp
         private BackgroundTaskDeferral _backgroundTaskDeferral;
        private AppServiceConnection _appServiceConnection;
```
 
Now that we have those two in place, let’s have a look inside the Run method:

``` csharp
        public void Run(IBackgroundTaskInstance taskInstance)
        {
            //get the task instance deferral
            this._backgroundTaskDeferral = taskInstance.GetDeferral();

            //hooking up to the Canceled event to close app connection
            taskInstance.Canceled += OnTaskCanceled;

            //getting the AppServiceTriggerDetails and hooking up to the RequestReceived event
            var details = (AppServiceTriggerDetails)taskInstance.TriggerDetails;
            _appServiceConnection = details.AppServiceConnection;
            _appServiceConnection.RequestReceived += OnRequestReceived;
        }
```
 
The first step is to get a background task deferral. This allows the service to run asynchronous code against the background Task without crashing. The next step is to handle the background task’s Canceled event. Even if the Task get’s cancelled, we must complete the process and tell the OS that we have finished. If we won’t do this, the AppService will stop working on the first cancel operation. To do so, add this code to your Canceled event handling method:

``` csharp
             this._backgroundTaskDeferral?.Complete();
```
 
Last but not least, we need to handle the request that the AppService received. To do so, we need to get the [AppServiceTriggerDetails](https://msdn.microsoft.com/en-us/library/windows/apps/xaml/dn921727(v=win.10).aspx?appid=dev14idef1&l=en-us&k=k(windows.applicationmodel.appservice.appservicetriggerdetails)%3bk(targetframeworkmoniker-.netcore,version%3dv5.0)%3bk(devlang-csharp)&rd=true) from our BackgroundTask. This allows us to get a reference to the [AppServiceConnection](https://msdn.microsoft.com/en-us/library/windows/apps/xaml/dn921704(v=win.10).aspx?appid=dev14idef1&l=en-us&k=k(windows.applicationmodel.appservice.appserviceconnection)%3bk(targetframeworkmoniker-.netcore,version%3dv5.0)%3bk(devlang-csharp)&rd=true), which provides the event ‘RequestReceived’ that we want to handle.

Within our RequestReceived event handling method, the effective work is done. As this is also an asynchronous action, we first need to get again a reference to the event’s Deferral:

``` csharp
 var msgDeferral = args.GetDeferral();
```
 
The next step is to get the data from the event’s [AppServiceRequestReceivedEventArgs](https://msdn.microsoft.com/en-us/library/windows/apps/xaml/dn921718(v=win.10).aspx?appid=dev14idef1&l=en-us&k=k(windows.applicationmodel.appservice.appservicerequestreceivedeventargs)%3bk(targetframeworkmoniker-.netcore,version%3dv5.0)%3bk(devlang-csharp)&rd=true):

``` csharp
             var input = args.Request.Message;
```
 
This gives us a [ValueSet](https://msdn.microsoft.com/en-us/library/windows/apps/windows.foundation.collections.valueset.aspx) (which works like a Dictionary), which we need to parse to get the input data we need:

``` csharp
 var response = (string) input["question"];
```
 
The ValueSet entries you are setting up here are the parameters your consumer has to provide. In my sample, I have only a single parameter called ‘question’, but it works also well with more parameters. After we pulled the parameters out of the ValueSet, we can do the work our AppService is supposed to do.

It is a good practice to separate the function you want to provide in the AppService in a separate project. You need to write the code only once, but can use it in your main application as well as in the AppService. In my Sample, this is done in the AppServiceResponder project. It may look like an overkill in this case, but if you have more complex logic than the sample it will absolutely make sense.

I am wrapping the work into a try/finally block as we need to complete the async operation in any case to keep our AppService running. This way, even if the work is not completed successfully, at least we come out clean of the AppService Task. Within the try part of the try/finally block, we are doing the work that the service is supposed to do. The result needs to be passed as a ValueSet again. To effectively send the result to the requesting app, we use this line of code:

``` csharp
 await args.Request.SendResponseAsync(result);
```
 
Here is the full method for reference:

``` csharp
         private async void OnRequestReceived(AppServiceConnection sender, AppServiceRequestReceivedEventArgs args)
        {
            //async operation needs a deferral
            var msgDeferral = args.GetDeferral();


            //picking up the data ValueSet
            var input = args.Request.Message;
            var result = new ValueSet();

            //parsing the ValueSet
            var response = (string) input["question"];

            try
            {
                //as long as the app service connection is established we are using the same instance even if data changes.
                //to avoid crashes, clear the result before getting the new one
                result.Clear();

                if (!string.IsNullOrEmpty(response))
                {
                    var responderResponse = AppServiceResponder.Responder.Instance.GetResponse(response);

                    result.Add("Status", "OK");
                    result.Add("response", responderResponse);
                }

                await args.Request.SendResponseAsync(result);
            }
            //using finally because we need to tell the OS we have finished, no matter of the result
            finally
            {
                msgDeferral?.Complete();
            }
        }
```
 
#### Declaring the App Service in the host app

The final step we need to apply is to declare the AppService in our Package.appxmanifest. Declaring the AppService is pretty simple. Just select ‘App Service’ in the dropdown and hit ‘Add’. Give it a name (Microsoft recommends ‘reverse domain name style ‘, so I used it. The last step is to declare the Entry point, which is AppServiceNamespace.ClassName (replace with yours).

The result should look like this:

![image](/assets/img/2016/03/image-2.png "image")

Now build the solution. If all is set up correct, you have made an application implementing an AppService.

#### Creating the AppService Connector

Like I recommend to extract functionality that runs inside the AppService, I do so for the code that connects the app into a separate project. This allows you to reuse the project in multiple apps. An additional advantage is that you can create a Nuget package for this separate project to provide this functionality to other developers (*I will write a separate post about this to keep this one focused on the AppService*).

After adding a new project to the Solution, rename/replace also here Class1. I named my handler class just Connector. To make it a no brainer to use the connector, I implemented the class as singleton:

``` csharp
         private static Connector _instance;
        public static Connector Instance => _instance ?? (_instance = new Connector());
```
 
After that, I added an asynchronous Task that returns the AppService’s response. Let’s have a look inside the task. Inside the using statement for the AppServiceconnection, the first thing we need to do is to call the AppService with these three lines of code:

``` csharp
                 //declaring the service and the package family name
                SampleAppServiceConnection.AppServiceName = "com.msiccdev.sampleappservice";

                //this one can be found in the Package.appxmanifest file
                SampleAppServiceConnection.PackageFamilyName = "acc75b1a-8b90-4f18-a2c4-08b0d700f1c6_62er76fr5b6k0";

                //trying to connect to he AppService
                AppServiceConnectionStatus status = await SampleAppServiceConnection.OpenAsync();
```
 
We’ll get a [AppServiceConnectionStatus](https://msdn.microsoft.com/en-us/library/windows/apps/xaml/dn921705(v=win.10).aspx?appid=dev14idef1&l=en-us&k=k(windows.applicationmodel.appservice.appserviceconnectionstatus)%3bk(targetframeworkmoniker-.netcore,version%3dv5.0)%3bk(devlang-csharp)&rd=true) back, which helps us to deside how to go on in the task. If we do not have success in getting a connection to the AppService, I am returning an error description. For this, I created a simple helper method:

``` csharp
         private string GetStatusDetail(AppServiceConnectionStatus status)
        {
            var result = "";
            switch (status)
            {
                case AppServiceConnectionStatus.Success:
                    result = "connected";
                    break;
                case AppServiceConnectionStatus.AppNotInstalled:
                    result = "AppServiceSample seems to be not installed";
                    break;
                case AppServiceConnectionStatus.AppUnavailable:
                    result =
                        "App is currently not available (could be running an update or the drive it was installed to is not available)";
                    break;
                case AppServiceConnectionStatus.AppServiceUnavailable:
                    result = "App is installed, but the Service does not respond";
                    break;
                case AppServiceConnectionStatus.Unknown:
                    result = "Unknown error with the AppService";
                    break;
            }

            return result;
        }
```
 
For the case we are successful with our connection attempt , I am sending the needed ValueSet with these lines:

``` csharp
                    var input = new ValueSet() {{"question", question}};
                    AppServiceResponse response = await SampleAppServiceConnection.SendMessageAsync(input);
```
 
I am using a simple ValueSet with just one value here, but I have already done it with a more complex structure and this works as well. I haven’t reached any limits by now, but I think that they are the same as for all Background Tasks (don’t throw stones at me if it is different). The only thing I then need to do is to handle the response according to its status with this switch statement:

``` csharp
                     switch (response.Status)
                    {
                        case AppServiceResponseStatus.Success:
                            result = (string) response.Message["response"];
                            break;
                        case AppServiceResponseStatus.Failure:
                            result = "app service called failed, most likely due to wrong parameters sent to it";
                            break;
                        case AppServiceResponseStatus.ResourceLimitsExceeded:
                            result = "app service exceeded the resources allocated to it and had to be terminated";
                            break;
                        case AppServiceResponseStatus.Unknown:
                            result = "unknown error while sending the request";
                            break;
                    }
```
 
As we already know by know, we receive a ValueSet as response message. In my sample, I am returning only the response string. That’s it, we are already able to run use the AppService via the Connector. For demo purposes, I added also a simple AppService consumer app. Once the user puts in a question and hits the answer button, the result from the AppService gets displayed. All we need is just one line of code:

``` csharp
             AnswerTextBlock.Text = await SampleAppServiceConnector.Connector.Instance.GetResponse(QuestionTextBox.Text);
```
 
Pretty easy to use, right? Here is a screen shot from the consumer app:

![image](/assets/img/2016/03/image-3.png "image")

That’s all we need for our AppService. Bonus of the project structure I used: You can provide a Nuget Package for other devs to consume your AppService. How? I will write about this in my next blog post.

Even if this is not a MVVM structured app, it absolutely works in there, too. If you want to have a look on a live in the Store app that uses MVVM and an AppService, [click here](https://www.microsoft.com/store/apps/9nblggh4qbxl). For using the AppService within your app, just download [this Nuget Package ](https://www.nuget.org/packages/SimpleLeetAppServiceHandler/)into your app. Otherwise, I created a complete working sample and pushed it[ on my Github account right here](https://github.com/MSiccDev/AppServiceSample). Feel free to play around with it to explore AppServices a bit more. As always, I hope this post is helpful for some of you.

Happy coding, everyone!