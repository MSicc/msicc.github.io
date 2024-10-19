---
id: 3957
title: 'How to integrate tickets from uservoice into your Windows Phone app'
date: '2014-01-22T04:43:16+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-integrate-tickets-from-uservoice-into-your-windows-phone-app/
categories:
    - Archive
tags:
    - feedback
    - idea
    - integration
    - list
    - oAuth
    - RestSharp
    - submit
    - tickets
    - uservoice
    - vote
    - WinPhanDev
    - wpdev
---

This time, we will have look on how to integrate tickets from uservoice into your Windows Phone app.

Users love to see a history of their tickets they submitted to a customer service. They can review them again, and maybe help also other users by showing or telling them about it.

However, if you want to get a list of all tickets from a specific user, we need to slightly change our request. We need to login as owner of the account to be allowed to search all tickets.

Let’s have a look on how to log in as owner:

``` csharp
         private void LoginAsOwner()
        {
            string loginAsOwnerPath = "/api/v1/users/login_as_owner";
            var client = new RestClient(BaseUrl)
            {
                Authenticator = OAuth1Authenticator.ForRequestToken(ConsumerKey, ConsumerSecret, oAuthCallBackUri)
            };

            //works only with POST!
            var request = new RestRequest(loginAsOwnerPath, Method.POST);
            request.AddHeader("Accept", "application/json");

            var response = client.ExecuteAsync(request, HandleLoginAsOwnerResponse);
        }

        private void HandleLoginAsOwnerResponse(IRestResponse restResponse)
        {
            var response = restResponse;
            var ownerTokens = JsonConvert.DeserializeObject<UserViaMailClass.Token>(response.Content);

            OwnerAccesToken = ownerTokens.oauth_token;
            OwnerAccesTokenSecret = ownerTokens.oauth_token_secret;

        }
```
 
The authorization request is pretty similar to the user’s authentication request. However, if we log in as owner, we are getting another AccessToken and another AccessTokenSecret. That’s why we are using the ‘ForRequestToken’-method with our RestClient.

Important to know is that the request itself works only with the HTTP-method ‘POST’, otherwise the login would be denied.

We are getting back the JSON string of our owner account, which can be deserialized with [JSON.net](http://james.newtonking.com/json) to get our AccessToken and AccessTokenSecret. I attached my ‘[UserViaMailClass](/assets/img/2014/01/UserViaMailClass.zip)‘ for easy deserialization (yes, it looks similar to the user class [from my authentication post](http://msicc.net/?p=3944), but has some differences in there).

Now that we have our OwnerAccesToken and OwnerAccessTokenSecret, we are able to search for all tickets from a specific user:

``` csharp
         public void GetAllTicketsFromUser()
        {
            string mailaddress = "<usersmailaddress>";
            string getSearchTicketsPath = "/api/v1/tickets/search.json";
            var client = new RestClient(BaseUrl)
            {
                Authenticator = OAuth1Authenticator.ForProtectedResource(ConsumerKey, ConsumerSecret, OwnerAccesToken, OwnerAccesTokenSecret)
            };

            var request = new RestRequest(getSearchTicketsPath, Method.GET);

            request.AddParameter("query", mailaddress);

            var response = client.ExecuteAsync(request, HandleGetAllTicketsFromUserResponse);
        }

        private void HandleGetAllTicketsFromUserResponse(IRestResponse restResponse)
        {
            var response = restResponse;

            var tickets = JsonConvert.DeserializeObject<TicketDataClass.TicketData>(response.Content);
        }
```
 
This request is again pretty similar to what we did to get a list of all suggestions. Please find attached my ‘[TicketDataClass](/assets/img/2014/01/TicketDataClass.zip)‘ for easy deserialization.

Of course, users want to be able to submit new tickets/support requests from our app, too. I will show you how to do that:

``` csharp
         public void CreateNewTicketAsUser()
        {
            string ticketsPath = "/api/v1/tickets.json";
            var client = new RestClient(BaseUrl)
            {
                Authenticator = OAuth1Authenticator.ForProtectedResource(ConsumerKey, ConsumerSecret, AccessToken, AccessTokenSecret)
            };

            var request = new RestRequest(ticketsPath, Method.POST);
            request.AddParameter("ticket[subject]", "testing the uservoice API");
            request.AddParameter("ticket[message]", "hi there, \n\nwe are just testing the creation of a new uservoice ticket.");

            var response = client.ExecuteAsync(request, HandleCreateNewTicketAsUserResponse);
        }

        private void HandleCreateNewTicketAsUserResponse(IRestResponse restResponse)
        {
            var response = restResponse;
        }
```
 
To submit a new ticket, we are using the user’s AccessToken and AccessTokenSecret. This way, the ticket gets automatically assigned to the ticket. We then need to pass the ‘ticket\[subject\]’ and ‘ticket\[message\]’ parameters to the request to make it being accepted by the uservoice API.

The response is a json string that contains the ticket id, which can be used to fetch the submitted ticket data. The Alternative is to call again the search method we created before to get the updated list.

Answering to already existing tickets as user seems to be not possible with the current API. Normally, if a user responds to the response mail we answer their support ticket with, it will get assigned to the existing ticket. If we create a new ticket with the same subject, it will be a new ticket that creates a new thread. I already reached out to uservoice if there is a way to do the same from the API. As soon as I have a response that enables me to do so, I will update this post.

Now that we have all important functions for our new support system, I am starting to make a small helper library for your Windows Phone 8 apps. I hope to have it finished by this weekend, and will of course blog about here.

As always, I hope this blog post is helpful for some of you. Until the next post,

Happy coding everyone!