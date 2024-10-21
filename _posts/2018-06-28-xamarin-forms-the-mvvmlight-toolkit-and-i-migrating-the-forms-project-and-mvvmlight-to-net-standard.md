---
id: 5684
title: 'Xamarin Forms, the MVVMLight Toolkit and I: migrating the Forms project and MVVMLight to .NET Standard'
date: '2018-06-28T14:00:59+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post, we will see how to convert a Xamarin.Forms PCL-project into a .NET Standard project. During that process, we will update MVVMLight to the latest version, plus I''ll show you how to fix common errors during that conversion.'
layout: post
permalink: /xamarin-forms-the-mvvmlight-toolkit-and-i-migrating-the-forms-project-and-mvvmlight-to-net-standard/
image: /assets/img/2018/06/lever-3090764_1280-1.jpg
categories:
    - 'Dev Stories'
    - Xamarin
tags:
    - '.NET Standard'
    - conversion
    - csproj
    - mvvmlight
    - project
    - xamarin
    - 'xamarin forms'
---

When I started this series, Xamarin and Xamarin.Forms did not fully support .NET Standard. The sample project for this series has still a portable class library, and as I wanted to blog on another topic, I got reminded that I never updated the project. This post will be all about the change to .NET Standard, focusing on the Xamarin.Forms project as well as the MVVMLight library, which is also available as a .NET Standard version in the meantime.

## Step-by-Step

I have done the necessary conversion steps already quite a few times, and to get you through the conversion very quickly, I will show you a set of screenshots. Where applicable, I will provide also some XML snippets that you can copy and paste.

#### Step 1

![step1_unload project](/assets/img/2018/06/step1_unload-project.png)


The first step is to unload the project file. To do so, select your Xamarin Forms project in Solution Explorer and right click again on it bring up the context menu. From there, select “*Unload Project*“.

#### Step 2

![step2_edit_csproj](/assets/img/2018/06/step2_edit_csproj.png)


Next, right-click the selected Xamarin.Forms project again to make the context menu visible once again. From there select “*Edit \[PROJECTNAME\]*.*csproj*” to open up the file.

#### Step 3

![step3_replace_all](/assets/img/2018/06/step3_replace_all.png)


Now that the file is open, press “CTRL + A”, followed by “DEL”. Seriously, just do it. Once the file is empty, copy these lines and paste them into your now empty file:

``` xml
 <Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>

</Project>
```
 
#### Step 4

![step4_open_packages_config](/assets/img/2018/06/step4_open_packages_config.png)


This step is not really necessary, but I want to point one thing out. The old PCL-project type has listed quite a bunch of libraries in the *packages.config* file. These are all NuGet packages necessary to make the app actually run (they get pulled in during the install of a NuGet package as dependencies). Now that we are converting, we are getting rid of the *packages.config* file. You can delete right away in the Solution Explorer. We will “install” the needed packages (marked with the arrows) in

#### Step 5

![step5_changed_csproj](/assets/img/2018/06/step5_changed_csproj.png)


For “installing” NuGet packages into the converted project, we are adding a new ItemGroup to the XML-File. The Package *System.ValueTuple* is referenced – because in this series’ sample project we have some code in there that uses it. The absolute minimum you need to get it running is:

``` xml
 <!--
Template for Nuget Packages:
<ItemGroup>
  <PackageReference Include="" Version="" />
</ItemGroup>
-->

<ItemGroup>
  <!--MvvmLight has changed, so do we!-->
  <PackageReference Include="MvvmLightLibsStd10" Version="5.4.1" />
  <!--needed references-->
  <PackageReference Include="Xamarin.Forms" Version="3.1.0.583944" />
</ItemGroup>
```
 
If you have other NuGet packages, just add them into this item group. They will get installed in the correct version if you follow this tutorial until the end.

You might have noticed that the MVVMLight package I am inserting here is not the same as before. This is absolutely true, but for a reason. Laurent Bugnion has published the .NET Standard version quite some time ago. If you want to read his blog post, [you can find it here](https://blog.galasoft.ch/posts/2018/02/publishing-mvvmlight-v5-4-1-with-net-standard-support/).

The second change I want to outline is `DebugType`settings. These are also not set in that way if you create a new project that is already .NET Standard. In order to enable debugging of your Xamarin.Forms code, however, you absolutely should also add these lines:

``` xml
 <!--these enable debugging in the Forms project-->
<PropertyGroup Condition=" '$(Configuration)'=='Debug' ">
  <DebugType>full</DebugType>
  <DebugSymbols>true</DebugSymbols>
  <GenerateDocumentationFile>false</GenerateDocumentationFile>
</PropertyGroup>
<PropertyGroup Condition=" '$(Configuration)'=='Release' ">
  <DebugType>pdbonly</DebugType>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```
 
These properties provide the debug symbols and the pdb-files a lot of analytic services are dependent on. Simple put, copy and paste them into your project. Hit the “Save”-Button and close the file.

#### Step 6

![step6_delete_obj](/assets/img/2018/06/step6_delete_obj.png)


Now that we have changed the project file, our already downloaded packages and created dll-files are no longer valid. To make sure we are able to compile, we need to delete the content of the obj folder in the *Folder View* of the *Solution Explorer* in Visual Studio.

#### Step 7

![step7_delete_bin](/assets/img/2018/06/step7_delete_bin.png)


Do the same to the content of the bin folder and switch back to the *Solution View* by hitting the *Folder View*-Button again

#### Step 8

![step8_reload_project](/assets/img/2018/06/step8_reload_project.png)


Now we are finally able to reload the project as the base conversion is already done.

#### Step 9

![step9_fix errors](/assets/img/2018/06/step9_fix-errors.png)


Trying to be optimistic and hit the Rebuild button will result in these (and maybe even more) errors. The first one is one that we can solve very fast, while the second one is only the first in a row of errors.

#### Step 10

![step10_delete_properties_folder](/assets/img/2018/06/step10_delete_properties_folder.png)


To solve the assembly attributes error, just go to the *Folder View* again and select the *Properties* folder. Bring up the context menu by right-clicking on it and select the delete option. Confirm the deletion to get rid of the folder. The new project style creates the assembly information based on the project file during build, which is causing the errors.

#### Step 11

![step11_unneeded_reference](/assets/img/2018/06/step11_unneeded_reference.png)


Now let’s face the next error. As the MVVMLight .NET Standard version does no longer rely on the `CommonServiceLocator`like before, we are able to remove this reference from <span style="display: inline !important; float: none; background-color: transparent; color: #333333; cursor: text; font-family: Georgia,'Times New Roman','Bitstream Charter',Times,serif; font-size: 16px; font-style: normal; font-variant: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;">our </span>`ViewModelLocator`.

#### Step 12

![step12_unneeded_instantiation](/assets/img/2018/06/step12_unneeded_instantiation.png)


Of course, we now can also remove the instantiation call for the removed `ServiceLocator`.

### Step 13

![step13_replace_servicelocator_calls](/assets/img/2018/06/step13_replace_servicelocator_calls.png)


In the ViewModel instance reference, replace `ServiceLocator.Current` with `SimpleIoc.Default`. Hit the save button again. You might have more errors to fix. Do so, and finally save your ViewModelLocator.

#### Step 14

![step14_rebuild_succeeded](/assets/img/2018/06/step14_rebuild_succeeded.png)


After all the work is done, we are now able to compile the .NET Standard version of our Xamarin.Forms project. If you fixed all of your errors in the step before, you should achieve a similar result like me and get a success message.

#### Final steps

Now that the Xamarin.Forms project is built, you might want to try to build all other projects in the solution as well. The changed structure of the Xamarin.Forms project will have an impact also on the platform projects, that’s why I absolutely recommend deleting the contents of *bin* and *object* folders there, too. This will solve you a lot of work to get things to compile again.

As always, I hope this post is helpful for some of you. If you have any questions, feel free to ping me on my social accounts or write a comment below. The next post in this series will involve again more code, so stay tuned – it will be out soon.

Please find the updated sample project [here on GitHub](https://github.com/MSiccDev/XfMvvmLight).

#### Happy coding, everyone!

[title image credit](https://pixabay.com/en/lever-rear-derailleur-conversion-3090764/)