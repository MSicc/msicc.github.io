---
id: 4054
title: 'How to use the WebAuthenticationBroker for oAuth in a Windows Phone Runtime WP8.1 app'
date: '2014-04-19T16:17:03+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-use-the-webauthenticationbroker-for-oauth-in-a-windows-phone-runtime-wp8-1-app/
categories:
    - Archive
tags:
    - authentication
    - ContinuationManager
    - oAuth
    - Tokens
    - WebAuthenticationBroker
    - 'Windows Phone'
    - WP81
    - wpdev
---

![oAuthDance](/assets/img/2014/04/oAuthDance.png "oAuthDance")

After playing around with WP8.1 for a few days (like everyone else), I decided to dig a bit into development of WP8.1.

As oAuth is the common authentication method nowadays for Apps and Websites, I was curios about the implementation of the WebAuthenticationBroker in a WINPRT app.

I used it before with Windows 8 for my TweeCoMinder app, but that’s a few month back (it didn’t make it into the Store yet (another goal, right – porting TweeCoMinder to Universal will be a lot of fun and learning for me ;-)).

Before we continue: This is a pretty huge topic. Be prepared that it may take you more than one time to read and understand what is going on.

Let’s dive into it. Unlike the Windows WebAuthenticationBroker, the Phone version does not use the [AuthenticateAsync](http://msdn.microsoft.com/en-us/library/windows/apps/windows.security.authentication.web.webauthenticationbroker.authenticateasync.aspx "AuthenticateAsync authenticateAsync methods") method. It uses [AuthenticateAndContinue](http://msdn.microsoft.com/en-us/library/windows/apps/windows.security.authentication.web.webauthenticationbroker.authenticateandcontinue.aspx "AuthenticateAndContinue authenticateAndContinue methods") instead. This is related to the lifecycle on phone, as it is more likely that an WINPRT app is suspended than on Windows (at least that’s the official reason).

### Preparing our App

But we are able to get it working, no worries. First, we need the so called ContinuationManager. This class brings the user back to the app where the fun begun.

You can download the complete class from [here](/assets/img/2014/04/ContinuationManager.zip) (it is a 1:1 copy from the official MSDN reference). The only thing you need to do is to add your app’s Namespace into it.

The next step we need to do: some modifications at the App.xaml.cs file.

1\. Add an OnActivated event (it isn’t there in the Phone template) \[updated: added missing continuationManager initialization\]

``` csharp
         protected async override void OnActivated(IActivatedEventArgs e)
        {

            continuationManager = new ContinuationManager();

            CreateRootFrame();

            // Restore the saved session state only when appropriate
            if (e.PreviousExecutionState == ApplicationExecutionState.Terminated)
            {
                try
                {
                    await SuspensionManager.RestoreAsync();
                }
                catch (SuspensionManagerException)
                {
                    //Something went wrong restoring state.
                    //Assume there is no state and continue
                }
            }

            //Check if this is a continuation
            var continuationEventArgs = e as IContinuationActivatedEventArgs;
            if (continuationEventArgs != null)
            {
                continuationManager.Continue(continuationEventArgs);
            }

            Window.Current.Activate();
        }
```
 
First, we check the SuspensionManager and let him restore a saved state – if there is one. If you do not have a Folder ”Common” with the SuspensionManager, just add a new Basic page. This will generate the Common folder with the SuspenstionManager class for you.

After that, we are checking if the activation is a Continuation. We need this check there, otherwise the app will not be able to receive the Tokens after returning from the WebAuthenticationBroker. Note: declare the ContinuationManager globally in App.xaml.cs with this to avoid multiple instances (which will crash the app for sure).

``` csharp
 public static ContinuationManager continuationManager { get; private set; }
```
 
2\. Add a CreateRootFrame() method with some little changes to the default behavior

``` csharp
         private void CreateRootFrame()
        {
            // Do not repeat app initialization when the Window already has content,
            // just ensure that the window is active
            if (rootFrame != null)
                return;

            // Create a Frame to act as the navigation context and navigate to the first page
            rootFrame = new Frame();

            //Associate the frame with a SuspensionManager key                                
            SuspensionManager.RegisterFrame(rootFrame, "AppFrame");

            // Set the default language
            rootFrame.Language = Windows.Globalization.ApplicationLanguages.Languages[0];

            rootFrame.NavigationFailed += OnNavigationFailed;

            // Place the frame in the current Window
            Window.Current.Content = rootFrame;
        }
```
 
This simply creates the RootFrame for our app. Place this one in the OnLaunched and OnActivated events. Instead of creating the RootFrame in the OnLaunched method, declare it globally in App.xaml.cs with this:

``` csharp
 public static Frame rootFrame { get; set; }
```
 
3\. Handle the HardwareButtons\_BackPressed event

``` csharp
         private void HardwareButtons_BackPressed(object sender, BackPressedEventArgs e)
        {
            Frame frame = Window.Current.Content as Frame;
            if (frame == null)
            {
                return;
            }

            if (frame.CanGoBack)
            {
                frame.GoBack();
                e.Handled = true;
            }
        }
```
 
This just handles the BackButton press globally (you can still handle it on individual pages/frames/controls).

4\. Handle Navigation Errors in the OnNavigationFailed event

``` csharp
 private void OnNavigationFailed(object sender, NavigationFailedEventArgs e)
{

    //change this code for your needs
    throw new Exception("Failed to load Page " + e.SourcePageType.FullName);
}
```
 
5\. Add the Save method to the OnSuspending event

``` csharp
 await SuspensionManager.SaveAsync();
```
 
This is just to save the current state of our app while the App gets suspended.

### Preparing oAuth

Now, we have our app prepared to be continued after the WebAuthenticationBroker has finished. Pretty much to do so far, but now we are going to start with the real fun: The oAuth process. We are going to use Twitter in this case, as it is very popular to be added as a sharing option in apps (yes, I recently saw a lot of them).

Before we are able to connect to Twitter, we need some preparing methods. The first one is to get a random character chain. Twitter demands a 32 digit chain that contains random alphanumeric values, the so called oAuth-nonce. A lot of samples use a chain that is shorter, which will be fine for the initial authentication, but not for additional request. Here is my method:

``` csharp
 string GetNonce()
{
   StringBuilder builder = new StringBuilder();
   Random random = new Random();
   char ch;
   for (int i = 0; i < 32; i++)
   {
     ch = Convert.ToChar(Convert.ToInt32(Math.Floor(26 * random.NextDouble() + 65)));
     builder.Append(ch);
   }

   return builder.ToString().ToLower();
 }
```
 
We also need a timestamp in Milliseconds, this one is pretty straight forward:

``` csharp
 string GetTimeStamp()
{
  TimeSpan SinceEpoch = DateTime.UtcNow - new DateTime(1970, 1, 1);
  return Math.Round(SinceEpoch.TotalSeconds).ToString();
}
```
 
On top, we need a signature for our Authentication request:

``` csharp
         string GetSignature(string sigBaseString, string consumerSecretKey, string oAuthTokenSecret=null)
        {
            IBuffer KeyMaterial = CryptographicBuffer.ConvertStringToBinary(consumerSecretKey + "&" + oAuthTokenSecret, BinaryStringEncoding.Utf8);
            MacAlgorithmProvider HmacSha1Provider = MacAlgorithmProvider.OpenAlgorithm("HMAC_SHA1");
            CryptographicKey MacKey = HmacSha1Provider.CreateKey(KeyMaterial);
            IBuffer DataToBeSigned = CryptographicBuffer.ConvertStringToBinary(sigBaseString, BinaryStringEncoding.Utf8);
            IBuffer SignatureBuffer = CryptographicEngine.Sign(MacKey, DataToBeSigned);
            string Signature = CryptographicBuffer.EncodeToBase64String(SignatureBuffer);

            return Signature;
        }
```
 
You will find a lot of samples that don’t use the oAuthTokenSecret in the basic method. We will need this for additional requests, so I am using a nullable string overload. This way, I need only one method after all.

### Performing the Authentication process

To start the Authentication process, we are using this code in our Button\_Click event:

``` csharp
         private async void connectToTwitterButton_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                string oAuth_Token = await GetTwitterRequestTokenAsync(TwitterCallBackUri, TwitterConsumerKey);
                string TwitterUrl = "https://api.twitter.com/oauth/authorize?oauth_token=" + oAuth_Token;

                WebAuthenticationBroker.AuthenticateAndContinue(new Uri(TwitterUrl), new Uri(TwitterCallBackUri), null, WebAuthenticationOptions.None);
            }
            catch
            { 
                //error handling here
            }
        }
```
 
First, we need a RequestToken that authenticates our request for the oAuthToken, that’s why we are awaiting the GetTwitterRequestTokenAsync(TwitterCallBackUri, TwitterConsumerKey) Task. The Task itself is this one:

``` csharp
         private async Task<string> GetTwitterRequestTokenAsync(string twitterCallbackUrl, string consumerKey)
        {
            //
            // Acquiring a request token
            //
            string TwitterUrl = "https://api.twitter.com/oauth/request_token";

            string nonce = GetNonce();
            string timeStamp = GetTimeStamp();
            string SigBaseStringParams = "oauth_callback=" + Uri.EscapeDataString(twitterCallbackUrl);
            SigBaseStringParams += "&" + "oauth_consumer_key=" + consumerKey;
            SigBaseStringParams += "&" + "oauth_nonce=" + nonce;
            SigBaseStringParams += "&" + "oauth_signature_method=HMAC-SHA1";
            SigBaseStringParams += "&" + "oauth_timestamp=" + timeStamp;
            SigBaseStringParams += "&" + "oauth_version=1.0";
            string SigBaseString = "GET&";
            SigBaseString += Uri.EscapeDataString(TwitterUrl) + "&" + Uri.EscapeDataString(SigBaseStringParams);
            string Signature = GetSignature(SigBaseString, TwitterConsumerSecret);

            TwitterUrl += "?" + SigBaseStringParams + "&oauth_signature=" + Uri.EscapeDataString(Signature);
            HttpClient httpClient = new HttpClient();
            string GetResponse = await httpClient.GetStringAsync(new Uri(TwitterUrl));

            string request_token = null;
            string oauth_token_secret = null;
            string[] keyValPairs = GetResponse.Split('&');

            for (int i = 0; i < keyValPairs.Length; i++)
            {
                string[] splits = keyValPairs[i].Split('=');
                switch (splits[0])
                {
                    case "oauth_token":
                        request_token = splits[1];
                        break;
                    case "oauth_token_secret":
                        oauth_token_secret = splits[1];
                        break;
                }
            }

            return request_token;
        }
```
 
After receiving the RequestToken, we are able to call the the [AuthenticateAndContinue(Uri, Uri, ValueSet, WebAuthenticationOptions)](http://msdn.microsoft.com/en-us/library/windows/apps/dn608116.aspx "AuthenticateAndContinue(Uri, Uri, ValueSet, WebAuthenticationOptions)") method. The WebAuthenticationBroker opens and the user needs to login. After finishing, the user will get redirected back into our app.

If you think that’s all of our code, I need to disappoint you. The fun goes on. We prepared our app to be recognized as continued app. To effectifely continue our app, we need to use the ContinueWebAuthentication(WebAuthenticationBrokerContinuationEventArgs args) event from our ContinuationManager class:

``` csharp
         public async void ContinueWebAuthentication(WebAuthenticationBrokerContinuationEventArgs args)
        {
            WebAuthenticationResult result = args.WebAuthenticationResult;

            if (result.ResponseStatus == WebAuthenticationStatus.Success)
            {
                await GetTwitterUserNameAsync(result.ResponseData.ToString());
            }
            else if (result.ResponseStatus == WebAuthenticationStatus.ErrorHttp)
            {
                MessageDialog HttpErrMsg = new MessageDialog(string.Format("There was an error connecting to Twitter: \n {0}", result.ResponseErrorDetail.ToString()), "Sorry");
                await HttpErrMsg.ShowAsync();
            }
            else
            {
                MessageDialog ErrMsg = new MessageDialog(string.Format("Error returned: \n{0}", result.ResponseStatus.ToString()), "Sorry");
                await ErrMsg.ShowAsync();
            }
        }
```
 
The WebAuthenticationResult holds our accessToken and accessTokenSecret, that we are going to exchange for our final oAuthToken and oAuthTokenSecret (we need them for all further requests).

If all goes well, we are now able to call the final Authentication Task GetTwitterUserNameAsync(string webAuthResultResponseData):

``` csharp
         private async Task GetTwitterUserNameAsync(string webAuthResultResponseData)
        {

            string responseData = webAuthResultResponseData.Substring(webAuthResultResponseData.IndexOf("oauth_token"));
            string request_token = null;
            string oauth_verifier = null;
            String[] keyValPairs = responseData.Split('&');

            for (int i = 0; i < keyValPairs.Length; i++)
            {
                String[] splits = keyValPairs[i].Split('=');
                switch (splits[0])
                {
                    case "oauth_token":
                        request_token = splits[1];
                        break;
                    case "oauth_verifier":
                        oauth_verifier = splits[1];
                        break;
                }
            }

            String TwitterUrl = "https://api.twitter.com/oauth/access_token";

            string timeStamp = GetTimeStamp();
            string nonce = GetNonce();

            String SigBaseStringParams = "oauth_consumer_key=" + TwitterConsumerKey;
            SigBaseStringParams += "&" + "oauth_nonce=" + nonce;
            SigBaseStringParams += "&" + "oauth_signature_method=HMAC-SHA1";
            SigBaseStringParams += "&" + "oauth_timestamp=" + timeStamp;
            SigBaseStringParams += "&" + "oauth_token=" + request_token;
            SigBaseStringParams += "&" + "oauth_version=1.0";
            String SigBaseString = "POST&";
            SigBaseString += Uri.EscapeDataString(TwitterUrl) + "&" + Uri.EscapeDataString(SigBaseStringParams);

            String Signature = GetSignature(SigBaseString, TwitterConsumerSecret);

            HttpStringContent httpContent = new HttpStringContent("oauth_verifier=" + oauth_verifier, Windows.Storage.Streams.UnicodeEncoding.Utf8);
            httpContent.Headers.ContentType = HttpMediaTypeHeaderValue.Parse("application/x-www-form-urlencoded");
            string authorizationHeaderParams = "oauth_consumer_key=\"" + TwitterConsumerKey + "\", oauth_nonce=\"" + nonce + "\", oauth_signature_method=\"HMAC-SHA1\", oauth_signature=\"" + Uri.EscapeDataString(Signature) + "\", oauth_timestamp=\"" + timeStamp + "\", oauth_token=\"" + Uri.EscapeDataString(request_token) + "\", oauth_version=\"1.0\"";

            HttpClient httpClient = new HttpClient();

            httpClient.DefaultRequestHeaders.Authorization = new HttpCredentialsHeaderValue("OAuth", authorizationHeaderParams);
            var httpResponseMessage = await httpClient.PostAsync(new Uri(TwitterUrl), httpContent);
            string response = await httpResponseMessage.Content.ReadAsStringAsync();

            String[] Tokens = response.Split('&');
            string oauth_token_secret = null;
            string oauth_token = null;
            string screen_name = null;

            for (int i = 0; i < Tokens.Length; i++)
            {
                String[] splits = Tokens[i].Split('=');
                switch (splits[0])
                {
                    case "screen_name":
                        screen_name = splits[1];
                        break;
                    case "oauth_token":
                        oauth_token = splits[1];
                        break;
                    case "oauth_token_secret":
                        oauth_token_secret = splits[1];
                        break;
                }
            }

            if (oauth_token != null)
            {
                App.SettingsStore.TwitteroAuthToken = oauth_token;
            }

            if (oauth_token_secret != null)
            {
                App.SettingsStore.TwitteroAuthTokenSecret = oauth_token_secret;
            }
            if (screen_name != null)
            {
                App.SettingsStore.TwitterName = screen_name;
            }
        }
```
 
At the end of this Task, we will have all Tokens and the ScreenName of our user and can save them for further usage.

The whole oAuth process is a pretty huge thing as you can see. I hope this blog post helps you all to understand how to use the WebAuthenticationBroker and get the required Tokens for all further requests.

Here are some useful links to the MSDN, if you want to dive deeper:

- [Quickstart: Connecting to an online identity provider (Windows Store apps using C#/VB/C++ and XAML)](http://msdn.microsoft.com/en-us/library/windows/apps/xaml/dn448945.aspx "http://msdn.microsoft.com/en-us/library/windows/apps/xaml/dn448945.aspx")
- [How to continue your Windows Phone app after calling a file picker (Windows Phone Store apps using C#/VB/C++ and XAML)](http://msdn.microsoft.com/en-us/library/windows/apps/xaml/dn614994.aspx)
- [How to continue your Windows Phone Store app after calling an AndContinue method](http://msdn.microsoft.com/en-us/library/windows/apps/xaml/dn631755.aspx)
- [Web authentication broker sample](http://code.msdn.microsoft.com/windowsapps/Web-Authentication-d0485122)

As always, this covers my experience. If anyone has tips or tricks to make this all easier (except of using already existing libraries), feel free to add a comment below.

Until the next post, happy coding!