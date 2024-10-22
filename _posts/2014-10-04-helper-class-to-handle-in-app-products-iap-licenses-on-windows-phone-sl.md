---
id: 4165
title: 'Helper class to handle In-App-Products (IAP) licenses on Windows Phone (SL)'
date: '2014-10-04T02:26:29+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /helper-class-to-handle-in-app-products-iap-licenses-on-windows-phone-sl/
categories:
    - Archive
tags:
    - app
    - IAP
    - In-App-Products
    - 'timed storage'
    - 'Windows Phone'
    - wpdev
---

Like a lot of other apps, also [UniShare](https://www.windowsphone.com/s?appid=ee42cb1d-8a68-41c6-9c0c-d3e3fc61d6ea) got hit by an outage in the licensing system of the Windows Phone Store recently.

There was a workaround to get the licenses back from the store, but even with that, users have left negative votes (even [after I posted the workaround](https://apps.msicc.net/2014/09/22/how-to-get-the-already-paid-premium-features-back-in-unishare/) and also [wpcentral wrote about it a few days later](https://www.google.com/url?q=https://www.wpcentral.com/fix-windows-phone-store-iap-breakage-close&sa=U&ei=S-4uVI_GLMvfPYyOgeAK&ved=0CA0QFjAE&client=internal-uds-cse&usg=AFQjCNHN3l9pHnh2SSwRIpYqXBeJu57Mlg)).

The problem with those negative reviews: they tend to remain, even after responding to them (if possible) or having conversation with the users via mail. I totally understand users that are annoyed by such facts – I got hit by it as well with other apps I am using regularly. So I was thinking about a better solution as the recommended way by Microsoft, which says you should check it on every app start.

[Rob Irving](https://twitter.com/robwirving) posted about his solution [at the same day wpcentral wrote about it](https://robwirving.com/2014/09/25/dont-lose-customers-durable-iaps-back-app-settings/), which is one possible solution. His motive was the same as my solution – improving the user experience with the IAP’s.

However, I am preferring to check the licenses against the Store from time to time to make sure that the licenses are still correct.

Here is my solution (for durable IAP):

First, let’s we need to declare an object for the ListingInformation, which will hold the information that the store returns for our IAP:

``` csharp
 ListingInformation IAPListing;
```
 
Then, we need to create these two classes:

``` csharp
         public class IAP
        {
            public string Key { get; set; }

            public string Name { get; set; }

            public string Price { get; set; }

            public ProductType Type { get; set; }

            public string Description { get; set; }

            public string Image { get; set; }

            public bool IsLicenseActive { get; set; }
        }

        public class IAPToSave
        {
            public List<IAP> IAPListToSave { get; set; }

            public DateTime date { get; set; }
        }
```
 
The class IAP is the class/model that holds a single IAP item information. The second class is needed for saving the fetched IAP information (we’ll see later how I did it).

Now we have prepared these, we can finally go to the store and fetch the IAP list. I created an async Task that returns a List&lt;IAP&gt; for it:

``` csharp
         public async Task<List<IAP>> GetAllIAP()
        {
            var list = new List<IAP>();

            IAPListing = await CurrentApp.LoadListingInformationAsync();

            foreach (var product in IAPListing.ProductListings)
            {
                list.Add(new IAP()
                {
                    Key = product.Key,
                    Name = product.Value.Name,
                    Description = product.Value.Description,
                    Price = product.Value.FormattedPrice,
                    Type = product.Value.ProductType,
                    Image = product.Value.ImageUri.ToString(),
                    IsLicenseActive = isPackageUnlocked(product.Key)
                });
            }
            return list;
        }
```
 
To immediately check if our user has already purchased the item, we are getting a Boolean for it. While we are getting the data from the store, we are using this to fill our IAP class with the desired value (see IsLicenseActive property in the IAP item above).

``` csharp
         public bool isPackageUnlocked(string productKey)
        {
            var licenseInformation = CurrentApp.LicenseInformation;

            if (licenseInformation.ProductLicenses[productKey].IsActive)
            {
                return true;
            }
            else
            {
                return false;
            }
        }
```
 
Now we have already all data that we need to display all IAP in our app. The usage is fairly simple:

``` csharp
 var iapHelper = new IAPHelper();
var iapList = await iapHelper.GetAllIAP();
```
 
You can now bind the iapList to a ListBox (or your proper control/view). Next thing we are creating is our helper that performs the purchase action and returns a result string to display in a message to the user in our IAPHelper class:

``` csharp
         public async Task<string> unlockPackage(string productKey, string productName)
        {
            var licenseInformation = CurrentApp.LicenseInformation;

            string response = string.Empty;

            if (!licenseInformation.ProductLicenses[productKey].IsActive)
            {
                try
                {
                    //opening the store to display the purchase page
                    await CurrentApp.RequestProductPurchaseAsync(productKey);

                    //getting the result after returning into app
                    var isUnlocked = isPackageUnlocked(productKey);

                    if (isUnlocked == true)
                    {
                        response = string.Format("You succesfully unlocked {0}.", productName);  
                    }
                    else
                    {
                        response = string.Format("There was an error while trying to unlock {0}. Please try again.", productName);
                    }
                }
                catch (Exception)
                {
                    response = string.Format("There was an error while trying to unlock {0}. Please try again.", productName);
                }
            }
            else
            {
                response = string.Format( "You already unlocked {0}", productName);
            }

            return response;
        }
```
 
You may of course vary the messages that are displayed to the user to your favor.

The usage of this Task is also pretty straight forward:

``` csharp
 var iapHelper = new IAPHelper();
string message = await iapHelper.unlockPackage(((IAPHelper.IAP)IAPListBox.SelectedItem).Key, ((IAPHelper.IAP)IAPListBox.SelectedItem).Name);
```
 
After that, we need to refresh the LicenseInformation by using GetAllIAP() again to refresh the iapList and of course our ListBox.

My goal was to save the LicenseInformation of my IAP for a limited time so the user is protected for future outages (or situations where no network connection is available). That’s why we need to add another Task to our IAPHelper class:

``` csharp
         public async Task<string> SerializedCurrentIAPList(List<IAP> iaplist, DateTime lastchecked)
        {
            string json = string.Empty;

            IAPToSave listToSave = new IAPToSave() { IAPListToSave = iaplist, date = lastchecked };

            if (iaplist.Count != 0)
            {
                json = await Task.Factory.StartNew(() => JsonConvert.SerializeObject(listToSave));
            }
            return json;
        }
```
 
As you can see, now is the point where we need the second class I mentioned in the beginning. It has one property for the List&lt;IAP&gt; and a DateTime property that we are saving. I am serializing the whole class to a JSON string. This way, we are able to save it as a string in our application’s storage.

The usage of this Task is as simple as the former ones:

``` csharp
 var savedIAPList = await iapHelper.SerializedCurrentIAPList(iapList, DateTime.Now);
```
 
The last thing we need to create is an object that helps us indicating if a refresh of the list is needed or not. Like I said, I want to do this action time based, so here is my way to get this value:

``` csharp
         public bool isReloadNeeded(DateTime lastchecked, TimeSpan desiredtimespan)
        {
            bool reloadNeeded = false;

            var now = DateTime.Now;

            TimeSpan ts_lastchecked = now - lastchecked;

            if (ts_lastchecked > desiredtimespan)
            {
                reloadNeeded = true;
            }

            return reloadNeeded;
        }
```
 
This method just checks if the desired TimeSpan has passed already and returns the matching Boolean value. The usage of this is likewise pretty simple:

``` csharp
 var savedlist = await Task.Factory.StartNew(() => JsonConvert.DeserializeObject<IAPHelper.IAPToSave>(App.SettingsStore.savedIAPList));

if (iapHelper.isReloadNeeded(savedlist.date, new TimeSpan(96, 0, 0)))
{
   //reload the list and perform your actions
}
else
{
   //use savedlist.IAPListToSave and perform your actions 
}
```
 
This way, we are able to make sure that all IAPs are available for a minimum of time and protect our users against store outages.

Note: I know that this approach does not follow the recommended way of Microsoft. It is up to us to deal with bad reviews if something on the store part is not working. This is my approach to avoid negative reviews because of store outages (at least for a limited time). However, like always, I hope this post is helpful for some of you.

Happy coding!