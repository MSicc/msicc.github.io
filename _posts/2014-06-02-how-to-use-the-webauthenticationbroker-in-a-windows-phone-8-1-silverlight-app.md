---
id: 4074
title: 'How to use the WebAuthenticationBroker in a Windows Phone 8.1 Silverlight app'
date: '2014-06-02T05:37:12+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-use-the-webauthenticationbroker-in-a-windows-phone-8-1-silverlight-app/
categories:
    - Archive
tags:
    - AuthenticateAndContinue
    - authentication
    - Continuation
    - networking
    - oAuth
    - Silverlight
    - 'social networks'
    - WebAuthenticationBroker
    - 'Windows Phone 8.1'
    - WP8.1
    - wpdev
---

![oAuthDance](/assets/img/2014/06/oAuthDance.png "oAuthDance")

I already wrote [how to use the WebAuthenticationBroker (WAB) in a Windows Phone Runtime app]({ post_url 2014-04-19-how-to-use-the-webauthenticationbroker-for-oauth-in-a-windows-phone-runtime-wp8-1-app }). With the need for me to switch back to Silverlight for UniShare, I needed to perform some rewriting of the WAB code.

The WAB needs some other code to work properly in a Silverlight project, and this post will got through all steps that are needed for this. I will reference to the above blog post where the methods itself are the same. I am just highlighting the changes that are needed in an 8.1 Silverlight project.

### Preparing our App

First, we need to set up the proper continuation event in App.xaml.cs. We are doing this by adding this line of code in the App class:

``` csharp
 public WebAuthenticationBrokerContinuationEventArgs WABContinuationArgs { get; set; }
```
 
This event is used when we are coming back to our app after the WAB logged the user in. The next step we need to do, is to tell our app that we have performed an authentication, and pass the values back to our main code. Unlike in the Runtime project, we are using the Application\_ContractActivated event for this:

``` csharp
         private void Application_ContractActivated(object sender, Windows.ApplicationModel.Activation.IActivatedEventArgs e)
        {
            var _WABContinuationArgs = e as WebAuthenticationBrokerContinuationEventArgs;
            if (_WABContinuationArgs != null)
            {
                this.WABContinuationArgs = _WABContinuationArgs;
            }
        }
```
 
If you are upgrading from a WP8 project, you might have to add this event manually. Our app is now prepared to get results from the WAB.

### Preparing oAuth

I am going to show you the oAuth process of Twitter to demonstrate the usage of the WAB. First, we need again the GetNonce(), GetTimeStamp () and GetSignature(string sigBaseString, string consumerSecretKey, stringoAuthTokenSecret=null) methods [from my former blog post.]({ post_url 2014-04-19-how-to-use-the-webauthenticationbroker-for-oauth-in-a-windows-phone-runtime-wp8-1-app })

### Performing the oAuth process

In the oAuth authentication flow, we need to obtain a so called Request Token, which allows our app to communicate with the API (in this case Twitter’s API).

Add the following code to your “connect to Twitter”- Button event:

``` csharp
                         string oAuth_Token = await GetTwitterRequestTokenAsync(TwitterCallBackUri, TwitterConsumerKey);
                        string TwitterUrl = "https://api.twitter.com/oauth/authorize?oauth_token=" + oAuth_Token;

                        WebAuthenticationBroker.AuthenticateAndContinue(new Uri(TwitterUrl), new Uri(TwitterCallBackUri));
```
 
Like in a Runtime app, we are getting the request token first (code is also in my [former blog post]({ post_url 2014-04-19-how-to-use-the-webauthenticationbroker-for-oauth-in-a-windows-phone-runtime-wp8-1-app }), once we obtained the request token, we are able to get the oAuth token that enables us to get the final user access tokens.

Once the user has authenticated our app, we’ll receive the above mentioned oAuth tokens. To use them, add the following code to your OnNavigatedTo event:

``` csharp
                 var appObject = Application.Current as App;

                if (appObject.WABContinuationArgs != null)
                {
                    WebAuthenticationResult result = appObject.WABContinuationArgs.WebAuthenticationResult;

                    if (result.ResponseStatus == WebAuthenticationStatus.Success)
                    {
                        await GetTwitterUserNameAsync(result.ResponseData.ToString());
                    }
                    else if (result.ResponseStatus == WebAuthenticationStatus.ErrorHttp)
                    {
                        MessageBox.Show(string.Format("There was an error connecting to Twitter: \n {0}", result.ResponseErrorDetail.ToString()), "Sorry", MessageBoxButton.OK);

                    }
                    else
                    {
                        MessageBox.Show(string.Format("Error returned: \n{0}", result.ResponseStatus.ToString()), "Sorry", MessageBoxButton.OK);
                        HideProgressIndicator();
                    }
                }
```
 
The WebAuthenticationResult now holds all values that we need to perform the final actions. To complete the oAuth process on Twitter, you can use the GetTwitterUserNameAsync(string webAuthResultResponseData) method from my [former blog post]({ post_url 2014-04-19-how-to-use-the-webauthenticationbroker-for-oauth-in-a-windows-phone-runtime-wp8-1-app }). If you are not using other methods to control the result of the WAB, don’t forget to set appObject.WABContinuationArgs to null after you finished obtaining all tokens and data from Twitter (or other services).

As you can see, there are some structural differences in using the WAB when creating a Silverlight app, but we are also able to use a lot of code from my Runtime project. I hope this post is helpful for some of you to get the oAuth dance going.

In my next post, I will show you how to authenticate your app with Yammer (Microsoft’s enterprise social network).

Until then, happy coding!