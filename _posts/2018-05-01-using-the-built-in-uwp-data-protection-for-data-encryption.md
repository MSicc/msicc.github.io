---
id: 5596
title: 'Using the built-in UWP data protection for data encryption'
date: '2018-05-01T08:00:22+02:00'
author: 'Marco Siccardi'
excerpt: 'To sum up my posts on data encryption for (mobile) applications, I''ll show you how encrypt and decrypt data in an Universal Windows Platform app with the built-in DataProtection API.'
layout: post
permalink: /using-the-built-in-uwp-data-protection-for-data-encryption/
image: /assets/img/2018/05/data_protection_uwp_title.jpg
categories:
    - 'Dev Stories'
    - Windows
    - Xamarin
tags:
    - asymmetric
    - 'data protection'
    - decryption
    - encryption
    - uwp
    - xamarin
---

## DataProtectionProvider

The UWP has an easy-to-use helper class to perform encryption tasks on data. Before you can use the [DataProtectionProvider class](https://docs.microsoft.com/en-us/uwp/api/Windows.Security.Cryptography.DataProtection.DataProtectionProvider), you have to decide on which level you want the encryption to happen. The level is derived from the protection descriptor, which derives the encryption material internally from the user data specified. For non-enterprise apps, you can choose between four levels (passed in as `protectionDescriptor `parameter with the constructor):

- “LOCAL=user”
- “LOCAL=machine”
- “WEBCREDENTIALS=MyPasswordName”
- “WEBCREDENTIALS=MyPasswordName,myweb.com”

If you want the encryption to be only for the currently logged-in user, use the first provider, for a machine-wide implementation use the second one in the list above. The providers for web credentials are intended to be used within the browser/web view. Personally, I never used them, because of this, this post will focus on local implementations.

## Encrypting data

There are two ways to encrypt data with the `DataProtectionProvider`. You can encrypt a s string or a stream. My implementation uses always the stream implementation. This way I can encrypt whole files if needed and simple strings with the same method. Here is the encryption `Task`:

``` csharp
 public async Task<byte[]> Encrypt(byte[] data)
{
    var provider = new DataProtectionProvider(UseForAllUsers ? _localMachineDescriptor : _localUserDescriptor);

    var contentBuffer = CryptographicBuffer.CreateFromByteArray(data);
    var contentInputStream = new InMemoryRandomAccessStream();
    var protectedContentStream = new InMemoryRandomAccessStream();

    //storing data in the stream
    IOutputStream outputStream = contentInputStream.GetOutputStreamAt(0);
    var dataWriter = new DataWriter(outputStream);
    dataWriter.WriteBuffer(contentBuffer);
    await dataWriter.StoreAsync();
    await dataWriter.FlushAsync();

    //reopening in input mode
    IInputStream encodingInputStream = contentInputStream.GetInputStreamAt(0);

    IOutputStream protectedOutputStream = protectedContentStream.GetOutputStreamAt(0);
    await provider.ProtectStreamAsync(encodingInputStream, protectedOutputStream);
    await protectedOutputStream.FlushAsync();

    //verify that encryption happened
    var inputReader = new DataReader(contentInputStream.GetInputStreamAt(0));
    var protectedReader = new DataReader(protectedContentStream.GetInputStreamAt(0));

    await inputReader.LoadAsync((uint)contentInputStream.Size);
    await protectedReader.LoadAsync((uint)protectedContentStream.Size);

    var inputBuffer = inputReader.ReadBuffer((uint)contentInputStream.Size);
    var protectedBuffer = protectedReader.ReadBuffer((uint)protectedContentStream.Size);

    if (!CryptographicBuffer.Compare(inputBuffer, protectedBuffer))
    {
       return protectedBuffer.ToArray();
    }
    else
    {
        return null;
    }
}
```
 
First, I am defining the provider based on a boolean property. The passed in byte array is then converted to an IBuffer, which in turn makes it easier to pass it to the InMemoryRandomAccessStream that we will encrypt. After writing the data into an this stream, we need to reopen the stream in input mode, which allows us to call the `ProtectStreamAsync`method. By comparing the both streams, we are finally verifying that encryption happened.

## Decrypting data

Of course it is very likely we want to decrypt data we encrypted with the above method at some point. We are basically reversing the process above with the following `Task`:

``` csharp
 public async Task<byte[]> Decrypt(byte[] encryptedBytes)
{
    var provider = new DataProtectionProvider();

    var encryptedContentBuffer = CryptographicBuffer.CreateFromByteArray(encryptedBytes);
    var contentInputStream = new InMemoryRandomAccessStream();
    var unprotectedContentStream = new InMemoryRandomAccessStream();

    IOutputStream outputStream = contentInputStream.GetOutputStreamAt(0);
    var dataWriter = new DataWriter(outputStream);
    dataWriter.WriteBuffer(encryptedContentBuffer);
    await dataWriter.StoreAsync();
    await dataWriter.FlushAsync();

    IInputStream decodingInputStream = contentInputStream.GetInputStreamAt(0);

    IOutputStream protectedOutputStream = unprotectedContentStream.GetOutputStreamAt(0);
    await provider.UnprotectStreamAsync(decodingInputStream, protectedOutputStream);
    await protectedOutputStream.FlushAsync();

    DataReader reader2 = new DataReader(unprotectedContentStream.GetInputStreamAt(0));
    await reader2.LoadAsync((uint)unprotectedContentStream.Size);
    IBuffer unprotectedBuffer = reader2.ReadBuffer((uint)unprotectedContentStream.Size);

    return unprotectedBuffer.ToArray();
}
```
 
For decryption, we do not need to specify the protection descriptor again. The OS will find the right one on its own. While I have read that this sometimes makes problems, I haven’t got any problems until now. Once again, we are creating `InMemoryRandomAccessStream`instances to perform decryption on it. Finally, after we unprotected our data with the `UnProtectStreamAsync`method, we are reading the data back into a byte array.

## Usage

The usage of these two helpers is once again pretty straight forward. Let’s have a look at data encryption:

``` csharp
 var textToCrypt = "this is just a plain test text that will be encrypted and decrypted";

//async method
var encrypted = await Encrypt(Encoding.UTF8.GetBytes(textToCrypt));
//non async method:
//var encrypted = Encrypt(Encoding.UTF8.GetBytes(textToCrypt)).GetAwaiter().GetResult();
```
 
Decryption of encoded data is done like this:

``` csharp
 //async method
var decrypted = await Decrypt(encrypted);

//non async method
//var decrypted = Decrypt(encrypted).GetAwaiter().GetResult();

//getting back the string:
var decryptedString = Encoding.UTF8.GetString(decrypted);
```
 
## Conclusion

Using the built in `DataProtection `API makes it very easy to protect sensitive data within an UWP app. You can use it on both a string or a stream, it is up to you if you follow my way to use always a stream or make it dependent on your scenario. If you want to have more control over the key size, the encryption method used or any other detail, I wrote a post about doing everything on my own a while back ([find it here]({% post_url 2015-05-20-string-encryption-in-windows-8-1-universal-apps %})). Even if it is from 2015 and based on WINRT (Win8.1), the APIs are still alive and the post should help you to get started.

You can have a look on Android encryption [here]({% post_url 2018-04-19-xamarin-android-asymmetric-encryption-without-any-user-input-or-hardcoded-values %}), while you’ll find the iOS post [here]({% post_url 2018-04-27-how-to-perform-asymmetric-encryption-without-user-input-hardcoded-values-with-xamarin-ios %}).

As always, I hope this post will be helpful for some of you. If you have any questions/feedback, feel free to comment below or connect with me via my social networks.

#### Happy coding, everyone!