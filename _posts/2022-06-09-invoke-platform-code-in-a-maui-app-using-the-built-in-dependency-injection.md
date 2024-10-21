---
id: 7037
title: 'Invoke platform code in a MAUI app using the built-in Dependency Injection'
date: '2022-06-09T16:00:00+02:00'
author: 'Marco Siccardi'
excerpt: 'I recently started to port my internal MVVM libraries over to .NET MAUI. It did not take long until I reached the point where I needed to invoke platform code. This post is about my experience with that.'
layout: post
permalink: /invoke-platform-code-in-a-maui-app-using-the-built-in-dependency-injection/
image: /assets/img/2022/06/MAUI_DI_Test_Running.jpeg
categories:
    - 'Dev Stories'
    - MAUI
    - Xamarin
tags:
    - .NET
    - '.NET MAUI'
    - 'Dependency Injection'
    - MAUI
    - mvvm
    - platforms
---

In Xamarin.Forms, my internal libraries for MVVM helped me to keep my applications cleanly structured and abstracted. I recently started the process of porting them over to .NET MAUI. I was quickly reaching the point where I needed to invoke platform specific code, so I read up the [documentation](https://docs.microsoft.com/en-us/dotnet/maui/platform-integration/invoke-platform-code).

### The suggested way

The documentation suggests creating a partial class with partial method definitions and a corresponding partial classes with the partial method implementations ([like described in the docs](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/partial-classes-and-methods)). I tried to follow the above-mentioned MAUI documentation and copy/pasted the code sample in there and thought everything is going to be fine. Well, it was not. I wasn’t even able to compile the solution with that code on my Mac in the first place.

In search for a possible cause of this, I did not find a solution immediately. In the end, it turned out that I needed to implement the partial class method on all platforms specified in the `TargetFrameworks` within the `.csproj` file. It should have been obvious due to the fact that MAUI is a single project with multiple target frameworks, but it wasn’t on that day.

On top of that, Visual Studio did some strange changes to the `.csproj` file specifying unnecessary `None`, `Compile` and `Include` targets that should not be generated explicitly, which added a lot to my confusion as well. After removing them from the project file and adding an implementation for all platforms, I was able to compile and test the code from the docs.

### But I love my interfaces!

Likewise, that’s why I did not stop there. Following the abstraction approach, interfaces allow us to define the common surface of the API without worrying about the implementation details. That’s not the case for the partial classes approach, like the problems I had showed.

Luckily for us, .NET MAUI supports multi targeting. This means an interface can have a platform specific implementation while being defined in the shared part of the application. If you have used the [MSBuildExtras](https://github.com/novotnyllc/MSBuildSdkExtras) package before, you know already how that works. Best part – .NET MAUI already provides the multi targeting configuration out of the box.

### Show me some code!

First, let’s define a simple interface for this exercise:

``` csharp
 namespace MAUIDITest.InterfaceDemo
{
    public interface IPlatformDiTestService
    {
        string SayYourPlatformName();
    }
}
```
 
Now we are going to implement the platform specific implementations. Go to the first platforms folder and add a new class named `PlatformDiTestService`. Then – and this is really important to make it work – adjust the namespace to be the same as the one of the interface. Last, but not least, implement the interface, for example like this:

``` csharp
 namespace MAUIDITest.InterfaceDemo
{
    public class PlatformDiTestService : IPlatformDiTestService
    {
        public string SayYourPlatformName()
        {
            return "I am MacOS!";
        }
    }
}
```
 
Repeat this for all platforms, and replace the platform’s name accordingly.

### Using Dependency Injection in MAUI

If you have been following along my past blog posts, you know that I recently switched to the CommunityToolkit MVVM (read more [here]({% post_url  2022-02-08-using-microsofts-extensions-dependencyinjection-package-in-xamarin-forms-mvvm-applications-part-1 %}) and [here]({% post_url 2022-02-19-using-microsofts-extensions-dependencyinjection-package-in-xamarin-forms-mvvm-applications-part-2 %})). I already heard that MAUI will get the same DI container built-in, so the choice was obvious. Now let’s have a look how easy we can inject our interface into our ViewModel. Head over to your MauiProgram.cs file and update the CreateMauiApp method:

``` csharp
 	public static MauiApp CreateMauiApp()
	{
		var builder = MauiApp.CreateBuilder();
		builder
			.UseMauiApp<App>()
			.ConfigureFonts(fonts =>
			{
				fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
				fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
			});

		//lowest dependency
		builder.Services.AddSingleton<IPlatformDiTestService, PlatformDiTestService>();
		//relies on IPlatformDiTestService
		builder.Services.AddTransient<MainPageViewModel>();
		//relies on MainPageViewModel
		builder.Services.AddTransient<MainPage>();

		return builder.Build();
	}
```
 
First, add the registration of the interface and the implementation. In the sample above, the `MainPageViewModel` relies on the interface and gets it automatically injected by the DI handler. For testing purposes, I even inject the `MainViewModel` into the `MainPage`‘s constructor. This is very likely to change in a real world application.

For completeness, here is the `MainPageViewModel` class:

``` csharp
 using System;
using System.ComponentModel;
using System.Windows.Input;
using MAUIDITest.InterfaceDemo;

namespace MAUIDITest.ViewModel
{
    public class MainPageViewModel : INotifyPropertyChanged
    {
        private readonly IPlatformDiTestService _platformDiTestService;
        private string sayYourPlatformNameValue = "Click the 'Reveal platform' button";
        private Command _sayYourPlatformNameCommand;

        public event PropertyChangedEventHandler PropertyChanged;


        public MainPageViewModel(IPlatformDiTestService platformDiTestService)
        {
            _platformDiTestService = platformDiTestService;
        }

        public void OnPropertyChanged(string propertyName)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }

        public string SayYourPlatformNameValue
        {
            get => sayYourPlatformNameValue;
            set
            {
                sayYourPlatformNameValue = value;
                OnPropertyChanged(nameof(this.SayYourPlatformNameValue));
            }
        }

        public Command SayYourPlatformNameCommand => _sayYourPlatformNameCommand ??=
            new Command(() => { this.SayYourPlatformNameValue = _platformDiTestService.SayYourPlatformName(); });
    }
}
```
 
And of course, you want to see the `MainPage.xaml.cs` file as well:

``` csharp
 using MAUIDITest.InterfaceDemo;
using MAUIDITest.ViewModel;

namespace MAUIDITest;

public partial class MainPage : ContentPage
{
    private readonly IPlatformDiTestService _platformDiTestService;
    int count = 0;

	public MainPage(MainPageViewModel mainPageViewModel)
	{
		InitializeComponent();

		this.BindingContext = mainPageViewModel;
    }

	private void OnCounterClicked(object sender, EventArgs e)
	{
		count++;

		if (count == 1)
			CounterBtn.Text = $"Clicked {count} time";
		else
			CounterBtn.Text = $"Clicked {count} times";

		SemanticScreenReader.Announce(CounterBtn.Text);
	}
}

```
 
Last, but not least, the updated `MainPage.xaml` file:

``` xml
 <?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MAUIDITest.MainPage">
			 
    <ScrollView>
        <VerticalStackLayout 
            Spacing="25" 
            Padding="30,0" 
            VerticalOptions="Center">

            <Image
                Source="dotnet_bot.png"
                SemanticProperties.Description="Cute dot net bot waving hi to you!"
                HeightRequest="200"
                HorizontalOptions="Center" />
                
            <Label 
                Text="Hello, World!"
                SemanticProperties.HeadingLevel="Level1"
                FontSize="32"
                HorizontalOptions="Center" />
            
            <Label 
                Text="Welcome to .NET Multi-platform App UI"
                SemanticProperties.HeadingLevel="Level2"
                SemanticProperties.Description="Welcome to dot net Multi platform App U I"
                FontSize="18"
                HorizontalOptions="Center" />

            <Button 
                x:Name="CounterBtn"
                Text="Click me"
                SemanticProperties.Hint="Counts the number of times you click"
                Clicked="OnCounterClicked"
                HorizontalOptions="Center" />


            <Label Text="Test of built in DI:" FontSize="Large" HorizontalOptions="Center"></Label>
            <Label x:Name="PlatformNameLbl" FontSize="Large" HorizontalOptions="Center" Text="{Binding SayYourPlatformNameValue}" />

            <Button Text="Reveal platform" HorizontalOptions="Center" Command="{Binding SayYourPlatformNameCommand}"/>

        </VerticalStackLayout>
    </ScrollView>
 
</ContentPage>
```
 
I did not change the default code that comes with the template. You can easily recreate this by using the default MAUI template of Visual Studio and copy/paste the code snippets above to play around with it.

## Conclusion

I only started my journey to update my internal libraries to .NET MAUI. I stumbled pretty fast with that platform invoking code, but luckily, I was able to move along. Platform specific code can be handled pretty much the same as before, which I hope I was able to show you in this post. I’ll write more posts on my updating experiences to MAUI as they happen.

##### Until the next post, happy coding, everyone!