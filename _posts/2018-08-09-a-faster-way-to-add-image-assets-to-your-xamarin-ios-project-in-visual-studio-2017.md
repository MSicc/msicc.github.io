---
id: 5751
title: 'A faster way to add image assets to your Xamarin.iOS project in Visual Studio 2017'
date: '2018-08-09T19:29:38+02:00'
author: 'Marco Siccardi'
excerpt: 'Recently, I was in the situation that I needed to add a bunch of images to a Xamarin.iOS project. In this post, I am going to show you a faster way to add image assets to your Xamarin.iOS project in Visual Studio.'
layout: post
permalink: /a-faster-way-to-add-image-assets-to-your-xamarin-ios-project-in-visual-studio-2017/
image: /assets/img/2018/08/ios-faster-add-assets-title.jpg
categories:
    - 'Dev Stories'
    - iOS
    - Xamarin
tags:
    - assets
    - csproj
    - iOS
    - 'project file'
    - 'quick tip'
    - tip
    - visualstudio
    - vs2017
    - xamarin
---

While Visual Studio has an editor that shall help to add new image assets to the project, it is pretty slow when adding a bunch of new images. If you have to add a lot of images to add, it even crashes once in while, which can be annoying. Because I was in the process of adding more than a hundred images for [porting my first app ever](https://msiccdev.net/windowsphone/fishing-knots/) from Windows Phone to iOS and Android, I searched for a faster way – and found it.

### What’s going on under the hood?

When you add an `imageset` to your assets structure, Visual Studio does quite some work. These are the steps that are done:

1. creation of a folder for the `imageset` in the Assets folder
2. creation of a `Contents.json`file
3. modification of the `Contents.json` file
4. modification of the .csproj file

This takes some time for *every* image, and Visual Studio seems to be quite busy with these steps. After analyzing the way `imagesets` get added, I immediately recognized that I am faster than Visual Studio if I do that work manually.

### How to add the assets – step by step

1. right click on the project in Solution Explorer and select ‘*Open Folder in File Explorer*‘ and find the ‘*Assets*‘ folder ![open_folder_in_file_explorer](/assets/img/2018/08/open_folder_in_file_explorer.png)

2. create a new folder in this format: ‘*\[yourassetname\].imageset*‘
3. add your image to the folder
4. create a new file with the name `Contents.json` ![add_image_and_contents_file](/assets/img/2018/08/add_image_and_contents_file.png)

5. open the file (I use [Notepad++](https://notepad-plus-plus.org/download/v7.5.8.html) for such operations) and add this minimum required `json`to it: 
``` json
     {
      "images": [
        {
          "scale": "1x",
          "idiom": "universal",
          "filename": "[yourimage].png"
        },
        {
          "scale": "2x",
          "idiom": "universal",
          "filename": "[yourimage].png"
        },
        {
          "scale": "3x",
          "idiom": "universal",
          "filename": "[yourimage].png"
        },
        {
          "idiom": "universal"
        }
      ],
      "properties": {
        "template-rendering-intent": ""
      },
      "info": {
        "version": 1,
        "author": ""
      }
    }
    ```
 6. go back to Visual Studio, right click on the project again and select ‘*Unload Project*‘![unload_project](/assets/img/2018/08/unload_project.png)

7. right click again and select ‘*Edit \[yourprojectname\].iOS.csproj*‘ ![edit_ios_csproj](/assets/img/2018/08/edit_ios_csproj.png)

8. find the `ItemGroup` with the Assets
9. inside the `ItemGroup`, add your `imageset` with these two entries: 
``` xml
     <ImageAsset Include="Assets.xcassets\[yourassetname].imageset\[yourimage].png">
      <Visible>false</Visible>
    </ImageAsset>
    <ImageAsset Include="Assets.xcassets\[yourassetname].imageset\Contents.json">
      <Visible>false</Visible>
    </ImageAsset>   
```

 10. close the file and reload the project by selecting it from the context menu in Solution Explorer

If you followed this steps, your assets should be visible immediately:  
![assets_with_added_imagesets](/assets/img/2018/08/assets_with_added_imagesets.png)


I did not measure the time exactly, but I felt I was significantly faster by adding all those images that way. Your mileage may vary, depending on the power of your dev machine.

### Conclusion

Features like the AssetManager for iOS in Visual Studio are nice to have, but some have some serious performance problems (most of them are already known). By taking over the process manually, one can be a lot faster than a built-in feature. That said, like always, I hope this post is helpful for some of you.

#### Happy coding, everyone!