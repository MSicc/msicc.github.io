---
id: 4041
title: 'How to use Text to Speech to read text aloud on Windows Phone 8'
date: '2014-03-28T22:45:45+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-use-text-to-speech-to-read-text-aloud-on-windows-phone-8/
categories:
    - Archive
tags:
    - InstalledVoices
    - 'speech synthesis'
    - Synthesis
    - 'Text to Speech'
    - tts
    - voice
    - VoiceInformation
    - 'Windows Phone 8'
    - WP8
    - wpdev
---

![tts](/assets/img/2014/03/tts.png "tts")

Today I started to update my [very first app](http://www.windowsphone.com/s?appid=ec3a70c1-6802-4197-918f-3506798ead93) I ever wrote to Windows Phone 8. The app has a read aloud feature that uses the Bing translation service (as TTS was not available on Windows Phone **7).**

Of course I am now using the new Windows Phone 8 API, but I had some trouble figuring out how to handle text to speech with different languages and if no language speech pack is installed on a device. I finally found a solution and want to share it with you.

First, we need to declare a new SpeechSynthesizer and an IEnumerable for VoiceInformation:

``` csharp
 SpeechSynthesizer speechSynth = new SpeechSynthesizer();
IEnumerable<VoiceInformation> voices = InstalledVoices.All;
```
 
Next, I declared a simple helper method to get the currently used language:

``` csharp
          public string GetCurrentCulture()
         {
             return CultureInfo.CurrentCulture.TwoLetterISOLanguageName.ToString();
         }
```
 
I am using the TwoLetterISOLanguageName property because there are different versions of some languages like German or English. This makes it easier to handle throughout other methods.

Now that we are able to read the installed voices and to read the currently used language, we can use a simple Linq query to get the speech language we want to use:

``` csharp
 var engVoice = from voice in voices where voice.Language.StartsWith("en") select voice;
```
 
The engVoice object contains now all English speech packages – if they are installed. To determine if we have items we can use, we just need to check the Count(). As long as the count is bigger than 0, we can start to let our app reading aloud our text. If not, we should inform the user that there is no matching language pack installed.

``` csharp
                  if (engVoice.Count() > 0)
                 {
                     speechSynth.SetVoice(engVoice.ElementAt(0));
                     await speechSynth.SpeakTextAsync(text);
                 }
                 else
                 {
                     MessageBox.Show("Language package missing");
                 }
```
 
My app supports three languages. Because of this, I am using if/else if statements to match the languages. If the user uses another language than the supported ones, I am cancelling all speech attempts and display a message to the user which languages are supported.

Here is my complete method:

``` csharp
          private async void StartReadAloud(string text)
         {
             IEnumerable<VoiceInformation> voices = InstalledVoices.All;

             if (GetCurrentCulture() == "en")
             {
                 var engVoice = from voice in voices where voice.Language.StartsWith("en") select voice;
                 if (engVoice.Count() > 0)
                 {
                     speechSynth.SetVoice(engVoice.ElementAt(0));
                     await speechSynth.SpeakTextAsync(text);
                 }
                 else
                 {
                     MessageBox.Show("Language package missing");
                 }
             }
             else if (GetCurrentCulture() == "it")
             {
                 var itVoice = from voice in voices where voice.Language.StartsWith("it") select voice;
                 if (itVoice.Count() > 0)
                 {
                     speechSynth.SetVoice(itVoice.ElementAt(0));
                     await speechSynth.SpeakTextAsync(text);
                 }                                  
                 else
                 {
                     //msgbox Italian
                 }
             }
             else if (GetCurrentCulture() == "de")
             {
                 var deVoice = from voice in voices where voice.Language.StartsWith("de") select voice;
                 if (deVoice.Count() > 0)
                 {
                     speechSynth.SetVoice(deVoice.ElementAt(0));
                     await speechSynth.SpeakTextAsync(text);
                 }
                 else
                 {
                     //msgbox German
                 }
             }
             else
             {
                 speechSynth.CancelAll();
                 //msgbox notSupported language               
             }
         }
```
 
I hope this is helpful for some of you and saves you some trouble.If you have anything to add/correct on my method, feel free to leave a comment below.

Happy coding, everyone!