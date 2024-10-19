---
id: 3398
title: 'Dev Story Series (Part 2 of many): Getting recent posts from WordPress into your Windows Phone and Windows 8 app'
date: '2013-01-12T09:37:05+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /dev-story-series-part-2-of-many-getting-recent-posts-from-wordpress-into-your-windows-phone-and-windows-8-app/
categories:
    - Archive
tags:
    - apps
    - binding
    - Dev
    - json
    - win8dev
    - WordPress
    - wpdev
    - XAML
---

Now that we have a [full WordPress JSON class](http://msicc.net/?p=2929), we are able to download our recent posts from WordPress into our apps for Windows Phone and Windows 8. I am still not using MVVM to keep it simple (and because I have to dive into it more deeply).

The first thing we need to do is to download the JSON string for the recent posts. The Uri scheme is pretty simple: *{yourblogadresshere}?json=get\_recent\_posts*

I declared a public string in my MainPage for that, so it is very easy to use it in our app.

The second thing we are going to do is to download the JSON string into the app.

For Windows Phone I used a WebClient, as I want to keep it compatible with the Windows Phone 7 OS. I will update the App with an dedicated WP8 version later, for the moment it is working on both OS versions. Add this code to you Page\_Loaded event:

``` csharp
              WebClient GetPostsClient = new WebClient();
                GetPostsClient.Headers[HttpRequestHeader.IfModifiedSince] = DateTime.Now.ToString();
                GetPostsClient.DownloadStringCompleted += new DownloadStringCompletedEventHandler(GetPostsClient_DownloadStringCompleted);
                GetPostsClient.DownloadStringAsync(new Uri(RecentPostJsonUri));
```
 

We will also have to add the Handler for GetPostsClient\_DownloadStringCompleted:

``` csharp
 void GetPostsClient_DownloadStringCompleted(object sender, DownloadStringCompletedEventArgs e)
        {
            App.jsonString_result = e.Result;
        }
```
 

In Windows 8 there is no WebClient, so I used an HttpClient:

``` csharp
                         HttpClient getJsonStringClient = new HttpClient();
                        getJsonStringClient.DefaultRequestHeaders.IfModifiedSince = DateTime.UtcNow;
                        App.jsonString_result = await getJsonStringClient.GetStringAsync(RecentPostJsonUri);
```
 

Both the Windows Phone and the Windows 8 apps are downloading the string asynchronously, the UI is reliable all the time. You may have noticed the additional Header that I request. This way, we are able to integrate a refresh function into our app. If we leave this out, our app uses the cached string, and users will have to exit the app to refresh the list of our posts.

You will have to declare a public static string variable for the downloaded string in App.xaml.cs, that keeps the downloaded string accessible through the whole app.

Until now we have only downloaded our JSON String, which looks like this:

![image](/assets/img/2013/01/image.png "image")

**Side note:** The WordPress JSON API has a dev mode. Just add “&amp;dev=1” to your above created Uri, and you will be able to see the whole JSON string in a readable form in your browser.

Back to our topic. Off course this is not a good format for users. They want to see only the content, without all the formatting and structuring code around.

What we need to do, is to deserialize our JSON String. This is possible with Windows Phone and Windows 8 own API, but I highly recommend to use the JSON.net library. You can download and learn more about it [here](http://json.codeplex.com/). To install the library, just go to Tools&gt;Library Package Manager&gt;Manage NuGet Packages for Solution, search for JSON.net, and install it.

After installing the package, we are able to use only one line of code to deserialize our JSON String to our data members:

``` csharp
var postList = JsonConvert.DeserializeObject<Posts>(App.jsonString_result);
```
 

Now we need the deserialized data to be displayed to the user. The desired control for Windows Phone is a ListBox, for Windows 8 you it is called ListView. We need to create an ItemTemplate in XAML and bind the data we want to show to the user (Just change ListBox to ListView for Windows 8 in XAML):

``` csharp
 <ListBox x:Name="PostListBox">
                <ListBox.ItemTemplate>
                    <DataTemplate>
				<StackPanel>
 				<Image x:Name="PostImage" 
				       Source="{Binding thumbnail}" />
                           	<TextBlock x:Name="TitleTextBlock" 
				           Text="{Binding title}" 
					   TextWrapping="Wrap" 
					   FontSize="20" />
                                <TextBlock x:Name="PublishedTextBlock" 
					   Text="{Binding date}" 
					   FontSize="12"/>
				</StackPanel>
                    </DataTemplate>
                </ListBox.ItemTemplate>
          </ListBox>
```
 

As you can see, we have set some Bindings in the code above. This Bindings rely on the DataContract Post, as every ListBox/ListView-Item represents one Post of our postList.

``` csharp
 [DataContract]
public class Post
    {
        [DataMember]
        public int id { get; set; }
        [DataMember]
        public string type { get; set; }
        [DataMember]
        public string slug { get; set; }
        [DataMember]
        public string url { get; set; }
        [DataMember]
        public string status { get; set; }
        [DataMember]
        public string title { get; set; }
        [DataMember]
        public string title_plain { get; set; }
        [DataMember]
        public string content { get; set; }
        [DataMember]
        public string excerpt { get; set; }
        [DataMember]
        public string date { get; set; }
        [DataMember]
        public string modified { get; set; }
        [DataMember]
        public List<Category> categories { get; set; }
        [DataMember]
        public List<object> tags { get; set; }
        [DataMember]
        public Author author { get; set; }
        [DataMember]
        public List<comment> comments { get; set; }
        [DataMember]
        public List<Attachment> attachments { get; set; }
        [DataMember]
        public int comment_count { get; set; }
        [DataMember]
        public string comment_status { get; set; }
        [DataMember]
        public string thumbnail { get; set; }
    }
```
 

Choose the fields you want to display to create your own DataTemplate to show only the data you want. Last but not least we have to tell our app that the ItemSource of our ListBox is the deserialized list, which is also done easily:

``` csharp
PostListBox.ItemsSource = postList.posts;
```
 

If you now hit F5 on your keyboard, the app should be built and the device/emulator should show your recent posts in a list. You don’t need to add additional code to download the images, as the image source points already to an image and will be downloaded automatically.

**Pro-Tip:**

The thumbnails from the our DataContract Post are looking really ugly sometimes. To get a better looking result in your ListBox/ListView, I recommend to use the attached images. To do this, you will need the following code:

``` csharp
          foreach (var item in postList.posts)
            {
                var postImagefromAttachement = item.attachments.FirstOrDefault();
                if (postImagefromAttachement == null)
                {
                    item.thumbnail = placeholderImage; //add your own placeholderimage here
                }
                else
                {
                    item.thumbnail = postImagefromAttachement.images.medium.url;
                }

            }
```
 

This code checks your list of attachments in your post, takes the first image, and downloads a higher quality (medium/full). I am using medium to get best results on quality and download speed.

I hope this is helpful for some of you to get forward for to create a WordPress blog app on Windows Phone and Windows 8.

Happy coding!