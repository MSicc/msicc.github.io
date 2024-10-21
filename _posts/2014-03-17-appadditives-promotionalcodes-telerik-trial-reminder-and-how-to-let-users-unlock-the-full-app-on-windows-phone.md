---
id: 4015
title: 'AppAdditives PromotionalCodes, Telerik trial reminder and how to let users unlock the full app on Windows Phone'
date: '2014-03-17T19:11:42+01:00'
author: 'Marco Siccardi'
layout: post
permalink: /appadditives-promotionalcodes-telerik-trial-reminder-and-how-to-let-users-unlock-the-full-app-on-windows-phone/
categories:
    - Archive
tags:
    - appadditives
    - code
    - components
    - exgrip
    - 'feature based trial'
    - 'full version'
    - promo
    - 'promo codes'
    - redeem
    - 'time based trial'
    - trial
    - unlock
    - validate
    - WinPhanDev
    - wpdev
---

I love it when I am discovering new awesome stuff to provide a unique User Experience in my apps. [AppAdditives](https://www.appadditives.com) by ExGrip LLC are the newest tools I felt in love with.

AppAdditives allow you to create promo images, social cards and widgets for your blog very easy. On top of that, they provide you an easy to use promotional code system. According to their developers, they are planning even more awesome stuff for Windows Phone and Windows 8 in future to help especially small and indie developers/publishers.

Let’s have a look on how easy it is to generate a list of promo codes after you registered for their free service.

![Screenshot (332)](/assets/img/2014/03/Screenshot-332.png "Screenshot (332)")

To generate a new list of promo codes, click on Promotional Codes. Then choose which way you provide – one time codes (every code gets invalid after being redeemed) or multi-user codes (one code for up to 2 million users). I am using the one time codes for my app.

![Screenshot (330)](/assets/img/2014/03/Screenshot-330.png "Screenshot (330)")

After you selected the proper time zone, you will get the following settings menu:

![settingsforpromocodes](/assets/img/2014/03/settingsforpromocodes.png "settingsforpromocodes")

Enter all your settings, and click on start to generate your list of promo codes. It will look like this:

![Screenshot (331)](/assets/img/2014/03/Screenshot-331.png "Screenshot (331)")

That’s all we need to do here. Let’s fire up Visual Studio. Open NuGet, and install the package ‘ExGrip.PromotionalCodes’ in your preferred way.

First, add your API Key and API Secret to your app (you will find them on the Promotional Codes page). Then, add the following code to your button/function that will redeem the code for your users:

``` csharp
if (PromoCodeTextBox.Text != string.Empty)
{
    PromotionCodeManager promoCodeMan = new PromotionCodeManager(AppAdditivesAPIKey, AppAdditivesAPISecret);

    bool validateCode = await promoCodeMan.ValidatePromoCode(this.PromoCodeTextBox.Text);

    if (validateCode == true)
    {
        progress.Text = "redeeming promo code...";

        bool redeemCode = await promoCodeMan.RedeemPromoCode(this.PromoCodeTextBox.Text);

        if (redeemCode == true)
        {
            App.isPromoCodeActivated = true;

            RedeemPromoCodeWindow.IsOpen = false;
            redeemPromoCode.IsEnabled = false;

            await RadMessageBox.ShowAsync("You unlocked the full version.", "Success", MessageBoxButtons.OK);

        }
        else
        {
            await RadMessageBox.ShowAsync("The code you entered cannot be redeemed. Please try again or contact our support.", "unable to redeem your code", MessageBoxButtons.OK);
        }
    }
    else
    {
        await RadMessageBox.ShowAsync("The code you entered is not valid. Please try again.", "invalid promo code", MessageBoxButtons.OK);
    }
}
```
 
As you can see, I am using an chain to first validate the promo code, and only if it is valid, I allow to redeem the code.

But that’s not all. I am using Telerik’s RadTrialApplicationReminder to manage the trial state of my app. Windows Phone does not allow to redeem a code for a full version via the Store, so we need to be creative here.

As I am not limiting features but use a time based trial, I can use three already existing properties of RadTrialApplicationReminder. If my Boolean is true after the app start, I am setting all periods to 9999 days and skip all further reminders:

``` csharp
if (isPromoCodeActivated == true)
{
    trialReminder.FreePeriod = TimeSpan.FromDays(9999);
    trialReminder.OccurrencePeriod = TimeSpan.FromDays(9999);
    trialReminder.AllowedTrialPeriod = TimeSpan.FromDays(9999);
    trialReminder.AreFurtherRemindersSkipped = true;
}
```
 
This way, I can easily provide this clearly missing features by using Promotional Codes from AppAdditives. The only thing I need to do is to tell users to only download the trial version of my app and give them a promo code.

Additional hint: If you have users that switch devices, they are not able to redeem the code again. Just tell that to your users, they will be either asking you for another code – or buy your app, anyways.

As always, I hope this post is helpful for some of you.

Happy coding, everyone!