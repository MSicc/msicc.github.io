---
id: 5555
title: 'Xamarin Android: asymmetric encryption without any user input or hardcoded values'
date: '2018-04-19T21:13:48+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post I am showing you how to securely create a key pair for asymmetric encryption in your Xamarin Android app, without any user interaction needed and - more important - without any hardcoded values in your code.'
layout: post
permalink: /xamarin-android-asymmetric-encryption-without-any-user-input-or-hardcoded-values/
image: /assets/img/2018/04/lock_encyprtion_title_android.jpg
categories:
    - Android
    - 'Dev Stories'
    - Xamarin
tags:
    - Android
    - asymmetric
    - decryption
    - 'Electronic Codebook'
    - encryption
    - hardcoded
    - KeySize
    - 'private key'
    - 'public key'
    - RSA
    - 'user input'
    - xamarin
---

## The problem

Android is often said to be one of the most unsecure platforms one can use. This problem is home-made, as there is still a lot of fragmentation. There are thousand of models that do not get the latest updates and security patches, mostly because OEMs (seem to) not care (for different reasons, biggest reason is of course money). On the other side, there are still developers that save user names and passwords in plain text (which is the worst) or have hardcoded values in their code that make it way to easy to compromise encrypted data.

In the past, a lot of us developers made some of those mistakes. Be it because most of the popular samples around the web use hardcoded values (e. g. for the IV) or because of blindly copy &amp; pasting from other websites or by using badly implemented libraries. Everyone should stop using these and use methods that are more secure. One of the most secure ways to do so is to use asymmetric encryption with a private/public key pair. The Android OS is doing a lot to help us generating such a key pair, and I am going to show you how to use it.

## Android<span class="pl-en">KeyStore</span>

As the name already implies, Android uses the `AndroidKeyStore`to keep keys secure. The`AndroidKeyStore` is derived from the Java Security implementations and provides:

- generation of keys and key pairs
- key material that is maintained out of any application process
- the key material can be bound to security hardware
- additional usage limits are implemented in the OS
- certificate store

Read more on that topic in the official [Android documentation](https://developer.android.com/training/articles/keystore.html).

## Encryption with a key pair explained

If you want to handle sensitive data securely in your app (and you should), there are only two ways. Either you are not saving them (which will often keep users not returning to your app or even uninstalling it just because they must type them in over and over again) or encrypt these data before saving it. One of the more secure ways to encrypt data is to use a private/public key pair, also known as asymmetric encryption (because you use one key for encryption and the other for decryption).

The private key is only known to the issuer of the key. In the case of Android, it is the OS or the security hardware that is in built into the device. The private key should always be private, and Android does handle this for us. The public key can be given to external parties (like us developers) to use them for decryption of sensitive data. The OS adds an additional layer and makes sure that only your app(s) are able to use the public key (aka ‘key access validation’).

Of course Google does not make all and everything about that encryption and validation process public (for obvious reasons).

In this post, I will focus on the creation of such a key pair, on how to retrieve a key from the `AndroidKeyStore`and in the end, we will of course encrypt some data. My implementation is based on [this article series](https://proandroiddev.com/secure-data-in-android-encryption-7eda33e68f58), which provides a whole lot of explanation. If you want to know more about this topic, I absolutely recommend reading it. I will not go to deep into details, if you want to know more, once again, just read the articles linked above.

## Let the OS create a key pair for you

The Android OS has two generators – a `KeyGenerator` and a `KeyPairGenerator`. The `KeyGenerator` provides a single key, while we will focus on the `KeyPairGenerator`, which will give us a brand new private/public key pair.

The first step is to initialize the [KeyStore](https://developer.xamarin.com/api/type/Java.Security.KeyStore/) itself, which I am doing in the constructor of my helper class:

``` csharp
        public PlatformEncryptionKeyHelper(Context context, string keyName)
       {
           _context = context;
           _keyName = keyName.ToLowerInvariant();

           _androidKeyStore = KeyStore.GetInstance(KEYSTORE_NAME);
           _androidKeyStore.Load(null);
       }
```
 
The essential step here is to load the instance with `null`, otherwise all other operations will not work. You should also never change the keystore’s name unless you know exactly what you are doing.

Now that we have the `KeyStore` initialized, let’s go ahead and create a new key pair. As I support Android 5.0 (Lollipop) in my apps, I have also a fallback in place, as the current iteration is only available for device with Android 6 (Marshmallow) and above. Here is the code:

``` csharp
         public void CreateKeyPair()
        {
            DeleteKey();

            KeyPairGenerator keyGenerator =
                KeyPairGenerator.GetInstance(KeyProperties.KeyAlgorithmRsa, KEYSTORE_NAME);

            if (Build.VERSION.SdkInt >= BuildVersionCodes.JellyBeanMr2 &&
                Build.VERSION.SdkInt <= BuildVersionCodes.LollipopMr1)
            {
                var calendar = Calendar.GetInstance(_context.Resources.Configuration.Locale);
                var endDate = Calendar.GetInstance(_context.Resources.Configuration.Locale);
                endDate.Add(CalendarField.Year, 20);

                //this API is obsolete after Android M, but I am supporting Android L
#pragma warning disable 618
                var builder = new KeyPairGeneratorSpec.Builder(_context)
#pragma warning restore 618
                              .SetAlias(_keyName).SetSerialNumber(BigInteger.One)
                              .SetSubject(new X500Principal($"CN={_keyName} CA Certificate"))
                              .SetStartDate(calendar.Time)
                              .SetEndDate(endDate.Time).SetKeySize(KeySize);

                keyGenerator.Initialize(builder.Build());
            }
            else if (Build.VERSION.SdkInt >= BuildVersionCodes.M)
            {
                var builder =
                    new KeyGenParameterSpec.Builder(_keyName, KeyStorePurpose.Encrypt | KeyStorePurpose.Decrypt)
                        .SetBlockModes(KeyProperties.BlockModeEcb)
                        .SetEncryptionPaddings(KeyProperties.EncryptionPaddingRsaPkcs1)
                        .SetRandomizedEncryptionRequired(false).SetKeySize(KeySize);

                keyGenerator.Initialize(builder.Build());
            }

            keyGenerator.GenerateKeyPair();
        }
```
 
As you can see, the creation of such a key pair is way easier with Android 6 (Marshmallow) and above. I will focus on this part, details for the fallback solution can be found in the articles I linked above. I am requesting a RSA key pair for encryption and decryption, which needs to be specified explicitly. We are using the so called ‘ Electronic Codebook’ encryption mode, which will cut the data to encrypt into blocks that will be encrypted. Also important: the key’s size. A bigger key means more security, but also more time for operations done with it. Android defaults to a key size of 2048 bits, which provides a good average of security and execution time. With this method in place, we are already able to create a brand new key pair.

Note: The `DeleteKey()`method call beforehand just makes sure we have only one valid key pair with that name available. I am also following Google’s recommendations by calling it before creating a new key.

## Retrieving the public key for encryption

Now that the `AndroidKeyStore`holds a key pair for us, let us have a look on how to retrieve the public key, which is used for encryption:

``` csharp
 public IKey GetPublicKey()
{
    if (!_androidKeyStore.ContainsAlias(_keyName))
        return null;

    return _androidKeyStore.GetCertificate(_keyName)?.PublicKey;
}
```
 
Android internally creates a self signed certificate for the key pair (that’s why we had to perform this action manually before Android 6 (Marshmallow). The API makes this visible to us in the case of the retrieval of the public key. Xamarin provides the [IKey interface](https://developer.xamarin.com/api/type/Java.Security.IKey/), which is once again inherited from the Java Security APIs.

## Retrieving the private key for decryption

Of course, we want to decrypt the data we encrypted at some point. That is as easy as getting the public key:

``` csharp
 public IKey GetPrivateKey()
{
    if (!_androidKeyStore.ContainsAlias(_keyName))
        return null;

    return _androidKeyStore.GetKey(_keyName, null);
}
```
 
As we did not set a password during the key pair creation, we are passing `null` in here to get our private key.

## Deleting a key pair

There may be situations where you want to delete a key. The `AndroidKeyStore` has an API available for that as well. You may guess it, it is also very easy to use:

``` csharp
 public bool DeleteKey()
{
    if (!_androidKeyStore.ContainsAlias(_keyName))
        return false;

    _androidKeyStore.DeleteEntry(_keyName);
    return true;
}
```
 
## Usage

As you probably remember, I created a helper class for handling all things related to the `AndroidKeyStore`. Let’s have a look on how to encrypt and decrypt a string with the help of this class.

``` csharp
 _encryptionKeyHelper = new PlatformEncryptionKeyHelper(Application.Context, KeyStoreName);
_encryptionKeyHelper.CreateKeyPair();

_privateKey = _encryptionKeyHelper.GetPrivateKey();
_publicKey = _encryptionKeyHelper.GetPublicKey();
```
 
After instantiating the helper class, we use the `CreateKeyPair()`method to get a key pair. In the full class I will share later in this post, I have another helper that will check if the key already exists. You can use this class to step over the creation part if there is already a key pair.

Now let’s see how encryption works:

``` csharp
 //we used these values to create the keys
//now we need to tell the OS to use the same values during encryption/decryption
var transformation = "RSA/ECB/PKCS1Padding";

var stringToEncrypt = "This is a simple string for demo purposes only. Nothing special here.";

var cipher = Cipher.GetInstance(transformation);
cipher.Init(CipherMode.EncryptMode, _publicKey);

var encryptedData = cipher.DoFinal(Encoding.UTF8.GetBytes(stringToEncrypt));
```
 
We are using the [Cipher class provided by Xamarin](https://developer.xamarin.com/api/type/Javax.Crypto.Cipher/), which inherits from the Java Crypto API. The transformation string consists of “algorithm/mode/padding” and needs to be passed to the `cipher `instance. After specifying that we want to encrypt with the public key, the `DoFinal`method encrypts the string and returns it as a byte array, which can be saved pretty easy.

Decryption works in a similar way:

``` csharp
 var transformation = "RSA/ECB/PKCS1Padding"; 

var cipher = Cipher.GetInstance(transformation);
cipher.Init(CipherMode.DecryptMode, _privateKey);

var decryptedBytes = cipher.DoFinal(encyrptedData);
var finalString = Encoding.UTF8.GetString(decryptedBytes);
```
 
Once again, we are using the `Cipher`class. Remember to initialize the cipher instance once again, because we are using now the decryption mode. The `DoFinal`<span style="display: inline !important; float: none; background-color: transparent; color: #333333; cursor: text; font-family: Georgia,'Times New Roman','Bitstream Charter',Times,serif; font-size: 16px; font-style: normal; font-variant: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;">method</span> will decrypt the encrypted byte array, which can be turned into a string once again.

I did not create a sample project this time. However, the full helper class is [available here on my GitHub account](https://gist.github.com/MSiccDev/6c3b8bc532192cd3abb41982198e2373) as Gist.

*Xamarin.Forms tipp*: You can make this class available by extracting an interface from it and use the `DependencyService` to get access from your forms project if necessary.

## Conclusion

The security of your user’s data should always be something you are concerned about. With this little helper, we are using the OS (and in some cases also the device) to secure data in your Xamarin.Android app. Sadly, a lot of samples require user interaction or even use some hardcoded values. This should not be used in a production app. Feel free to use my helper class as a starting point.

As always, I hope this post is helpful for some of you. [In the next post]({% post_url 2018-04-27-how-to-perform-asymmetric-encryption-without-user-input-hardcoded-values-with-xamarin-ios %}), I will show you how to use a similar mechanism in your Xamarin.iOS app.

#### Until then, happy coding, everyone!