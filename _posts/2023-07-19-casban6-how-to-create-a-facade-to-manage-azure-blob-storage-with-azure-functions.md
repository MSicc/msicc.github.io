---
id: 7362
title: '#CASBAN6 – How to create a facade to manage Azure Blob Storage with Azure Functions'
date: '2023-07-19T17:00:00+02:00'
author: 'Marco Siccardi'
excerpt: 'In this blog post, I will show you how to create an Azure Function that manages upload, download, and deletion of files with Azure Blob Storage.'
layout: post
permalink: /casban6-how-to-create-a-facade-to-manage-azure-blob-storage-with-azure-functions/
image: /assets/img/2023/07/casban6-azfunction-for-azblob-title.png
categories:
    - 'Dev Stories'
    - Azure
    - Web
tags:
    - '#CASBAN6'
    - Azure
    - 'Azure Functions'
    - AzureDev
    - Blob
    - 'Blob Storage'
    - DefaultAzureCredential
    - download
    - files
    - secure
    - upload
---

### Why the facade?

You might wonder why I am creating this facade to manage file uploads to the Azure Blob Storage. There are two main reasons. The first one is that I wanted to have a common endpoint for all interactions with Azure, also for file handling. The second one is security. The function app already has a high level of security by enforcing a Microsoft account, which is equally needed for calls the Blob endpoint.

### Preparation

Before we can start to use the Azure SDK internally, we need to add two new NuGet packages to our our Azure Function project:

- [`Azure.Storage.Blobs`](https://www.nuget.org/packages/Azure.Storage.Blobs)
- [`Azure.Identity`](https://www.nuget.org/packages/Azure.Identity)

Once we have these two added, we create a new HTTP trigger function called `BlobFunction` in our project.

Last but not least, we also need to set up an environment variable for our storage account name. For our local development environment, add the property `StorageAccountName` to the `local.settings.json` file. Remember, settings in this file are basically your local copy of the Azure Function app Configuration settings. This means, we’ll add that later on there as well.

### Authentication

If you have read [my last post](https://msicc.net/casban6-how-to-configure-azurite-to-use-defaultazurecredential-with-docker-on-macos/), you might already know that we are going to use the `AzureDefaultCredential` to authenticate our requests against the Blob storage API internally. This leads to the first method we implement to create an authenticated `BlobServiceClient`:

``` csharp
 private BlobServiceClient? GetBlobServiceClient()
{

    BlobServiceClient? blobServiceClient = null;
    string? storageAccountName = Environment.GetEnvironmentVariable("StorageAccountName");
    string? blobServiceUrl = null;

#if DEBUG
    if (!string.IsNullOrWhiteSpace(storageAccountName))
        blobServiceUrl = $"https://127.0.0.1:10000/{storageAccountName}";
#else
        if (!string.IsNullOrWhiteSpace(storageAccountName))
             blobServiceUrl = $"https://{storageAccountName}.blob.core.windows.net";
#endif

    if (!string.IsNullOrWhiteSpace(blobServiceUrl))
    {
        _logger.LogInformation("Using Blob Service Url: {Url}", blobServiceUrl);
        blobServiceClient = new BlobServiceClient(new Uri(blobServiceUrl), new DefaultAzureCredential());
    }
    else
    {
        _logger.LogError("Cannot read StorageAccountName setting for creation of blobServiceUrl");
    }

    return blobServiceClient;
}
```
 
Let’s break that down. As mentioned before, we are reading the configuration to get the storage account name. Then we use the `DEBUG` preprocessor directive to determine between the Azurite service url and the Azure service url. The later one is then used to create the authenticated `BlobServiceClient`.

### Upload files

Uploading files will be one of our most commonly used tasks besides downloading/reading them. Let’s create a new DTO for bundling the upload data:

``` csharp
 public class FileUploadRequest
{
    [JsonConstructor]
    public FileUploadRequest()
    {

    }

    public FileUploadRequest(byte[] fileBytes, string fileName, string containerName)
    {
        this.Base64Content = Convert.ToBase64String(fileBytes, Base64FormattingOptions.None);
        this.FileName = fileName;
        this.ContainerName = containerName;
    }

    [JsonProperty(Required = Required.Always)]
    public string Base64Content { get; set; }

    [JsonProperty(Required = Required.Always)]
    public string FileName { get; set; }

    [JsonProperty(Required = Required.Always)]
    public string ContainerName { get; set; }

}
```
 
When creating the request, the constructor creates a Base64 string from the file’s byte array and sets the file name. We also need to specify the container name the file should be stored into separately. The parameterless constructor is only used internally for deserialising.

The second DTO we are creating is the response object, which is pretty simple and follows the same scheme:

``` csharp
 public class FileUploadResponse
{
    [JsonConstructor]
    public FileUploadResponse()
    {

    }

    public FileUploadResponse(string fileName, string containerName)
    {
        this.FileName = fileName;
        this.ContainerName = containerName;
    }

    [JsonProperty(Required = Required.Always)]
    public string FileName { get; set; }

    [JsonProperty(Required = Required.Always)]
    public string ContainerName { get; set; }
}
```
 
With our DTO objects in place, we are now already able to create our uploading function. As with all other endpoints so far, we are also adding Swagger to the mix to make the API easily testable in our browser. Here’s the full method:

``` csharp
 [OpenApiOperation("CREATE", "Blob", Description = "Creates a new blob for the attached file.", Visibility = OpenApiVisibilityType.Important)]
[OpenApiRequestBody("application/json", typeof(FileUploadRequest), Required = true, Description = "The file to upload")]
[OpenApiParameter("overwrite", In = ParameterLocation.Query, Type = typeof(bool), Required = false, Description = "overwrite existing files with the same name", Visibility = OpenApiVisibilityType.Important)]
[OpenApiParameter("ensureUnique", In = ParameterLocation.Query, Type = typeof(bool), Required = false, Description = "make sure the file name is unique", Visibility = OpenApiVisibilityType.Important)]
[OpenApiResponseWithoutBody(HttpStatusCode.Created, Description = "OK response if the file upload operation succeeded")]
[OpenApiResponseWithoutBody(HttpStatusCode.Unauthorized, Description = "Response for unauthenticated requests.")]
[OpenApiResponseWithBody(HttpStatusCode.BadRequest, "text/plain", typeof(string), Description = "Request cannot not be processed, see response body why")]
[Function($"{nameof(BlobFunction)}_{nameof(Create)}")]
public async Task<HttpResponseData> Create([HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = Route)] HttpRequestData req)
{
    try
    {
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

        FileUploadRequest? fileUploadRequest = JsonConvert.DeserializeObject<FileUploadRequest>(requestBody);

        if (fileUploadRequest != null)
        {
            byte[] fileBytes = Convert.FromBase64String(fileUploadRequest.Base64Content);

            BlobServiceClient? blobServiceClient = GetBlobServiceClient();

            if (blobServiceClient != null)
            {
                BlobContainerClient? containerClient = blobServiceClient.GetBlobContainerClient(fileUploadRequest.ContainerName);

                _ = bool.TryParse(req.GetProperty("ensureUnique"), out bool ensureUnique);

                string blobName = ensureUnique ? $"{Path.GetFileNameWithoutExtension(fileUploadRequest.FileName)}_{Guid.NewGuid()}{Path.GetExtension(fileUploadRequest.FileName)}" : fileUploadRequest.FileName;
                
                BlobClient? blobClient = containerClient.GetBlobClient(blobName);

                _ = bool.TryParse(req.GetProperty("overwrite"), out bool overwrite);
                
                try
                {
                    await blobClient.UploadAsync(new BinaryData(fileBytes), overwrite);
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Error creating blob container client");

                    if (ex is RequestFailedException requestFailedException)
                    {
                        if (requestFailedException.ErrorCode == "ContainerNotFound")
                        {
                            await blobServiceClient.CreateBlobContainerAsync(fileUploadRequest.ContainerName);

                            await blobClient.UploadAsync(new BinaryData(fileBytes), overwrite);
                        }
                    }
                    else
                    {
                        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
                    }
                }

                return await req.CreateResponseDataWithJsonAsync(HttpStatusCode.Created, new FileUploadResponse(blobName, fileUploadRequest.ContainerName), _jsonSerializerSettings);
                
            }

            _logger.LogError("Error creating blob object because a BlobServiceClient couldn't be created");
            return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
        }

        return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Submitted file upload is invalid, blob cannot be created.");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error creating blob object");
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
    }
}
```
 
Let’s break that method down. First, we are reading the submitted data and try to create a `FileUploadRequest` object from it. Once we have the uploaded object, we are converting the Base64 string back into a byte array, which we will use later on.

We are then checking the optional parameter to ensure uniqueness, which adds a `GUID` if it is true. On top, we are able to overwrite files with the exact same file name if we want/need to do so.

Finally, we are trying to upload the byte array. If we succeed, we return a `201 Created` response with the blob name and the container name. This way, the url of the file is never exposed.

There is also some error handling in this method. If the specified container does not exist, for example, it will be created and the file then uploaded. This adds some responsibility to the developer using it, but also the flexibility of using different containers. All other cases are handled by error responses.

### Download files

As uploading files is essential for our API, this is the case for downloading files as well. Let’s have a look at the download endpoint:

``` csharp
 [OpenApiOperation("GET", "Blob", Description = "Gets a file from the Azure Blob Storage.", Visibility = OpenApiVisibilityType.Important)]
[OpenApiParameter("containerName", In = ParameterLocation.Query, Type = typeof(string), Required = true, Description = "container name to store the files in", Visibility = OpenApiVisibilityType.Important)]
[OpenApiParameter("fileName", In = ParameterLocation.Query, Type = typeof(string), Required = true, Description = "Name of the file to download", Visibility = OpenApiVisibilityType.Important)]
[OpenApiResponseWithBody(HttpStatusCode.OK, "application/octet-stream", typeof(byte[]), Description = "Gets a file by its name in the Azure Blob Storage")]
[OpenApiResponseWithoutBody(HttpStatusCode.Unauthorized, Description = "Response for unauthenticated requests.")]
[OpenApiResponseWithoutBody(HttpStatusCode.NotFound, Description = "No file with the specified file name was found")]
[OpenApiResponseWithBody(HttpStatusCode.BadRequest, "text/plain", typeof(string), Description = "Request cannot not be processed, see response body why")]
[Function($"{nameof(BlobFunction)}_{nameof(GetFile)}")]
public async Task<HttpResponseData> GetFile([HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = Route)] HttpRequestData req)
{
    string? fileName = req.GetProperty("fileName");
    if (string.IsNullOrWhiteSpace(fileName))
    {
        _logger.LogError("Error: file cannot be found without providing the file name");
        return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Please provide a valid file name.");
    }

    try
    {
        BlobServiceClient? blobServiceClient = GetBlobServiceClient();

        if (blobServiceClient != null)
        {
            string? containerName = req.GetProperty("containerName");
            if (string.IsNullOrWhiteSpace(containerName))
            {
                _logger.LogError("Error: file cannot be found without providing a container name");
                return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Please provide a valid container name.");
            }

            BlobContainerClient? containerClient = blobServiceClient.GetBlobContainerClient(containerName);
            BlobClient? blobClient = containerClient.GetBlobClient(fileName);

            try
            {
                // ReSharper disable UseAwaitUsing
                using Stream? stream = await blobClient.OpenReadAsync();
                // ReSharper restore UseAwaitUsing

                return await req.CreateBytesResponseAsync(HttpStatusCode.OK, stream, fileName);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error getting file blob with Name \'{Name}\'", fileName);

                if (ex is RequestFailedException requestFailedEx)
                {
                    if (requestFailedEx.ErrorCode == "BlobNotFound")
                        return await req.CreateResponseDataAsync(HttpStatusCode.NotFound, "The specified file does not exist on the server.");

                    return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, requestFailedEx.Message);
                }
                
                return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
            }
        }

        _logger.LogError("Error getting file because a BlobServiceClient couldn't be created");
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");

    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error getting file blob with Name \'{Name}\'", fileName);
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
    }
}
```
 
Look at this method, there should be no big surprises. We are checking the file name, if there is one, we are moving on to check the container name. If both are provided, we ask for the file from the container. If it is found, we send the raw bytes of the blob down to the client. The whole process is, once again, surrounded by a bunch of error handling code.

### Delete files

Occasionally, we also might need to delete some of our files. This is why we also have an endpoint for that:

``` csharp
 [OpenApiOperation("DELETE", "Blob", Description = "Delete a blob from the Azure Storage.", Visibility = OpenApiVisibilityType.Important)]
[OpenApiParameter("containerName", In = ParameterLocation.Query, Type = typeof(string), Required = true, Description = "container name to store the files in", Visibility = OpenApiVisibilityType.Important)]
[OpenApiParameter("fileName", In = ParameterLocation.Query, Type = typeof(string), Required = true, Description = "Name of the file to delete", Visibility = OpenApiVisibilityType.Important)]
[OpenApiResponseWithoutBody(HttpStatusCode.OK, Description = "OK response if the delete operation succeeded")]
[OpenApiResponseWithoutBody(HttpStatusCode.NotFound, Description = "No file with the specified file name was found")]
[OpenApiResponseWithoutBody(HttpStatusCode.Unauthorized, Description = "Response for unauthenticated requests.")]
[OpenApiResponseWithBody(HttpStatusCode.BadRequest, "text/plain", typeof(string), Description = "Request cannot not be processed, see response body why")]
[Function($"{nameof(BlobFunction)}_{nameof(Delete)}")]
public async Task<HttpResponseData> Delete([HttpTrigger(AuthorizationLevel.Anonymous, "delete", Route = Route)] HttpRequestData req)
{
    string? fileName = req.GetProperty("fileName");
    if (string.IsNullOrWhiteSpace(fileName))
    {
        _logger.LogError("Error: file cannot be deleted without providing the file name");
        return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Submitted file upload is invalid, blob cannot be created.");
    }

    try
    {
        BlobServiceClient? blobServiceClient = GetBlobServiceClient();

        if (blobServiceClient != null)
        {
            string? containerName = req.GetProperty("containerName");
            if (string.IsNullOrWhiteSpace(containerName))
            {
                _logger.LogError("Error: file cannot be found without providing a container name");
                return await req.CreateResponseDataAsync(HttpStatusCode.BadRequest, "Please provide a valid container name.");
            }
            
            BlobContainerClient? containerClient = blobServiceClient.GetBlobContainerClient(containerName);
            BlobClient? blobClient = containerClient.GetBlobClient(fileName);

            try
            {
                await blobClient.DeleteIfExistsAsync();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error deleting file blob with Name \'{Name}\'", fileName);
                return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
            }

            return req.CreateResponse(HttpStatusCode.OK);
        }

        _logger.LogError("Error deleting file because a BlobServiceClient couldn't be created");
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");

    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error deleting file blob with Name \'{Name}\'", fileName);
        return await req.CreateResponseDataAsync(HttpStatusCode.InternalServerError, "An internal server error occured. Error details logged.");
    }
}
```
 
As with the prior endpoints, we first determine if we have all needed parameters to find the file. If so, we are moving on and delete the file. If this is successful, we return a `200 OK` response. Also in this method, we are handling all kinds of error and respond accordingly to the client.

### Conclusion

Using Azure Functions, we can easily create a facade for file handling. This adds not only a common API, but also an additional layer of security to our blog engine. As always, I hope this blog post is helpful for some of you.

You can find the code above [in the GitHub Repo](https://github.com/MSiccDev/ServerlessBlog) for this blog series.

#### Until the next post, happy coding!

---

[Title image created with Bing Image Creator](https://www.bing.com/images/create/create-me-a-featured-image-for--22how-to-create-a-f/64b77ccfb54a4f158b915b2d3a197c8a?id=b5vJoceqJdmO7bgbjytx5w%3d%3d&view=detailv2&idpp=genimg&FORM=GCRIDP&mode=overlay)