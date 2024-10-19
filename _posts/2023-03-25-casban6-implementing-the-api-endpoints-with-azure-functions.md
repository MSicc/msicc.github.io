---
id: 7238
title: '#CASBAN6: Implementing the API endpoints with Azure Functions'
date: '2023-03-25T08:50:51+01:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I will show you how to use the base class from the last post to create the actual implementations for our Azure Functions API endpoints.'
layout: post
permalink: /casban6-implementing-the-api-endpoints-with-azure-functions/
image: /assets/img/2023/03/CASBAN6-Azure-Function-endpoint-implementation.png
categories:
    - 'Dev Stories'
    - Azure
tags:
    - '#CASBAN6'
    - API
    - Azure
    - 'Azure Functions'
    - CRUD
    - EF
    - 'EF Core'
    - endpoints
    - 'Entity Framework'
    - json
    - REST
---

[After my last post](https://msicc.net/casban6-function-base-class-and-an-update-to-the-dto-models/), we have the base implementation ready to be used for our endpoints. As we already know, our endpoints will feature the *CRUD* pattern to interact with the endpoints. Please note: the [code on GitHub](https://github.com/MSiccDev/ServerlessBlog) is already a few steps ahead and may look a bit different from what I am posting here (mostly due to the `OpenApi` attributes, but you will be able to follow along).

I will use the `AuthorFunction` to demonstrate the implementation. All other function implementations besides the `BlogFunction` follow the same pattern.

### Let’s dive in

First, we create a new class that derives from our base class. The constructor initializes the EF context as well as the `ILogger `for the author function. On top, we are defining the `Route` template that our functions are going to use as constant, so we can refer it in the function’s attributes. You should now have something similar to this:

``` csharp
 public class AuthorFunction : BlogFunctionBase
{
    private const string Route = "blog/{blogId}/author";

    public AuthorFunction(BlogContext blogContext, ILoggerFactory loggerFactory) : base(blogContext) =>
        Logger = loggerFactory.CreateLogger<AuthorFunction>();
}
```
 
### Override the `Create` method

The Create function is obviously responsible for creating a new entry in our database. First, we check if the `blogId` query parameter was specified and if it is parsable as a Guid. This step is the same for all endpoints. If these checks succeed, we are moving on to the next step, in this case deserializing the submitted `Author` DTO.

We are using then the `CreateFrom` mapping extension method to transform the DTO into the `EntityModel.Author` object ([read my post on DTOs and mappings here](https://msicc.net/casban6-the-dtos-and-mappings/)). The latter one can then be added to the context’s authors list and saved. If all goes well, we create a `201 Created` response indicating the direct API url to read the newly created author. In all other cases, we have some error handling in place.

``` csharp
 [Function($"{nameof(AuthorFunction)}_{nameof(Create)}")]
public override async Task<HttpResponseData> Create([HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = Route)] HttpRequestData req, string blogId)
{
    try
    {
        if (string.IsNullOrWhiteSpace(blogId) || Guid.Parse(blogId) == default)
            return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Required parameter 'blogId' (GUID) is not specified or cannot be parsed.");


        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

        Author? author = JsonConvert.DeserializeObject<Author>(requestBody);

        if (author != null)
        {

            EntityModel.Author? newAuthorEntity = author.CreateFrom(Guid.Parse(blogId));

            EntityEntry<EntityModel.Author> createdAuthor =
                BlogContext.Authors.Add(newAuthorEntity);

            await BlogContext.SaveChangesAsync();

            return await req.CreateNewEntityCreatedResponseDataAsync(createdAuthor.Entity.AuthorId);
        }
        else
        {
            return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Submitted data is invalid, author cannot be created.");
        }
    }
    catch (Exception ex)
    {
        Logger.LogError(ex, "Error creating author object on blog with Id {BlogId}", blogId);
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
    }
}
```
 
### Override the `GetList` method

We are using the `GetList` method to retrieve a list of entities. This method employs also the count and skip parameters, which is a simple way of implementing paging. We are using the `ToDto` method on the `EnityModel.Author` list to return the entities to the caller.

If something is going wrong during the function call, we have some error handling in place.

``` csharp
 [Function($"{nameof(AuthorFunction)}_{nameof(GetList)}")]
public override async Task<HttpResponseData> GetList([HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = Route)] HttpRequestData req, string blogId)
{
    try
    {
        Logger.LogInformation("Trying to get authors...");

        if (string.IsNullOrWhiteSpace(blogId) || Guid.Parse(blogId) == default)
            return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Required parameter 'blogId' (GUID) is not specified or cannot be parsed.");

        (int count, int skip) = req.GetPagingProperties();

        List<EntityModel.Author> entityResultSet = await BlogContext.Authors.
                                                                     Include(author => author.UserImage).
                                                                     ThenInclude(media => media.MediumType).
                                                                     Where(author => author.BlogId == Guid.Parse(blogId)).
                                                                     Skip(skip).
                                                                     Take(count).
                                                                     ToListAsync();

        List<Author> resultSet = entityResultSet.Select(entity => entity.ToDto()).ToList();

        return await req.CreateOkResponseDataWithJsonAsync(resultSet, JsonSerializerSettings);
    }
    catch (Exception ex)
    {
        Logger.LogError(ex, "Error getting author list for blog with Id \'{Id}\'", blogId);
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
    }
}
```
 
### Override the `GetSingle` method

The GetSingle endpoint needs also the id of the desired entity to be executed. This call is for getting just a single author from the database. Also here we have our error handling in place.

``` csharp
 [Function($"{nameof(AuthorFunction)}_{nameof(GetSingle)}")]
public override async Task<HttpResponseData> GetSingle([HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = Route + "/{id}")] HttpRequestData req, string blogId, string id)
{
    try
    {
        if (string.IsNullOrWhiteSpace(blogId) || Guid.Parse(blogId) == default)
            return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Required parameter 'blogId' (GUID) is not specified or cannot be parsed.");
        
        if (!string.IsNullOrWhiteSpace(id))
        {
            Logger.LogInformation("Trying to get author with Id: {Id}...", id);

            EntityModel.Author? existingAuthor =
                await BlogContext.Authors.
                                  Include(author => author.UserImage).
                                  ThenInclude(media => media.MediumType).
                                  SingleOrDefaultAsync(author => author.BlogId == Guid.Parse(blogId) &&
                                                                 author.AuthorId == Guid.Parse(id));

            if (existingAuthor == null)
            {
                Logger.LogWarning("Author with Id {Id} not found", id);
                return req.CreateResponse(HttpStatusCode.NotFound);
            }

            return await req.CreateOkResponseDataWithJsonAsync(existingAuthor.ToDto(), JsonSerializerSettings);
        }
        else
        {
            return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Submitted data is invalid, must specify BlogId");
        }
    }
    catch (Exception ex)
    {
        Logger.LogError(ex, "Error getting author with Id '{AuthorId}' for blog with Id \'{BlogId}\'", id, blogId);
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
    }
}
```
 
### Override the `Update` method

The structure of the `Update` function should be no surprise. First check the blog’s id, then read the submitted `Author` DTO. If the author already exists, update the information of the `Author` in the database. Otherwise, tell the caller there is no such author.

``` csharp
 [Function($"{nameof(AuthorFunction)}_{nameof(Update)}")]
public override async Task<HttpResponseData> Update([HttpTrigger(AuthorizationLevel.Anonymous, "put", Route = Route + "/{id}")] HttpRequestData req, string blogId, string id)
{
    try
    {
        if (string.IsNullOrWhiteSpace(blogId) || Guid.Parse(blogId) == default)
            return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Required parameter 'blogId' (GUID) is not specified or cannot be parsed.");

        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

        Author? authorToUpdate = JsonConvert.DeserializeObject<Author>(requestBody);

        if (authorToUpdate != null)
        {
            EntityModel.Author? existingAuthor =
                await BlogContext.Authors.
                                  Include(author => author.UserImage).
                                  ThenInclude(media => media.MediumType).
                                  SingleOrDefaultAsync(author => author.BlogId == Guid.Parse(blogId) &&
                                                                 author.AuthorId == Guid.Parse(id));

            if (existingAuthor == null)
            {
                Logger.LogWarning("Author with Id {Id} not found", id);
                return req.CreateResponse(HttpStatusCode.NotFound);
            }

            existingAuthor.UpdateWith(authorToUpdate);

            await BlogContext.SaveChangesAsync();

            return req.CreateResponse(HttpStatusCode.Accepted);
        }
        else
        {
            return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Submitted data is invalid, author cannot be modified.");
        }
    }
    catch (Exception ex)
    {
        Logger.LogError(ex, "Error updating author with Id '{AuthorId}' for blog with Id \'{BlogId}\'", id, blogId);
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
    }
}
```
 
### Override the `Delete` method

Last but not least, we sometimes need to delete entities for whatever reason. This is where the `Delete` function comes into play. This function requires both the blog’s id and the entity’s id to be executed. As with all other functions, there is some error handling in place.

``` csharp
 [Function($"{nameof(AuthorFunction)}_{nameof(Delete)}")]
public override async Task<HttpResponseData> Delete([HttpTrigger(AuthorizationLevel.Anonymous, "delete", Route = Route + "/{id}")] HttpRequestData req, string blogId, string id)
{
    try
    {
        if (string.IsNullOrWhiteSpace(blogId) || Guid.Parse(blogId) == default)
            return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Required parameter 'blogId' (GUID) is not specified or cannot be parsed.");

        EntityModel.Author? existingAuthor = await BlogContext.Authors.
                                                               Include(author => author.UserImage).
                                                               SingleOrDefaultAsync(author => author.BlogId == Guid.Parse(blogId) &&
                                                                                              author.AuthorId == Guid.Parse(id));

        if (existingAuthor == null)
        {
            Logger.LogWarning("Author with Id {Id} not found", id);
            return req.CreateResponse(HttpStatusCode.NotFound);
        }

        BlogContext.Authors.Remove(existingAuthor);

        await BlogContext.SaveChangesAsync();

        return req.CreateResponse(HttpStatusCode.OK);
    }
    catch (Exception ex)
    {
        Logger.LogError(ex, "Error deleting author with Id '{AuthorId}' from blog with Id \'{BlogId}\'", id, blogId);
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
    }
}
```
 
### Helper methods

You might have noticed that there are some extensions methods in the code samples above you haven’t seen so far. I got you, here they are.

#### Paging properties

To get the paging properties (I use simple paging here), we have the query parameters `count` and `skip`. To extract them from the request, we do some parsing on the parameters that the `HttpRequestData` provides us.

``` csharp
 private static Dictionary<string, string> GetQueryParameterDictionary(this HttpRequestData req)
{
    Dictionary<string, string> result = new Dictionary<string, string>();

    string queryParams = req.Url.GetComponents(UriComponents.Query, UriFormat.UriEscaped);

    if (!string.IsNullOrWhiteSpace(queryParams))
    {
        string[] paramSplits = queryParams.Split('&');

        if (paramSplits.Any())
        {
            foreach (string split in paramSplits)
            {
                string[] valueSplits = split.Split('=');

                if (valueSplits.Any() && valueSplits.Length == 2)
                    result.Add(valueSplits[0], valueSplits[1]);
            }
        }
    }
    return result;
}

public static (int count, int skip) GetPagingProperties(this HttpRequestData req)
{
    Dictionary<string, string> queryParams = req.GetQueryParameterDictionary();

    int count = 10;
    int skip = 0;

    if (queryParams.Any(p => p.Key == nameof(count)))
        _ = int.TryParse(queryParams[nameof(count)], out count);

    if (queryParams.Any(p => p.Key == nameof(skip)))
        _ = int.TryParse(queryParams[nameof(skip)], out skip);

    return (count, skip);
}
```
 
#### HttpResponseData helpers

Our function will run in an isolated process. Besides having a bunch of advantages like easier dependency injection, it brings also some syntax changes. As you can see [from the docs](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide#http-trigger), we are now responding with a `HttpResponseData` object. For easier creation of these objects, I wrote these extensions:

``` csharp
 public static async Task<HttpResponseData> CreateResponseDataAsync(this HttpRequestData req, HttpStatusCode statusCode, string? message)
{
    HttpResponseData response = req.CreateResponse(statusCode);

    if (string.IsNullOrWhiteSpace(message))
        message = statusCode.ToString();

    await response.WriteStringAsync(message);

    return response;
}

public static async Task<HttpResponseData> CreateNewEntityCreatedResponseDataAsync(this HttpRequestData req, Guid createdResourceId)
{
    HttpResponseData response = req.CreateResponse(HttpStatusCode.Created);
    response.Headers.Add("Location", $"{req.Url}/{createdResourceId}");

    await response.WriteStringAsync("OK");

    return response;
}

public static async Task<HttpResponseData> CreateOkResponseDataWithJsonAsync(this HttpRequestData req, object responseData, JsonSerializerSettings? settings)
{
    string json = JsonConvert.SerializeObject(responseData, settings);
    
    HttpResponseData response = req.CreateResponse(HttpStatusCode.OK);
    response.Headers.Add("Content-Type", "application/json; charset=utf-8");

    await response.WriteStringAsync(json);

    return response;
}
```
 
The first one creates a response with the specified `HttpStatusCode` and an optional message. The second one is for the `201 Created` responses, while the last one is for the `200 OK` responses.

### The `BlogFunction`

The BlogFunction implmentation is the only one not deriving from the base class. As the blog is the root entity, there are some differences from the pattern above.

The `Create` method in this function works without the blog’s id, but otherwise is the same as for all other `Create` methods.

The `GetBlogList` method features count properties for its child entities (authors, posts, tags, media) and does also not need the blog’s id. Details of child entities should be loaded via their function implementations.

The `GetBlog` method tries to load a blog completely with all child entities. This may result in a very large data set and should be used with caution and for exports only.

The Update and Delete methods are once again following the pattern of the other functions, except they just need the blog’s id.

You can have a look at the `BlogFunction` right [here on GtiHub](https://github.com/MSiccDev/ServerlessBlog/blob/dev/src/BlogFunctions/BlogFunction.cs).

### Anonymous authorization

If you are wondering why all the functions have the `AuthorizationLevel` set to `Anonymous`, I got you. Once the function is deployed to Azure, we will use the Azure Active Directory to force a login to a Microsoft account (others may follow) to call our functions. We have a strong protection this way without big efforts.

### Conclusion

In this post, I showed you how to use the base class we created in the last post of the series, and also showed you how the `BlogFunction` differs from that. In the next post, we will have a look at how to add Swagger to our Functions that will make API testing a lot easier as we advance with our project. With each post, we are getting closer to deploy the API functions to Azure, so stay tuned for the next post(s)!

#### Until the next post, happy coding, everyone!