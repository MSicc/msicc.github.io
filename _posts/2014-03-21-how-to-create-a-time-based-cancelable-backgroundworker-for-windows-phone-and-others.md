---
id: 4028
title: 'How to create a time based &#038; cancelable BackgroundWorker for Windows Phone (and others)'
date: '2014-03-21T20:29:21+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-create-a-time-based-cancelable-backgroundworker-for-windows-phone-and-others/
categories:
    - Archive
tags:
    - Background
    - 'background worker'
    - 'C#'
    - 'multi threading'
    - 'sleeping thread'
    - threading
    - 'time based'
    - timed
    - 'Windows Phone'
    - WinPhanDev
    - wpdev
---

![backgroundworkerTimebased](/assets/img/2014/03/backgroundworkerTimebased.png "backgroundworkerTimebased")

While working on my current project, I needed a solution for a running code with a time offset. Well, you might say, no problem after all, and it is true.

The challenge at this point was to find a way to keep it cancelable at any time. And so the fun started.

After digging a bit deeper into the MSDN documentation, the [BackgroundWorker](http://msdn.microsoft.com/en-us/library/System.ComponentModel.BackgroundWorker(v=vs.110).aspx) class already have a bunch of methods and properties that are very helpful for this case.

As it is always the case when you work with multiple threads, it can cause some headache. But I got around it and thought it might be helpful for some of you.

Here is how I solved the scenario:

First, declare a static BackgroundWorker so you need to set it up only once on the page you want to use it. In the Loaded or OnNavigatedTo event, we are instantiating our BackgroundWorker.

``` csharp
 worker = new BackgroundWorker();
worker.WorkerSupportsCancellation = true;
worker.WorkerReportsProgress = true;
```
 
Now, let’s add two very important properties: WorkerSupportsCancellation and WorkerReportsProgress. Both need to be set to true, otherwise you will get one InvalidOperationException after the other when running the code.

In my case, I created a separate method to start the BackgroundWorker and hook up to all important events:

``` csharp
         private void RunBackgroundWorker()
        {
            worker.RunWorkerCompleted += worker_RunBackgroundWorkerCompleted;

            //delegating the DoWork event
            worker.DoWork += ((s, args) =>
            {
                //generating a loop
                for (int i = 0; i < 100; i++)
                {

                    if (worker.CancellationPending == true)
                    {
                        //set cancel to true to finish the cancellation on the next run in the loop
                        args.Cancel = true;
                    }
                    else
                    {
                        //calculate your time: seconds * 1000 / 100
                        Thread.Sleep(50);
                        worker.ReportProgress(i);
                    }
                }
            });

            //can be used to fill a progress bar/show percentage
            worker.ProgressChanged += worker_ProgressChanged;

            //start the BackgroundWorker
            worker.RunWorkerAsync();
        }
```
 
You might notice that I created a loop that runs up to 100. This is necessary for my solution, as this is the key to make the BackgroundWorker cancelable in my solution. Every 50 milliseconds, the CancellationPending property gets checked as I am looping through until the count reaches 100. I am setting an offset of 50 milliseconds, as I want to have a total offset of 5 seconds.

Next, let us set up a button that cancels the background task (can be used in other events/methods, too):

``` csharp
 public void CancelButton_Click(object sender, RoutedEventArgs e)
{
    //check this always to avoid InvalidOperationExceptions
   if (worker.WorkerSupportsCancellation == true)
   {
         //request cancellation of the BackgroundWorker
         //this sets the CancellationPending property to true
        worker.CancelAsync();
    }
 }
```
 
Important: Always check if the Cancellation is supported by your BackgroundWorker to avoid those ugly InvalidOperationExceptions. Then, simply call the CancelAsync() method of your BackgroundWorker to set the CancellationPending property to true.

When the BackgroundWorker has completed (or cancelled), the value of the CancellationPending property is passed to the RunBackgroundWorkerCompletedEvent, where you can simply use the e.Cancelled property to continue your code (which will then be back on your main application thread):

``` csharp
 private void worker_RunBackgroundWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)
{
    if (e.Cancelled == true)
    {
       //your code in case the BackgroundWorker was cancelled
    }
    else
    {
       //your code in case the BackgroundWorker was running until the end
    }
}
```
 
The last event I want to mention is the ProgressChanged event. It can be used to display a percentage or similar things, I just use it to see when the cancellation gets active:

``` csharp
 private void worker_ProgressChanged(object sender, ProgressChangedEventArgs e)
{
     //your code here, for example display percentage or fill a progress bar

     //use the line below to check the percentage and if CancellationPending property gets changed
     //Debug.WriteLine(e.ProgressPercentage + "  " + worker.CancellationPending);
}
```
 
With this few methods we can create a time based offset on a BackgroundWorker while keeping it cancelable at any time. The code above should work in similar form also on other platforms like Windows 8 or Xamarin.

As always, I hope this will be helpful for some of you.

Happy coding!