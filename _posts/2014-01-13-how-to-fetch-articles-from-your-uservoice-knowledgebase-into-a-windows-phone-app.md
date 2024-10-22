---
id: 3935
title: 'How to fetch articles from your uservoice knowledgebase into a Windows Phone app'
date: '2014-01-13T19:27:50+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-fetch-articles-from-your-uservoice-knowledgebase-into-a-windows-phone-app/
categories:
    - Archive
tags:
    - articles
    - authentication
    - FAQ
    - fetch
    - httpclient
    - json
    - 'knowledge base'
    - request
    - uservoice
    - WinPhanDev
    - wpdev
---

As some of you might know, I recently switched to uservoice.com for feedback, support and also FAQ hosting (read more [here]({% post_url 2014-01-06-experiment-using-uservoice-com-for-support-feedback-and-faq %})). Of course I want to integrate all those features into my app(s) to make the user experience as native as possible.

First, you need to generate a new app in uservoice.com. Log into your account, click on ‘Admin Console’, ‘Settings’ and finally ‘Integrations’. Then add your API client, you will have something like this:

![uservoice_api_client](/assets/img/2014/01/uservoice_api_client.png "uservoice_api_client")

Today, we are starting with getting our knowledge base articles into our app.

This is the easiest part besides assigning the support mail address to a button.

Here is how we are doing it:

First, we need to declare some constants:

``` csharp
 //consumer key and secret are needed to authorize our requests (oAuth)
 //KB articles only need the ConsumerKey
 const string ConsumerKey = "<youKey>";
 const string ConsumerSecret = "<yourSecret>";
 //all KB articles:
 const string KnowledgebaseString = "https://<yoursubdomain>.uservoice.com/api/v1/articles.json?client={0}";
 //KB topic articles:
 //using sort=oldest ensures that you will get the right order of your articles
 const string KnowledgebaseTopicString = "https://<yousubdomain>.uservoice.com/api/v1/topics/{0}/articles.json?client={1}&sort=oldest";

static string articleJsonString;
```
 
As you can see, we have two options to fetch our knowledgebase articles – all articles (if you have only one app, you’re fine with that) or topic based.

To get the needed topic id, just open the topic in your browser. The topic id is part of the url:

![uservoice_topic_id](/assets/img/2014/01/uservoice_topic_id.png "uservoice_topic_id")

For getting authorized to receive the JSON string of our knowledge base, we need to pass the consumer key as parameter “client” to the base url of our request. To get our list sorted, I am using the sort parameter as well.

To receive the JSON string, we are creating an async Task&lt;string&gt; that fetches our article. To make the result reloadable, add the IfModifiedSince Header (otherwise Windows Phone caches the result during the app’s current lifecycle).

``` csharp
 //receiving the JSON string for our KB does not need any advanced requests: 
//a basic HttpClient handles everything for us, as we don't need any authentication here
public async Task<string> GetKBJsonString()
{
      string getKBJSonStringFromUserVoice = string.Empty;

     HttpClient getKBJsonClient = new HttpClient();
     getKBJsonClient.DefaultRequestHeaders.IfModifiedSince = DateTime.Now;

     getKBJSonStringFromUserVoice = await getKBJsonClient.GetStringAsync(new Uri(string.Format(KnowledgebaseTopicString, "47463", ConsumerKey), UriKind.RelativeOrAbsolute));

      return getKBJSonStringFromUserVoice;
 }
```
 
Of course we want to have a list shown to our users – to display our JSON string in a ListBox, we need to deserialize it. To be able to deserialize it, we need a data class. You can use [json2sharp.com](https://json2sharp.com) to generate the base class or use this one (~~download Link~~). It fits for both all articles or topic based articles.

First, create a ListBox with the corresponding DataTemplate (I am only using question and answer text for this demo).

``` xml
<ListBox x:Name="FAQListBox">
<ListBox.ItemTemplate>
    <DataTemplate>
        <Grid>
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"></RowDefinition>
                <RowDefinition Height="Auto"></RowDefinition>
            </Grid.RowDefinitions>
            <TextBlock Grid.Row="0" x:Name="questionTB" Style="{StaticResource PhoneTextTitle2Style}" Text="{Binding question}" TextWrapping="Wrap"></TextBlock>
            <TextBlock Grid.Row="1" x:Name="answerTB" Style="{StaticResource PhoneTextSubtleStyle}" Text="{Binding text}" TextWrapping="Wrap"></TextBlock>
        </Grid>
    </DataTemplate>
</ListBox.ItemTemplate>
</ListBox>
```
 
I am using [JSON.net](https://james.newtonking.com/json) for everything around JSON strings. Here is how to deserialize the JSON string and set the ItemsSource of our Listbox:

``` csharp
articleJsonString = await GetKBJsonString();

var articlesList = JsonConvert.DeserializeObject<KBArticleDataClass.KBArticleData>(articleJsonString);

FAQListBox.ItemsSource = articlesList.articles;
```
 
You are now already able to run the project. Here is the result of my test app, displaying the FAQ of my [NFC Toolkit](https://www.windowsphone.com/s?appid=2c33cb7d-c97b-4204-aa8b-1e8712718519) app:

![uservoice_listbox_testapp_screenshot](/assets/img/2014/01/uservoice_listbox_testapp_screenshot.png "uservoice_listbox_testapp_screenshot")

As you can see, it takes only about ten minutes to get the knowledge base into your app. Using a remote source has a lot of advantages, the most important one is you don’t need to update your app when you add new answered questions.

I am now starting to work on integrating the feedback forum. It requires an oAuth authentication, and will be a bit more complicated than this one. Of course I will share it with you all here on my blog – stay tuned.

Until then – happy coding!