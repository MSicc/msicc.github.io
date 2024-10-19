---
id: 5641
title: 'Xamarin.Forms, Akavache and I: storing, retrieving and deleting data'
date: '2018-05-21T08:06:58+02:00'
author: 'Marco Siccardi'
excerpt: 'Following up the initial post on using Akavache and Xamarin.Forms together, I will show you some typical methods for storing, retrieving and deleting data from the cache instance we implemented in the first post of this series.'
layout: post
permalink: /xamarin-forms-akavache-and-i-storing-retrieving-and-deleting-data/
image: /assets/img/2018/05/storing-data-title.jpg
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - akavache
    - Android
    - cache
    - caching
    - database
    - 'dependency service'
    - interface
    - iOS
    - json
    - reactive
    - 'reactive extensions'
    - save
    - sqlite
    - uwp
    - Windows
    - xamarin
    - 'xamarin forms'
---

Caching always has the same job: provide data that is frequently used in very little time. As I mentioned in [my first post of this series](https://msicc.net/xamarin-forms-akavache-and-i-initial-setup-new-series/), Akavache is my first choice because it is fast. It also provides a very easy way to interact with it (once one gets used to [Reactive Extensions](http://introtorx.com/)). The code I am showing here is living in the Forms project, but can also be called from the platform projects thanks to the interface we defined already before.

### Enabling async support

First things first: we should write our code asynchronously, that’s why we need to enable async support by adding `using System.Reactive.Linq;`to the using statements in our class. This one is not so obvious, and I read a lot of questions on the web where this was the simple solution. So now you know, let’s go ahead.

### Simple case

The most simple case of storing data is just throwing data with a key into the underlying database:

``` csharp
 //getting a reference to the cache instance
var cache = SimpleIoc.Default.GetInstance<IBlobCacheInstanceHelper>().LocalMachineCache;
var dataToSave = "this is a simple string to save into the database";
await cache.InsertObject<string>("YourKeyHere", dataToSave);
```
 
Of course, we need a reference to the `IBlobCache`instance we have already in place. I am saving a simple string here for demo purposes, but you can also save more complex types like a list of blog posts into the cache. Akavache uses [Json.NET ,](https://www.newtonsoft.com/json) which will serialize the data into a valid json string that you can be saved. Similarly, it is very easy to get the data deserialized from the database:

``` csharp
 var dataFromCache = cache.GetObject<string>("YourKeyHere");
```
 
That’s it. For things like storing Boolean values, simple strings (unencrypted), dates etc., this might already be everything you need.

### Caching data from the web

Of course it wouldn’t be necessary to implement an advanced library if we would have only this scenario. More often, we are fetching data from the web and need to save it in our apps. There are several reasons to do this, with saving (mobile) data volume and performance being the two major reasons.

Akavache provides a bunch of very useful Extensions. The most prominent one I am using is the `GetOrFetchObject<T>`method. A typical implementation looks like this:

``` csharp
 var postsCache = await cache.GetOrFetchObject<List<BlogPost>>(feedName,
    async () =>
    {
        var newPosts = await _postsHandler.GetPostsAsync(BaseUrl, 20, 20, 1, feedName.ToCategoryId()).ConfigureAwait(false);

        await cache.InsertObject<List<BlogPost>>(feedName, newPostsDto);

        return newPosts;
    });
```
 
The `GetOrFetchObject<T>`method’s minimum parameters are the key of the cache entry and an asynchronous function that shall be executed when there is no data in the cache. In the sample above, it loads the latest 20 posts from a WordPress blog (utilizing [my WordPressReader lib](https://github.com/MSiccDev/WordPressReaderStd)) and saves it into the cache instance before returning the downloaded data. The method has an optional parameter of `DateTimeOffset`, which may be interesting if you need to expire the saved data after some time.

### Saving images/documents from the web

If you need to download files, be it images or other documents, from the web, Akavache provides another helper extension:

``` csharp
 byte[] bytes = await cache.DownloadUrl("YourFileKeyHere", url);
```
 
Personally, I am loading all files with this method, even though there are some special image loading methods available as well ([see the readme at Akavache’s repo](https://github.com/reactiveui/Akavache)). The main reason I am doing so is that until now, I always have a platform specific implementation for such cases – mainly due to performance reasons. I one of the following blog posts you will see such an implementation for image caching using a custom renderer on each platform.

### Deleting data from the cache

When working with caches, one cannot avoid the situation that data needs to be removed manually from the cache.

``` csharp
 //delete a single entry by key:
cache.Invalidate("KeyToDelete");

//delete all entries with the same type:
cache.InvalidateAllObjects<BlogPost>();

//delete all entries
cache.InvalidateAll();
```
 
If you want to continue with some other action after deletion completes, you can use the `Subscribe` method to invoke this action:

``` csharp
 cache.InvalidateAll().Subscribe(x => YourMethodToInvoke());
```
 
### Conclusion

Even though Akavache provides more methods to store and retrieve data, the ones I mentioned above are those that I use frequently and without problems in my Xamarin.Forms applications, while still being able to invoke them in platform specific code as well. If you want to have a look at the other methods that are available, click the link above to the GitHub repo of Akavache. As always, I hope this blog post is helpful for some of you.

#### Until the next post, happy coding, everyone!