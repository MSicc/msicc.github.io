---
id: 4182
title: 'How to create a folder in Windows Phone 8.1 Pictures Library (and save/read files into/from it)'
date: '2014-10-25T07:32:00+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-create-a-folder-in-windows-phone-8-1-pictures-library-and-saveread-files-intofrom-it/
categories:
    - Archive
tags:
    - file
    - image
    - 'image file'
    - jpeg
    - library
    - photo
    - pictures
    - 'pictures library'
    - png
    - storage
    - StorageFile
    - 'Windows Phone 8.1'
---

Some of you might have noticed that [UniShare](https://www.windowsphone.com/s?appid=ee42cb1d-8a68-41c6-9c0c-d3e3fc61d6ea) has its own folder in your devices picture library. Also some other apps like WhatsApp or Tweetium have it. The advantages of your app’s own folder are clear:

- easier to get images into your app
- user can always reflect which pictures come from your app
- another presence of your app within the OS
- higher remember rate for your app at the user site (which leads to more frequent usage of your app)

This post will show you how easy it is to generate a folder into the Pictures Library as well as save and read files into/from this folder.

#### Preparation

First, you need to add this using statements to your app:

``` csharp
 using Windows.Storage;
using Windows.Storage.Streams;
using Windows.UI.Xaml.Media;
using Windows.UI.Xaml.Media.Imaging;
```
 
Next, add the Picture Library capabilities to your app’s Package.appmanifest.

![image](/assets/img/2014/10/image.png "image")

If you have a 8.1 Silverlight app, you need to add it to both the Package.appmanifest as well as the WMAppManifest.xml:

![image](/assets/img/2014/10/image1.png "image")

Then we are already able to generate our folder with this single line of code (counts for both Silverlight and Runtime apps):

``` csharp
 StorageFolder appFolder= await KnownFolders.PicturesLibrary.CreateFolderAsync("myCustomAppFolder", CreationCollisionOption.OpenIfExists);
```
 
You should always use the CreateFolderAsync method together with the CollisionOption ‘OpenIfExists’. This way, your app will open it every time you are going to save a file, but creates the folder if it does not exist yet. If you now go to your pictures library, you will not see your folder yet, although it is there (use a [File Manager app](https://www.windowsphone.com/s?appid=762e837f-461d-4847-8399-3526f54fc25e) to check it if you want). Folders do only get populated when they have content. This is what the next step is about.

#### Save an image file

Saving an image is also pretty straight forward. First we are generating a StorageFile within our folder:

``` csharp
 StorageFile myfile= await appFolder.CreateFileAsync("myfile.jpg", CreationCollisionOption.ReplaceExisting);
```
 
This generates a File Container that we can write our image to. To save the image we are going to asynchronously write the Stream of our image into it:

``` csharp
 //asuming we have an Image control, replace this with your local code
var img = myImage.Source as WriteableBitmap;
//get fresh drawn image 
img.Invalidate();

using (Stream stream = await myfile.OpenStreamForWriteAsync())
{
   img.SaveJpeg(stream, img.PixelWidth, img.PixelHeight, 0, 100);
}
```
 
This code works for both a Windows Phone 8.1 Silverlight and Runtime apps. If you now go to your Pictures library, you will see your app’s folder as well as your saved image. Pretty easy, right?

#### Read images from our app’s folder

Reading an image file is pretty easy as well. Here is the code:

``` csharp
 //open the picture library
StorageFolder libfolder = KnownFolders.PicturesLibrary;
//get all folders first
IReadOnlyList<StorageFolder> folderList = await libfolder.GetFoldersAsync();
//select our app's folder
var appfolder = folderList.FirstOrDefault(f => f.Name.Contains("myCustomAppFolder"));
//get the desired file (assuming you know the file name)
StorageFile picfile = await appPicturesFolder.GetFileAsync("myfile.jpg");
//generate a stream from the StorageFile
var stream = await picfile.OpenAsync(FileAccessMode.Read);
//generate a new image and set the source to our stream
BitmapImage img = new BitmapImage();
img.SetSource(stream);

//todo: work with the image
```
 
To get our generated folder, we need to fetch a list of folders in the library using the StorageFolder.GetFoldersAsync() method. We then query this list for our app’s folder. If you want to get a list of all pictures in your folder, you can use the [StorageFile.GetFilesAsync()](https://msdn.microsoft.com/en-us/library/windows/apps/br227276.aspx) method. What I have done above is to load our saved single file. Finally, I opened a stream from this file and assigned it to a new BitmapImage, which can be used in our app.

There are also a lot of other options one can do with these folders and files, this is a very common scenario.

As always, I hope this post is helpful for some of you.