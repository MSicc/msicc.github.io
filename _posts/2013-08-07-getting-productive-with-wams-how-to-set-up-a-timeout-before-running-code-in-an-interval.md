---
id: 3678
title: 'Getting productive with WAMS: how to set up a timeout before running code in an interval'
date: '2013-08-07T06:33:42+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /getting-productive-with-wams-how-to-set-up-a-timeout-before-running-code-in-an-interval/
categories:
    - Azure
    - 'Dev Stories'
tags:
    - Interval
    - JavaScript
    - 'Mobile Services'
    - Timeout
    - WAMS
    - 'Windows Azure'
---

![WAMS](/assets/img/2013/08/WAMS.png "WAMS")

I truly love Windows Azure Mobile Services, as it provides an easy way to connect Windows Phone apps to the cloud and also manages Live Tiles and Toast Notifications via Push. This way, we don’t have an impact on the users battery life by running background agents.

However, scheduled scripts are only able to run in predefined time windows, with 15 minutes at lowest.

It may happen that you want to run your script in shorter intervals, as I am currently implementing into mine.

### The functions: setTimeout() and setInterval()

The difference between those two is pretty simple:

- setTimeout() is running the function included only once after the time in milliseconds has passed
- setInterval() is running the function included in an interval set up in milliseconds until it gets stopped

Knowing this, it is pretty simple to set up those two functions each for itself:

``` js 
setTimeout(function() { 
//code to run 
}, time in milliseconds);

setInterval(function() {
//code to run
}, time in milliseconds);
```
 
I highly recommend to declare an interval as a variable. If you won’t do that, the interval will run forever as you never can stop it!

### But what if we need first a timeout and then an interval?

That is the problem I was trying to solve over the last two days. I was playing around with all kinds of code constellations to achieve this goal. And I found a solution.

We need to set up a few things to achieve that goal:

- global variables for the number of runs and the interval
- a function that only proves if the interval is still valid to run or needs to be stopped
- our function that should be run in an interval

In my example, I want the code to be executed five times, then the interval should be stopped (cleared). I achieve this goal with a pretty simple if/else clause, like you can see:

``` js
if (NumberOfRuns < 6)
{
        console.log("interval ran " + NumberOfRuns + " times");
        doSomething();
        }
        else
        {
        //stoping the interval       
        clearInterval(Interval);
        console.log("interval stopped!");
}
```
 
As this is a sample script, I use the function doSomething() to be executed. This represents your code that should be executed if the interval is still valid. Only important thing: counting up the number of runs every time the function is hit.

``` js
  //counting up the number of runs with every call of the function
    NumberOfRuns++;

    //your code runs here
```
 
And in our starting point (the function that has the same name like your script), we finally declare the the timeout as well as we start to count:

``` js
  //setting the NumberofRuns to 1 as after timeout the first run starts
   NumberOfRuns = 1;
   console.log("start")

   //settings timeout to call the function that proves if the interval is still valid
   setTimeout(function(){
       //interval has to be defined in this scope, otherwise it will not be accepted
       //we also need a variable for the interval to be able to stop the interval
       Interval = setInterval(RunInterval, 5000);
       console.log("timeout is over");
   },15000);
```
 
First, we start our counter with 1, as after the timeout the code will be executed for the first time. I added also some logging functions, so you can prove everything is running fine.

The content of the global variable “Interval” has to be declared within the timeout, otherwise we will not be able to set the interval for our function.

Once you figured all the points above, it is pretty easy to implement this into your running script.

If you want to play around with this and see what the log looks like, here is the full WAMS scheduler script:

``` js
//declaring global variables
var NumberOfRuns;
var Interval;

//function to execute in interval
function doSomething()
{
    //counting up the number of runs with every call of the function
    NumberOfRuns++;

    //your code runs here
}

//this function proves if the the interval is still valid
function RunInterval()
{
     if (NumberOfRuns < 6)
        {
        console.log("interval ran " + NumberOfRuns + " times");
        doSomething();
        }
        else
        {
        //stoping the interval       
        clearInterval(Interval);
        console.log("interval stopped!");
        }
}

//main script function (start)
function TestIntervalScript() 
{
   //setting the NumberofRuns to 1 as after timeout the first run starts
   NumberOfRuns = 1;
   console.log("start")

   //settings timeout to call the function that proves if the interval is still valid
   setTimeout(function(){
       //interval has to be defined in this scope, otherwise it will not be accepted
       //we also need a variable for the interval to be able to stop the interval
       Interval = setInterval(RunInterval, 5000);
       console.log("timeout is over");
   },15000);
}
```
 
And here is a screen shot from the desired log file:

![Screenshot (192)](/assets/img/2013/08/Screenshot-192.png "Screenshot (192)")

As I am still pretty new to JavaScript, there might be also other ways to achieve this. Feel free to leave a comment if you have anything to add/improve on this code.

And as always, I hope this is helpful for some of you when playing around with Windows Azure Mobile Services.

Happy coding!