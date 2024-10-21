---
id: 3969
title: '[Update 3] UserVoice WP8 library for user features'
date: '2014-03-15T20:08:33+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /introducing-the-uservoice-wp8-library-for-user-features/
categories:
    - Archive
tags:
    - async
    - customer
    - 'customer service'
    - feedback
    - request
    - RestSharp
    - suggestions
    - support
    - tickets
    - uservoice
---

It is done. I made my first library for Windows Phone 8. This blog post is about why I did it and how to use it.

Update: I changed some method names of the library to make it easier to use the library. I updated the code in this post to reflect those changes.

Update 2: I added support to load more pages with clients. On top, I also added my EmojiDetector class, as the UserVoice API does not support Emojis. Check the code below.

Update 3: Now the library supports comments on suggestions and mark as helpful for knowledge base items.

### Why UserVoice?

We indie developers have a big problem: We do not have a support team to explain our users how to use our apps or how to solve certain problems/issues. Which leads to our next problem: users are customers. Customers want to be satisfied. It is our job to do this with our apps by providing them a high level user experience and feature rich apps. Often users don’t go the extra mile to send us an email to tell us what is wrong. Or they plan it, but forget about it. Or even worse: they get annoyed and uninstall our apps.

As some of you know, I am working at the hardware support team of a German phone carrier. Over the years, I learned how important it is to listen to customers, pick up their ideas and wishes and work to get them done if possible. And if it is not possible, you need to tell them that – even that is an important part of customer service!

Many of us have set up a twitter account, a separate mail address, maybe an extra online form to catch all requests from users up. But users tend to not use them for one reason: they are not integrated in our apps. So I spend some time “googling with Bing” (Thanks to [@robwirving](https://twitter.com/robwirving) for that awesome phrase!) on possible solutions.

Uservoice as the best value if using a free subscription, and they have an API that we can use ([see also this post]({% post_url 2014-01-06-experiment-using-uservoice-com-for-support-feedback-and-faq %}) on how to get started with uservoice). I made it a very slim library and concentrated on the features we really need in our app on the user side.

### The Library!

You can get the library easily via [NuGet](https://www.nuget.org/packages/uservoice_WP8_Userfeatures/) directly into your app. Just add this package to your app’s packages list:

![uservoice_lib_nuget](/assets/img/2014/01/uservoice_lib_nuget.png "uservoice_lib_nuget")

The library also needs [RestSharp](https://restsharp.org/), which gets automatically added to your project if you install the library.

After you installed it, you need to declare some variables that we need over and over again while using the library:

``` csharp
 Urls.subDomain = "<your subdomain>";
Urls.oAuthCallBackUri = "<your callback url>";
Tokens.ConsumerKey = "<your ConsumerKey>";
Tokens.ConsumerSecret = "<your ConsumerSecret>";
```
 
You can get this values out of Settings/Integrations in your UserVoice account.

Additionally, you should save these Tokens to not ask the user for login again and again.

``` csharp
 Tokens.AccessToken
Tokens.AccessTokenSecret  
Tokens.OwnerAccessToken
Tokens.OwnerAccessTokenSecret
```
 
Another important class you should be aware of is the RequestParamaters class:

![Screenshot (309)](/assets/img/2014/01/Screenshot-309.png "Screenshot (309)")

It contains the needed variables for all requests, and you can easily use them to save them for TombStoning or anything else you want to save them.

Let’s have a look at the possible requests:

- Knowledge Base:

``` csharp
 KnowledgeBase kb = new KnowledgeBase();`   
//load complete knowledge base  
UservoiceRequests.KnowledgeBase = await kb.getAll();  
//load specific page in your knowledge base
int page = 2;
 UservoiceRequests.KnowledgeBase = await kb.getAll(page);
//load specific topic 
UservoiceRequests.KnowledgeBaseTopic = await kb.getTopic(RequestParameters.topicId);
//load specific page in topic
int page = 2;
UservoiceRequests.KnowledgeBaseTopic = await kb.getTopic(RequestParameters.topicId, page);
//mark article as helpful
var ratingResponse = await kb.markHelpful(RequestParamaeters.articleId);
```
 
- Suggestions:

``` csharp
 Suggestion suggestion = new Suggestion();  
//load all suggestions  
UservoiceRequests.allSuggestions = await suggestion.getAll(RequestParameters.forumId);  
//load specific page in all suggestions
int page = 2;
UservoiceRequests.allSuggestions = await suggestion.getAll(RequestParameters.forumId, page); 
//vote on a suggestion 
UservoiceRequests.voteForSuggestion = await suggestion.vote(RequestParameters.forumId, RequestParameters.suggestionId, RequestParameters.vote);  
//submit new suggestion  
UservoiceRequests.postSuggestion = await suggestion.create(RequestParameters.forumId, RequestParameters.newSuggestionTitle, RequestParameters.newSuggestionText, RequestParameters.newSuggestionReferrer, RequestParameters.newSuggestionVotes); 
//search suggestions  
UservoiceRequests.searchSuggestion = await suggestion.search(RequestParameters.suggestionsSearchQuery);
//get all comments for suggestion:
UservoiceRequests.allCommentsForSuggestion = await suggestion.getComments(RequestParameters.forumId, RequestParameters.suggestionId);
//submit a new comment on suggestion:
UservoiceRequests.postCommentOnSuggestion = await suggestion.comment(RequestParameters.forumId, RequestParameters.suggestionId, RequestParameters.newCommentText);
```
 
- User data:

``` csharp
 User user = new User();  
UservoiceRequests.User = await user.getUser();
```
 
- Tickets:

``` csharp
 //Tickets are not associated with the user from the API side. However, you are able to show all tickets from a user with this:  
ticket = new Ticket();  
UservoiceRequests.AllTicketsFromUser = await ticket.getAll(RequestParameters.userMail); 
//load specific page in all tickets:
int page = 2;
UservoiceRequests.AllTicketsFromUser = await ticket.getAll(RequestParameters.userMail, page);
//submit a new ticket on behalf of the user
UservoiceRequests.newTicket = await ticket.create(RequestParameters.TicketSubject, RequestParameters.TicketMessage);
```
 
As you can see, all requests are async.

You don’t need explicitly authenticate a user, because the library is built to detect this automatically. If an authenticated user is required, the user will be redirected to the authentication page of UserVoice.

In the current version, you will need to manual send the request again after the user is authenticated, but I will update the library to make also this automatically soon.

- EmojiDetector

``` csharp
string text = "here would be the string from your textboxes that contain emojis";

//check if Emojis are in text:
if (EmojiDetector.HasEmojis(text) == true)
{
    //your code here
}
//remove Emojis in text:
text = EmojiDetector.RemoveEmojis(text);
```
 
The UserVoice API does not support Emojis, that’s why I wrote this little helper that you can easily use to remove them with only one line of code or to display a message to your users.

One last point: I don’t use RestSharp’s serializer – all request return the corresponding JSON string. This way, everyone of you can use the serializer of choice (I absolutely recommend [JSON.net](https://james.newtonking.com/json), though).

Please consider the current version as beta release, and report any issues with that to me via [Twitter](https://twitter.com/msicc) or [mail](mailto:msiccdev@hotmail.com).

And now enjoy my library &amp; happy coding!