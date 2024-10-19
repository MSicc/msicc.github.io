---
id: 3944
title: 'How to authenticate users with the uservoice API on Windows Phone'
date: '2014-01-19T19:13:59+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-authenticate-users-with-the-uservoice-api-on-windows-phone/
categories:
    - Archive
tags:
    - authentication
    - feedback
    - oAuth
    - RestSharp
    - suggestion
    - support
    - uservoice
    - WinPhanDev
    - wpdev
---

I am really enjoying playing around with the uservoice API. It enables me to integrate a lot of cool new features that take the user experience to a new level, while helping me to improve my apps.

Today, I am going to write about how to authenticate a user with the uservoice API.

uservoice uses the oAuth authentication in version 1.0a. Therefore, we need a consumer key and a consumer secret for our app. Log into &lt;yourname&gt;.uservoice.com, and go to ‘Settings/Integration’, where you create a new API client for your app.

I recommend to set it up as a trusted client, as you will have more rights with a trusted client. After your set up your app, you should have an entry like this under integrations:

![uservoice_api_client](/assets/img/2014/01/uservoice_api_client1.png "uservoice_api_client")

Now we have everything set up on the uservoice part. Let’s go to our app. I am using the [RestSharp](http://restsharp.org/) library that makes it a little easier to authenticate users. You should do the same if you want to follow along this post. In Visual Studio, right click the project name and select ‘Manage NuGet packages’, and search for RestSharp in the ‘Online’ section.

After we have integrated the RestSharp library in our app, we are going to set up some objects for authentication:

``` csharp
  const string ConsumerKey = "<yourkey>";
 const string ConsumerSecret = "<yoursecret>";
 static string oAuthToken = "";
 static string oAuthTokenSecret = "";
 static string oAuthVerifier = "";
 static string AccessToken = "";
 static string AccessTokenSecret = "";
 const string BaseUrl = "http://<yourname>.uservoice.com";
 const string oAuthCallBackUri = "<yourcallbackUrl>";
 const string requestTokenPath = "oauth/request_token";
 const string authorizePath = "/oauth/authorize?";
 const string accessTokenPath = "oauth/access_token";
```
 
Setting up these static and constant string object are needed for our authentication flow.

Next, add a WebBrowser control in XAML. We need this to let the user authorize our app to communicate with the uservoice servers:

``` xml
 <Grid x:Name="authBrowserGrid" Visibility="Collapsed">
 <phone:WebBrowser x:Name="authBrowser" Height="800" Width="460"></phone:WebBrowser>
 </Grid>
```
 
You might have noticed that I set the Visibility property to collapsed. With oAuth, we are typically authenticating users by referring them to the web site, that handles the login process for us. After the user has authorized our app, the website transfers the user to our callback url that we define with our API client.

As I told you already above, RestSharp helps us a lot when it comes to authentication.

There are several ways to use the RestSharp API, but for the moment, I am using it directly to authenticate my users.

This is the start method:

``` csharp
 public void GetUserAuthenticated()
 {

            authBrowserGrid.Visibility = Visibility.Visible;

            var client = new RestClient(BaseUrl)
 {
 Authenticator = OAuth1Authenticator.ForRequestToken(ConsumerKey, ConsumerSecret, oAuthCallBackUri)
 };

            var request = new RestRequest(requestTokenPath, Method.GET);
 var response = client.ExecuteAsync(request, HandleRequestTokenResponse);
 }
```
 
We are creating a new RestClient that calls our BaseUrl which we defined before. In the same call, we are telling our client that we want to use oAuth 1.0(a), passing the parameters ConsumerKey, ConsumerSecret and our callback url with the client. As we want to receive a RequestToken, we are generating a new RestRequest, passing the requestTokenPath and the ‘GET’ method as parameters.

Finally, we want to continue with the response, that should contain our RequestToken. Windows Phone only supports the ExecuteAsync method, which needs a specific response handler to be executed. That was the first lesson I had to learn with using RestSharp on Windows Phone. To handle the response, we are creating a new handler method that implements ‘IRestResponse’:

``` csharp
 private void HandleRequestTokenResponse(IRestResponse restResponse)
 {
    var response = restResponse;

          //tbd: write an extraction helper for all tokens, verifier, secrets
 oAuthToken = response.Content.Substring((response.Content.IndexOf("oauth_token=") + 12), (response.Content.IndexOf("&oauth_token_secret=") - 12));
 var tempStringForGettingoAuthTokenSecret = response.Content.Substring(response.Content.IndexOf("&oauth_token_secret=") + 20);
 oAuthTokenSecret = tempStringForGettingoAuthTokenSecret.Substring(0,  tempStringForGettingoAuthTokenSecret.IndexOf("&oauth_callback_confirmed"));

var authorizeUrl = string.Format("{0}{1}{2}",BaseUrl, authorizePath, response.Content);

           authBrowser.Navigate(new Uri(authorizeUrl));
 authBrowser.IsScriptEnabled = true;
 authBrowser.Navigating += authBrowser_Navigating;

}
```
 
We now have an object that we are able to work with (the variable response). We need to extract the oauth\_token and the oauth\_token\_secret and save them for later use.

After that, we can build our authorization url, that we pass to the WebBrowser control we created earlier. Important: you should set the ‘IsScriptEnabled’ property to true, as authentication websites often use scripts to verify the entered data. Last but not least, we are subscribing to the Navigating event of the browser control. In this event, we are handling the response to the authorization:

``` csharp
 void authBrowser_Navigating(object sender, NavigatingEventArgs e)
 {
      if (e.Uri.AbsolutePath == "<yourAbsolutePath>")
      {
           oAuthVerifier = e.Uri.Query.Substring(e.Uri.Query.IndexOf("&oauth_verifier=") + 16);

           GetAccessToken();
      }
 }
```
 
We only need the verification string for out next request we have to send, so we are extracting the parameter “oauth\_verifier” and going over to get our AccessToken for all further requests:

``` csharp
 private void GetAccessToken()
 {
 var client = new RestClient(BaseUrl)
 {
 Authenticator = OAuth1Authenticator.ForAccessToken(ConsumerKey, ConsumerSecret, oAuthToken, oAuthTokenSecret, oAuthVerifier)
 };

            var request = new RestRequest(accessTokenPath, Method.GET);
 var response = client.ExecuteAsync(request, HandleAccessTokenResponse);

        }
```
 
This time, we are telling our RestClient that we want to get the AccessToken, and we need also the oAuthToken and the oAuthTokenSecret we saved before we navigated our users to the uservoice authentication site. Of course, we also need a handler for the response to this request:

``` csharp
 private void HandleAccessTokenResponse(IRestResponse restResponse)
 {
 var response = restResponse;

            AccessToken = response.Content.Substring(12, response.Content.IndexOf("&oauth_token_secret=") -12);
 AccessTokenSecret = response.Content.Substring(response.Content.IndexOf("&oauth_token_secret=") + 20);

            authBrowserGrid.Visibility = Visibility.Collapsed;

            //continue with your code (new method call etc.)
 }
```
 
The only thing we now need to do is to extract the AccessToken and the AccessTokenSecret and save them permanently in our app (as long as the user is authenticated). We need them for a lot of calls to the uservoice API.

Let’s call the uservoice API to get more information about the user that has now authorized our app:

``` csharp
 public void GetUser()
 {
 var client = new RestClient(BaseUrl)
 {
 Authenticator = OAuth1Authenticator.ForProtectedResource(ConsumerKey, ConsumerSecret, AccessToken, AccessTokenSecret)
 };

           var request = new RestRequest(getUserPath, Method.GET);
 var response = client.ExecuteAsync(request, HandleGetUserResponse);
 }
```
 
As you can see, we are now only using the AccessToken and the AccessTokenSecret to call the uservoice API, no additional login is required. To complete this sample, here is again the handler for the response:

``` csharp
         private void HandleGetUserResponse(IRestResponse restResponse)
       {
              var response = restResponse;

            //tbd: do something with the result (e.g. checking response.StatusCode)

       }
```
 
We have now received a JSON string that tells us a lot of information about our user and the date we have on uservoice:

![Screenshot (304)](/assets/img/2014/01/Screenshot-304.png "Screenshot (304)")

As you can see, it is relatively easy to get our users authenticated and calling the uservoice API. In my next blog post, I will write about suggestions (that’s the idea forum). I will go into details on getting a list of all suggestions, how to let a user post a new idea and letting a user vote for ideas – all from within your app!

In the meantime, I hope this post is helpful for some of you.

Happy coding, everyone!