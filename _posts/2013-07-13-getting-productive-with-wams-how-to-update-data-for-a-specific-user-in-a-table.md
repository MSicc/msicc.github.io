---
id: 3652
title: 'Getting productive with WAMS: how to update data for a specific row in a table'
date: '2013-07-13T07:42:09+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /getting-productive-with-wams-how-to-update-data-for-a-specific-user-in-a-table/
categories:
    - Azure
    - 'Dev Stories'
tags:
    - AzureDev
    - 'Mobile Services'
    - 'specific row'
    - SQL
    - update
    - 'user data'
    - WAMS
    - 'Windows Azure'
    - wpdev
---

![WAMS.png](/assets/img/2013/07/WAMS.png)


Like I promised, I will share some of the Azure goodness I learned during creating my last app.

This post is all about how to update a specific table entry (like a user’s data) in a Azure SQL table from an Windows Phone app.

First, we need to make sure that there is some data from the user we want to update. I used the LookupAsync () method to achieve that.

``` csharp
 IMobileServiceTable<userItems> TableToUpdate = App.MobileService.GetTable<userItems>();
IMobileServiceTableQuery<userItems> query = TableToUpdate.Where(useritem => useritem.TwitterId == App.TwitterId);

var useritemFromAzure = await query.ToListAsync();
var useritemLookUp = await TableToUpdate.LookupAsync(useritemFromAzure.FirstOrDefault<userItems>().Id);
```
 
If we want a specific entry, we need a search criteria to find our user and fetch the id of the user’s table entry. In my case, I used the Twitter Id for the query as every user has this on my project.

Now that we have the table row id, we can easily update the data of this specific row with the UpdateAsync() method.

We need to declare which columns should be updated and asign the values to it first. After that, we simply call the UpdateAsync() method.

``` csharp
useritemLookUp.TwitterId = App.TwitterId;
useritemLookUp.LastCheckedAt = DateTime.Now;
useritemLookUp.OSVersion = "WP8";
useritemLookUp.AppVersion = App.VersionNumber;

await TableToUpdate.UpdateAsync(useritemLookUp);
```
 
Please note that you need a items class/model to create the update data (which you should have already before thinking about updating the data).

You should wrap this code in a try{}/catch{} block to be able to react to the Exceptions that possibly can be thrown and display a matching message to the user.

That’s already all about updating a specific row in a WAMS SQL table.

As always I hope this post is helpful for some of you.

Happy coding!