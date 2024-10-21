---
id: 3787
title: 'Make your app listening to the user&#8217;s voice'
date: '2013-10-29T06:19:13+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /make-your-app-listening-to-the-users-voice/
categories:
    - Archive
tags:
    - grammar
    - recognition
    - speech
    - SpeechRecognizer
    - srgs
    - VCD
    - voice
    - 'Voice Command Definition'
    - winphand
    - WP8
    - wp8dev
    - wpdev
---

![speech_recognition_WPLogo](/assets/img/2013/10/speech_recognition_WPLogo.png "speech_recognition_WPLogo")

In one of my recent projects, I added speech recognition to start the app as well as for using the app. This blog post gives you a short overview on what’s possible and how to do it.

### How to start your app with certain conditions:

Any app can be started by the simple voice command “open \[AppName\]” on Windows Phone. But what if we want to start certain functions when calling our app via speech? Then you’ll need a so called Voice Command Definition (VCD) file. This is a xml file that tells the OS to launch a certain function of your app.

Before we can start, please make sure you enabled ID\_CAP\_SPEECH\_RECOGNITION, ID\_CAP\_MICROPHONE, and ID\_CAP\_NETWORKING capabilities in your app manifest.

Let’s have a look at a VCD with two different launch arguments:

``` xml
 <?xml version="1.0" encoding="utf-8"?>

<VoiceCommands xmlns="https://schemas.microsoft.com/voicecommands/1.0">
 <CommandSet xml:lang="en-US">
 <CommandPrefix>speech test app</CommandPrefix>
 <Example> speech test app</Example>

<Command Name="open MainPage">
 <Example> open the MainPage </Example>
 <ListenFor> open the MainPage </ListenFor>
 <ListenFor> bring me to Mainpage </ListenFor>
 <ListenFor> let me see my MainPage </ListenFor>
 <Feedback> loading Mainpage....</Feedback>
 <Navigate />
 </Command>

<Command Name="open SettingsPage">
 <Example> open the settings page</Example>
 <ListenFor>open settings</ListenFor>
 <ListenFor>bring me to settings page</ListenFor>
 <ListenFor>let me see my settings page</ListenFor>
 <Feedback>opening settings ...</Feedback>
 <Navigate />

</Command>

</CommandSet>
 </VoiceCommands>
```
 

As you can see, the syntax is pretty easy to use. Here is some short explanation about the commands:

- &lt;CommandPrefix&gt; is for telling the OS how to launch your app
- &lt;Command Name=”xxx”&gt; is for telling the OS which launch parameter should be passed to your app
- &lt;ListenFor&gt; defines which speech cases you want to allow to let the OS launch your app (pretty important point)
- &lt;Example&gt; Text entered here is shown in TellMe’s speech Window and on the “What can I say?” page
- &lt;Feedback&gt; this is the answer of TellMe to the user

The next part is to load the VCD file once the app starts. Therefore, you will need to add the VCD file in the App constructor. I created an async Task for that:

``` csharp
private async Task InitializeVoiceCommands()
 {
 await VoiceCommandService.InstallCommandSetsFromFileAsync(new Uri("ms-appx:///VoiceCommandDefinitions.xml"));
 }
```
 
It is recommended that you set “Copy to Output Directory” property of the VCD file to “Copy if newer”.

Now that we have implemented and loaded our VCD file, let’s have a look on how we can open our app based on the arguments. Based on that VCD file, our app gets key/value pairs of the launch arguments. The implementation is pretty easy in a few lines of code:

``` csharp
if (NavigationContext.QueryString.ContainsKey("voiceCommandName"))
 {
   string voiceCommandName = NavigationContext.QueryString["voiceCommandName"];
   switch (voiceCommandName)
    {
       case "open MainPage":
       //let the App launch normally
       //you can also pass other launcharguments via the VCD file that could launch other functions
       break;

       case "open SettingsPage":
       //let the App lauch immediately to the settings page
       NavigationService.Navigate(new Uri("/SettingsPage.xaml", UriKind.Relative));
       break;
     }
 }
```
 
And with this few lines we already added our app with individual voice commands to the speech function of the OS.

### Using your app with in app voice commands:

We now learned how to start our app with certain conditions. However, this is not how we are using the speech recognition within our app.

There are several ways on how you can implement in app voice commands. I will show you two of them.

First of all, using the Windows button does not work in app. You will need to add a dedicated button to start the voice recognition.

Let’s start with a List&lt;string&gt; or string\[\] array, and launch the SpeechRecognizerUI:

``` csharp
try
{
    //initalize speech recognition
    SpeechRecognizerUI speech = new SpeechRecognizerUI();

    List<string> InAppCommandList = new List<string>
    {
            "goto home",
            "goto settings",
            "set name",
            "set age",
            //add more to that list as you need
    };

    //load the List as a Grammar
    speech.Recognizer.Grammars.AddGrammarFromList("InAppCommandList", InAppCommandList);

    //show some examples what the user can say:
    speech.Settings.ExampleText = " goto home, goto settings, set name, set age ";

    //get the result
    SpeechRecognitionUIResult result = await speech.RecognizeWithUIAsync();

    //delegate the results
    switch (result.RecognitionResult.Text)
    {
        case "goto home":
            //your code here
            break;
        case "goto settings":
            //your code here
            break;
        case "set name":
            //your code here
            break;
        case "set age":
            //your code here
            break;
    }
}
catch (System.NullReferenceException)
{
    //do something with the Exception
}
```
 
As you can see, there are only a few steps needed:

- initalize speech recognition
- load the List&lt;string&gt;/string\[\] as a Grammar
- get the result
- delegate the results

You might have noticed that I wrapped the code into a try/catch block. I needed to do that as I often got a NullReferenceException on closing without voice input or also if no text was heard. This way, your app will not crash.

There is also another way to add a Grammar to the speech recognition engine: SRGS (Speech Recognition Grammar Specification), which is again a XML file that contains which input is accepted.

While I was working on this app, I noticed that it is pretty powerful – and complex. This is why I decided to mix both methods to get things done (you know, timelines and such). I will dive into that topic deeper and then write another blog post about it. But for now, I’ll show you a simple example.

In this case, I wanted the SpeechRecognizerUI to only accept numbers as input. This is how my SRGS file looks like:

``` xml
 <?xml version="1.0" encoding="utf-8" ?>

<grammar version="1.0" xml:lang="en-US" root="Command" tag-format="semantics/1.0"
 xmlns="https://www.w3.org/2001/06/grammar"
 xmlns:sapi="https://schemas.microsoft.com/Speech/2002/06/SRGSExtensions">

<rule id="Command" scope="public">
 <item repeat="1-20">
 <ruleref uri="#Numbers"/>
 </item>
 </rule>

<rule id="Numbers">
 <one-of>
 <item >1</item>
 <item >2</item>
 <item >3</item>
 <item >4</item>
 <item >5</item>
 <item >6</item>
 <item >7</item>
 <item >8</item>
 <item >9</item>
 <item >0</item>
 </one-of>
 </rule>

</grammar>
```
 
If you add a SRGS file as to your project, the correct structure is already implemented. You only need to set the root to the first rule you want to be executed. In my case, I defined a rule with the id Command in a public scope (this way, you would also be able to call it from another Grammar).

The rule returns items that will occur minimum 1 time and maximum 20 times. Basically, this defines the range of the input length.

To make the SpeechRecognitionUI only accepting numbers, I defined another rule that contains a list of numbers. the &lt;one-of&gt; property defines here that any of the numbers is accepted. With &lt;ruleref uri=”#Numbers”/&gt; I am telling my app to accept any of the numbers as often as they occur until the count of 20.

Now let’s have a look on how to implement this Grammar into our app and connect it to our SpeechRecognizerUI:

``` csharp
SpeechRecognizerUI speech = new SpeechRecognizerUI();

//generate a Numberonly Input Scope based on a SRGS file
//the ms-appx:/// prefix will not work here and return a FileNotFoundException!
Uri NumberGrammarUriPath = new Uri("file://" + Windows.ApplicationModel.Package.Current.InstalledLocation.Path + @"/NumberOnlyGrammar.xml", UriKind.RelativeOrAbsolute);

speech.Recognizer.Grammars.AddGrammarFromUri("NumberGrammarUriPath", NumberGrammarUriPath);

SpeechRecognitionUIResult speechRecognitionResult = await speech.RecognizeWithUIAsync();

if (speechRecognitionResult.ResultStatus == SpeechRecognitionUIStatus.Succeeded)
{
    //using Regex to remove all whitespaces
    string NumberInputString = Regex.Replace(speechRecognitionResult.RecognitionResult.Text, @"\s+", "");
    // do somethin with the string
}
```
 
We are again calling the SpeechRecognizerUI, but this time we are loading our XML formatted SRGS file as Grammar. Notice that loading the file only works with the way above. Using the “ms-appx:///” prefix will return a FileNotFoundException and crash your app.

Instead of AddGrammarFromList we are using AddGrammarFromUri to load the file.

By checking the SpeechRecognitionUIStatus, we are performing actions only if the recognition was succesful based on our SRGS Grammar. If you want to perform other actions on non-success status, you will be able to implement the following enumerations:

- Succeeded (0)
- Busy (1)
- Cancelled (2)
- Preempted (3)
- PrivacyPolicyDeclined (4)

As you can see, basic speech recognition is fast to implement. As always, I hope this post is helpful for some of you.

I also recommend to read these pages on MSDN about speech for Windows Phone: [https://msdn.microsoft.com/en-us/library/windowsphone/develop/jj207021(v=vs.105).aspx](https://msdn.microsoft.com/en-us/library/windowsphone/develop/jj207021(v=vs.105).aspx "https://msdn.microsoft.com/en-us/library/windowsphone/develop/jj207021(v=vs.105).aspx")

Happy coding everyone!