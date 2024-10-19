---
id: 3812
title: 'How to work with the user&#8217;s music library on Windows Phone'
date: '2013-11-11T20:13:00+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-work-with-the-users-music-library-on-windows-phone/
categories:
    - Archive
tags:
    - app
    - MediaHistory
    - MediaLibrary
    - music
    - 'now playing'
    - 'Windows Phone'
    - wpdev
---

![wp_ss_20131111_0002b](/assets/img/2013/11/wp_ss_20131111_0002b.png "wp_ss_20131111_0002b")

I am currently exploring a lot of APIs that I never used before in my Windows Phone apps as I am adding more features to my [NFC Toolkit](http://www.windowsphone.com/s?appid=2c33cb7d-c97b-4204-aa8b-1e8712718519).

Like I always do, I create sample applications to explore what is possible and then integrate them into my main app.

One of those APIs is the MediaLibrary class on Windows Phone. You might think: there are tons of examples out there, why another post? Well, I agree, there are a lot of samples, but I want to cover a whole scenario.

In my sample, I am getting a list of all Albums stored in Library, start them playing, and display the current playing song as well as handle state managements of the playing song.

First, you will need to add the capabilities ID\_CAP\_MEDIALIB\_AUDIO and ID\_CAP\_MEDIALIB\_PLAYBACK to your app. If you will not do that, every call against the APIs will end up with an NotAuthorizedException.

For any access to the MediaLibrary and the MediaPlayer classes we are using the Namespace “[Microsoft.Xna.Framework.Media](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.aspx)”. Here our trouble begins already.

Xna is Microsoft’s framework to create games on Windows Phone 7. As Windows Phone 8 supports also native code, this is mainly used in 7 games. That’s why you might run in some TargetInvocationExceptions while you debug apps that use them.

To avoid this, we need to add a Framework Dispatcher that simulates a game loop for us. Add the following method to your App.xaml.cs:

``` csharp
        private void StartGameLoop()
        {
            GameTimer gameTimer = new GameTimer();
            gameTimer.UpdateInterval = TimeSpan.FromMilliseconds(33);

            gameTimer.Update += delegate
            {
                try
                {
                    FrameworkDispatcher.Update();
                }
                catch
                { }
            };

            gameTimer.Start();

            FrameworkDispatcher.Update();
        }
```
 
We are starting a new [GameTimer](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.gametimer.aspx) and delgate it to our UI using the [FrameworkDispatcher.Update()](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.frameworkdispatcher.update.aspx) method. By adding it to App.xaml.cs and calling this method in Lauching and Activated event of our app, we have it running through our whole app and are done with this.

There are several methods to this, I found this the most easy version. I would pay credits, but don’t remember where I saw this – sorry.

Before we came to our first call against the MediaLibrary, we need to add a class for our List&lt;T&gt; or ObservableCollection&lt;T&gt;. I don’t know if this is the best practice in this case, but it made working with the List of Albums very easy for me:

``` csharp
        public class AlbumFromStorage
        {
            public string AlbumName {get; set;}

            public string AlbumArtist {get; set;}

            public WriteableBitmap AlbumCover {get; set;}

            public SongCollection AlbumSongs { get; set; }

        }
```
 
To use this class, just add a new List&lt;T&gt;/ObservableCollection&lt;T&gt; to the Page class. Do the same for a MediaLibrary object and an AlbumCollection object:

``` csharp
public ObservableCollection<AlbumFromStorage> AlbumsList = new ObservableCollection<AlbumFromStorage>();
public MediaLibrary lib = new MediaLibrary();
public AlbumCollection albumslist;
```
 
Now we are prepared for fetching all local stored albums. Add the following code to your corresponding method:

``` csharp
            albumslist = lib.Albums;

            if (albumslist.Count != 0)
            {
                foreach (var item in albumslist)
                {
                    AlbumsList.Add(new AlbumFromStorage()
                    {
                        AlbumArtist = item.Artist.ToString(),
                        AlbumName = item.Name,
                        AlbumCover = PictureDecoder.DecodeJpeg(item.GetThumbnail()),
                        AlbumSongs = item.Songs
                    });
                }

                MusicAlbumListBox.ItemsSource = AlbumsList;
            }
            else
            {
                await MessageBox.Show("It seems you have not stored any music on your phone.", "Sorry :(", MessageBoxButton.OK);
            }
```
 
Let me explain the code. First, we are using our AlbumCollection object to asign the [MediaLibrary.Albums](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.medialibrary.albums.aspx) property. This will give us a collection of all albums. Then we need to check first, if the count in the collection is not 0 (it will throw ugly exceptions if a user doesn’t have any music stored if you won’t to this).

Then we add these albums to our ObservableCollection&lt;AlbumFromStorage&gt; picking the interesting properties of each album for us.

As the thumbnail for the album cover is a Stream, we need to use a WriteableBitmap for calling the [GetThumbnail()](http://msdn.microsoft.com/en-us/library/ff827731.aspx) method. The last step adds our ObservableCollection&lt;AlbumFromStorage&gt; as ItemSource to our ListBox.

This will be the result (based on my current albums list):

![wp_ss_20131111_0002](/assets/img/2013/11/wp_ss_20131111_0002.png "wp_ss_20131111_0002")

Now all we need to do is to make a tapped album playing with Windows Phone’s media Player. Add the following code to the Listbox ItemTap Event:

``` csharp
            if (Microsoft.Xna.Framework.Media.MediaPlayer.State != MediaState.Playing)
             {
                 //to delete the recent song (if there is any), just add this
                 MediaPlayer.Stop();

                 var selectedItem = this.MusicAlbumListBox.SelectedItem as AlbumFromStorage;

                 // play the SongCollection of the selected Album
                 MediaPlayer.Play(selectedItem.AlbumSongs);
             }
             else if (Microsoft.Xna.Framework.Media.MediaPlayer.State == MediaState.Playing)
             {
                 MessageBox.Show("You are already playing music. Please stop the music first.");

                 //your own handling could be different here, like asking the user for the desired action and perform  it then

             }
```


As you can see, I use the [MediaPlayer.State](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.mediaplayer.state.aspx) property to get the album playing. The album is a [SongCollection](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.songcollection.aspx) that holds all songs of the album. I recommend you to stop the last played song (if there is any) first with the [MediaPlayer.Stop()](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.mediaplayer.stop.aspx) method before start playing the first album song with the [MediaPlayer.Play()](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.mediaplayer.play.aspx) method. Otherwise, it *may* happen that the user hears a second from the old song.

Very important: you must handle the case that the user has already music playing. If not, your app is likely to not pass certification.

After we started playing the music, we naturally want to display the current song.

To achieve this, we need to add two events to our page constructor: [MediaPlayer.ActiveSongChanged](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.mediaplayer.activesongchanged.aspx) and [MediaPlayer.MediaStateChanged](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.mediaplayer.mediastatechanged.aspx).

Within the MediaPlayer\_MediaStateChanged event, add the following code:

``` csharp
           if (MediaPlayer.State == MediaState.Playing)
            {
                var activeSong = MediaPlayer.Queue.ActiveSong;
                Dispatcher.BeginInvoke(() =>
                {
                    MusicTileTitleBlock.Text = string.Format("playing:\n{0}\n{1}", activeSong.Name, activeSong.Artist);

                });

            }
            else if (MediaPlayer.State == MediaState.Paused)
            {
                var activeSong = MediaPlayer.Queue.ActiveSong;
                Dispatcher.BeginInvoke(() =>
                {
                    MusicTitleBlock.Text = string.Format("paused:\n{0}\n{1}", activeSong.Name, activeSong.Artist);                    
                });
            }
            else if (MediaPlayer.State == MediaState.Stopped)
            {
                Dispatcher.BeginInvoke(() =>
                {
                    MusicTitleBlock.Text = "music stopped"; 
                 });
            }
```
 
This way, we use the MediaState to display the the current state as well as the title and artist. As we are updating the UI thread, as Dispatcher is used to update the Text.

To change the title and the artist of the current playing song, use this code within the MediaPlayer\_ActiveSongChanged event:

``` csharp
            var activeSong = MediaPlayer.Queue.ActiveSong;
            Dispatcher.BeginInvoke(() =>
            {
                MusicTileTitleBlock.Text = string.Format("{0}\n{1}", activeSong.Name, activeSong.Artist);

            });
```
 
If you want to display also the album image, you will need to cache the image from the ItemTap event before, which would work fine with albums. If you choose [Playlists](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.medialibrary.playlists.aspx) instead of Albums, this will not work out well as you will not be able to get the image from [MediaPlayer.Queue.ActiveSong](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.media.mediaqueue.activesong.aspx).

I did a lot of research to get this feature(s) working the way they should. I took me several hours to figure everything out exactly, and I hope this post helps some of you out there to save some time on this.

With this post, you will be able to generate the whole experience your users deserve from the beginning to the end.

Until the next post, happy coding!