---
id: 7218
title: '#CASBAN6: Function base class (and an update to the DTO models)'
date: '2023-01-20T16:00:00+01:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I am going to show you the base class for all entity functions (except for the Blog entity). Additionally, I remark some changes to the DTO models that were occurring as development advanced.'
layout: post
permalink: /casban6-function-base-class-and-an-update-to-the-dto-models/
image: /assets/img/2023/01/CASBAN6_FunctionBase_Title.png
categories:
    - 'Dev Stories'
    - Azure
    - MAUI
tags:
    - .NET6
    - '#CASBAN6'
    - API
    - Azure
    - 'Azure Functions'
    - 'base class'
    - client
    - CRUD
    - DTO
    - github
    - refactoring
    - SDK
    - update
---

After we have been setting up our Azure Function project last time, we are now able to create a base class for our Azure functions. The main goal is to achieve a common configuration for all functions to make our life easier later on.

### CRUD defintion

Our API endpoints should provide us a CRUD (Create-Read-Update-Delete) interface. Our implementation will reflect this pattern as follows:

- A creation endpoint
- Two read endpoints – one for list results (like a list of posts) and one for receiving details of an entity
- An update endpoint to change existing entities
- A delete endpoint

As all of our entities are tied to a single blog’s Id, we will use this base class for all entities besides the `Blog` entity itself.

### The base class

``` csharp
     public abstract class BlogFunctionBase
    {
        internal readonly BlogContext BlogContext;
        internal ILogger? Logger;
        internal JsonSerializerSettings? JsonSerializerSettings;

        protected BlogFunctionBase(BlogContext blogContext)
        {
            BlogContext = blogContext ?? throw new ArgumentNullException(nameof(blogContext));

            CreateNewtonSoftSerializerSettings();
        }

        private void CreateNewtonSoftSerializerSettings()
        {
            JsonSerializerSettings = NewtonsoftJsonObjectSerializer.CreateJsonSerializerSettings();

            JsonSerializerSettings.ContractResolver = new CamelCasePropertyNamesContractResolver();

            JsonSerializerSettings.NullValueHandling = NullValueHandling.Ignore;
            JsonSerializerSettings.Formatting = Formatting.Indented;
            JsonSerializerSettings.ReferenceLoopHandling = ReferenceLoopHandling.Ignore;
            JsonSerializerSettings.DateFormatHandling = DateFormatHandling.IsoDateFormat;
            JsonSerializerSettings.DateParseHandling = DateParseHandling.DateTimeOffset;
        }

        public virtual Task<HttpResponseData> Create([HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = null)] HttpRequestData req,
            string blogId) =>
            throw new NotImplementedException();

        public virtual Task<HttpResponseData> GetList([HttpTriggerAttribute(AuthorizationLevel.Anonymous, "get", Route = null)] HttpRequestData req, string blogId) =>
            throw new NotImplementedException();

        public virtual Task<HttpResponseData> GetSingle([HttpTriggerAttribute(AuthorizationLevel.Anonymous, "get", Route = null)] HttpRequestData req, string blogId, string id) =>
            throw new NotImplementedException();

        public virtual Task<HttpResponseData> Update([HttpTriggerAttribute(AuthorizationLevel.Anonymous, "put", Route = null)] HttpRequestData req, string blogId, string id) =>
            throw new NotImplementedException();

        public virtual Task<HttpResponseData> Delete([HttpTriggerAttribute(AuthorizationLevel.Anonymous, "delete", Route = null)] HttpRequestData req, string blogId, string id) =>
            throw new NotImplementedException();

    }
```
 
Knowing the definition of our API endpoints, there shouldn’t be any surprises with that implementation. As a base class, the definition is of course `abstract`.

In the constructor, I am setting up how JSON objects will be handled. I am following common practices in formatting. The only thing that is different from using JSON.NET directly is the fact we need to explicitly use the `NewtonsoftJsonObjectSerializer.CreateJsonSerializerSettings` method to create an instance of the `JsonSerializerSettings` property.

We also have an internal` ILogger?` property, which will be used in the derived class to create a typed instance for logging purposes. Last but not least, I am enforcing passing in a `BlogContext` instance from our `Entity Framework` implementation.

If you look at the method declaration, you’ll see the authorization level is set to Anonymous. As I am using Azure Active Directory, I am handling the authorization on a separate layer (there will be a post on that topic as well). Besides that, there is nothing special. All methods need a `blogId` and those endpoints that interact with a resource need a resource `id` as well.

The blog entity has a similar structure, with some differences in the parameter definitions. To keep things simple, I decided to let it be different from the base class definition above. We will see this in the next post of this series.

### Update to the DTO models

While the blog series is ongoing, it still lags a bit behind on what I am currently working on ([you can follow the dev branch on GitHub for an up-to-date view](https://github.com/MSiccDev/ServerlessBlog/tree/dev)). As I am currently working on the administration client for our blog, I started to implement an SDK that can be used by all clients (the blog’s website will also just be a client). [See the original post on DTOs here](https://msicc.net/casban6-the-dtos-and-mappings/).

To be able to create a generic implementation of the calls to the API endpoints, I needed to create also a base class for the DTO models. It is a very simple class, as you can see:

``` csharp
 public abstract class DtoModelBase
{
    public virtual Guid? BlogId { get; set; }

    public virtual Guid? ResourceId { get; set; }
}
```
 
This base class allows me to specify a `Type` that derives from it in the SDK API calls – which was my main goal. Second, I have now a common `ResourceId` property instead of the `Id` being named after the class name of the DTO. Both properties are virtual to allow me to specify the `Required` attributes in the derived classes as needed. [You can see these changes on GitHub](https://github.com/MSiccDev/ServerlessBlog/blob/dev/src/DtoModel/DtoModelBase.cs).

The reason I am writing about the change already today is that it will have impact on how the functions are implemented, as both the API functions and the Client SDK use the same DTO classes.

### Conclusion

In this post, we had a look at the base class for our Azure functions we will use as an API and on the updated DTO models. With this prerequisite post in place, we will have a look at the function implementations in the next post.

#### Until the next post, happy coding, everyone!