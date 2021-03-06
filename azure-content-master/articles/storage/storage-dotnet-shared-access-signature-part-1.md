<properties
	pageTitle="Shared Access Signatures: Understanding the SAS Model | Microsoft Azure"
	description="Learn about delegating access to Azure Storage resources, including blobs, queues, tables, and files, using shared access signatures (SAS). Shared access signatures protect your storage account key while granting access to resources in your account to other users. You can control the permissions that you grant and the interval over which the SAS is valid. If you also establish a stored access policy, you can revoke the SAS should you fear your account security is compromised."
	services="storage"
	documentationCenter=""
	authors="tamram"
	manager="carmonm"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="05/23/2016"
	ms.author="tamram"/>



# Shared Access Signatures, Part 1: Understanding the SAS model

## Overview

Using a shared access signature (SAS) is a powerful way to grant limited access to objects in your storage account to other clients, without having to expose your account key. In Part 1 of this tutorial on shared access signatures, we'll provide an overview of the SAS model and review SAS best practices. [Part 2](storage-dotnet-shared-access-signature-part-2.md) of the tutorial walks you through the process of creating shared access signatures with the Blob service.

## What is a shared access signature?

A shared access signature provides delegated access to resources in your storage account. This means that you can grant a client limited permissions to objects in your storage account for a specified period of time and with a specified set of permissions, without having to share your account access keys. The SAS is a URI that encompasses in its query parameters all of the information necessary for authenticated access to a storage resource. To access storage resources with the SAS, the client only needs to pass in the SAS to the appropriate constructor or method.

## When should you use a shared access signature?

You can use a SAS when you want to provide access to resources in your storage account to a client that can't be trusted with the account key. Your storage account keys include both a primary and secondary key, both of which grant administrative access to your account and all of the resources in it. Exposing either of your account keys opens your account to the possibility of malicious or negligent use. Shared access signatures provide a safe alternative that allows other clients to read, write, and delete data in your storage account according to the permissions you've granted, and without need for the account key.

A common scenario where a SAS is useful is a service where users read and write their own data to your storage account. In a scenario where a storage account stores user data, there are two typical design patterns:


1\. Clients upload and download data via a front-end proxy service, which performs authentication. This front-end proxy service has the advantage of allowing validation of business rules, but for large amounts of data or high-volume transactions, creating a service that can scale to match demand may be expensive or difficult.

![sas-storage-fe-proxy-service][sas-storage-fe-proxy-service]

2\.	A lightweight service authenticates the client as needed and then generates a SAS. Once the client receives the SAS, they can access storage account resources directly with the permissions defined by the SAS and for the interval allowed by the SAS. The SAS mitigates the need for routing all data through the front-end proxy service.

![sas-storage-provider-service][sas-storage-provider-service]

Many real-world services may use a hybrid of these two approaches, depending on the scenario involved, with some data processed and validated via the front-end proxy while other data is saved and/or read directly using SAS.

Additionally, you will need to use a SAS to authenticate the source object in a copy operation in certain scenarios:

- When you copy a blob to another blob that resides in a different storage account, you must use a SAS to authenticate the source blob. With version 2015-04-05, you can optionally use a SAS to authenticate the destination blob as well.
- When you copy a file to another file that resides in a different storage account, you must use a SAS to authenticate the source file. With version 2015-04-05, you can optionally use a SAS to authenticate the destination file as well.
- When you copy a blob to a file, or a file to a blob, you must use a SAS to authenticate the source object, even if the source and destination objects reside within the same storage account.

## Types of shared access signatures

Version 2015-04-05 of Azure Storage introduces a new type of shared access signature, the account SAS. You can now create either of two types of shared access signatures:

- **Account SAS.** The account SAS delegates access to resources in one or more of the storage services. All of the operations available via a service SAS are also available via an account SAS. Additionally, with the account SAS, you can delegate access to operations that apply to a given service, such as **Get/Set Service Properties** and **Get Service Stats**. You can also delegate access to read, write, and delete operations on blob containers, tables, queues, and file shares that are not permitted with a service SAS. See [Constructing an Account SAS](https://msdn.microsoft.com/library/mt584140.aspx) for in-depth information about about constructing the account SAS token.

- **Service SAS.** The service SAS delegates access to a resource in just one of the storage services: the Blob, Queue, Table, or File service. See [Constructing a Service SAS](https://msdn.microsoft.com/library/dn140255.aspx) and [Service SAS Examples](https://msdn.microsoft.com/library/dn140256.aspx) for in-depth information about constructing the service SAS token.

## How a shared access signature works

A shared access signature is a URI that points to one or more storage resources and includes a token that contains a special set of query parameters. The token indicates how the resources may be accessed by the client. One of the query parameters, the signature, is constructed from the SAS parameters and signed with the account key. This signature is used by Azure Storage to authenticate the SAS.

The account SAS and service SAS tokens include some common parameters, and also take a few parameters that that are different.

### Parameters common to account SAS and service SAS tokens

- **Api version** An optional parameter that specifies the storage service version to use to execute the request.
- **Service version** A required parameter that specifies the storage service version to use to  authenticate the request.
- **Start time.** This is the time at which the SAS becomes valid. The start time for a shared access signature is optional; if omitted, the SAS is effective immediately. Must be expressed in UTC (Coordinated Universal Time), with a special UTC designator ("Z") i.e. 1994-11-05T13:15:30Z.
- **Expiry time.** This is the time after which the SAS is no longer valid. Best practices recommend that you either specify an expiry time for a SAS, or associate it with a stored access policy. Must be expressed in UTC (Coordinated Universal Time), with a special UTC designator ("Z") i.e. 1994-11-05T13:15:30Z (see more below).
- **Permissions.** The permissions specified on the SAS indicate what operations the client can perform against the storage resource using the SAS. Available permissions differ for an account SAS and a service SAS.
- **IP.** An optional parameter that specifies an IP address or a range of IP addresses outside of Azure (see the section [Routing session configuration state](../expressroute/expressroute-workflows.md#routing-session-configuration-state) for Express Route) from which to accept requests.
- **Protocol.** An optional parameter that specifies the protocol permitted for a request. Possible values are both HTTPS and HTTP (https,http), which is the default value, or HTTPS only (https). Note that HTTP only is not a permitted value.
- **Signature.** The signature is constructed from the other parameters specified as part token and then encrypted. It's used to authenticate the SAS.

### Parameters for an account SAS token

- **Service or services.** An account SAS can delegate access to one or more of the storage services. For example, you can create an account SAS that delegates access to the Blob and File service. Or you can create a SAS that delegates access to all four services (Blob, Queue, Table, and File).
- **Storage resource types.** An account SAS applies to one or more classes of storage resources, rather than a specific resource. You can create an account SAS to delegate access to:
	- Service-level APIs, which are called against the storage account resource. Examples include **Get/Set Service Properties**, **Get Service Stats**, and **List Containers/Queues/Tables/Shares**.
	- Container-level APIs, which are called against the container objects for each service: blob containers, queues, tables, and file shares. Examples include **Create/Delete Container**, **Create/Delete Queue**, **Create/Delete Table**, **Create/Delete Share**, and **List Blobs/Files and Directories**.
	- Object-level APIs, which are called against blobs, queue messages, table entities, and files. For example, **Put Blob**, **Query Entity**, **Get Messages**, and **Create File**.

### Parameters for a service SAS token

- **Storage resource.** Storage resources for which you can delegate access with a service SAS include:
	- Containers and blobs
	- File shares and files
	- Queues
	- Tables and ranges of table entities.

## Examples of SAS URIs

Here is an example of a service SAS URI that provides read and write permissions to a blob. The table breaks down each part of the URI to understand how it contributes to the SAS:

	https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt?sv=2015-04-05&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D

Name|SAS portion|Description
---|---|---
Blob URI|https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt |The address of the blob. Note that using HTTPS is highly recommended.
Storage services version|sv=2015-04-05|For storage services version 2012-02-12 and later, this parameter indicates the version to use.
Start time|st=2015-04-29T22%3A18%3A26Z|Specified in UTC time. If you want the SAS to be valid immediately, omit the start time.
Expiry time|se=2015-04-30T02%3A23%3A26Z|Specified in UTC time.
Resource|sr=b|The resource is a blob.
Permissions|sp=rw|The permissions granted by the SAS include Read (r) and Write (w).
IP range|sip=168.1.5.60-168.1.5.70|The range of IP addresses from which a request will be accepted.
Protocol|spr=https|Only requests using HTTPS are permitted.
Signature|sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D|Used to authenticate access to the blob. The signature is an HMAC computed over a string-to-sign and key using the SHA256 algorithm, and then encoded using Base64 encoding.

And here is an example of an account SAS that uses the same common parameters on the token. Since these parameters are described above, they are not described here. Only the parameters that are specific to account SAS are described in the table below.

	https://myaccount.blob.core.windows.net/?restype=service&comp=properties&sv=2015-04-05&ss=bf&srt=s&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=F%6GRVAZ5Cdj2Pw4tgU7IlSTkWgn7bUkkAg8P6HESXwmf%4B

Name|SAS portion|Description
---|---|---
Resource URI|https://myaccount.blob.core.windows.net/?restype=service&comp=properties|The Blob service endpoint, with parameters for getting service properties (when called with GET) or setting service properties (when called with SET).
Services|ss=bf|The SAS applies to the Blob and File services
Resource types|srt=s|The SAS applies to service-level operations.
Permissions|sp=rw|The permissions grant access to read and write operations.  

Given that permissions are restricted to the service level, accessible operations with this SAS are **Get Blob Service Properties** (read) and **Set Blob Service Properties** (write). However, with a different resource URI, the same SAS token could also be used to delegate access to **Get Blob Service Stats** (read).

## Controlling a SAS with a stored access policy ##

A shared access signature can take one of two forms:

- **Ad hoc SAS:** When you create an ad hoc SAS, the start time, expiry time, and permissions for the SAS are all specified on the SAS URI (or implied, in the case where start time is omitted). This type of SAS may be created as an account SAS or a service SAS.

- **SAS with stored access policy:** A stored access policy is defined on a resource container - a blob container, table, queue, or file share - and can be used to manage constraints for one or more shared access signatures. When you associate a SAS with a stored access policy, the SAS inherits the constraints - the start time, expiry time, and permissions - defined for the stored access policy.

>[AZURE.NOTE] Currently, an account SAS must be an ad hoc SAS. Stored access policies are not yet supported for account SAS.

The difference between the two forms is important for one key scenario: revocation. A SAS is a URL, so anyone who obtains the SAS can use it, regardless of who requested it to begin with. If a SAS is published publicly, it can be used by anyone in the world. A SAS that is distributed is valid until one of four things happens:

1.	The expiry time specified on the SAS is reached.
2.	The expiry time specified on the stored access policy referenced by the SAS is reached (if a stored access policy is referenced, and if it specifies an expiry time). This can either occur because the interval elapses, or because you have modified the stored access policy to have an expiry time in the past, which is one way to revoke the SAS.
3.	The stored access policy referenced by the SAS is deleted, which is another way to revoke the SAS. Note that if you recreate the stored access policy with exactly the same name, all existing SAS tokens will again be valid according to the permissions associated with that stored access policy (assuming that the expiry time on the SAS has not passed). If you are intending to revoke the SAS, be sure to use a different name if you recreate the access policy with an expiry time in the future.
4.	The account key that was used to create the SAS is regenerated.  Note that doing this will cause all application components using that account key to fail to authenticate until they are updated to use either the other valid account key or the newly regenerated account key.

>[AZURE.IMPORTANT] A shared access signature URI is associated with the account key used to create the signature, and the associated stored access policy (if any). If no stored access policy is specified, the only way to revoke a shared access signature is to change the account key.

## Examples: Create and use shared access signatures

Below are some examples of both types of shared access signatures, account SAS and service SAS.

To run these examples, you'll need to download and reference these packages:

- [Azure Storage Client Library for .NET](http://www.nuget.org/packages/WindowsAzure.Storage), version 6.x or later (to use account SAS).
- [Azure Configuration Manager](http://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager)

### Example: Account SAS

The following code example creates an account SAS that is valid for the Blob and File services, and gives the client permissions read, write, and list permissions to access service-level APIs. The account SAS restricts the protocol to HTTPS, so the request must be made with HTTPS.

    static string GetAccountSASToken()
    {
        // To create the account SAS, you need to use your shared key credentials. Modify for your account.
	    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
	    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

        // Create a new access policy for the account.
        SharedAccessAccountPolicy policy = new SharedAccessAccountPolicy()
            {
                Permissions = SharedAccessAccountPermissions.Read | SharedAccessAccountPermissions.Write | SharedAccessAccountPermissions.List,
                Services = SharedAccessAccountServices.Blob | SharedAccessAccountServices.File,
                ResourceTypes = SharedAccessAccountResourceTypes.Service,
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
                Protocols = SharedAccessProtocol.HttpsOnly
            };

        // Return the SAS token.
        return storageAccount.GetSharedAccessSignature(policy);
    }

To use the account SAS to access service-level APIs for the Blob service, construct a Blob client object using the SAS and the Blob storage endpoint for your storage account.

    static void UseAccountSAS(string sasToken)
    {
        // In this case, we have access to the shared key credentials, so we'll use them
        // to get the Blob service endpoint.
	    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
	    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        // Create new storage credentials using the SAS token.
        StorageCredentials accountSAS = new StorageCredentials(sasToken);
        // Use these credentials and the Blob storage endpoint to create a new Blob service client.
        CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, blobClient.StorageUri, null, null, null);
        CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();

        // Now set the service properties for the Blob client created with the SAS.
        blobClientWithSAS.SetServiceProperties(new ServiceProperties()
        {
            HourMetrics = new MetricsProperties()
            {
                MetricsLevel = MetricsLevel.ServiceAndApi,
                RetentionDays = 7,
                Version = "1.0"
            },
            MinuteMetrics = new MetricsProperties()
            {
                MetricsLevel = MetricsLevel.ServiceAndApi,
                RetentionDays = 7,
                Version = "1.0"
            },
            Logging = new LoggingProperties()
            {
                LoggingOperations = LoggingOperations.All,
                RetentionDays = 14,
                Version = "1.0"
            }
        });

        // The permissions granted by the account SAS also permit you to retrieve service properties.
        ServiceProperties serviceProperties = blobClientWithSAS.GetServiceProperties();
        Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
        Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
        Console.WriteLine(serviceProperties.HourMetrics.Version);
    }

### Example: Service SAS with stored access policy

The following code example creates a stored access policy on a container and then generates a service SAS for the container. This SAS can then be given to clients for read-write permissions on the container. Change the code to use your own account name:

    // Parse the connection string for the storage account.
    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

    // Create the storage account with the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageConnectionString);

    // Create the blob client object.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Get a reference to the container for which shared access signature will be created.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");
    container.CreateIfNotExists();

    // Get the current permissions for the blob container.
    BlobContainerPermissions blobPermissions = container.GetPermissions();

    // Clear the container's shared access policies to avoid naming conflicts.
    blobPermissions.SharedAccessPolicies.Clear();

    // The new shared access policy provides read/write access to the container for 24 hours.
    blobPermissions.SharedAccessPolicies.Add("mypolicy", new SharedAccessBlobPolicy()
    {
       // To ensure SAS is valid immediately, don???t set the start time.
       // This way, you can avoid failures caused by small clock differences.
       SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
       Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Create | SharedAccessBlobPermissions.Add
    });

    // The public access setting explicitly specifies that
    // the container is private, so that it can't be accessed anonymously.
    blobPermissions.PublicAccess = BlobContainerPublicAccessType.Off;

    // Set the new stored access policy on the container.
    container.SetPermissions(blobPermissions);

    // Get the shared access signature token to share with users.
    string sasToken =
       container.GetSharedAccessSignature(new SharedAccessBlobPolicy(), "mypolicy");

A client in possession of the service SAS can use it from their code to authenticate a request to read or write to a blob in the container. For example, the following code uses the SAS token to create a new block blob in the container. Change the code to use your own account name:

    Uri blobUri = new Uri("https://<myaccount>.blob.core.windows.net/mycontainer/myblob.txt");

    // Create credentials with the SAS token. The SAS token was created in previous example.
    StorageCredentials credentials = new StorageCredentials(sasToken);

    // Create a new blob.
    CloudBlockBlob blob = new CloudBlockBlob(blobUri, credentials);

    // Upload the blob.
    // If the blob does not yet exist, it will be created.
    // If the blob does exist, its existing content will be overwritten.
    using (var fileStream = System.IO.File.OpenRead(@"c:\Temp\myblob.txt"))
    {
    	blob.UploadFromStream(fileStream);
    }


## Best practices for using shared access signatures

When you use shared access signatures in your applications, you need to be aware of two potential risks:

- If a SAS is leaked, it can be used by anyone who obtains it, which can potentially compromise your storage account.
- If a SAS provided to a client application expires and the application is unable to retrieve a new SAS from your service, then the application's functionality may be hindered.  

The following recommendations for using shared access signatures will help balance these risks:

1. **Always use HTTPS** to create a SAS or to distribute a SAS.  If a SAS is passed over HTTP and intercepted, an attacker performing a man-in-the-middle attack will be able to read the SAS and then use it just as the intended user could have, potentially compromising sensitive data or allowing for data corruption by the malicious user.
2. **Reference stored access policies where possible.** Stored access policies give you the option to revoke permissions without having to regenerate the storage account keys.  Set the expiration on these to be a very long time (or infinite)  and make sure that it is regularly updated to move it farther into the future.
3. **Use near-term expiration times on an ad hoc SAS.** In this way, even if a SAS is compromised unknowingly, it will only be viable for a short time duration. This practice is especially important if you cannot reference a stored access policy. This practice also helps limit the amount of data that can be written to a blob by limiting the time available to upload to it.
4. **Have clients automatically renew the SAS if necessary.** Clients should renew the SAS well before the expected expiration, in order to allow time for retries if the service providing the SAS is unavailable.  If your SAS is meant to be used for a small number of immediate, short-lived operations, which are expected to be completed within the expiration time given, then this may not be necessary, as the SAS is not expected be renewed.  However, if you have client that is routinely making requests via SAS, then the possibility of expiration comes into play.  The key consideration is to balance the need for the SAS to be short-lived (as stated above) with the need to ensure that the client is requesting renewal early enough to avoid disruption due to the SAS expiring prior to successful renewal.
5. **Be careful with SAS start time.** If you set the start time for a SAS to **now**, then due to clock skew (differences in current time according to different machines), failures may be observed intermittently for the first few minutes.  In general, set the start time to be at least 15 minutes ago, or don't set it at all, which will make it valid immediately in all cases.  The same generally applies to expiry time as well - remember that you may observe up to 15 minutes of clock skew in either direction on any request.  Note for clients using a REST version prior to 2012-02-12, the maximum duration for a SAS that does not reference a stored access policy is 1 hour, and any policies specifying longer term than that will fail.
6.	**Be specific with the resource to be accessed.** A typical security best practice is to provide a user with the minimum required privileges.  If a user only needs read access to a single entity, then grant them read access to that single entity, and not read/write/delete access to all entities.  This also helps mitigate the threat of the SAS being compromised, as the SAS has less power in the hands of an attacker.
7.	**Understand that your account will be billed for any usage, including that done with SAS.** If you provide write access to a blob, a user may choose to upload a 200GB blob.  If you've given them read access as well, they may choose do download it 10 times, incurring 2TB in egress costs for you.  Again, provide limited permissions, to help mitigate the potential of malicious users.  Use short-lived SAS to reduce this threat (but be mindful of clock skew on the end time).
8.	**Validate data written using SAS.** When a client application writes data to your storage account, keep in mind that there can be problems with that data. If your application requires that that data be validated or authorized before it is ready to use, you should perform this validation after the data is written and before it is used by your application. This practice also protects against corrupt or malicious data being written to your account, either by a user who properly acquired the SAS, or by a user exploiting a leaked SAS.
9. **Don't always use SAS.** Sometimes the risks associated with a particular operation against your storage account outweigh the benefits of SAS.  For such operations, create a middle-tier service that writes to your storage account after performing business rule validation, authentication, and auditing. Also, sometimes it's simpler to manage access in other ways. For example, if you want to make all blobs in a container publically readable, you can make the container Public, rather than providing a SAS to every client for access.
10.	**Use Storage Analytics to monitor your application.** You can use logging and metrics to observe any spike in authentication failures due to an outage in your SAS provider service or to the inadvertent removal of a stored access policy. See the [Azure Storage Team Blog](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/08/03/windows-azure-storage-logging-using-logs-to-track-storage-requests.aspx) for additional information.

## Conclusion ##

Shared access signatures are useful for providing limited permissions to your storage account to clients that should not have the account key.  As such, they are a vital part of the security model for any application using Azure Storage.  If you follow the best practices listed here, you can use SAS to provide greater flexibility of access to resources in your storage account, without compromising the security of your application.

## Next Steps ##

- [Shared Access Signatures, Part 2: Create and use a SAS with Blob storage](storage-dotnet-shared-access-signature-part-2.md)
- [Get Started with Azure File storage on Windows](storage-dotnet-how-to-use-files.md)
- [Manage anonymous read access to containers and blobs](storage-manage-access-to-resources.md)
- [Delegating Access with a Shared Access Signature](http://msdn.microsoft.com/library/azure/ee395415.aspx)
- [Introducing Table and Queue SAS](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)
[sas-storage-fe-proxy-service]: ./media/storage-dotnet-shared-access-signature-part-1/sas-storage-fe-proxy-service.png
[sas-storage-provider-service]: ./media/storage-dotnet-shared-access-signature-part-1/sas-storage-provider-service.png
