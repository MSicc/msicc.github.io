---
id: 4084
title: 'How to connect your Windows Phone 8 &#038; 8.1 app to Yammer'
date: '2014-06-08T19:20:32+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-connect-your-windows-phone-8-8-1-app-to-yammer/
categories:
    - Archive
tags:
    - authentication
    - enterprise
    - oAuth
    - social
    - 'social networks'
    - 'uri scheme'
    - WebAuthenticationBroker
    - 'Windows Phone 8.1'
    - WP8.1
    - wpdev
    - Yammer
---

In my recent project UniShare I also added Yammer, because some of users asked for it and we use it also at Telefónica.

This post is all about how to connect you Yammer. First, we need to prepare our app. Five steps are needed for preparation.

First, we need an uri scheme to launch our app. To add an uri scheme, right click on the WMAppManifest.xml and click on ‘Open With…’ and select ‘XML (Text) Editor with Encoding’. After the ‘Tokens’ section, add your custom uri scheme within the ‘Extensions’ tag, for example:

``` xml
       <Protocol Name="yourapp-yammercallback" NavUriFragment="encodedLaunchUri=%s" TaskID="_default" />
```
 
Next step is to add a proper UriMapper:
 
``` csharp
 public class UriMapper : UriMapperBase
{
    public override Uri MapUri(Uri uri)
    {
        if (uri.ToString().Contains("yourapp-yammercallback"))
        {
            string decodedUri = HttpUtility.UrlDecode(uri.ToString());

            int redirectUriIndex = decodedUri.IndexOf("yourapp-yammercallback");
            string redirectParams = new Uri(decodedUri.Substring(redirectUriIndex)).Query;
            // Map the OAuth response to the app page
            return new Uri("/MainPage.xaml" + redirectParams, UriKind.Relative);
        }

        else return uri;
    }
}
```
 
If you want to learn more about uri schemes, you can read [my blog post here]({% post_url 2013-12-08-why-windows-phone-apps-deserve-a-custom-uri-scheme-and-how-to-add-a-simple-launch-uri %}).

The third step is to generate your app on Yammer to get your Tokens. Open [https://www.yammer.com/client\_applications](https://www.yammer.com/client_applications), click on ‘Register New App’ and fill in the form:

![Screenshot (354)](/assets/img/2014/06/Screenshot-354.png "Screenshot (354)")

After clicking on continue, click on ‘Basic Info’ and add your uri scheme in the field ‘Redirect Uri’ and click on ‘Save’.

![Screenshot (356)](/assets/img/2014/06/Screenshot-356.png "Screenshot (356)")

The fourth step starts with downloading the [Yammer oAuth SDK for Windows Phone from Github](https://github.com/yammer/windows-phone-oauth-sdk-demo/archive/master.zip). Open the solution and built all projects. After that, you can close the solution. Go back into your project, and right click on ‘References’, followed by ‘Add Reference’. Browse to the folder ‘\\WPYammer-oauth-sdk-demo-master\\windows-phone-oauth-sdk-demo-master\\Yammer.OAuthSDK\\Bin\\Release’ and add the file with the name ‘Yammer.OAuthSDK.dll’.

The last step is to add your keys and the redirect uri in your app’s resource dictionary:

``` xml
 <model:OAuthClientInfo xmlns:model="clr-namespace:Yammer.OAuthSDK.Model;assembly=Yammer.OAuthSDK" x:Key="MyOAuthClientInfo"
    ClientId="your client id" 
    ClientSecret="your client secret" 
    RedirectUri="yourapp-yammercallback" />
```
 
Now that our app is prepared, we can finally launch the oAuth process. First, we need to read our values from our app’s resource dictionary. Add this to the constructor to do so:

``` csharp
 YammerClientId = App.MyOAuthClientInfo.ClientId;
YammerClientSecret = App.MyOAuthClientInfo.ClientSecret;
YammerCallbackUri = App.MyOAuthClientInfo.RedirectUri;
```
 
To kick the user out to the oAuth process, which happens in the phone’s browser, just add these two lines to your starting event (for example a button click event).

``` csharp
 OAuthUtils.LaunchSignIn(YammerClientId, YammerCallbackUri);
App.Current.Terminate();
```
 
You might notice the app gets terminated. We have to do this because only this way, the uri scheme we added earlier can do its work. If the app is not terminated, the values cannot be passed into our app. Technically, it would also be possible to use the WebAuthenticationBroker on 8.1 for this. Sadly, the Yammer authorization pages do not work well with the WAB. The best way to use it with this library, is to kick the user to the browser.

Once we are receiving the values of the authentication, we can continue the authentication process to get the final access token. Add this code to your OnNavigatedTo event:

``` csharp
 ShowProgressIndicator("connecting to Yammer...");

if (NavigationContext.QueryString.ContainsKey(Constants.OAuthParameters.Code) && NavigationContext.QueryString.ContainsKey(Constants.OAuthParameters.State) && e.NavigationMode != NavigationMode.Back)
{
    OAuthUtils.HandleApprove(
        YammerClientId,
        YammerClientSecret,
        NavigationContext.QueryString[Constants.OAuthParameters.Code],
        NavigationContext.QueryString[Constants.OAuthParameters.State],
        onSuccess: () =>
        {
            GetYammerCurrentUserData();
        }, onCSRF: () =>
        {
            MessageBox.Show("Please contact us via the 'help & support' page.", "Invalid redirect", MessageBoxButton.OK);
        }, onErrorResponse: errorResponse =>
        {
            Dispatcher.BeginInvoke(() => MessageBox.Show(errorResponse.OAuthError.ToString(), "Invalid operation", MessageBoxButton.OK));
            HideProgressIndicator();
        }, onException: ex =>
        {
            Dispatcher.BeginInvoke(() => MessageBox.Show(ex.ToString(), "Unexpected error", MessageBoxButton.OK));
            HideProgressIndicator();
        }
    );
}
// "Deny"
else if (NavigationContext.QueryString.ContainsKey(Constants.OAuthParameters.Error) && e.NavigationMode != NavigationMode.Back)
{
    string error, errorDescription;
    error = NavigationContext.QueryString[Constants.OAuthParameters.Error];
    NavigationContext.QueryString.TryGetValue(Constants.OAuthParameters.ErrorDescription, out errorDescription);

    string msg = string.Format("error: {0}\nerror_description:{1}", error, errorDescription);
    MessageBox.Show(msg, "Error", MessageBoxButton.OK);

    if (NavigationContext.QueryString != null)
    {
        string[] NavigationKeys = NavigationContext.QueryString.Keys.ToArray();
        ClearNavigationContext(NavigationKeys);
    }

    OAuthUtils.DeleteStoredToken();

}
```
 
With this, you are handling all errors and events properly. If the call ends successfully, we are finally able to get our user’s data:

``` csharp
 public void GetYammerCurrentUserData()
{
    ShowProgressIndicator("connecting to Yammer...");

    var baseUrl = new Uri("https://www.yammer.com/api/v1/users/current.json", UriKind.Absolute);

    OAuthUtils.GetJsonFromApi(baseUrl,
        onSuccess: response =>
        {
            //do something with the result (json string)

            HideProgressIndicator();

        }, onErrorResponse: errorResponse =>
        {
            Dispatcher.BeginInvoke(() =>
            {
                MessageBox.Show(errorResponse.OAuthError.ToString(), "Invalid operation", MessageBoxButton.OK);
                HideProgressIndicator();

            });
        }, onException: ex =>
        {
            Dispatcher.BeginInvoke(() =>
            {
                MessageBox.Show(ex.ToString(), "Unexpected error", MessageBoxButton.OK);
                HideProgressIndicator();

            });
        }
    );
}
```
 
As you can see above, the result is a json string that contains the current user’s data. You can either create a class/model to deserialize the values or use anonymous deserialization methods.

One important point: you cannot publish your app outside your home network until it got approved as a global app by Yammer. I am waiting for nearly two weeks now for UniShare to get approved and hope it will become available for all soon.

As always, I hope this post is helpful for some of you.

Happy coding!