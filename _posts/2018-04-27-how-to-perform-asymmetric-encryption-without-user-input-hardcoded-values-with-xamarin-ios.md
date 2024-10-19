---
id: 5583
title: 'How to perform asymmetric encryption without user input/hardcoded values with Xamarin iOS'
date: '2018-04-27T19:54:48+02:00'
author: 'Marco Siccardi'
excerpt: 'Like I promised already in my blog post about asymmetric encryption on Android with Xamarin, this post is all about asymmetric encryption on iOS, utilizing the built-in KeyChain API of iOS - once again, without any user input and any hardcoded values.'
layout: post
permalink: /how-to-perform-asymmetric-encryption-without-user-input-hardcoded-values-with-xamarin-ios/
image: /assets/img/2018/04/key_chain_encryption_title_ios.jpg
categories:
    - 'Dev Stories'
    - iOS
    - Xamarin
tags:
    - asymmetric
    - decryption
    - encryption
    - hardcoded
    - iOS
    - ipad
    - iPhone
    - KeySize
    - 'private key'
    - 'public key'
    - RSA
    - 'user input'
    - xamarin
---

I am not repeating all the initial explanations why you should use this way of encryption to secure sensible data in your app(s), as I did this already in the post on [how to do that on Android](https://msicc.net/xamarin-android-asymmetric-encryption-without-any-user-input-or-hardcoded-values/).

## iOS KeyChain

The [KeyChain API](https://developer.apple.com/documentation/security/certificate_key_and_trust_services/keys) is the most important security element on iOS (and MacOS). The interaction with it is not as difficult as one thinks, after getting the concept for the different using scenarios it supports. In our case, we want to create a public/private key pair for use within our app. Pretty much like on Android, we want no user input and no hardcoded values to get this pair.

## Preparing key (pair) creation

On iOS, things are traditionally a little bit more complex than on other platforms. This is also true for things like encryption. The first step would be to prepare the RSA parameters we want to use for encryption. However, that turned out to be a bit challenging because we need to pass in some keys and values that live in the native iOS security library and Xamarin does not fully expose them. Luckily, there is at least a [Xamarin API that helps us to extract those values](https://developer.xamarin.com/api/type/ObjCRuntime.Dlfcn/). I found [this SO post](https://stackoverflow.com/questions/49381392/c-sharp-how-to-store-rsa-key-pair-on-ios-keychain-xamarin) helpful to understand what is needed for the creation of the key pair. I adapted some of the snippets into my own helper class, this is also true for the `IosConstants`class:

``` csharp
     internal class IosConstants
    {
        private static IosConstants _instance;

        public static IosConstants Instance => _instance ?? (_instance = new IosConstants());

        public readonly NSString KSecAttrKeyType;
        public readonly NSString KSecAttrKeySize;
        public readonly NSString KSecAttrKeyTypeRSA;
        public readonly NSString KSecAttrIsPermanent;
        public readonly NSString KSecAttrApplicationTag;
        public readonly NSString KSecPrivateKeyAttrs;
        public readonly NSString KSecClass;
        public readonly NSString KSecClassKey;
        public readonly NSString KSecPaddingPKCS1;
        public readonly NSString KSecAccessibleWhenUnlocked;
        public readonly NSString KSecAttrAccessible;

        public IosConstants()
        {
            var handle = Dlfcn.dlopen(Constants.SecurityLibrary, 0);

            try
            {
                KSecAttrApplicationTag = Dlfcn.GetStringConstant(handle, "kSecAttrApplicationTag");
                KSecAttrKeyType = Dlfcn.GetStringConstant(handle, "kSecAttrKeyType");
                KSecAttrKeyTypeRSA = Dlfcn.GetStringConstant(handle, "kSecAttrKeyTypeRSA");
                KSecAttrKeySize = Dlfcn.GetStringConstant(handle, "kSecAttrKeySizeInBits");
                KSecAttrIsPermanent = Dlfcn.GetStringConstant(handle, "kSecAttrIsPermanent");
                KSecPrivateKeyAttrs = Dlfcn.GetStringConstant(handle, "kSecPrivateKeyAttrs");
                KSecClass = Dlfcn.GetStringConstant(handle, "kSecClass");
                KSecClassKey = Dlfcn.GetStringConstant(handle, "kSecClassKey");
                KSecPaddingPKCS1 = Dlfcn.GetStringConstant(handle, "kSecPaddingPKCS1");
                KSecAccessibleWhenUnlocked = Dlfcn.GetStringConstant(handle, "kSecAttrAccessibleWhenUnlocked");
                KSecAttrAccessible = Dlfcn.GetStringConstant(handle, "kSecAttrAccessible");

            }
            finally
            {
                Dlfcn.dlclose(handle);
            }
        }
    }
```
 
This class picks out the values we need to create the RSA parameters that will be passed to the KeyChain API later. No we have everything in place to create those with this helper method:

``` csharp
 private NSDictionary CreateRsaParams()
{
    IList<object> keys = new List<object>();
    IList<object> values = new List<object>();

    //creating the private key params
    keys.Add(IosConstants.Instance.KSecAttrApplicationTag);
    keys.Add(IosConstants.Instance.KSecAttrIsPermanent);
    keys.Add(IosConstants.Instance.KSecAttrAccessible);

    values.Add(NSData.FromString(_keyName, NSStringEncoding.UTF8));
    values.Add(NSNumber.FromBoolean(true));
    values.Add(IosConstants.Instance.KSecAccessibleWhenUnlocked);

    NSDictionary privateKeyAttributes = NSDictionary.FromObjectsAndKeys(values.ToArray(), keys.ToArray());

    keys.Clear();
    values.Clear();

    //creating the keychain entry params
    //no need for public key params, as it will be created from the private key once it is needed
    keys.Add(IosConstants.Instance.KSecAttrKeyType);
    keys.Add(IosConstants.Instance.KSecAttrKeySize);
    keys.Add(IosConstants.Instance.KSecPrivateKeyAttrs);

    values.Add(IosConstants.Instance.KSecAttrKeyTypeRSA);
    values.Add(NSNumber.FromInt32(this.KeySize));
    values.Add(privateKeyAttributes);

    return NSDictionary.FromObjectsAndKeys(values.ToArray(), keys.ToArray());
}
```
 
In order to use the `SecKey`API to create a random key for us, we need to pass in a `NSDictionary`that holds a list of private key attributes and is attached to a parent `NSDictionary` that holds it together with some other configuration values for the KeyChain API. If you want, you could also create a `NSDictionary` for the public key, but that is not needed for my implementation as I request it later from the private key (we’ll have a look on that as well).

## Finally, let the OS create a private key

Now we have all our parameters in place, we are able to create a new private key by calling the [SecKey.CreateRandomKey()](https://developer.xamarin.com/api/member/Security.SecKey.CreateRandomKey/p/Foundation.NSDictionary/Foundation.NSError@/) method:

``` csharp
 public bool CreatePrivateKey()
{
    Delete();
    var keyParams = CreateRsaParams();

    SecKey.CreateRandomKey(keyParams, out var keyCreationError);

    if (keyCreationError != null)
    {
        Debug.WriteLine($"{keyCreationError.LocalizedFailureReason}\n{keyCreationError.LocalizedDescription}");
    }

    return keyCreationError == null;
}
```
 
Like on Android, it is a good idea to call into the Delete() method before creating a new key (I’ll show you that method later). This makes sure your app uses just one key with the specified name. After that, we create a new random key with the help of the OS. Because we specified it to be a private key for RSA before, it will be exactly that. If there is an error, we will return false and print it in the Debug console.

## Retrieving the private key

Now we have created the new private key, we are able to retrieve it like this:

``` csharp
 public SecKey GetPrivateKey()
{
    var privateKey = SecKeyChain.QueryAsConcreteType(
        new SecRecord(SecKind.Key)
        {
            ApplicationTag = NSData.FromString(_keyName, NSStringEncoding.UTF8),
            KeyType = SecKeyType.RSA,
            Synchronizable = shouldSyncAcrossDevices
        },
        out var code);

    return code == SecStatusCode.Success ? privateKey as SecKey : null;
}
```
 
We are using the [QueryAsConcreteType method](https://developer.xamarin.com/api/member/Security.SecKeyChain.QueryAsConcreteType/p/Security.SecRecord/Security.SecStatusCode@/) to find our existing key in the Keychain. If the OS does not find the key, we are returning null. In this case, we would need to create a new key.

## Retrieving the public key for encryption

Of course, we need a public key if we want to encrypt our data. Here is how to get this public key from the private key:

``` csharp
 public SecKey GetPublicKey()
{
    return GetPrivateKey()?.GetPublicKey();
}
```
 
Really, that’s it. Even if we are not creating a public key explicitly when we are creating our private key, we are getting a valid public key for encryption from the `GetPublicKey`[method](https://developer.xamarin.com/api/member/Security.SecKey.GetPublicKey()/), called on the private key instance.

## Deleting the key pair

Like I said already earlier, sometimes we need to delete our encryption key(s). This little helper method does the job for us:

``` csharp
 public bool Delete()
{
    var findExisting = new SecRecord(SecKind.Key)
    {
        ApplicationTag = NSData.FromString(_keyName, NSStringEncoding.UTF8),
        KeyType = SecKeyType.RSA,
        Synchronizable = _shouldSyncAcrossDevices
    };

    SecStatusCode code = SecKeyChain.Remove(findExisting);

    return code == SecStatusCode.Success;
}
```
 
This time, we are searching for a [SecRecord](https://developer.xamarin.com/api/type/Security.SecRecord/) with the kind key, and calling the `Remove`method of the [SecKeyChain](https://developer.xamarin.com/api/type/Security.SecKeyChain/) API. Based on the status code, we finally return a bool that indicates if we were successful. Note: When we create a new key, (actually) I do not care about the status and just create a new one in my helper class. If we delete the key from another place, we are probably going to work with that status code.

As I did with the Android version, I did not create a demo project, but you can [have a look at the full class in this Gist](https://gist.github.com/MSiccDev/498d8e3a9f747c2299713a4aca99d89a) on GitHub.

## Usage

Now that we have our helper in place, we are able to encrypt and decrypt data in a very easy way. First, we need to obtain a private and a public key:

``` csharp
 var helper = new PlatformEncryptionKeyHelper("testKeyHelper");

if (!helper.KeysExist())
{
    helper.CreateKeyPair();
}

var privKey = helper.GetPrivateKey();
var pubKey = helper.GetPublicKey();
```
 
The [encryption method](https://developer.xamarin.com/api/member/Security.SecKey.Encrypt/p/Security.SecPadding/System.Byte[]/System.Byte[]@/) needs to be called directly on the public key instance:

``` csharp
 var textToCrypt = "this is just a plain test text that will be encrypted and decrypted";
pubKey.Encrypt(SecPadding.PKCS1, Encoding.UTF8.GetBytes(textToCrypt), out var encBytes);
```
 
For getting the plain value back, we need to call the [decryption method](https://developer.xamarin.com/api/member/Security.SecKey.Decrypt/p/Security.SecPadding/System.Byte[]/System.Byte[]@/) on the private key instance:

``` csharp
 privKey.Decrypt(SecPadding.PKCS1, encBytes, out var decBytes);
var decrypted = Encoding.UTF8.GetString(decBytes);
```
 
It may make sense to wrap these calls into helper methods, but you could also just use it like I did for demoing purposes. Just remember to use always the same padding method, otherwise you will not get any value back from the encrypted byte array.

Once again, if you need encryption of data in a *Xamarin.Forms* project, just extract an interface from the class or match it the interface you may already have extracted from the Android version. As I stated already before, every developer should use the right tools to encrypt data really securely in their apps. With that post, you now have also a starting point for your own Xamarin iOS implementation.

Like always, I hope this will be helpful for some of you. In my next post, we will have a look into the OS provided options for encryption and decryption on Windows 10 (UWP).

### Until then, happy coding, everyone!