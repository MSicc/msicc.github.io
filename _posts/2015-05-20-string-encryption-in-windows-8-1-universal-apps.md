---
id: 4337
title: '[Updated] String Encryption in Windows 8.1 Universal apps'
date: '2015-05-20T16:59:39+02:00'
author: 'Marco Siccardi'
layout: post
permalink: /string-encryption-in-windows-8-1-universal-apps/
categories:
    - Archive
tags:
    - encryption
    - 'extension methods'
    - 'roaming settings'
    - 'roaming storage'
    - saving
    - security
    - string
    - 'universal app'
    - 'Windows 8.1'
    - 'Windows Phone 8.1'
---

[![image credit: Maksim Kabakou, Fotolia.com](/assets/img/2015/05/encryption-credits-Maksim-Kabakou-Fotolia.jpg)](/assets/img/2015/05/encryption-credits-Maksim-Kabakou-Fotolia.jpg)

\[Updated: This post caused a lot of controversy and bad voices, but luckily also some constructive feedback. This is not the initial, but the updated version of this post.\]

One of the goals of a Universal Windows app project is to simplify the life of our users. Microsoft provides access to [RoamingSettings](https://msdn.microsoft.com/en-us/library/windows/apps/windows.storage.applicationdata.roamingsettings.aspx) as well as the [RoamingFolder](https://msdn.microsoft.com/en-us/library/windows/apps/windows.storage.applicationdata.roamingfolder.aspx) to achieve this goal.

The usage of this storage is fairly simple (it is the same as using the local counterparts). However, one big point is how to save those settings and files securely. Nowadays, a lot of users are even more concerned about security than before, and it makes all sense to encrypt synchronized data.

This is how I came a long with a helper class string encryption. I have searched a lot throughout the Web, but apparently this is not a topic to be discussed openly but behind closed doors. I partly understand that, but I want to say thanks to [Ginny Caughey](https://twitter.com/gcaughey) at this point for her feedback on security, and also thank [Joost van Schaik](https://twitter.com/LocalJoost) for his feedback that “forced” me to change my helper class to extension methods, which makes the usage more readable. If you want to learn more about Extension Methods, [this link](https://msdn.microsoft.com/en-us/library/bb311042.aspx) helped me to understand them and made me changing my code in about 5 minutes.

Let’s talk about security. There are two patterns for encrypting sensitive data: using [symmetric algorithm](https://en.wikipedia.org/wiki/Symmetric-key_algorithm) and using [asymmetric algorithm](https://en.wikipedia.org/wiki/Public-key_cryptography). The symmetric key encryption is secure, as long as you have a truly unique identifier that you can use as encryption key. Being very fast, using the symmetric algorithm is only as secure as the key that is used for encryption (obviously, I did it wrong in the first place, that’s why the code is updated). More security is achieved with the asymmetric algorithm, where a public and a private key are used for encryption. The downside of this higher level of security is that it takes longer to perform those actions. There are also more differences between those methods, but that would fill a whole book.

I have learned that on Android and iOS, often the AdvertisingId is used for such operations. Also Windows 8.1 (both phone and PC/tablet) have such an Id, provided by the [AdvertisingManager](https://msdn.microsoft.com/en-us/library/windows/apps/windows.system.userprofile.advertisingmanager.aspx) class. Be aware that the id is per-user and per-device and users can switch this off or even reset their advertising id, so this is not a good idea to use. In Windows 8.1 Runtime projects (both phone and tablet/PC, we luckily have the [PasswordVault](https://msdn.microsoft.com/en-us/library/windows/apps/windows.security.credentials.passwordvault.aspx) class. This brings us some key advantages: the PasswordVault is encrypted and it does roam to [trusted devices](https://windows.microsoft.com/en-us/windows-8/what-is-trusted-device). This is what I am using for saving the keys, being it the symmetric one or the asymmetric ones.

Before we will have a look at my Extension methods, I have to put in a disclaimer. *I am not saying that my way is the nonplus ultra way to encrypt strings. I do also not say that my way provides 100% security (which in fact does not even exist). My way provides security at a good level, but that level surely can be improved. I will take no responsibility for the security of apps that are using this.*

Let’s have a look at my Extension methods that will help you securing your data.

### Symmetric Key Encryption

My symmetric encryption methods were using a pre shared key. I received a lot of feedback that this does not make any sense as it throws away the security factor, especially as the Guid can be recalculated. So I changed the method to use random data, that is generated from the CryptographiBuffer.GenerateRandom() method. Then I am putting this into an Base64 encoded string to pass it over to the PasswordVault:

``` csharp
         public static string GetKeyMaterialString(string resource = null, string username = null)
        {
            string key = "";

            if (string.IsNullOrEmpty(resource) && string.IsNullOrEmpty(username))
            {
                //replace with your resource name if suitable
                resource = "symmetricKey";
                //replace with your user's name/identifier
                username = "sampleUserName";                                
            }            

            //using try catch as FindAllByResource will throw an exception anyways if the specified resource is not found
            try
            {
                //search for our saved symmetric key
                var findSymmetricKey = _passwordVault.FindAllByResource(resource);
                //calling RetrievePassword you MUST!
                findSymmetricKey[0].RetrievePassword();
                key = findSymmetricKey[0].Password;
            }
            catch (Exception)
            {
                //getting a true random key buffer with a length of 32 bytes
                IBuffer randomKeyBuffer = CryptographicBuffer.GenerateRandom(32);

                key = CryptographicBuffer.EncodeToBase64String(randomKeyBuffer); 

                _passwordVault.Add(new PasswordCredential(resource, username, key));
            }
            
            return key;
        }
```
 
The resource string as well as the username is needed to save the key into the PasswordVault. The FindAllByResource() method will throw an Exception if no credentials are found, in this case we are generating them from scratch in the catch block.

The encryption method is pretty straight forward as well. Microsoft does all the complicated calculating stuff, we just need to call the proper methods. What you get is a Base64 encoded string that easily can be saved into the ApplicationSettings or wherever you need it. Here is the complete extension method:

``` csharp
         public static string EncryptStringSymmetric(this string text)
        {
            string encryptedString = "";

            try
            {
                //load the alghorithm providers
                var symmetricKeyProvider = SymmetricKeyAlgorithmProvider.OpenAlgorithm(SymmetricAlgorithmNames.AesCbcPkcs7);

                //create the symmetric key that is used to encrypt the string from random keystring
                var cryptoKey = symmetricKeyProvider.CreateSymmetricKey(CryptographicBuffer.DecodeFromBase64String(GetKeyMaterialString()));

                //create the IBuffer that is for the string
                IBuffer buffer = CryptographicBuffer.CreateFromByteArray(Encoding.UTF8.GetBytes(text));

                //encrypt the byte array with the symmetric key
                encryptedString = CryptographicBuffer.EncodeToBase64String(CryptographicEngine.Encrypt(cryptoKey, buffer, null));

                //return the Base64 string representation of the encrypted string byte array
                return encryptedString;
            }
            catch (Exception)
            {
                return null;
            }

        }
```
 
The decryption method of course reverses all this. First, it loads the saved key from the password fault, and then decrypts and returns then the plain text string:

``` csharp
         public static string DecryptStringSymmetric(this string text)
        {
            string decryptedString = "";

            try
            {
                //load the alghorithm providers
                var symmetricKeyProvider = SymmetricKeyAlgorithmProvider.OpenAlgorithm(SymmetricAlgorithmNames.AesCbcPkcs7);

                //create the symmetric key that is used to encrypt the string from random keystring
                var cryptoKey = symmetricKeyProvider.CreateSymmetricKey(CryptographicBuffer.DecodeFromBase64String(GetKeyMaterialString()));

                //decode the input Base64 string
                IBuffer buffer = CryptographicBuffer.DecodeFromBase64String(text);
                //declare new byte array
                byte[] dectryptedBytes;
                //decrypt the IBuffer back to byte array
                CryptographicBuffer.CopyToByteArray(CryptographicEngine.Decrypt(cryptoKey, buffer, null), out dectryptedBytes);
                //get string back from the byte array
                decryptedString = Encoding.UTF8.GetString(dectryptedBytes, 0, dectryptedBytes.Length);

                //return plain text
                return decryptedString;
            }
            catch (Exception)
            {
                return null;
            }
        }
```
 
Both methods return null if anything did not work like excepted. This should help you to easily check if something went wrong in there. Here is how to use those methods:

#### Encrypting the string:

``` csharp
 var encryptedString = stringToEncrypt.EncryptStringSymmetric();
```
 
#### Decrypting the string:

``` csharp
 var dectryptedString = stringToDecrypt.DecryptStringSymmetric();
```
 
# Asymmetric Key Encryption

Like I already wrote, for asymmetric encryption always two keys are used. The method to get the keys is there slightly different:

``` csharp
         public static Dictionary<string, string> GetAsymmetricKeyPair(string username = null)
        {
            Dictionary<string, string> keyDictionary;
            const string privKey = "asymmetricPrivateKey";
            const string pubKey = "asymmetricPublicKey";

            if (string.IsNullOrEmpty(username))
            {
                //replace with your user's name/identifier 
                username = "sampleUserName";
            }

            //using try catch as FindAllByResource will throw an exception anyways if the specified resource is not found
            try
            {
                //search for our save asymmetric keys
                var findAsymmetricPrivateKey = _passwordVault.FindAllByResource(privKey);
                //calling RetrievePassword you MUST!
                findAsymmetricPrivateKey[0].RetrievePassword();
                var findAsymmetricPublicKey = _passwordVault.FindAllByResource(pubKey);
                //calling RetrievePassword you MUST!
                findAsymmetricPublicKey[0].RetrievePassword();

                //loading our keys into a new Dictionary
                keyDictionary = new Dictionary<string, string>()
                {
                    {privKey, findAsymmetricPrivateKey[0].Password},
                    {pubKey, findAsymmetricPublicKey[0].Password}
                };
            }
            catch (Exception)
            {
                //declaring the Key Algortihm Provider and creating the KeyPair
                var asymmetricKeyProvider =
                    AsymmetricKeyAlgorithmProvider.OpenAlgorithm(AsymmetricAlgorithmNames.RsaPkcs1);
                CryptographicKey cryptographicKeyPair = asymmetricKeyProvider.CreateKeyPair(512);

                //converting the KeyPair into IBuffers
                IBuffer privateKeyBuffer =
                    cryptographicKeyPair.Export(CryptographicPrivateKeyBlobType.Pkcs1RsaPrivateKey);
                IBuffer publicKeyBuffer =
                    cryptographicKeyPair.ExportPublicKey(CryptographicPublicKeyBlobType.Pkcs1RsaPublicKey);

                //encoding the key IBuffers into Base64 Strings and adding them to a new Dictionary
                keyDictionary = new Dictionary<string, string>
                {
                    {privKey, CryptographicBuffer.EncodeToBase64String(privateKeyBuffer)},
                    {pubKey, CryptographicBuffer.EncodeToBase64String(publicKeyBuffer)}
                };

                //saving the newly generated keys in PasswordVault
                //it is recommended to save the both keys separated from each other, though
                _passwordVault.Add(new PasswordCredential(privKey, username, keyDictionary[privKey]));
                //_passwordVault.Add(new PasswordCredential(pubKey, username, keyDictionary[pubKey]));
            }

            //return new Dictionary
            return keyDictionary;
        }
```
 
As you can see, I am creating a Dictionary that holds the two keys so we can work with. I also use another key generating method, specially made for asymmetric encryption. I store those two keys separated from each other into the PasswordVault in the end.

The encryption follows basically the same structure as the symmetric key encryption method. Due to the fact I already have two different keys, I think this is ok for the scenario of passing values between the Windows Phone and Windows project. Eventually I will change that in future when I see the need for it. Notice that for encryption, always the public key is used. The public key should also be saved separated from the private key. So here is the complete method:

``` csharp
         public static string EncryptStringAsymmetric(this string text, string publicKey = null)
        {
            //making sure we are providing a public key
            if (string.IsNullOrEmpty(publicKey))
            {
                var keyPairs = GetAsymmetricKeyPair();
                publicKey = keyPairs["asymmetricPublicKey"];
            }

            try
            {
                //converting the public key into an IBuffer
                IBuffer keyBuffer = CryptographicBuffer.DecodeFromBase64String(publicKey);
                
                //load the public key and the algorithm provider
                var asymmetricAlgorithmProvider = AsymmetricKeyAlgorithmProvider.OpenAlgorithm(AsymmetricAlgorithmNames.RsaPkcs1);
                var cryptoKey = asymmetricAlgorithmProvider.ImportPublicKey(keyBuffer, CryptographicPublicKeyBlobType.Pkcs1RsaPublicKey);

                //converting the string into an IBuffer
                IBuffer buffer = CryptographicBuffer.CreateFromByteArray(Encoding.UTF8.GetBytes(text));

                string encryptedString = "";

                //perform the encryption
                encryptedString = CryptographicBuffer.EncodeToBase64String(CryptographicEngine.Encrypt(cryptoKey, buffer, null));

                //return the Base64 string representation of the encrypted string
                return encryptedString;
            }
            catch (Exception)
            {
                return null;
            }

        }
```
 
The decryption of the encrypted string works also in a similar way to the symmetric one, except we are using the private key of our asymmetric key pair. To be able to do this, we need to use the ImportKeyPair method, whose name is a bit misleading in my opinion. In fact, we even need to specify that we want to import the private key and not the whole key pair. But even with that, decryption works like expected:

``` csharp
         public static string DecryptStringAsymmetric(this string text, string privateKey = null)
        {
            //making sure we are providing a public key
            if (string.IsNullOrEmpty(privateKey))
            {
                    var keyPairs = GetAsymmetricKeyPair();
                privateKey = keyPairs["asymmetricPrivateKey"];
            }

            try
            {
                //converting the private key into an IBuffer
                IBuffer keyBuffer = CryptographicBuffer.DecodeFromBase64String(privateKey);

                //load the private key and the algorithm provider
                var asymmetricAlgorithmProvider = AsymmetricKeyAlgorithmProvider.OpenAlgorithm(AsymmetricAlgorithmNames.RsaPkcs1);
                var cryptoKey = asymmetricAlgorithmProvider.ImportKeyPair(keyBuffer, CryptographicPrivateKeyBlobType.Pkcs1RsaPrivateKey);

                //converting the encrypted text into an IBuffer
                IBuffer buffer = CryptographicBuffer.DecodeFromBase64String(text);

                //cdecrypting the IBuffer and convert its content into a Byte array 
                byte[] decryptedBytes;
                CryptographicBuffer.CopyToByteArray(CryptographicEngine.Decrypt(cryptoKey, buffer, null), out decryptedBytes);

                string decryptedString = "";

                //getting back the plain text 
                decryptedString = Encoding.UTF8.GetString(decryptedBytes, 0, decryptedBytes.Length);

                return decryptedString;
            }
            catch (Exception)
            {
                return null;
            }
        }
```
 
Like the symmetric methods, also the asymmetric methods return null if something went wrong. Here is how to use them:

#### Encrypting the string:

``` csharp
 var encryptedString = stringToEncyrpt.EncryptStringAsymmetric();
```
 
#### Decrypting the string:

``` csharp
 var decryptedString = stringToDecrypt.DecryptStringAsymmetric();
```
 
As you can see, securing strings does not have to be more complicated than this. Using the PasswordVault, we have a truly secure place to store the keys (that need to be saved privately and secure), the rest is done by the methods provided by the operating system. I am by far not a security researcher or expert, but this class should provide a good level of security for string encryption in Windows 8.1 universal apps.

If you know more about security or have ideas to improve this helper class, I put up a [sample project on Github](https://github.com/MSiccDev/StringEncryptionExtension) where you also can play around with the class and the simple app I created for it. Feel free to contribute to this class or discuss in the comments below.

Happy coding, everyone!