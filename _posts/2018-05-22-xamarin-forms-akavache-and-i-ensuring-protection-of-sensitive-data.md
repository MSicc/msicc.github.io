---
id: 5648
title: 'Xamarin.Forms, Akavache and I: ensuring protection of sensitive data'
date: '2018-05-22T18:30:22+02:00'
author: 'Marco Siccardi'
excerpt: 'Nowadays it is more important than ever to ensure sensitive data to be protected. In this post, I''ll show you how to ensure data protection within your Akavache powered Xamarin.Forms app.'
layout: post
permalink: /xamarin-forms-akavache-and-i-ensuring-protection-of-sensitive-data/
image: /assets/img/2018/05/akavache-data-protection-title.png
categories:
    - Android
    - 'Dev Stories'
    - iOS
    - Windows
    - Xamarin
tags:
    - akavache
    - Android
    - asymmetric
    - cache
    - caching
    - 'data protection'
    - database
    - decryption
    - encryption
    - iencryptionprovider
    - iOS
    - 'secure blob cache'
    - sqlite
    - uwp
    - Windows
    - xamarin
    - 'xamarin forms'
---

### Recap

Some of you might remember my posts about encryption for Android, iOS and Windows 10. If not, take a look here:

> [Xamarin Android: asymmetric encryption without any user input or hardcoded values](https://msicc.net/xamarin-android-asymmetric-encryption-without-any-user-input-or-hardcoded-values/)

<iframe class="wp-embedded-content" data-secret="TVnfOXk5J0" frameborder="0" height="338" loading="lazy" marginheight="0" marginwidth="0" sandbox="allow-scripts" scrolling="no" security="restricted" src="https://msicc.net/xamarin-android-asymmetric-encryption-without-any-user-input-or-hardcoded-values/embed/#?secret=82byWdeXKC#?secret=TVnfOXk5J0" style="position: absolute; clip: rect(1px, 1px, 1px, 1px);" title="“Xamarin Android: asymmetric encryption without any user input or hardcoded values” — MSicc's Blog" width="600"></iframe>

> [How to perform asymmetric encryption without user input/hardcoded values with Xamarin iOS](https://msicc.net/how-to-perform-asymmetric-encryption-without-user-input-hardcoded-values-with-xamarin-ios/)

<iframe class="wp-embedded-content" data-secret="G6zzgBfCsd" frameborder="0" height="338" loading="lazy" marginheight="0" marginwidth="0" sandbox="allow-scripts" scrolling="no" security="restricted" src="https://msicc.net/how-to-perform-asymmetric-encryption-without-user-input-hardcoded-values-with-xamarin-ios/embed/#?secret=TxyGt27Sdt#?secret=G6zzgBfCsd" style="position: absolute; clip: rect(1px, 1px, 1px, 1px);" title="“How to perform asymmetric encryption without user input/hardcoded values with Xamarin iOS” — MSicc's Blog" width="600"></iframe>

> [Using the built-in UWP data protection for data encryption](https://msicc.net/using-the-built-in-uwp-data-protection-for-data-encryption/)

<iframe class="wp-embedded-content" data-secret="AZiCz6k5iC" frameborder="0" height="338" loading="lazy" marginheight="0" marginwidth="0" sandbox="allow-scripts" scrolling="no" security="restricted" src="https://msicc.net/using-the-built-in-uwp-data-protection-for-data-encryption/embed/#?secret=bZsNatx5Gv#?secret=AZiCz6k5iC" style="position: absolute; clip: rect(1px, 1px, 1px, 1px);" title="“Using the built-in UWP data protection for data encryption” — MSicc's Blog" width="600"></iframe>

It is no coincidence that I wrote these three posts before starting with this Akavache series, as we’ll use those techniques to protect sensitive data with Akavache. So you might have a look first before you read on.

## Creating a secure blob cache in Akavache

Akavache has a special type for saving sensitive data – based on the interface `ISecureBlobCache`. The first step is to extend the `IBlobCacheInstanceHelper`interface we implemented [in the first post of this series](https://msicc.net/xamarin-forms-akavache-and-i-initial-setup-new-series/):

``` csharp
     public interface IBlobCacheInstanceHelper
    {
        void Init();

        IBlobCache LocalMachineCache { get; set; }

        ISecureBlobCache SecretLocalMachineCache { get; set; }
    }
```
 
Of course, all three platform implementations of the `IBlobCacheInstanceHelper`interface need to be updated as well. The code to add for all three platform is the same:

``` csharp
 public ISecureBlobCache SecretLocalMachineCache { get; set; }     

private void GetSecretLocalMachineCache()
{
    var secretCache = new Lazy<ISecureBlobCache>(() =>
                                                 {
                                                     _filesystemProvider.CreateRecursive(_filesystemProvider.GetDefaultSecretCacheDirectory()).SubscribeOn(BlobCache.TaskpoolScheduler).Wait();
                                                     return new SQLiteEncryptedBlobCache(Path.Combine(_filesystemProvider.GetDefaultSecretCacheDirectory(), "secret.db"), new PlatformCustomAkavacheEncryptionProvider(), BlobCache.TaskpoolScheduler);
                                                 });

    this.SecretLocalMachineCache = secretCache.Value;
}
```
 
As we will use the same name for all platform implementations, that’s already all we have to do here.

## Platform specific encryption provider

Implementing the platform specific code is nothing new. Way before I used Akavache, others have already implemented solutions. The main issue is that there is no platform implementation for Android and iOS (and maybe others). My solution is [inspired by this blog post](https://kent-boogaart.com/blog/password-protected-encryption-provider-for-akavache) by [Kent Boogart](https://twitter.com/kent_boogaart), which is (as far as I can see), also broadly accepted amongst the community. The only thing I disliked about it was the requirement for a password – which either would be something reversible or causing a (maybe) bad user experience.

Akavache provides the `IEncryptionProvider`interface, which contains two methods. One for encryption, the other one for decryption. Those two methods are working with `byte[]`both for input and output. You should be aware and know how to convert your data to that.

### Implementing the IEncryptionProvider interface

The implementation of Akavache’s encryption interface is following the same principle on all three platforms.

- provide a reference to the internal `TaskpoolScheduler`in the constructor
- get an instance of our platform specific encryption provider
- get or create keys (Android and iOS)
- provide helper methods that perform encryption/decryption

Let’s have a look at the platform implementations. I will show the full class implementation and remarking them afterwards.

#### Android
 
``` csharp
 [assembly: Xamarin.Forms.Dependency(typeof(PlatformCustomAkavacheEncryptionProvider))]
namespace XfAkavacheAndI.Android.PlatformImplementations
{
    public class PlatformCustomAkavacheEncryptionProvider : IEncryptionProvider
    {
        private readonly IScheduler _scheduler;

        private static readonly string KeyStoreName = $"{BlobCache.ApplicationName.ToLower()}_secureStore";

        private readonly PlatformEncryptionKeyHelper _encryptionKeyHelper;

        private const string TRANSFORMATION = "RSA/ECB/PKCS1Padding";
        private IKey _privateKey = null;
        private IKey _publicKey = null;

        public PlatformCustomAkavacheEncryptionProvider()
        {
            _scheduler = BlobCache.TaskpoolScheduler ?? throw new ArgumentNullException(nameof(_scheduler), "Scheduler is null");

            _encryptionKeyHelper = new PlatformEncryptionKeyHelper(Application.Context, KeyStoreName);
            GetOrCreateKeys();
        }

        public IObservable<byte[]> DecryptBlock(byte[] block)
        {
            if (block == null)
            {
                throw new ArgumentNullException(nameof(block), "block cannot be null");
            }

            return Observable.Start(() => Decrypt(block), _scheduler);
        }

        public IObservable<byte[]> EncryptBlock(byte[] block)
        {
            if (block == null)
            {
                throw new ArgumentNullException(nameof(block), "block cannot be null");
            }

            return Observable.Start(() => Encrypt(block), _scheduler);
        }


        private void GetOrCreateKeys()
        {
            if (!_encryptionKeyHelper.KeysExist())
                _encryptionKeyHelper.CreateKeyPair();

            _privateKey = _encryptionKeyHelper.GetPrivateKey();
            _publicKey = _encryptionKeyHelper.GetPublicKey();
        }


        public byte[] Encrypt(byte[] rawBytes)
        {
            if (_publicKey == null)
            {
                throw new ArgumentNullException(nameof(_publicKey), "Public key cannot be null");
            }

            var cipher = Cipher.GetInstance(TRANSFORMATION);
            cipher.Init(CipherMode.EncryptMode, _publicKey);

            return cipher.DoFinal(rawBytes);
        }

        public byte[] Decrypt(byte[] encyrptedBytes)
        {
            if (_privateKey == null)
            {
                throw new ArgumentNullException(nameof(_privateKey), "Private key cannot be null");
            }

            var cipher = Cipher.GetInstance(TRANSFORMATION);
            cipher.Init(CipherMode.DecryptMode, _privateKey);

            return cipher.DoFinal(encyrptedBytes);
        }
    }
```
 
As you can see, I am getting Akavache’s internal `TaskpoolScheduler`in the constructor, like initial stated. Then, for this sample, I am using RSA encryption. The helper methods pretty much implement the same code like in the [post about my KeyStore implementation](https://msicc.net/xamarin-android-asymmetric-encryption-without-any-user-input-or-hardcoded-values/). The only thing to do is to use these methods in the EncryptBlock and DecyrptBlock method implementations, which is done asynchronously via [Observable.Start](https://msdn.microsoft.com/en-us/library/hh211971(v=vs.103).aspx).

### iOS

``` csharp
 [assembly: Xamarin.Forms.Dependency(typeof(PlatformCustomAkavacheEncryptionProvider))]
namespace XfAkavacheAndI.iOS.PlatformImplementations
{
    public class PlatformCustomAkavacheEncryptionProvider : IEncryptionProvider
    {
        private readonly IScheduler _scheduler;

        private readonly PlatformEncryptionKeyHelper _encryptionKeyHelper;


        private SecKey _privateKey = null;
        private SecKey _publicKey  = null;

        public PlatformCustomAkavacheEncryptionProvider()
        {
            _scheduler = BlobCache.TaskpoolScheduler ??
                         throw new ArgumentNullException(nameof(_scheduler), "Scheduler is null");

            _encryptionKeyHelper = new PlatformEncryptionKeyHelper(BlobCache.ApplicationName.ToLower());
            GetOrCreateKeys();
        }

        public IObservable<byte[]> DecryptBlock(byte[] block)
        {
            if (block == null)
            {
                throw new ArgumentNullException(nameof(block), "block can't be null");
            }

            return Observable.Start(() => Decrypt(block), _scheduler);
        }

        public IObservable<byte[]> EncryptBlock(byte[] block)
        {
            if (block == null)
            {
                throw new ArgumentNullException(nameof(block), "block can't be null");
            }

            return Observable.Start(() => Encrypt(block), _scheduler);
        }


        private void GetOrCreateKeys()
        {
            if (!_encryptionKeyHelper.KeysExist())
                _encryptionKeyHelper.CreateKeyPair();

            _privateKey = _encryptionKeyHelper.GetPrivateKey();
            _publicKey = _encryptionKeyHelper.GetPublicKey();
        }

        private byte[] Encrypt(byte[] rawBytes)
        {
            if (_publicKey == null)
            {
                throw new ArgumentNullException(nameof(_publicKey), "Public key cannot be null");
            }

            var code = _publicKey.Encrypt(SecPadding.PKCS1, rawBytes, out var encryptedBytes);

            return code == SecStatusCode.Success ? encryptedBytes : null;
        }

        private byte[] Decrypt(byte[] encyrptedBytes)
        {
            if (_privateKey == null)
            {
                throw new ArgumentNullException(nameof(_privateKey), "Private key cannot be null");
            }

            var code = _privateKey.Decrypt(SecPadding.PKCS1, encyrptedBytes, out var decryptedBytes);

            return code == SecStatusCode.Success ? decryptedBytes : null;
        }

    }
}
```
 
The iOS implementation follows the same schema as the Android implementation. However, [iOS uses the KeyChain](https://msicc.net/how-to-perform-asymmetric-encryption-without-user-input-hardcoded-values-with-xamarin-ios/), which makes the encryption helper methods itself different.

### UWP

``` csharp
 [assembly: Xamarin.Forms.Dependency(typeof(PlatformCustomAkavacheEncryptionProvider))]
namespace XfAkavacheAndI.UWP.PlatformImplementations
{
    public class PlatformCustomAkavacheEncryptionProvider : IEncryptionProvider
    {
        private readonly IScheduler _scheduler;

        private string _localUserDescriptor = "LOCAL=user";
        private string _localMachineDescriptor = "LOCAL=machine";

        public bool UseForAllUsers { get; set; } = false;

        public PlatformCustomAkavacheEncryptionProvider()
        {
            _scheduler = BlobCache.TaskpoolScheduler ??
                         throw new ArgumentNullException(nameof(_scheduler), "Scheduler is null");
        }

        public IObservable<byte[]> EncryptBlock(byte[] block)
        {
            if (block == null)
            {
                throw new ArgumentNullException(nameof(block), "block can't be null");
            }

            return Observable.Start(() => Encrypt(block).GetAwaiter().GetResult(), _scheduler);
        }

        public IObservable<byte[]> DecryptBlock(byte[] block)
        {
            if (block == null)
            {
                throw new ArgumentNullException(nameof(block), "block can't be null");
            }

            return Observable.Start(() => Decrypt(block).GetAwaiter().GetResult(), _scheduler);
        }


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
    }   
}
```
 
Last but not least, we have also an implementation for Windows applications. It is using the DataProtection API, [which does handle all that key stuff](https://msicc.net/using-the-built-in-uwp-data-protection-for-data-encryption/) and let’s us focus on the encryption itself. As the API is asynchronously, I am using `.GetAwaiter().GetResult()`Task extensions to make it compatible with `Observable.Start`.

### Conclusion

Using the implementations above paired with our instance helper makes it easy to protect data in our apps. With all those data breach scandals and law changes around, this is *one possible way* secure way to handle sensitive data, as we do not have hardcoded values or any user interaction involved.

For better understanding of all that code, [I made a sample project available](https://github.com/MSiccDev/XfAkavacheAndISample) that has all the referenced and mentioned classes implemented. Feel free to fork it and play with it (or even give me some feedback). For using the implementations, [please refer to my post about common usages I wrote a few days ago](https://msicc.net/xamarin-forms-akavache-and-i-storing-retrieving-and-deleting-data/). The only difference is that you would use `SecretLocalMachineCache`instead of `LocalMachineCache` for sensitive data.

As always, I hope this post is helpful for some of you.

#### Until the next post, happy coding!

---

P.S. Feel free to download my official app for msicc.net, which – of course – uses the implementations above:  
[iOS](https://itunes.apple.com/de/app/msiccs-blog/id1359113195) [Android ](https://play.google.com/store/apps/details?id=com.msiccdev.msiccsblog) [Windows 10](https://www.microsoft.com/store/apps/9WZDNCRDPQLK)