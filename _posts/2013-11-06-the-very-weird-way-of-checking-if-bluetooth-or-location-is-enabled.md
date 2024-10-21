---
id: 3801
title: 'The very weird way of checking if Bluetooth or Location is enabled'
date: '2013-11-06T20:05:41+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /the-very-weird-way-of-checking-if-bluetooth-or-location-is-enabled/
categories:
    - Archive
tags:
    - '0x80004004'
    - '0x8007048F'
    - API
    - GeoLocation
    - GeoPosition
    - HResult
    - Location
    - Proxmitiy
    - 'Windows Phone'
    - WP8
    - wp8dev
    - wpdev
---

![BT_GPS_WP_Blog](/assets/img/2013/11/BT_GPS_WP_Blog.png "BT_GPS_WP_Blog")

As I am in constant development of new features for my [NFC Toolkit](https://www.windowsphone.com/s?appid=2c33cb7d-c97b-4204-aa8b-1e8712718519), I came to the point where I needed to detect if Bluetooth and Location is enabled or not.

I searched about an hour across the internet, searched all well known WPDev sites as well as the MSDN Windows Phone documentation.

The solution is a very weird one.

As I change the opacitiy of an Image depending on the stauts (on/off), I created the following async Task to check:

``` csharp
 private async Task GetBluetoothState()
 {
 PeerFinder.AlternateIdentities["Bluetooth:Paired"] = "";

            try
 {
 var peers = await PeerFinder.FindAllPeersAsync();

                Dispatcher.BeginInvoke(() =>
 {
 BluetoothTileImage.Opacity = 1;
 });
 }
 catch (Exception ex)
 {
 if ((uint)ex.HResult == 0x8007048F)
 {
 Dispatcher.BeginInvoke(() =>
 {
 BluetoothTileImage.Opacity = 0.5;
 });
 }
 }
 }
```
 
As you can see above, we are searching for already paired devices with the [Proximity API](https://msdn.microsoft.com/en-us/library/windowsphone/develop/windows.networking.proximity.aspx) of Windows Phone. If we don’t have any of our already paired devices reachable, and we don’t throw an exception with the HResult of “0x8007048F”, Bluetooth is on. If the exception is raised, Bluetooth is off.

In a very similar way we need to check if the location setting is on:

``` csharp
 private async Task GetLocationServicesState()
 {
 Geolocator geolocator = new Geolocator();

try
 {
 Geoposition geoposition = await geolocator.GetGeopositionAsync(
 maximumAge: TimeSpan.FromMinutes(5),
 timeout: TimeSpan.FromSeconds(10)
 );

Dispatcher.BeginInvoke(() =>
 {
 LocationStatusTileImage.Opacity = 1;
 });

}
 catch (Exception ex)
 {
 if ((uint)ex.HResult == 0x80004004)
 {
 Dispatcher.BeginInvoke(() =>
 {
 LocationStatusTileImage.Opacity = 0.5;
 });
 }
 else
 {
 //tbd.
 }

}
}
```
 
For the location services, the HResult is “0x80004004”. We are trying to get the actual GeoLocation, and if the exception is thrown, location setting is off.

On Twitter, I got for the later one also another suggestion to detect if the location settings is enabled or not, by [Kunal Chowdhury](https://twitter.com/kunal2383) (=&gt;follow him!):

``` chsarp
geoLocator.LocationStatus == PositionStatus.Disabled;
```
 
This would work technically, but PositionStatus has 6 enumarations. Also, as stated here in the [Nokia Developer Wiki](https://developer.nokia.com/Community/Wiki/Get_Phone_Location_with_Windows_Phone_8), this can be a battery intese call (depends on the implementation). I leave it to you which one you want to use.

Back to the header of this post. Catching an exception to determine the Status of wireless connections just seems wrong to me. I know this is a working “solution” and we can use that. But it could have been better implemented (for example like the networking API).

I hope this post is helpful for some of you.

Until then, happy coding!