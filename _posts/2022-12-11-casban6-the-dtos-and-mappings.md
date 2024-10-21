---
id: 7160
title: '#CASBAN6: the DTOs and mappings'
date: '2022-12-11T08:38:17+01:00'
author: 'Marco Siccardi'
excerpt: 'In this post of my #CASBAN6 series, I am going to show the data transfer objects (DTO) used when calling the API and the corresponding mapping helpers for my serverless blog engine.'
layout: post
permalink: /casban6-the-dtos-and-mappings/
image: /assets/img/2022/12/CASBAN6_DTO_mappings_title.png
categories:
    - 'Dev Stories'
    - Azure
    - Database
tags:
    - '#CASBAN6'
    - API
    - Azure
    - 'Azure Functions'
    - DTO
    - EF
    - 'EF Core'
    - 'Entity Framework'
    - 'Entity Framework Core'
    - functions
    - Mappings
---

[We already have created our database and our entities]({% post_url 2022-11-19-casban6-implementing-the-data-model-using-entityframework-core-separate-libraries %}), so let’s have a look at how we bring the data to our API consuming applications.

If we recap, our entity models contain all the relations and identifiers. This could lead to some issues like circular references during serialization and unnecessary data repetition. Luckily for us, there is already a solution for this—it’s called data transfer object (DTO). The main purposes of a DTO is to serve data while being serializable ([see also Wikipedia](https://en.wikipedia.org/wiki/Data_transfer_object)).

### The DTO project

If you have been following along, you might already have guessed that I have created a [separate project for the DTO model classes](https://github.com/MSiccDev/ServerlessBlog/tree/main/src/DtoModel). The overall structure is similar to what you have already seen in my last post, where I showed you the entity model.

### Implementation

Let’s have an exemplary look at the `Medium` entity class:

``` csharp
 using System;
using System.Collections.Generic;

namespace MSiccDev.ServerlessBlog.EntityModel
{
    public class Medium
    {
        public Guid MediumId { get; set; }

        public Uri MediumUrl { get; set; }

        public string AlternativeText { get; set; }

        public string Description { get; set; }

        public Guid MediumTypeId { get; set; }
        public MediumType MediumType { get; set; }

        public Guid BlogId { get; set; }
        public Blog Blog { get; set; }

        public ICollection<Post> Posts { get; set; }
        public ICollection<Author> Authors { get; set; }

        public List<PostMediumMapping> PostMediumMappings { get; set; }

    }
}
```
 
The entity contains all relationships on the database. Our API will constrain a lot of them already down (we will see in a later post how), for example by requiring the `BlogId` for every call as primary identifier. There are a lot of other connection points, but we also want to be able to use the Medium endpoint just for managing media.

Here is the `Medium` DTO:

``` csharp
 using System;
namespace MSiccDev.ServerlessBlog.DtoModel
{
    public class Medium
    {
        public Guid MediumId { get; set; }

        public Uri MediumUrl { get; set; }

        public string AlternativeText { get; set; }

        public string Description { get; set; }

        public MediumType MediumType { get; set; }

        public bool? IsPostImage { get; set; } = null;
    }
}


```
 
The class contains all the information we need. With this DTO, we will be able to manage media files alone but also in its usage context (which is mostly within posts of a blog).

### Mapping helpers

To convert entity objects to data transfer objects and vice versa, we are using mappings. Mappings are converters that bring the data into the desired shape. On the contrary to our model classes, mappings are allowed to modify data during the conversion.

#### No library this time

If you are wondering why I am not using one of the established libraries for mappings, there are several reasons. When I came to the point of DTO implementation in the developing process, I evaluated the options for the mappings.

All of them had quite a learning curve, in the end, I was faster writing my own mappings. On bigger systems like shops or similar projects, I would probably have chosen the other path. There is also a small chance I change my mind one day, which would result in a refactoring session then.

Both mapping helper classes are, once again, in their own project.

### Converting entities to DTOs

As you can see in [the `EntityToDtoMapExtensions` class](https://github.com/MSiccDev/ServerlessBlog/blob/main/src/ModelHelper/EntityToDtoMapExtensions.cs), I created extension methods for all entity objects. To remain on the Medium class, here are the particular implementations (there should be no surprise):

``` csharp
 public static DtoModel.Medium ToDto(this EntityModel.Medium entity)
{
    return new DtoModel.Medium()
    {
        MediumId = entity.MediumId,
        MediumType = entity.MediumType.ToDto(),
        MediumUrl = entity.MediumUrl,
        AlternativeText = entity.AlternativeText,
        Description = entity.Description
    };
}

public static DtoModel.MediumType ToDto(this EntityModel.MediumType entity)
{
    return new DtoModel.MediumType()
    {
        MediumTypeId = entity.MediumTypeId,
        MimeType = entity.MimeType,
        Name = entity.Name,
        Encoding = entity.Encoding
    };
}
```
 
You may have noticed that I am not setting the `IsPostImage` property from within the extension. The information is only important in the context of a post, which is why the ToDto method for the post is setting it to true or false. Otherwise, it will be null and can be omitted in the API response.

### Converting DTOs to entities

There are two scenarios where we need to convert DTOs to entities: one is the creation of new entities, the other is updating existing entities. Being very creative with the names, I implemented a `CreateFrom` and an `UpdateWith` method for each DTO type.

You can have a look at [all implementations on Github](https://github.com/MSiccDev/ServerlessBlog/blob/main/src/ModelHelper/DtoToEntityMapExtensions.cs), like above, here we are focusing on the Medium DTO extensions:

``` csharp
 public static EntityModel.Medium CreateFrom(this DtoModel.Medium dto, Guid blogId)
{
    return new EntityModel.Medium()
    {
        BlogId = blogId,
        MediumId = dto.MediumId,
        MediumTypeId = dto.MediumType?.MediumTypeId ?? default,
        MediumUrl = dto.MediumUrl,
        AlternativeText = dto.AlternativeText,
        Description = dto.Description,
    };
}

public static EntityModel.Medium UpdateWith(this EntityModel.Medium existingMedium, DtoModel.Medium updatedMedium)
{
    if (existingMedium.MediumId != updatedMedium.MediumId)
        throw new ArgumentException("MediumId must be equal in UPDATE operation.");

    if (existingMedium.AlternativeText != updatedMedium.AlternativeText)
        existingMedium.AlternativeText = updatedMedium.AlternativeText;

    if (existingMedium.Description != updatedMedium.Description)
        existingMedium.Description = updatedMedium.Description;

    if (existingMedium.MediumTypeId != updatedMedium.MediumType.MediumTypeId)
        existingMedium.MediumTypeId = updatedMedium.MediumType.MediumTypeId;

    if (existingMedium.MediumUrl != updatedMedium.MediumUrl)
        existingMedium.MediumUrl = updatedMedium.MediumUrl;

    return existingMedium;
}
```
 
Once again, there should be no surprise in the implementation. If you have a look at the other methods, you will find them implemented similarly.

### Conclusion

In this post, I explained why we need DTOs and showed you how I implemented them. We also had a look at the mapping extensions to convert the entities to data transfer objects and vice versa. Now that we have them in place, we are able to start implementing our Azure Functions, which is where we are heading to next in [the #CASBAN6 blog series]({% post_url 2022-09-05-casban6-creating-a-serverless-blog-on-azure-with-net-6-new-series %}).

#### Until the next post, happy coding, everyone!