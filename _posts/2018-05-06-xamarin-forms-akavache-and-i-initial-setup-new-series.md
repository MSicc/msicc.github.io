---
id: 5606
title: 'Xamarin.Forms, Akavache and I: Initial setup (new series)'
date: '2018-05-06T16:35:27+02:00'
author: 'Marco Siccardi'
excerpt: 'Caching is very important, especially in mobile applications. With this post, I kick off my new series about how I use Akavache in my Xamarin.Forms applications and quite a few gotchas while doing so.'
layout: post
permalink: /xamarin-forms-akavache-and-i-initial-setup-new-series/
image: /assets/img/2018/05/storage-1209606_1280.jpg
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - akavache
    - Android
    - caching
    - database
    - 'dependency service'
    - interface
    - iOS
    - json
    - save
    - sqlite
    - uwp
    - Windows
    - xamarin
    - 'xamarin forms'
---

Caching is never a trivial task. Sometimes, we can use built-in storages, but more often, these take quite some time when we are storing a large amount of data (eg. large datasets or large json strings). I tried quite a few approaches, including:

- built-in storage
- self handled files
- plugins that use a one or all of the above
- [Akavache](https://github.com/akavache/Akavache) (which uses SQLite under the hood)

#### Why Akavache wins

Well, the major reason is quite easy. It is fast. Really fast. At least compared to the other options. You may not notice the difference until you are using a background task that relies on the cached data or until you try to truly optimize startup performance of your Xamarin Android app. Those two where the reason for me to switch, because once implemented, it does handle both jobs perfectly. Because it is so fast, there is quite an amount of apps that uses it. Bonus: there are a lot of tips on [StackOverflow](https://stackoverflow.com/questions/tagged/Akavache) as well as on [GitHub](https://github.com/akavache/Akavache/issues), as it is already used by a lot of developers.

## Getting your projects ready

Well, as often, it all starts with the installation of NuGet packages. As I am trying to follow good practices wherever I can, I am using .netStandard whenever possible. The latest stable version of Akavache does work partially in .netStandard projects, but I recommend to use the latest alpha (by the time of this post) in your .netStandard project (even if VisualStudio keeps telling you that a pre release dependency is not a good idea). If you are using the package reference in your project files, there might be some additional work to bring everything to build and run smoothly, especially in a Xamarin.Android project.

You mileage may vary, but in my experience, you should install the following dependencies and Akavache separately:

- [System.Reactive, version 3.1.1](https://www.nuget.org/packages/System.Reactive/3.1.1)
- [Splat, minimum version 3.0.0](https://www.nuget.org/packages/Splat/3.0.0)
- [SQLitePCLRaw.bundle\_e\_sqlite3, version 1.1.10](https://www.nuget.org/packages/SQLitePCLRaw.bundle_e_sqlite3/1.1.10)
- [akavache.sqlite3, version 6.0.0-alpha0038](https://www.nuget.org/packages/akavache.sqlite3/6.0.0-alpha0038)
- [akavache.core, version 6.0.0-alpha0038](https://www.nuget.org/packages/akavache.core/6.0.0-alpha0038)
- [akavache, version 6.0.0-alpha0038](https://www.nuget.org/packages/akavache/6.0.0-alpha0038)

After installing this packages in your Xamarin.Forms and platform projects, we are ready for the next step.

## Initializing Akavache

Basically, you *should* be able to use Akavache in a very simple way, by just defining the application name like this during application initialization:

``` csharp
 BlobCache.ApplicationName = "MyAkavachePoweredApp";
```
 
You can do this assignment in your platform project as well as in your Xamarin.Forms project, both ways will work. Just remember to do this, as also to get my code working, this is a needed step.

There are static properties like `BlobCache.LocalMachine`one can use to cache data. However, once your app will use an advanced library like Akavache, it is very likely that he complexity of your app will force you into a more complex scenario. In my case, the usage of a scheduled job on Android was the reason why I am doing the initialization on my own. The scheduled job starts a process for the application, and the job updates data in the cache that the application uses. There were several cases where the standard initialization did not work, so I decided to make the special case to a standard case. The following code will also work in simple scenarios, but keeps doors open for more complex ones as well. The second reason why I did my own implementation is the MVVM structure of my apps.

## IBlobCacheInstanceHelper rules them all

Like often when we want to use platform implementations, all starts with an interface that dictates the functionality. Let’s start with this simple one:

``` csharp
 public interface IBlobCacheInstanceHelper
{
    void Init();
    IBlobCache LocalMachineCache { get; set; }
}
```
 
We are defining our own `IBlobCache`instance, which we will initialize with the `Init() `method on each platform. Let’s have a look on the platform implementations:

``` csharp
 [assembly: Xamarin.Forms.Dependency(typeof(PlatformBlobCacheInstanceHelper))]
namespace [YOURNAMESPACEHERE]
{
    public class PlatformBlobCacheInstanceHelper : IBlobCacheInstanceHelper
    {
        private IFilesystemProvider _filesystemProvider;

        public PlatformBlobCacheInstanceHelper() { }

        public void Init()
        {
            _filesystemProvider = Locator.Current.GetService<IFilesystemProvider>();
            GetLocalMachineCache();
        }

        public IBlobCache LocalMachineCache { get; set; }

        private void GetLocalMachineCache()
        {

            var localCache = new Lazy<IBlobCache>(() => 
                                                  {
                                                      _filesystemProvider.CreateRecursive(_filesystemProvider.GetDefaultLocalMachineCacheDirectory()).SubscribeOn(BlobCache.TaskpoolScheduler).Wait();
                                                      return new SQLitePersistentBlobCache(Path.Combine(_filesystemProvider.GetDefaultLocalMachineCacheDirectory(), "blobs.db"), BlobCache.TaskpoolScheduler);
                                                  });

            this.LocalMachineCache = localCache.Value;
        }

        //TODO: implement other cache types if necessary at some point
    }
}
```
 
Let me explain what this code does.

As SQLite, which is powering Akavache, is file based, we need to provide a file path. The `Init()` method assigns Akavache’s internal `IFileSystemProvider`interface to the internal member. After getting an instance via Splat’s Locator, we can now use it to get the file path and create the `.db`-file for our local cache. The `GetLocalMachineCache()`method is basically a copy of [Akavache’s internal registration](https://github.com/akavache/Akavache/blob/501b397d8c071366c3b6783aae3e98695b3d7442/src/Akavache.Sqlite3/Registrations.cs). It lazily creates an instance of `BlobCache` through the `IBlobCache`interface. The create instance is then passed to the `LocalMachineCache`property, which we will use later on. Finally, we will be using the `DependencyService`of Xamarin.Forms to get an instance of our platform implementation, which is why we need to define the Dependency attribute as well.

*Note*: you can name the file whatever you want. If you are already using Akavache and want to change the instance handling, you should keep the original names used by Akavache. This way, your users will not lose any data.

This implementation can be used your Android, iOS and UWP projects within your Xamarin.Forms app. If you are wondering why I do this separately for every platform, you are right. Until now, there is no need to do it that way. The code above would also work solely in your Xamarin.Forms project. Once you are coming to the point where you need encrypted data in your cache, the platform implementations will change on every platform. This will be topic of a future blog post, however.

If you have been reading my series about MVVMLight, you may guess the next step already. This is how I initialize the platform implementation within my ViewModelLocator:

``` csharp
 //register:
var cacheInstanceHelper = DependencyService.Get<IBlobCacheInstanceHelper>();
if (!SimpleIoc.Default.IsRegistered<IBlobCacheInstanceHelper>())
     SimpleIoc.Default.Register<IBlobCacheInstanceHelper>(()=> cacheInstanceHelper);

//initialize:
//cacheInstanceHelper.Init();
//or
SimpleIoc.Default.GetInstance<IBlobCacheInstanceHelper>().Init();
```
 
So that’s it, we are now ready to use our local cache powered by Akavache within our Xamarin.Forms project. In the next post, we will have a look on how to use akavache for storing and retrieving data.

### Until then, happy coding, everyone!