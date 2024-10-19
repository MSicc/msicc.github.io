---
id: 6776
title: 'Some helpful extensions when dealing with types in .NET'
date: '2020-10-10T08:00:00+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I am sharing some extension methods that became helpful when dealing with types in .NET applications or libraries.'
layout: post
permalink: /some-helpful-extensions-when-dealing-with-types-in-net/
image: /assets/img/2020/10/extension_pier_illustration.jpg
categories:
    - 'Dev Stories'
tags:
    - assembly
    - 'extension methods'
    - helper
    - interface
    - mvvm
    - type
---

If you are writing reusable code, chances are high that you will write quite some code that deals with types, generics, and interfaces. Over the years, the collection of my helper extensions for that have grown. As some of my upcoming posts use them, I share them (also) for future reference.

### 1. Check if a Type is deriving from another Type

Deriving types is a common practice. To some extent, you can use pattern matching. Sometimes, that isn’t enough, though (especially if you have a multi-level derivation path). This is when I use one of these two extensions:

``` csharp
         public static bool IsDerivingFrom(this Type type, Type searchType)
        {
            if (type == null) throw new NullReferenceException();

            return
                type.BaseType != null &&
                (type.BaseType == searchType ||
                type.BaseType.IsDerivingFrom(searchType));
        }

        public static bool IsDerivingFromGenericType(this Type type, Type searchGenericType)
        {
            if (type == null) throw new ArgumentNullException(nameof(type));
            if (searchGenericType == null) throw new ArgumentNullException(nameof(searchGenericType));

            return
                type != typeof(object) &&
                (type.IsGenericType &&
                searchGenericType.GetGenericTypeDefinition() == searchGenericType ||
                IsDerivingFromGenericType(type.BaseType, searchGenericType));
        }
```
 
*Update 1: [Type.IsSubclassOf](https://docs.microsoft.com/en-us/dotnet/api/system.type.issubclassof?view=netcore-3.1#:~:text=You%20can%20call%20the%20IsSubclassOf%20method%20to%20determine,a%20type%20is%20an%20enumeration.%20More%20items...%20) will give you the same result as IsDerivingFrom. The main purpose was (is) to use my implementation when having multiple levels of derivation and being able to debug the whole detection process.*

### 2. Get type of T from IEnumerable&lt;T&gt;

Sometimes, one needs to know the item type of an IEnumerable&lt;T&gt;. These two extensions will help you in this case:

``` csharp
         [System.Diagnostics.CodeAnalysis.SuppressMessage("Style", "IDE0060:Remove unused parameter", Justification = "Extension method")]
        public static Type GetItemType<T>(this IEnumerable<T> enumerable) => typeof(T);

        public static Type? GetItemType(this object enumerable)
            => enumerable == null ? null :
            (enumerable.GetType().GetInterface(typeof(IEnumerable<>).Name)?.GetGenericArguments()[0]);
```
 
### 3. Check if a type implements a certain interface

Interfaces are supposed to make the life of a developer easier. Like with the type derivation, sometimes we need to know if a type implements a certain interface. This extension answers the question for you:

``` csharp
         public static bool ImplementsInterface(this Type? type, Type? @interface)
        {
            bool result = false;

            if (type == null || @interface == null)
                return result;

            var interfaces = type.GetInterfaces();
            if (@interface.IsGenericTypeDefinition)
            {
                foreach (var item in interfaces)
                {
                    if (item.IsConstructedGenericType && item.GetGenericTypeDefinition() == @interface)
                        result = true;
                }
            }
            else
            {
                foreach (var item in interfaces)
                {
                    if (item == @interface)
                        result = true;
                }
            }

            return result;
        }
```
 
*Update 2: [Type.IsAssignableFrom](https://docs.microsoft.com/en-us/dotnet/api/system.type.isassignablefrom?view=netcore-3.1) will also tell you if a type implements an interface. As for the IsDerivingFrom method, I wanted to be able to debug the detection, which is – besides from having an explicit implementation – the main reason for this method.*

### 4. Find a Type in an external **assembly** 

If you need a type that lives in an external assembly but you only have it’s `Name` (or `FullName`, it’s easy to adapt), this one will resolve the Type for you as long as its assembly is loaded into your `AppDomain`:

``` csharp
         public static Type? FindType(this string name)
        {
            Type? result = null;
            var nonDynamicAssemblies = AppDomain.CurrentDomain.GetAssemblies().Where(a => !a.IsDynamic);
            try
            {
                result = nonDynamicAssemblies.
                     SelectMany(a => a.GetExportedTypes()).
                     FirstOrDefault(t => t.Name == name);
            }
            catch
            {
                result = nonDynamicAssemblies.
                     SelectMany(a => a.GetTypes()).
                     FirstOrDefault(t => t.Name == name);
            }

            return result;
        }
```
 
### **Conclusion** 

Like I said in the beginning, this post will be used for future reference. These extensions made a lot of controls, MVVM implementations and business logic I wrote in the past (both for work and my private projects) a whole lot easier. Like always, I hope this post will be helpful for some of you.

Please find the full class in this [Gist](https://gist.github.com/MSicc/aac961bb30e9889cbd3a80efd59479b3).

##### Until the next post, happy coding!

Title Image by [Tim Hill](https://pixabay.com/users/timhill-5727184/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=5436525) from [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=5436525)