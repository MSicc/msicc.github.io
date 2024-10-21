---
id: 3760
title: 'Getting productive with WAMS: How to call Twitter REST API 1.1 from a scheduled script'
date: '2013-08-27T04:32:27+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /getting-productive-with-wams-how-to-call-twitter-rest-api-1-1-from-a-scheduled-script/
categories:
    - Azure
    - 'Dev Stories'
tags:
    - 'API 1.1'
    - Azure
    - AzureDev
    - 'Mobile Services'
    - 'REST API'
    - twitter
---

![WAMS.png](/assets/img/2013/08/WAMS.png)


Like I promised in my first post about Windows Azure Mobile Services, I will show you how to call the Twitter Rest API 1.1 from a scheduled script. The documentation of the http request object does only use Twitter API 1.0 (which is no longer available).

First, you will need a Consumer key and a Consumer secret for your app. Just go to [dev.twitter.com](https://dev.twitter.com), register with your Twitter account and then add a new application.

The second thing you will need, is the so called Access token and Access token secret. Both are user dependent, without them Twitter will give you an error that your app is not authorized to use this account for anything on Twitter.

There are several ways to obtain these values. As I am registering the user within my phone app, I am uploading these values from phone and store it in my Mobile Services database.

To generate the requested data, we need several additional data for our request to Twitter:

- a timestamp for the oAuth Header and the signature string
- a random number to secure the request (= nonce)
- an oAuth signature (signed array of the user’s data)
- a HMAC encoded Hash string

These data is used for our request to Twitter.

Let’s start with the “simple” things:

**generate Timestamp:**

``` js 
//generating the timestamp for the OAuth Header and signature string
var timestamp  = new Date() / 1000;
timestamp = Math.round(timestamp);
```
 
**generate nonce**

``` js
function generateNonce() {
    var code = "";
    for (var i = 0; i < 20; i++) {
        code += Math.floor(Math.random() * 9).toString();
    }
    return code;
}
```
 

**oAuth signature**

``` js
//generating the oAuth signatured array for the Twitter request
function generateOAuthSignature(method, url, data) {
    //remove query string parameters
    var index = url.indexOf('?');
    if (index > 0)
        url = url.substring(0, url.indexOf('?'));

    var signingToken = encodeURIComponent(ConsumerSecret) + "&" + encodeURIComponent(twitterAccessTokenSecret);

    var keys = [];
    for (var d in data) {
        if (d != 'oauth_signature') {
            //console.log('data:', d);
            keys.push(d);
        }
    }

    keys.sort();
    var output = "GET&" + encodeURIComponent(url) + "&";
    var params = "";
    keys.forEach(function (k) {
        params += "&" + encodeURIComponent(k) + "=" + encodeURIComponent(data[k]);
    });
    params = encodeURIComponent(params.substring(1));

    return hashString(signingToken, output + params, "base64");
}
```
 
**generate the HMAC encoded hash string**

``` js
//generate Hash-string, encoded in HMAC-SHA1 as required by Twitter's API v1.1
function hashString(key, str, encoding) {
    //console.log('basestring:', str);
    var hmac = crypto.createHmac("sha1", key);
    hmac.update(str);
    return hmac.digest(encoding);
}
```
 
Now we have prepared all of these functions, we are well prepared to call the Twitter API. In this example we are calling the user’s profile data:

``` js
function requestToTwitter()
{

    //the url declaration has to be in this function to make the request working!
    //declaring it in another function would cause an error 401 from Twitter's API
    url = 'https://api.twitter.com/1.1/users/show.json?user_id=' + twitterId;

    //generate data for sending the request to Twitter
    //this is the data used in the signature string as well as in the Authorization header
    var oAuthData = {oauth_consumer_key: ConsumerKey, oauth_nonce: nonce, oauth_signature: null, oauth_signature_method: "HMAC-SHA1", oauth_timestamp: timestamp, oauth_token: twitterAccessToken, oauth_version: "1.0"};
    var sigData = {};
    for (var k in oAuthData) {
        sigData[k] = oAuthData[k];
    }
    sigData['user_id'] = twitterId;

    var sig = generateOAuthSignature('GET', url, sigData);
    oAuthData.oauth_signature = sig;

    var oAuthHeader = "";
    for (k in oAuthData) {
        oAuthHeader += "," + encodeURIComponent(k) + "=\"" + encodeURIComponent(oAuthData[k]) + "\"";
    }
    oAuthHeader = oAuthHeader.substring(1);
    //very important to not miss the space after OAuth!
    authHeader = 'OAuth '+oAuthHeader;

    var reqOptions = {
            uri: url,
            headers: { 'Accept': 'application/json', 'Authorization': authHeader }
    };

    var httpRequest = require('request');
        httpRequest(reqOptions,callback );

}

var callback = function(err, response, body) {
    //console.log("in requestToTwitter = callback"); 
            if (err) {
            console.log(err)
            } else if (response.statusCode !== 200) {
                console.log("from twitter callback " + response.statusCode + " response: " + response.body);
            } else {
                var userProfile = JSON.parse(body);
                UserIdFromTwitter = userProfile.id;
                twitterScreenName = userProfile.screen_name;

}
}
```
 
You may have noticed that there are several variables that are not declared within these functions. Just declare them globally in your scheduled script.

You can read more about the oAuth authorization process at [https://oauth.net/](https://oauth.net/ "https://oauth.net/").

There are more services out there that use the oAuth process, so you should be able to convert this for other requests, like getPocket.com (formerly Read It later) and others.

As always, I hope this post was helpful for some of you.

Happy coding!