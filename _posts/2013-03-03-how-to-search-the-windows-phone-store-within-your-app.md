---
id: 3511
title: 'How to search the Windows Phone Store within your app'
date: '2013-03-03T21:55:45+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-search-the-windows-phone-store-within-your-app/
categories:
    - Archive
tags:
    - apps
    - search
    - 'Windows 8'
    - 'Windows Phone'
    - 'Windows Phone Store'
    - wp8dev
    - wpdev
---

![MPSearch](/assets/img/2013/03/MPSearch.png "MPSearch")

I am currently working on my NFC app, and I want to make it easier for the end user to search for the AppId which you need for the LaunchApp record. So I thought about a possible solution for this, and of course the easiest way is to search the app.

If you only want to search the Windows Phone Store and let show some search results, there is the [MarketplaceSearchTask](http://msdn.microsoft.com/en-us/library/windowsphone/develop/hh394001(v=vs.105).aspx), which you can call as a launcher. The only problem is, you cannot get any values into your app this way. But I found a way to get the results into my app. This post is about how I did it.

The first thing you will need to add to your project is the [HTMLAgilitypack](http://htmlagilitypack.codeplex.com/). It helps you parsing links from an HTML-based document. Huge thanks to [@AWSOMEDEVSIGNER](https://twitter.com/AWSOMEDEVSIGNER) (follow him!), who helped me to get started with it and understand XPath. Xpath is also important for the HAP to work with Windows Phone. You will need to reference to System.Xml.Xpath.dll, which you will find in

> %ProgramFiles(x86)%Microsoft SDKsMicrosoft SDKsSilverlightv4.0LibrariesClient or  
> %ProgramFiles%Microsoft SDKsMicrosoft SDKsSilverlightv4.0LibrariesClient

Ok, if we have this, we can continue in creating the search. Add a TextBox, a Button and a ListBox to your XAML:

``` xml
           <Grid.RowDefinitions>
                <RowDefinition Height="90"></RowDefinition>
                <RowDefinition Height="90"></RowDefinition>
                <RowDefinition Height="*"></RowDefinition>
            </Grid.RowDefinitions>
            <TextBox x:Name="MPsearchTerm" Height="80" Grid.Row="0"></TextBox>
            <Button x:Name="searchButton" Height="80" Grid.Row="1" Content="search" Click="searchButton_Click_1"></Button>
            <ListBox Grid.Row="2" x:Name="ResultListBox">
                <ListBox.ItemTemplate>
                    <DataTemplate>
                        <StackPanel Orientation="Horizontal">
                            <Image Source="{Binding AppLogo}" 
                                   Height="50" 
                                   Width="50" 
                                   Margin="10,0,0,0">
                            </Image>
                            <TextBlock Text="{Binding AppName}"
                                       FontSize="32"
                                       Margin="12,0,0,0" >
                            </TextBlock>

                        </StackPanel>

                    </DataTemplate>
                </ListBox.ItemTemplate>
            </ListBox>
```
 

After creating this, we will hook up the Click-Event of our Button to create our search. We are going to use the search of windowsphone.com to get all the information we want. You can parse any information that you need like ratings etc., but we focus on AppId, Name of the App and of course the store Logo of each app.

First, we need to create the search Uri. The Uri is country dependent like your phone. This is how we create the Uri:

``` csharp
string currentLanguage = CultureInfo.CurrentCulture.Name;
string searchUri = string.Format("http://www.windowsphone.com/{0}/store/search?q={1}", currentLanguage, MPsearchTerm.Text);
```
 
After that, we are using a WebClient to get the HTML-String of the search. I used the WebClient as I want to make it usable on WP7.x and WP8.

``` csharp
//start WebClient (this way it will work on WP7 & WP8)
WebClient MyMPSearch = new WebClient();
 //Add this header to asure that new results will be downloaded, also if the search term has not changed
// otherwise it would not load again the result string (because of WP cashing)
MyMPSearch.Headers[HttpRequestHeader.IfModifiedSince] = DateTime.Now.ToString();
//Download the String and add new EventHandler once the Download has completed
 MyMPSearch.DownloadStringCompleted += new DownloadStringCompletedEventHandler(MyMPSearch_DownloadStringCompleted);
MyMPSearch.DownloadStringAsync(new Uri(searchUri));
```
 
In our DownloadStringCompletedEvent we now are parsing the HTML-String. First thing we need to do is to create a HTML-Document that loads our string:

```
 chsarp
//HAP needs a HTML-Document as it is based on Linq/Xpath
HtmlDocument doc = new HtmlDocument();
doc.LoadHtml(e.Result);
```
 
The next step is a bit tricky if you do not know Xpath. You need to got through all the HTML elements to find the one that holds the data you want to parse. In our case it is the class called “medium” within the result table “appList”.

```
csharp
var nodeList = doc.DocumentNode.SelectNodes("//table[@class='appList']/tbody/tr/td[@class='medium']/a");
```
 
Note that you have to use ‘ instead of ” for the class names in Xpath. I recommend to open a sample search page in an internet browser and look into the code view of the page to find the right path.

![image](/assets/img/2013/03/image.png "image")

Now that we have a NodeList, we can parse the data we want:

``` csharp
foreach(var node in nodeList)
{
    //get AppId from Attributes
    cutAppID = node.Attributes["data-ov"].Value.Substring(5, 36);

    //get ImageUri for Image Source 
    var ImageMatch = Regex.Match(node.OuterHtml, "src");
    cutAppLogo = node.OuterHtml.Substring(ImageMatch.Index + 5, 92);

    // get AppTitle from node
    //Beginning of the AppTitle String
    var AppTitleMatch = Regex.Match(node.InnerHtml, "alt=");
    var StringToCut = Regex.Replace(node.InnerHtml.Substring(AppTitleMatch.Index),"alt="",string.Empty);
    //End of the AppTitle String
    var searchForApptitleEnd = Regex.Match(StringToCut, "">");
    //FinalAppName - cutting away the rest of the Html String at index of searchForApptitelEnd
    // if we won't do that, it would not display the name correctly
    cutAppName = StringToCut.Remove(searchForApptitleEnd.Index);
}
```
 
As you can see, we need to perform some String operations, but this is the easiest way I got the result I want – within my app. As always I hope this will be helpful for some of you.

You can download full working sample here: [MarketplaceSearch.zip](/assets/img/2013/03/MarketplaceSearch.zip)

Happy coding!