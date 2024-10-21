---
id: 4088
title: 'How to load a list of a user&#8217;s twitter friends/followers in your Windows Phone app'
date: '2014-06-28T04:40:42+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-load-a-list-of-a-users-twitter-friendsfollowers-in-your-windows-phone-app/
categories:
    - Archive
tags:
    - 'do while'
    - file
    - followers
    - friends
    - friendslist
    - 'json file'
    - loop
    - 'rate limit'
    - 'save json file'
    - twitter
    - 'Windows Phone 8'
    - 'Windows Phone 8.1'
    - WP8
    - WP8.1
    - wpdev
---

![TwitterFriendsList_WP_Blog](/assets/img/2014/06/TwitterFriendsList_WP_Blog.png "TwitterFriendsList_WP_Blog") During the last days, I was adding a new function to [UniShare](https://www.windowsphone.com/s?appid=ee42cb1d-8a68-41c6-9c0c-d3e3fc61d6ea) that suggests users the Twitter handles while typing in some text. The base of this new function is a list of the user’s twitter friends. In this post, I am going to show you how to generate this list and save it for further use. First thing I need to talk about: you can only load a maximum of 3000 friends at once due to the rate limit on Twitter’s part. At the end of the post, I’ll give you also an idea how to continue this after the first 3000 users were loaded. For the most scenarios, 3000 should be enough to cover the major part of your users. To be able to load all friends we first create a Task that fetches 200 entries of our user’s friends list. To be able to perform paging through all entries, Twitter uses so called cursors, which are 64-bit signed integers (long). You can read more about that here on Twitter’s [API documentation](https://dev.twitter.com/docs/misc/cursoring). Let’s have a look on this Task:

``` csharp
         private async Task<string> TwitterFriendsListString(long twitterCursor)
        {
            ShowProgressIndicator("generating mention suggestion base...");

            //needed for generating the Signature
            string twitterRequestUrl = "https://api.twitter.com/1.1/friends/list.json";

            //used for sending the request to the API
            //for usage of the url parameters, go to https://dev.twitter.com/docs/api/1.1/get/friends/list
            string twitterUrl = string.Format("https://api.twitter.com/1.1/friends/list.json?cursor={0}&screen_name={1}&count=200&skip_status=true&include_user_entities=false", twitterCursor, App.SettingsStore.twitterName);

            string timeStamp = GetTimeStamp();
            string nonce = GetNonce();

            string response = string.Empty;

            //we need to include all url parameters in the signature
            //IMPORTANT: Twitter's API demands them to be sorted alphabetically, otherwise the Signature will not be accepted
            string sigBaseStringParams = "count=200";
            sigBaseStringParams += "&" + "cursor=" + twitterCursor;
            sigBaseStringParams += "&" + "include_user_entities=false";
            igBaseStringParams += "&" + "oauth_consumer_key=" + twitterConsumerKey;
            sigBaseStringParams += "&" + "oauth_nonce=" + nonce;
            sigBaseStringParams += "&" + "oauth_signature_method=HMAC-SHA1";
            sigBaseStringParams += "&" + "oauth_timestamp=" + timeStamp;
            sigBaseStringParams += "&" + "oauth_token=" + App.SettingsStore.twitteroAuthToken;
            sigBaseStringParams += "&" + "oauth_version=1.0";
            sigBaseStringParams += "&" + "screen_name=" + App.SettingsStore.twitterName;
            sigBaseStringParams += "&" + "skip_status=true";
            string sigBaseString = "GET&";
            sigBaseString += Uri.EscapeDataString(twitterRequestUrl) + "&" + Uri.EscapeDataString(sigBaseStringParams);

            string signature = GetSignature(sigBaseString, twitterConsumerSecret, App.SettingsStore.twitteroAuthTokenSecret);

            string authorizationHeaderParams =
                "oauth_consumer_key=\"" + twitterConsumerKey +
                "\", oauth_nonce=\"" + nonce +
                "\", oauth_signature=\"" + Uri.EscapeDataString(signature) +
                "\", oauth_signature_method=\"HMAC-SHA1\", oauth_timestamp=\"" + timeStamp +
                "\", oauth_token=\"" + Uri.EscapeDataString(App.SettingsStore.twitteroAuthToken) +
                "\", oauth_version=\"1.0\"";

            HttpClient httpClient = new HttpClient();

            httpClient.DefaultRequestHeaders.Authorization = new HttpCredentialsHeaderValue("OAuth", authorizationHeaderParams);
            var httpResponseMessage = await httpClient.GetAsync(new Uri(twitterUrl));

            response = await httpResponseMessage.Content.ReadAsStringAsync();

            return response;
        }
```
 
As you can see, we need to create our signature based on the url parameters we are requesting. Twitter demands them to be sorted alphabetically. If you would not sort them this way, the Signature is wrong and your request gets denied by Twitter. We are overloading this Task with the above mentioned cursor to get the next page of our user’s friends list. If you need the GetNonce() and GetSignature() methods, [take a look at this post]({% post_url 2014-04-19-how-to-use-the-webauthenticationbroker-for-oauth-in-a-windows-phone-runtime-wp8-1-app %}), I am using them for all requests to Twitter. Now that we have this Task ready to make our requests, lets load the complete list. First, declare these objects in your page/model:

``` csharp
         public List<TwitterMentionSuggestionBase> twitterMentionSuggestionsList;
        public long nextfriendlistCursor = -1;
        public string friendsResponseString = string.Empty;
        public string friendsJsonFileName = "twitterFriends.json";
```
 
Then we need to create a loop to go through all pages Twitter allows us until we are reaching the rate limit of 15 requests per 15 minutes. I using a do while loop, as this is the most convenient way for this request:

``` csharp
             //to avoid double entries, make sure the list is we are using is empty
            if (twitterMentionSuggestionsList.Count != 0)
            {
                twitterMentionSuggestionsList.Clear();
            }
            //starting the do while loop
            do
            {
                //fetching the Json response
                //to get the first page of the list from Twitter, the cursor needs to be -1
                friendsResponseString = await TwitterFriendsListString(nextfriendlistCursor);

                //using Json.net to deserialize the string
                var friendslist = JsonConvert.DeserializeObject<TwitterFriendsClass.TwitterFriends>(friendsResponseString);
                //setting the cursor for the next request
                nextfriendlistCursor = friendslist.next_cursor;

                //adding 200 users to our list
                foreach (var user in friendslist.users)
                {
                    //using the screen name ToLower() to make the further usage easier
                    TwitterMentionSuggestionsList.Add(new TwitterMentionSuggestionBase() { name = "@" + user.screen_name.ToLower() });
                }

                //once the last page is reached, the cursor will be 0
                if (nextfriendlistCursor == 0)
                {
                    //break will continue the code after the do while loop
                    break;
                }
            }
            //the while condition needs to return true if you would set up a Boolean for it!
            while (nextfriendlistCursor != 0);
```
 
As long as the cursor is not 0, the loop will load the json string with 200 entries. The class you’ll need for deserializing can be easily generated. Just set a breakpoint after the call of our Task, copy the whole response string and go to <https://json2csharp.com> to get your class. The TwitterMentionSuggestionBase class only contains a string property for the name and can be extended for you needs:

``` csharp
     public class TwitterMentionSuggestionBase
    {
        public string name { get; set; }
    }
```
 
Of course we need to save the list for further use. The code I use is pretty much the same you’ll find when you search for “saving json string to file” with your favorite search engine and works with Windows Phone 8 and 8.1:

``` csharp
             //saving the file to Isolated Storage
            var JsonListOfFriends = JsonConvert.SerializeObject(twitterMentionSuggestionsList);
            try
            {
                //getting the localFolder our app has access to
                StorageFolder localFolder = ApplicationData.Current.LocalFolder;
                //create/overwrite a file
                StorageFile friendsJsonFile = await localFolder.CreateFileAsync(friendsJsonFileName, CreationCollisionOption.ReplaceExisting);
                //write the json string to the file
                using (IRandomAccessStream writeFileStream = await FriendsJsonFile.OpenAsync(FileAccessMode.ReadWrite))
                {
                    using (DataWriter streamWriter = new DataWriter(writeFileStream))
                    {
                        streamWriter.WriteString(JsonListOfFriends);
                        await streamWriter.StoreAsync();
                    }
                }
            }
            catch
            {

            }
```
 
This way, we can easily generate a friends list for our users, based on their Twitter friends. Some points that might be helpful:

- users with more than 3000 friends will receive an error after reaching this point on our task. Catch this error, but continue to save the already loaded list. Save the latest cursor for the next page and prompt the user to continue loading after 15 minutes to continue loading.
- if you want to have a list with more details about a user’s friends, extend the TwitterMentionSuggestionBase class with the matching properties.
- this code can also be used to determine the followers of a user, you just need to change the url parameters as well as the signature parameters [according to the matching Twitter endpoint](https://dev.twitter.com/docs/api/1.1/get/followers/list)

As always, I hope this is helpful for some of you. Until the next post, happy coding!