---
id: 4070
title: 'How to send a mail from your Windows Phone 8.1 app'
date: '2014-04-24T11:04:18+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /how-to-send-a-mail-from-your-windows-phone-8-1-app/
categories:
    - Archive
tags:
    - addresses
    - Email
    - 'EMailManager class'
    - EmailRecipient
    - mail
    - 'send mail'
    - 'Windows Phone 8.1'
    - WP8.1
    - wpdev
---

### The old way

Sometimes, we need to pass some values to a newly generated E-Mail. In Windows Phone, we always had the EmailComposeTask for that:

``` csharp
 EmailComposeTask mailTask = new EmailComposeTask();
mailTask .To = "mail@domain.com";
mailTask .Subject = "this is the Subject";
mailTask.Body = "this is the Body";

mailTask.Show();
```
 
This way was pretty straight forward. It will work in a Windows Phone 8.1 Silverlight app, but not in a WINPRT app.

### The new way

I looked around and found the [EmailManager](http://msdn.microsoft.com/en-us/library/windowsphone/develop/windows.applicationmodel.email.emailmanager.aspx) class, which is also pretty easy to use. Let’s go through the code. First, we need to declare the [EmailRecipient](http://msdn.microsoft.com/en-us/library/windowsphone/develop/windows.applicationmodel.email.emailrecipient.aspx)(s):

``` csharp
 EmailRecipient sendTo = new EmailRecipient()
{
    Address = "mail@domain.com"
};
```
 
After that, we are now able to set up our [EmailMessage](http://msdn.microsoft.com/en-us/library/windowsphone/develop/windows.applicationmodel.email.emailmessage.aspx), which works in a similar way like the old EmailComposeTask:

``` csharp
 EmailMessage mail = new EmailMessage();
mail.Subject = "this is the Subject";
mail.Body = "this is the Body";
```
 
After setting up the Subject and the Body, we need to add our recipients to the EMailMessage. This is the only way to add “To”, “Bcc” and “CC” objects to our EMailMessage:

``` csharp
 mail.To.Add(sendTo);
//mail.Bcc.Add(sendTo);
//mail.CC.Add(sendTo);
```
 
Last but not least, we are calling the [ShowComposeNewEmailAsync()](http://msdn.microsoft.com/en-us/library/windowsphone/develop/windows.applicationmodel.email.emailmanager.showcomposenewemailasync.aspx) method of the EmailManager class, which will open the Share contract with mail only:

``` csharp
 await EmailManager.ShowComposeNewEmailAsync(mail);
```
 
![wp_ss_20140424_0002](/assets/img/2014/04/wp_ss_20140424_0002.png "wp_ss_20140424_0002")

Here is once again the complete code for you to copy:

``` csharp
 //predefine Recipient
EmailRecipient sendTo = new EmailRecipient()
{
    Address = "mail@domain.com"
};

//generate mail object
EmailMessage mail = new EmailMessage();
mail.Subject = "this is the Subject";
mail.Body = "this is the Body";

//add recipients to the mail object
mail.To.Add(sendTo);
//mail.Bcc.Add(sendTo);
//mail.CC.Add(sendTo);

//open the share contract with Mail only:
await EmailManager.ShowComposeNewEmailAsync(mail);
```
 
The new way works for both Windows Phone 8.1 Runtime and Silverlight apps.

### The dirty way:

I found also another way that works with an Uri including the values as parameters launched by LaunchUriAsync() that gets recognized by the OS:

``` csharp
 var mail = new Uri("mailto:?to=tickets@msiccdev.uservoice.com&subject=this is the Subject&body=this is the Body");
await Launcher.LaunchUriAsync(mail);
```
 
The parameters get automatically parsed by the Mail app.

I’ll leave it up to you which way you are using.

As always, I hope this is helpful for some of you.

Happy coding!