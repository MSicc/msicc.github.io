---
id: 7343
title: '#CASBAN6: How to configure Azurite to use DefaultAzureCredential with Docker on macOS'
date: '2023-07-07T08:00:00+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I''ll show you how to configure the Azurite storage emulator to use the DefaultAzureCredential while running in a Docker container on macOS.'
layout: post
permalink: /casban6-how-to-configure-azurite-to-use-defaultazurecredential-with-docker-on-macos/
image: /assets/img/2023/07/CASBAN6-docker-container-azurite-title.png
categories:
    - 'Dev Stories'
    - Azure
    - macOS
tags:
    - Azure
    - AzureDev
    - Azurite
    - Blob
    - 'Blob Storage'
    - container
    - credentials
    - debugging
    - DefaultAzureCredential
    - Docker
    - local
    - macOS
    - setup
    - storage
---

Before we will have a look at the promised facade for uploading files to Azure’s Blob storage, I want to show you how to configure Azurite to use the Microsoft Identity framework with Docker on macOS for local debugging. Using the Identity framework, our local debug environment and the productive rolled out functions will behave the same.

If you have been following along, you should already have Docker Desktop on your machine. If not, install the application from[ their website](https://www.docker.com/products/docker-desktop/).

### Get the docker image

The docker image is available in the [Docker Hub](https://hub.docker.com/_/microsoft-azure-storage-azurite?tab=description). You can download it once you have installed Docker Desktop by entering the following command in a Terminal window:

``` shell
 docker pull mcr.microsoft.com/azure-storage/azurite
```
 
### Get a local certificate

To use the `AzureDefaultCredential` within Azurite, we need a locally-trusted development certificate. You can either use OpenSSL or [mkcert](https://github.com/FiloSottile/mkcert) to generate the needed certificate. We will move on with mkcert.

If you have installed other Azure stuff on your Mac before, you probably have already installed Homebrew. If not, follow [this link](https://brew.sh) and install it. Once you are ready, install mkcert with the following command:

``` shell
 brew install mkcert
```
 
Next, create a folder that can be used as a local data store for the Azurite container we will create later. We will also use this folder to store the SSL certificate on our host machine. To create the certificate and its key, run the following commands:

``` shell
 mkcert -install
mkcert -key-file /Users/{YourLocalFolder}/azurite/127.0.0.1-key.pem -cert-file /Users/{YourLocalFolder}/azurite/127.0.0.1.pem 127.0.0.1
```
 
The first command creates the root certificate that is needed for certificate issuance. The second command creates a derived certificate that we can use for Azurite (replace `{YourLocalFolder}` with your local path). You can explore more options by typing in `mkcert -help` into your Terminal.

### Creating the Docker container

Now we have everything in place to finally spin up our Azurite container. To do so, run the following command in your Terminal:

``` shell
 docker run -p 10000:10000 -v /Users/{YourLocalFolder}/azurite:/workspace -l /workspace mcr.microsoft.com/azure-storage/azurite azurite-blob --blobHost 0.0.0.0 --oauth basic --cert /workspace/127.0.0.1.pem --key /workspace/127.0.0.1-key.pem 
```
 
Let’s break down what this does. The `docker run -p 10000:10000` portion is to create the container listening to port 10000.

The `-v /Users/{YourLocalFolder}/azurite:/workspace` portion creates a virtual directory that can be mapped with the `-l /workspace `portion to the workspace folder inside the container. This is very important to make the certificate we created earlier accessible for the container.

The `mcr.microsoft.com/azure-storage/azurite` `azurite-blob --blobHost 0.0.0.0` portion activates just the Blob storage in Azurite and makes it accessible from the host system.

The `--oauth basic` portion activates the usage of the `DefaultAzureCredential` but requires the earlier created certificate to actually work. As we mounted our local folder already to the container, we can easily reference the certificate using the virtual directory we assigned earlier in the command: -cert `/workspace/127.0.0.1.pem --key /workspace/127.0.0.1-key.pem`

### Conclusion

It took me some time to figure out all the options and the right configuration on my Mac. [The docs](https://github.com/Azure/Azurite/blob/main/README.md) are available on GitHub. This post should help to put everything together as fast as possible on macOS.

As always, I hope this post will be helpful for some of you. In the next post, we finally will have a look into the Azure Function facade for handling files in the Azure Blob storage.

#### Until the next post, happy coding!

---

[Title Image created with Bing Image Creator](https://www.bing.com/images/create/create-me-a-featured-image-for--22how-to-configure-/64a78d89d64f4ae78d143ad47bb8d236?id=E5HUvb5eGTfiqzBiUUsCQg%3d%3d&view=detailv2&idpp=genimg&FORM=GCRIDP&ajaxhist=0&ajaxserp=0)