<div>

WebCenter Content REST API

# About the REST APIs

The WebCenter Content REST APIs allow you to interact with content in WebCenter Content.

## REST Overview

REST, or Representational State Transfer, uses basic methods such as GET, POST, PUT, and DELETE over a standard protocol, such as HTTPS, to submit requests from a client application to operate on objects stored on a server.

A request is in the form of a uniform resource identifier (URI) that identifies the resource (such as a dDocName, dID, or GUID) on which to operate and includes any parameters necessary for the operation.

Each REST request from the client to a server contains all of the information necessary to understand the request and does not rely on the server to store information about the individual request or about any relationship between requests. Session state is stored (cached) entirely on the client.

Each REST request is translated to an `IdcService` request and the results of the service are returned as a JSON response. The primary `IdcService` called by each REST API is noted in the description of each API.

## Getting Started

To use WebCenter Content REST APIs, install a REST client like [Postman](https://www.postman.com/){target="_blank"}.

## Authorization

The WebCenter Content REST API supports multiple authorization types. The following types are supported:

-   Basic Auth
-   OAuth Token (needs to be setup by an administrator; for setup details see [Configure Two-Legged or Three-Legged OAuth](https://docs.oracle.com/en/cloud/paas/webcenter-content/webcenter-content-marketplace/index.html#configure-two-legged-or-three-legged-oauth).)
-   Anonymous calls with no authentication

## Using the Universal Query Format

The WebCenter Content REST API supports Universal Query Format. The Universal Query Format is intended to allow client applications to retrieve search results with a query syntax that could be recognized across all WebCenter Content supported search engines. For details see the knowledge base article [UniversalQueryFormat for WebCenter Content Doc ID 1670824.1](https://support.oracle.com/epmos/faces/DocumentDisplay?sourceId=FAQ&id=1670824.1){target="_blank"}.

## CORS Support

Cross-Origin Resource Sharing (CORS) is a protocol to allow handshake between the client and the server using HTTP Headers and Methods to ensure that the cross-origin request is allowed by the server. This handshake is handled seamlessly by the client when using the Javascript XMLHttpRequest object. Most browsers enforce a "Same-Origin" policy but if the server responds to the CORS handshake appropriately then the cross-origin request will be allowed by the browser.

If a customer has a browser application that integrates with WebCenter Content but is hosted in a different domain, adding the browser application domain to the CORS origins list will whitelist the domain.

### Whitelist Configuration

There are two config.cfg settings that control CORS behavior for both WebCenter Content REST APIs and Native (IdcService) APIs. A user with Admin privileges needs to set these in config.cfg to control CORS behavior, and when these settings are changed, the server must be restarted.

::: table-responsive
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                        Default           Description                                                                                                Example
  --------------------------- ----------------- ---------------------------------------------------------------------------------------------------------- --------------------------------------------------------------------
  AllowedCrossOriginDomains   null              A comma separated list of allowed domain origins. The `*` character is supported as a wildcard character   `AllowedCrossOriginDomains=https://*.company.com`\
                                                                                                                                                           Allows both https://front.company.com and https://back.company.com

  DisableCORSCredentials      false             When false, the `Access-Control-Allow-Credentials` response header is returned as true                     `DisableCORSCredentials=true`\
                                                                                                                                                           The `Access-Control-Allow-Credentials` header is not returned
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

### REST Preflight

All resources support an OPTIONS method. These OPTIONS methods always return 200 with no HTTP body. If the `Origin` header is set on the request AND allowed, the following headers are returned.

::: table-responsive
  Header                             Description
  ---------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Access-Control-Allow-Origin        A list of the domains permitted to make cross origin requests.
  Access-Control-Allow-Methods       A list of the HTTP methods that can be called on the resource.
  Access-Control-Allow-Credentials   A boolean indicating if requests must be made with credentials to the server, by default this is `true`.
  Access-Control-Allow-Headers       A list of allowed response headers. If request header `Access-Control-Request-Headers` is set, that list is returned as the response headers; otherwise the list is `Authorization, Accept, Content-Type`.
  Access-Control-Max-Age             The seconds a CORS preflight request can be cached. The server always returns 300.
:::

### REST Cross-Site Protection

All resources validate the `Origin` header. If the `Origin` header is set on the request AND allowed, the following headers are returned.

::: table-responsive
  Header                             Description
  ---------------------------------- -----------------------------------------------------------------------------------------------------------
  Access-Control-Allow-Origin        A list of the domains permitted to make cross origin requests.
  Access-Control-Allow-Credentials   A boolean indicating if requests must be made with credentials to the server. By default, this is `true`.
:::

## Background Jobs

WebCenter Content supports background jobs. These jobs are performed in the background by the server.

### Starting a Background Job

Background (or bulk) jobs are started with a POST call. This POST returns a `Location` header, a URI to get the status about the job. Only the user that started the job or an administrator can access the job. Background jobs are removed from the server a month after the job completes.

When a job is started, very little validation is done on the job; users will need to access the status URI to monitor the progress of the job.

#### Background Delete Job

The bulk delete job allows a user to delete up to 1000 files per job, see [Start Background Delete Job](#start-background-delete-job) for details about starting the job. The job will only delete files that the user has access to.

#### Background Download Job

The bulk download job allows a user to select up to 1000 files (or a file size total of 2 gigabytes) to be stored in a zip file that can be downloaded after the zip is complete, see [Start Background Download Job](#start-background-download-job) for details on starting the job. The job will only download files that the user has access to.

After the zip file is created, it is checked into the `BulkFile` DocType and `SecureBulkFile` Security Group, and is only accessible by the user that started the job or an administrator.

There are two config.cfg settings that control how zip files are checked in.

A user with administrator privileges can edit the config.cfg file to control behavior.

Care must be taken when changing these settings as existing zip files are not moved if these settings are changed.

::: table-responsive
  Name                                  Default          Description
  ------------------------------------- ---------------- ---------------------------------------------
  BulkDownloadZipArchiveDocType         BulkFile         The DocType used to store zip files.
  BulkDownloadZipArchiveSecurityGroup   SecureBulkFile   The security group used to store zip files.
:::

### Canceling a Background Job

Background (or bulk) jobs can be canceled before the job completes with a POST call. See [Cancel a Background Job](#cancel-a-background-job). Only the user that started the job or an administrator can cancel the job. The `dJobID` GUID is the last part of the `Location` header returned when the job was started.

Sending a cancel request does not ensure the job will terminate. The job decides if or when the job can safely terminate.

### Monitoring Background Job Status

When a Background (or bulk) job is started, a `Location` header is returned. This URI should be used to monitor the status of the job. See [Get Status on a Background Job](#get-status-on-a-background-job). Only the user that started the job or an administrator can access the status of the job.

## Resource URI

A REST call includes an HTTPS method, such as OPTIONS, POST, GET, PUT, or DELETE, and a resource identified by a Uniform Resource Identifier (URI).

The URI is in this format:

> https://{WCCHost}\[:WCCPort\]/documents/wcc/api/version/resourcePath

For example:

> https://www.example.com/documents/wcc/api/v1.1

## REST Request

The WebCenter Content REST API supports direct URL and JSON data format for REST requests. In general, the WebCenter Content REST API uses parameter names similar to the IdcService names. The WebCenter Content imposes no limits on REST request (URI and parameters). Good practice keeps URI length under 2048 characters and REST request bodies under 4096 characters.

### Path Parameters

Path parameters are resource parameters passed directly in the URI resource before the `?` character.

### Query Parameters

Query parameters are resource parameters passed directly in the URI resource after the `?` character.

### Body Parameters

Body parameters are resource parameters passed in the HTTP body of the REST request.

### FormData Parameters

FormData parameters are used in multi-part requests and passed in the HTTP body of the REST request. WebCenter Content REST APIs make use of this when uploading files.

### Range Requests

The WebCenter Content REST API supports single part range requests via the Range header when downloading files. The Range header can be used to return part of a file. A client can use the Range header to specify the bytes to that are requested.

The following Range header formats are supported:

::: table-responsive
  Range Format   Description                                                                      Example
  -------------- -------------------------------------------------------------------------------- ----------------------------------------------------------------
  bytes=s-e      Return the bytes starting with s to end bit e. Note that the range is 0-based.   To return 31 bytes: Range: bytes=10-40
  bytes=-e       Return the last e bytes of the file.                                             To return the last 10 bytes: Range: bytes=-10
  bytes=s-       Return all except the first s bytes of the file.                                 To return the file except the first 10 bytes: Range: bytes=10-
:::

When the requested range can be returned, the server returns 206 Partial Content and the Content-Range header. The Content-Range header is in the format:

> Content-Range: bytes `range-start`-`range-end`/`fileSize`

Where:

-   `range-start` is the first byte returned

-   `range-end` is the last byte returned

-   `fileSize` is the total size of the file

When the requested range cannot be returned, the server returns 416 Range Not Satisfiable.

## REST Response

The REST API supports returning the following data:

-   The response may return a file.
-   The response may return in the JSON format.
-   The response may be empty in which case standard HTTP returns codes that provide an indication of the success or failure of a response.

In general, the responses of the WebCenter Content REST APIs use parameter names similar to the IdcService names.

### HTTP Response Codes

The WebCenter Content REST APIs may return any valid HTTP response code. The most commonly returned codes are defined in the following table.

::: table-responsive
  HTTP Status Code            Description
  --------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  200 OK                      The request was successfully completed.
  201 Created                 The request has been fulfilled and resulted in a new resource being created. The response may include a `Location` header containing the canonical URI for the newly created resource.
  204 No Content              The request has been fulfilled and the server has no more details.
  206 Partial Content         The request has been fulfilled and the byte range has been returned.
  400 Bad Request             The request could not be processed because it contains missing or invalid information (such as a validation error on an input field or a missing required value).
  401 Unauthorized            The request is not authorized. The authentication credentials included with this request are missing or invalid.
  403 Forbidden               The user cannot be authenticated. The user does not have authorization to perform this request.
  404 Not Found               The request includes a resource URI that does not exist.
  409 Conflict                The request could not be processed because there is a conflict with the current state of the resource.
  416 Range Not Satisfiable   The server cannot return the bytes requested.
  500 Internal Server Error   The server encountered an unexpected condition that prevented it from fulfilling the request.
  503 Service Unavailable     The server is temporarily unavailable.
:::

### Error Responses

The following table describes the fields for error messages that can be returned when an error is returned.

::: table-responsive
  Name          Description
  ------------- ----------------------------------------------------------------------------------------------------------------
  type          A link that describes the type of error.
  title         A brief summary error message.
  detail        Details about the error from the server; the service `StatusMessage`.
  errorKey      When the error comes from the service layer; the service `StatusMessageKey`
  o:errorCode   When the error comes from the service layer `StatusCode`. A negative number indicates that the service failed.
:::

## Supported Capabilities

Objects in the server can have capabilities. The list of supported capabilties is:

::: table-responsive
  Name                         Description
  ---------------------------- -------------------------------------------------------------------------------------------------------------------------------------
  CREATE_CHILD_FOLDER          The capability to create a new child folder of the specified folder.
  CREATE_CHILD_DOCUMENT        The capability to create a new document in the specified folder.
  FILE_DOCUMENT                The capability to create an owner link to the specified document.
  UNFILE_DOCUMENT              The capability to delete the owner link to the specified document (without deleting the document itself or other soft links to it).
  UPDATE_METADATA              The capability to update the metadata of the specified item (other than access control metadata).
  UPDATE_SECURITY_GROUP        The capability to update the security group of the specified folder or revision.
  UPDATE_ACCOUNT               The capability to update the account of the specified folder or revision.
  UPDATE_ACCESS_CONTROL_LIST   The capability to update the access control list of the specified folder or revision.
  UPDATE_AUTHOR                The capability to update the author of the specified folder or revision.
  MOVE                         The capability to move the specified folder, document, or shortcut.
  COPY                         The capability to copy the specified folder, document, or shortcut.
  DELETE                       The capability to delete the specified folder, document, or shortcut.
  PROPAGATE_METADATA           The capability to propagate the metadata defaults of the specified folder.
  CHECK_OUT                    The capability to check out the specified document.
  CANCEL_CHECK_OUT             The capability to cancel the check out of the specified document.
  CHECK_IN                     The capability to check in a new revision of the specified document.
  DELETE_REVISION              The capability to delete the specified revision.
  RESTORE_REVISION             The capability to create a new revision of a document as a copy of the specified revision.
  LATEST_REVISION              The capability to test if a document revision is the latest revision.
  FOLLOW                       The capability to follow a folder or document.
  UNFOLLOW                     The capability to unfollow a folder or document.
  APPROVE                      The capability to approve a document.
  SIGN_AND_APPROVE             The capability to sign and approve a document.
  REJECT                       The capability to reject a document.
:::

# REST API Definitions

# Files

## Upload Document

#### POST /documents/wcc/api/v1.1/files/data {#post-documentswccapiv11filesdata}

#### Description

Upload a new file, creating a new document. (CHECKIN_UNIVERSAL)

#### Parameters

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name             Located in     Description                                                                                                                                                                                  Required       Schema
  ---------------- -------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -------------- --------------
  metadataValues   formData       The metadata to associate with this document in JSON format. Some fields are required, some are optional; this can vary based on how the server is configured. These fields are required:\   Yes            string
                                  •dDocType -- The type of document being uploaded\                                                                                                                                                           
                                  •dDocTitle -- The title of the document being uploaded\                                                                                                                                                     
                                  •dRevLabel -- The revision of the document being uploaded\                                                                                                                                                  
                                  •dSecurityGroup -- The security group of the document being uploaded\                                                                                                                                       
                                  Additional fields might be required.\                                                                                                                                                                       
                                  \                                                                                                                                                                                                           
                                  When the FrameworkFolders component is enabled, to upload to a folder include:\                                                                                                                             
                                  •fParentGUID -- The folder GUID of the uploaded file.                                                                                                                                                       

  primaryFile      formData       The primary file for this document.                                                                                                                                                          Yes            file

  alternateFile    formData       The alternate file for this document.                                                                                                                                                        No             file
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Document successfully uploaded.\                                                                
                          Returns Header `Location` which is a URI to get the metadata of the new document.              

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example

Upload a file.

##### Request

> POST .../documents/wcc/api/v1.1/files/data

##### Request Body

FormData Parameters

> metadataValues = {"dDocTitle":"Rest","dSecurityGroup":"Public","dDocType":"Document"}
>
> primaryFile = \[filePath\]

##### HTTP Response

> Status = 201

Header

> Location = .../documents/wcc/api/v1.1/files/ID14006201

#### Example 2

Upload a file to the `Users:weblogic` folder.

##### Request

> POST .../documents/wcc/api/v1.1/files/data

##### Request Body

FormData Parameters

> metadataValues = {"dDocTitle":"Rest","dSecurityGroup":"Public","dDocType":"Document", "fParentGUID":"FLD_USER:weblogic"}
>
> primaryFile = \[filePath\]

##### HTTP Response

> Status = 201

Header

> Location = .../documents/wcc/api/v1.1/folders/files/5D6FC0B59C968931834B5BF2D6DAA52C

## Upload Document Revision

#### POST /documents/wcc/api/v1.1/files/{dDocName}/data {#post-documentswccapiv11filesddocnamedata}

#### Description

Uploads a new document as a new revision of an existing document. (CHECKIN_UNIVERSAL)

#### Parameters

::: table-responsive
  Name             Located in   Description                                                                                                                                                                                Required   Schema
  ---------------- ------------ ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------- --------
  dDocName         path         The dDocName of the document to add the revision to.                                                                                                                                       Yes        string
  metadataValues   formData     The metadata to associate with this revision in JSON format. Fields that are not provided will be inherited. The `dDocName` field may be required depending on the server configuration.   Yes        string
  primaryFile      formData     The primary file for this revision.                                                                                                                                                        Yes        file
  alternateFile    formData     The alternate file for this revision.                                                                                                                                                      No         file
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Revision successfully uploaded.\                                                                
                          Returns Header `Location` which is a URI to get the metadata of the new document.              

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example

Upload a new revision with a new dDocTitle.

##### Request

> POST .../documents/wcc/api/v1.1/files/ID14006201/data

##### Request Body

FormData Parameters

> metadataValues = {"dDocTitle":"Rest Update Title"}
>
> primaryFile = \[filePath\]

##### HTTP Response

> Status = 201

Header

> Location = .../documents/wcc/api/v1.1/files/ID14006201

## Download Document

#### GET /documents/wcc/api/v1.1/files/{dDocName}/data {#get-documentswccapiv11filesddocnamedata}

#### Description

Downloads file content for the specified document as a stream. (GET_FILE)

#### Parameters

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name           Located in     Description                                                                                                                                    Required       Schema
  -------------- -------------- ---------------------------------------------------------------------------------------------------------------------------------------------- -------------- --------------------------
  Range          header         The byte range of the document to download.                                                                                                    No             [Range](#range-requests)

  dDocName       path           The dDocName of the document to download.                                                                                                      Yes            string

  version        query          The version of the document to download. If not provided, the latest released version is assumed.                                              No             string

  rendition      query          The rendition of the document to retrieve. If rendition is not specified, primary rendition of the document is assumed. Allowed renditions:\   No             string
                                •primary\                                                                                                                                                     
                                •alternate\                                                                                                                                                   
                                •web\                                                                                                                                                         
                                •rendition:T\                                                                                                                                                 
                                •custom attachments by name                                                                                                                                   
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the complete stream of the document                                       
  206    Successfully returned the byte range of the document                                            
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  416    The server cannot return the bytes requested                                                    
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-2}

Download a document

##### Request

> GET .../documents/wcc/api/v1.1/files/ID14006201/data

##### Request Body

None

##### HTTP Response

> Status = 200

Body

> stream of file contents as application/octet-stream

## Download Revision of a Document by its dID

#### GET /documents/wcc/api/v1.1/files/.by.did/{dID}/data {#get-documentswccapiv11filesbydiddiddata}

#### Description

Downloads file content for the specified document revision as a stream by providing its dID. (GET_FILE)

#### Parameters

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name           Located in     Description                                                                                                                                    Required       Schema
  -------------- -------------- ---------------------------------------------------------------------------------------------------------------------------------------------- -------------- --------------------------
  Range          header         The byte range of the document to download.                                                                                                    No             [Range](#range-requests)

  dID            path           Revision identifier of the content item.                                                                                                       Yes            string

  rendition      query          The rendition of the document to retrieve. If rendition is not specified, primary rendition of the document is assumed. Allowed renditions:\   No             string
                                •primary\                                                                                                                                                     
                                •alternate\                                                                                                                                                   
                                •web\                                                                                                                                                         
                                •rendition:T\                                                                                                                                                 
                                •custom attachments by name                                                                                                                                   
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the complete stream of the document                                       
  206    Successfully returned the byte range of the document                                            
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  416    The server cannot return the bytes requested                                                    
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-3}

Download a document by dID

##### Request

> GET .../documents/wcc/api/v1.1/files/.by.did/6205/data

##### Request Body

None

##### HTTP Response

> Status = 200

Body

> stream of file contents as application/octet-stream

## Copy Document Revision

#### POST /documents/wcc/api/v1.1/files/{dDocName}/{dID}/copyRevision {#post-documentswccapiv11filesddocnamedidcopyrevision}

#### Description

Create a new document by copying the revision of an existing document. (COPY_REVISION)

#### Parameters

::: table-responsive
  Name       Located in   Description                                                                                                                                                                                                                                                                                                                                   Required   Schema
  ---------- ------------ --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- -----------------------------------------------------------------
  dDocName   path         The dDocName of the document to be used as the source of the copy.                                                                                                                                                                                                                                                                            Yes        string
  dID        path         The dID of the document to be used as the source of the copy.                                                                                                                                                                                                                                                                                 Yes        string
  body       body         The metadata as JSON to apply to the newly created copy. Any field not set in the JSON will be inherited from the source file. If the `dDocName` used does not exist on the server, a new document is created; if the `dDocName` exists, a new revision is created. The `dDocName` field may be required depending on server configuration.   No         [MetadataChangeObjectParameter](#metadatachangeobjectparameter)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Revision copy successfully created.\                                                            
                          Returns Header `Location` which is a URI to get the metadata of the new document.              

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-4}

Create a copy of ID14006201 and dID 6202 as a new content item.

##### Request

> POST .../documents/wcc/api/v1.1/files/ID14006201/6202/copyRevision

##### Request Body

Set `dDocName` and `dDocTitle` on the new content item.

> { "dDocName": "createCopyRevision",
>
> "dDocTitle":"Rest copyRevision" }

##### HTTP Response

> Status = 201

Body

> Location = .../documents/wcc/api/v1.1/files/ID14006205

## Get Document Information

#### GET /documents/wcc/api/v1.1/files/{dDocName} {#get-documentswccapiv11filesddocname}

#### Description

Get the document information for a document. (DOC_INFO_LATESTRELEASE or DOC_INFO_SIMPLE_BYREV)

#### Parameters

::: table-responsive
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name           Located in     Description                                                                                                                                                                                                                                                                                                                                                               Required       Schema
  -------------- -------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -------------- --------------
  dDocName       path           The dDocName of the document to return information.                                                                                                                                                                                                                                                                                                                       Yes            string

  version        query          The version of the document to return information. If not provided, the latest released version is returned.                                                                                                                                                                                                                                                              No             string

  getFullInfo    query          If true, addition arrays like `docInfo`, `wfInfo`, `revisionHistory`, and `fileInfo` may be returned. This can be used with `fields` parameter to further filter what is returned.                                                                                                                                                                                        No             boolean

  fields         query          By default, all fields that would be returned by the DOC_INFO family of services are returned. When provided, the field is a comma separated list of explicit fields returned. When used with `getFullinfo=true`, the format is: `\{array\}.\{fieldsName\}`. For example, `fields=docInfo.dDocName,revisionHistory.dID,wfInfo.*,root.dDocTitle`. There are 2 keywords:\   No             string
                                •`root` refers to the top level of the JSON being returned; fields can be referred to by using either keyword `root.fieldName` or without the keyword as `fieldName`.\                                                                                                                                                                                                                   
                                •The `\*` character means all fields in an array.                                                                                                                                                                                                                                                                                                                                        
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------
  200    Successfully returned the metadata                                                             [MetadataObjectResponse](#MetadataObjectResponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-5}

Get the metadata for CREATECOPYREVISION.

##### Request

> GET .../documents/wcc/api/v1.1/files/CREATECOPYREVISION

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"dDocName": "CREATECOPYREVISION",
"dID": 6203,
"dDocType": "Document",
"dDocTitle": "Rest copyRevision",
"dRevLabel": "1",
"dSecurityGroup": "Public",
"dDocAuthor": "weblogic",
"dStatus": "RELEASED",
"dOriginalName": "exif-xmp.jpg",
"dFormat": "image/jpeg",
"dFileSize": 129756,
"dDocCreator": "weblogic",
"dDocLastModifier": "weblogic",
"dDocOwner": "weblogic",
"dIndexedID": 6203,
"dIsWebFormat": false,
"dCharacterSet": null,
"dRendition1": null,
"dPublishType": null,
"dRendition2": null,
"dDocClass": null,
"dInDate": "2024-05-21 16:08:00Z",
"dRevRank": "0",
"dDocID": 6405,
"dWebExtension": "jpg",
"dCheckoutUser": null,
"dLocation": null,
"dIsPrimary": false,
"dExtension": "jpg",
"dReleaseState": "Y",
"dProcessingState": "Y",
"dWorkflowState": null,
"dIndexerState": null,
"dReleaseDate": "2024-05-21 16:08:00Z",
"dFlag1": null,
"dCreateDate": "2024-05-21 16:08:00Z",
"dOutDate": null,
"dRevClassID": 6203,
"dLanguage": null,
"xExternalDataSet": null,
"dRevisionID": 1,
"dIsCheckedOut": false,
"dDocAccount": null,
"dPublishState": null,
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Get Document Information by dID

#### GET /documents/wcc/api/v1.1/files/.by.did/{dID} {#get-documentswccapiv11filesbydiddid}

#### Description

Get the document information for a document by dID. (DOC_INFO_SIMPLE_BYREV)

#### Parameters

::: table-responsive
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name           Located in     Description                                                                                                                                                                                                                                                                                                                                                               Required       Schema
  -------------- -------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -------------- --------------
  dID            path           The dID/Revision identifier of the document.                                                                                                                                                                                                                                                                                                                              Yes            string

  getFullInfo    query          If true, addition arrays like `docInfo`, `wfInfo`, `revisionHistory`, and `fileInfo` may be returned. This can be used with `fields` parameter to further filter what is returned.                                                                                                                                                                                        No             boolean

  fields         query          By default, all fields that would be returned by the DOC_INFO family of services are returned. When provided, the field is a comma separated list of explicit fields returned. When used with `getFullinfo=true`, the format is: `\{array\}.\{fieldsName\}`. For example, `fields=docInfo.dDocName,revisionHistory.dID,wfInfo.*,root.dDocTitle`. There are 2 keywords:\   No             string
                                •`root` refers to the top level of the JSON being returned; fields can be referred to by using either keyword `root.fieldName` or without the keyword as `fieldName`.\                                                                                                                                                                                                                   
                                •The `\*` character means all fields in an array.                                                                                                                                                                                                                                                                                                                                        
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------
  200    Successfully returned the metadata                                                             [MetadataObjectResponse](#metadataobjectresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-6}

Get the metadata for content item with dID 6203.

##### Request

> GET .../documents/wcc/api/v1.1/files/.by.did/6203

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
 "dDocName": "CREATECOPYREVISION",
"dID": 6203,
"dDocType": "Document",
"dDocTitle": "Rest copyRevision",
"dRevLabel": "1",
"dSecurityGroup": "Public",
"dDocAuthor": "weblogic",
"dStatus": "RELEASED",
"dOriginalName": "exif-xmp.jpg",
"dFormat": "image/jpeg",
"dFileSize": 129756,
"dDocCreator": "weblogic",
"dDocLastModifier": "weblogic",
"dDocOwner": "weblogic",
"dIndexedID": 6203,
"dIsWebFormat": false,
"dCharacterSet": null,
"dRendition1": null,
"dPublishType": null,
"dRendition2": null,
"dDocClass": null,
"dInDate": "2024-05-21 16:08:00Z",
"dRevRank": "0",
"dDocID": 6405,
"dWebExtension": "jpg",
"dCheckoutUser": null,
"dLocation": null,
"dIsPrimary": false,
"dExtension": "jpg",
"dReleaseState": "Y",
"dProcessingState": "Y",
"dWorkflowState": null,
"dIndexerState": null,
"dReleaseDate": "2024-05-21 16:08:00Z",
"dFlag1": null,
"dCreateDate": "2024-05-21 16:08:00Z",
"dOutDate": null,
"dRevClassID": 6203,
"dLanguage": null,
"xExternalDataSet": null,
"dRevisionID": 1,
"dIsCheckedOut": false,
"dDocAccount": null,
"dPublishState": null,
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Delete Document

#### DELETE /documents/wcc/api/v1.1/files/{dDocName} {#delete-documentswccapiv11filesddocname}

#### Description

Deletes the document and all its revisions. (DELETE_DOC)

#### Parameters

::: table-responsive
  Name       Located in   Description                                                                                                                             Required   Schema
  ---------- ------------ --------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocName   path         The dDocName of the document to be deleted.                                                                                             Yes        string
  version    query        The version number of the document to delete. If the version is not specified, the content item and all its versions will be deleted.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted document                                                                   
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-7}

Delete the content item with dDocName CREATECOPYREVISION.

##### Request

> DELETE .../documents/wcc/api/v1.1/files/CREATECOPYREVISION

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Delete Revision of a Document by its dID

#### DELETE /documents/wcc/api/v1.1/files/.by.did/{dID} {#delete-documentswccapiv11filesbydiddid}

#### Description

Deletes a document revision. (DELETE_REV_EX)

#### Parameters

::: table-responsive
  Name   Located in   Description                                         Required   Schema
  ------ ------------ --------------------------------------------------- ---------- --------
  dID    path         The dID (revision) of the document to be deleted.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted the document.                                                              
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-8}

Delete the content item with dID 6206.

##### Request

> DELETE .../documents/wcc/api/v1.1/files/.by.did/6206

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Get Document Version Information

#### GET /documents/wcc/api/v1.1/files/{dDocName}/versions {#get-documentswccapiv11filesddocnameversions}

#### Description

Get all the revision information of a document. (DOC_INFO_BY_NAME)

#### Parameters

::: table-responsive
  Name       Located in   Description                                                                                                                                                                                                               Required   Schema
  ---------- ------------ ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocName   path         The dDocName of the document to return revision information.                                                                                                                                                              Yes        string
  fields     query        This parameter is optional. When it is not provided, all fields that would be returned by the DOC_INFO family of services are returned. When provided, the field is a comma separated list of explicit fields returned.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -------------------------------------------------------
  200    Successfully returned the metadata                                                             [MetadataVersionsResponse](#metadataversionsresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-9}

Get the metadata for all the versions of the content item with dDocName ID14006204.

##### Request

> GET .../documents/wcc/api/v1.1/files/ID14006204/versions

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy {
"dDocName": "ID14006204",    
"dRevLabelLatest": "2",
"count": 2,
"items": [
    {
        "dDocName": "ID14006204",
        "dID": 6205,
        "dDocType": "Document",
        "dDocTitle": "Rest Update Title",
        "dRevLabel": "2",
        "dSecurityGroup": "Public",
        "dDocAuthor": "weblogic",
        "dStatus": "RELEASED",
        "dOriginalName": "exif-xmp.jpg",
        "dFormat": "image/jpeg",
        "dFileSize": 129756,
        "dDocCreator": "weblogic",
        "dDocLastModifier": "weblogic",
        "dDocOwner": "weblogic",
        "dIndexedID": 6205,
        "xIdcProfile": null,
        "dIsWebFormat": false,
        "xIsACLReadOnlyOnUI": 0,
        "dCharacterSet": null,
        "xComments": null,
        "xDecimal": null,
        "dRendition1": null,
        "dPublishType": null,
        "dRendition2": null,
        "dDocClass": null,
        "xReqText": null,
        "dInDate": "2024-05-21 19:19:14Z",
        "dRevRank": "0",
        "dDocID": 6409,
        "dMessage": null,
        "dWebExtension": "jpg",
        "dCheckoutUser": null,
        "dLocation": null,
        "xPartitionId": null,
        "xWebFlag": null,
        "dIsPrimary": false,
        "dExtension": "jpg",
        "xStorageRule": "DispByContentId",
        "dReleaseState": "Y",
        "dProcessingState": "Y",
        "dWorkflowState": null,
        "dIndexerState": null,
        "xLongText": null,
        "xTest": 0,
        "xOptionList": null,
        "xAnnotationDetails": 0,
        "xMemo": "Test memo",
        "dReleaseDate": "2024-05-21 19:24:05Z",
        "dDocFunction": null,
        "dFlag1": null,
        "dCreateDate": "2024-05-21 19:19:14Z",
        "dOutDate": null,
        "dRevClassID": 6204,
        "dLanguage": null,
        "xExternalDataSet": null,
        "dRevisionID": 2,
        "dIsCheckedOut": false,
        "dDocAccount": null,
        "dPublishState": null,
        "xDate": null,
        "xLibraryGUID": null
    },
    {
        "dDocName": "BB14006204",
        "dID": 6204,
        "dDocType": "Document",
        "dDocTitle": "Rest",
        "dRevLabel": "1",
        "dSecurityGroup": "Public",
        "dDocAuthor": "weblogic",
        "dStatus": "RELEASED",
        "dOriginalName": "Mugs.jpg",
        "dFormat": "image/jpeg",
        "dFileSize": 53834,
        "dDocCreator": "weblogic",
        "dDocLastModifier": "weblogic",
        "dDocOwner": "weblogic",
        "dIndexedID": 6205,
        "xIdcProfile": null,
        "dIsWebFormat": false,
        "xIsACLReadOnlyOnUI": 0,
        "dCharacterSet": null,
        "xComments": null,
        "xDecimal": null,
        "dRendition1": null,
        "dPublishType": null,
        "dRendition2": null,
        "dDocClass": null,
        "xReqText": null,
        "dInDate": "2024-05-21 19:18:39Z",
        "dRevRank": "1",
        "dDocID": 6407,
        "dMessage": null,
        "dWebExtension": "jpg",
        "dCheckoutUser": null,
        "dLocation": null,
        "xPartitionId": null,
        "xWebFlag": null,
        "dIsPrimary": false,
        "dExtension": "jpg",
        "xStorageRule": "DispByContentId",
        "dReleaseState": "O",
        "dProcessingState": "Y",
        "dWorkflowState": null,
        "dIndexerState": null,
        "xLongText": null,
        "xTest": 0,
        "xOptionList": null,
        "xAnnotationDetails": 0,
        "xMemo": "Test memo",
        "dReleaseDate": "2024-05-21 19:18:41Z",
        "dDocFunction": null,
        "dFlag1": null,
        "dCreateDate": "2024-05-21 19:18:39Z",
        "dOutDate": null,
        "dRevClassID": 6204,
        "dLanguage": null,
        "xExternalDataSet": null,
        "dRevisionID": 1,
        "dIsCheckedOut": false,
        "dDocAccount": null,
        "dPublishState": null,
        "xDate": null,
        "xLibraryGUID": null
    }
    ]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Document Search

#### GET /documents/wcc/api/v1.1/files/search/items {#get-documentswccapiv11filessearchitems}

#### Description

Search for documents in the server. (GET_SEARCH_RESULTS)

#### Parameters

::: table-responsive
  Name      Located in   Description                                                                                                                                                      Required   Schema
  --------- ------------ ---------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- ---------
  q         query        The search query to search for content items. See [Using Universal Query Format](#using-the-universal-query-format)                                              No         string
  fields    query        The names of metadata fields to be returned for each content item . There is a set of fields that are always returned whether or not this parameter is passed.   No         string
  orderBy   query        The sort field and sort order which will be used to arrange the filtered content items.                                                                          No         string
  limit     query        The maximum number of items listed per page.                                                                                                                     No         integer
  offset    query        Specifies the point from which items are listed for the response.                                                                                                No         integer
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -------------------------------------------------
  200    Successfully returned the search results                                                       [SearchResultsResponse](#searchresultsresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-10}

Search for all files in the `Public` security group.

##### Request

> GET .../documents/wcc/api/v1.1/files/search/items

##### Request Body

> query parameter q = dSecurityGroup \<matches\> \`Public\`

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"totalResults": 5,
"query": "dSecurityGroup <matches> \`Public\`",
"offset": 0,
"limit": 20,
"count": 5,
"hasMore": "false",
"pageNumber": 1,
"items": [
    {
        "dDocName": "ID14006204",
        "dID": 6205,
        "dDocType": "Document",
        "dDocTitle": "Rest Update Title",
        "dRevLabel": "2",
        "dSecurityGroup": "Public",
        "dDocAuthor": "weblogic",
        "dStatus": "Released",
        "dOriginalName": "exif-xmp.jpg",
        "dFormat": "image/jpeg",
        "dFileSize": 129756,
        "dDocCreatedDate": "2024-05-21 19:18:39Z",
        "dDocCreator": "weblogic",
        "dDocLastModifiedDate": "2024-05-21 19:24:04Z",
        "dDocLastModifier": "weblogic",
        "dIndexedID": 6205,
        "dDocOwner": "weblogic",
        "dRendition1": null,
        "dPublishType": null,
        "URL": "/cs/groups/public/documents/document/yje0/mda2/~edisp/id14006204.jpg",
        "dRendition2": null,
        "VaultFileSize": 129756,
        "dDocClass": null,
        "dInDate": "2024-05-21 19:19:14Z",
        "dWebExtension": "jpg",
        "xWebFlag": null,
        "dExtension": "jpg",
        "xStorageRule": "DispByContentId",
        "dDocFunction": null,
        "dCreateDate": "2024-05-21 19:19:14Z",
        "dOutDate": null,
        "dRevClassID": 6204,
        "xExternalDataSet": null,
        "dRevisionID": 2,
        "dDocAccount": null,
    },
    {
        "dDocName": "ID14006201",
        "dID": 6202,
        "dDocType": "Document",
        "dDocTitle": "Rest Update Title",
        "dRevLabel": "2",
        "dSecurityGroup": "Public",
        "dDocAuthor": "weblogic",
        "dStatus": "Released",
        "dOriginalName": "exif-xmp.jpg",
        "dFormat": "image/jpeg",
        "dFileSize": 129756,
        "dDocCreatedDate": "2024-05-21 14:54:46Z",
        "dDocCreator": "weblogic",
        "dDocLastModifiedDate": "2024-05-21 15:49:16Z",
        "dDocLastModifier": "weblogic",
        "dIndexedID": 6202,
        "dDocOwner": "weblogic",
        "dRendition1": null,
        "dPublishType": null,
        "URL": "/cs/groups/public/documents/document/yje0/mda2/~edisp/id14006201.jpg",
        "dRendition2": null,
        "VaultFileSize": 129756,
        "dDocClass": null,
        "dInDate": "2024-05-21 15:49:16Z",
        "dWebExtension": "jpg",
        "xWebFlag": null,
        "dExtension": "jpg",
        "xStorageRule": "DispByContentId",
        "dDocFunction": null,
        "dCreateDate": "2024-05-21 15:49:16Z",
        "dOutDate": null,
        "dRevClassID": 6201,
        "xExternalDataSet": null,
        "dRevisionID": 2,
        "dDocAccount": null,
    },
    {
        "dDocName": "ID14006002",
        "dID": 6002,
        "dDocType": "Document",
        "dDocTitle": "Rest",
        "dRevLabel": "1",
        "dSecurityGroup": "Public",
        "dDocAuthor": "weblogic",
        "dStatus": "Released",
        "dOriginalName": "Mugs.jpg",
        "dFormat": "image/jpeg",
        "dFileSize": 53834,
        "dDocCreatedDate": "2024-05-13 16:53:42Z",
        "dDocCreator": "weblogic",
        "dDocLastModifiedDate": "2024-05-13 16:53:42Z",
        "dDocLastModifier": "weblogic",
        "dIndexedID": 6002,
        "dDocOwner": "weblogic",
        "dRendition1": null,
        "dPublishType": null,
        "URL": "/cs/groups/public/documents/document/yje0/mda2/~edisp/id14006002.jpg",
        "dRendition2": null,
        "VaultFileSize": 53834,
        "dDocClass": null,
        "dInDate": "2024-05-13 16:53:42Z",
        "dWebExtension": "jpg",
        "xPartitionId": null,
        "xWebFlag": null,
        "dExtension": "jpg",
        "dDocFunction": null,
        "dCreateDate": "2024-05-13 16:53:42Z",
        "dOutDate": null,
        "dRevClassID": 6002,
        "xExternalDataSet": null,
        "dRevisionID": 1,
        "dDocAccount": null,
    },
    {
        "dDocName": "ID14005802",
        "dID": 5802,
        "dDocType": "Document",
        "dDocTitle": "file",
        "dRevLabel": "1",
        "dSecurityGroup": "Public",
        "dDocAuthor": "UserB",
        "dStatus": "Released",
        "dOriginalName": "Mugs.jpg",
        "dFormat": "image/jpeg",
        "dFileSize": 53834,
        "dDocCreatedDate": "2024-05-09 16:25:20Z",
        "dDocCreator": "UserB",
        "dDocLastModifiedDate": "2024-05-09 16:25:20Z",
        "dDocLastModifier": "UserB",
        "dIndexedID": 5802,
        "dDocOwner": "UserB",
        "dRendition1": null,
        "dPublishType": null,
        "URL": "/cs/groups/public/documents/document/yje0/mda1/~edisp/id14005802.jpg",
        "dRendition2": null,
        "VaultFileSize": 53834,
        "dDocClass": null,
        "dInDate": "2024-05-09 16:24:00Z",
        "dWebExtension": "jpg",
        "xWebFlag": null,
        "dExtension": "jpg",
        "xStorageRule": "DispByContentId",
        "dDocFunction": null,
        "dCreateDate": "2024-05-09 16:25:20Z",
        "dOutDate": null,
        "dRevClassID": 5802,
        "xExternalDataSet": null,
        "dRevisionID": 1,
        "dDocAccount": null,
    },
    {
        "dDocName": "1715255331863",
        "dID": 5801,
        "dDocType": "Document",
        "dDocTitle": "Copy Title",
        "dRevLabel": "1",
        "dSecurityGroup": "Public",
        "dDocAuthor": "weblogic",
        "dStatus": "Released",
        "dOriginalName": "CSASampleImage.jpg",
        "dFormat": "image/jpeg",
        "dFileSize": 25382,
        "dDocCreatedDate": "2024-05-09 11:48:53Z",
        "dDocCreator": "weblogic",
        "dDocLastModifiedDate": "2024-05-09 11:48:53Z",
        "dDocLastModifier": "weblogic",
        "dIndexedID": 5801,
        "dDocOwner": "weblogic",
        "dRendition1": null,
        "dPublishType": null,
        "URL": "/cs/groups/public/documents/document/mju1/mzmx/~edisp/1715255331863.jpg",
        "dRendition2": null,
        "VaultFileSize": 25382,
        "dDocClass": null,
        "dInDate": "2024-05-09 11:48:53Z",
        "dWebExtension": "jpg",
        "xPartitionId": null,
        "xWebFlag": null,
        "dExtension": "jpg",
        "dDocFunction": null,
        "dCreateDate": "2024-05-09 11:48:53Z",
        "dOutDate": null,
        "dRevClassID": 5801,
        "xExternalDataSet": null,
        "dRevisionID": 1,
        "dDocAccount": null,
    }
],
"sortOrder": "dInDate:Desc"
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Resubmit a document

#### POST /documents/wcc/api/v1.1/files/{dDocName}/resubmitConversion {#post-documentswccapiv11filesddocnameresubmitconversion}

#### Description

Resubmit a document that failed conversion. (RESUBMIT_FOR_CONVERSION)

#### Parameters

::: table-responsive
  Name             Located in   Description                                                                                                                                                     Required   Schema
  ---------------- ------------ --------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- ---------
  dDocName         path         The dDocName of the document to be resubmitted.                                                                                                                 Yes        string
  version          query        The version number of the document to be resubmitted. If the version is not specified, the latest version will be resubmitted.                                  No         string
  alwaysResubmit   query        By default, only files in a failed conversion state can be resubmitted. Setting this to true allows files that were successfully converted to be resubmitted.   No         boolean
:::

#### Responses

::: table-responsive
  Code   Description                                                                                                  Schema
  ------ ------------------------------------------------------------------------------------------------------------ -----------------------------------------------------------------
  202    The file has been resubmitted.                                                                                
  401    Unauthorized                                                                                                  
  409    The file is not in a failed converted state, and could be resubmitted with the `alwaysResubmit` parameter.   [ResubmitConflictErrorResponse](#resubmitconflicterrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request.                [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-11}

Resubmit content item ID14006204.

##### Request

> POST .../documents/wcc/api/v1.1/files/ID14006204/resubmitConversion

##### Request Body

None

##### HTTP Response

> Status = 202

Body

None.

## Resubmit a document by dID

#### POST /documents/wcc/api/v1.1/files/.by.did/{dID}/resubmitConversion {#post-documentswccapiv11filesbydiddidresubmitconversion}

#### Description

Resubmit a document that failed conversion. (RESUBMIT_FOR_CONVERSION)

#### Parameters

::: table-responsive
  Name   Located in   Description                Required   Schema
  ------ ------------ -------------------------- ---------- --------
  dID    path         The dID of the document.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                                 Schema
  ------ ----------------------------------------------------------------------------------------------------------- -----------------------------------------------------------------
  202    The file has been resubmitted                                                                                
  401    Unauthorized                                                                                                 
  409    The file is not in a failed converted state, and could be resubmitted with the `alwaysResubmit` parameter   [ResubmitConflictErrorResponse](#resubmitconflicterrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request                [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-12}

Resubmit revision 6205 (but it has not failed conversion).

##### Request

> POST .../documents/wcc/api/v1.1/files/.by.did/6205/resubmitConversion

##### Request Body

None

##### HTTP Response

> Status = 409

Body

> {
>
> "type": "https://www.rfc-editor.org/rfc/rfc9110.html#name-409-conflict",
>
> "title": "Resubmit Not Required",
>
> "detail": "The content item is not in a failed conversion state.",
>
> "o:errorCode": -1
>
> }

## Move a Document to a Different Storage Tier

#### POST /documents/wcc/api/v1.1/files/{dDocName}/storage/.updateStorageTier {#post-documentswccapiv11filesddocnamestorageupdatestoragetier}

#### Description

Move a document revision to a different OCI Object Storage tier. (OCI_UPDATE_REV_STORAGE_TIER)

Refer to [Using WebCenter Content with Low-Cost Object Storage Tiers](https://docs.oracle.com/en/cloud/paas/webcenter-content/wc-objectstorage-integration/index.html#managing-documents-in-oci-object-storage-tiers) for more information on managing documents in OCI Object Storage tiers.

#### Parameters

::: table-responsive
  Name          Located in   Description                                                                                                               Required   Schema
  ------------- ------------ ------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocName      path         The dDocName of the document to be moved.                                                                                 Yes        string
  storageTier   query        The storage tier the document should be moved to. Supported tiers include 'Standard', 'InfrequentAccess' and 'Archive'.   Yes        string
  version       query        The version of the document to move. If the version is not specified, the latest version will be moved.                   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    The document was moved to the specified storage tier                                            
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The document could not be found                                                                 
  409    The document could not be moved to the specified storage tier                                   
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-13}

Move the latest revision for a content item with Content ID "ID14006204" to the "Infrequent Access" storage tier.

##### Request

> POST .../documents/wcc/api/v1.1/files/ID14006204/storage/.updateStorageTier?storageTier=InfrequentAccess

##### Request Body

None.

##### HTTP Response

> Status = 204

## Move a Document to a Different Storage Tier by dID

#### POST /documents/wcc/api/v1.1/files/.by.did/{dID}/storage/.updateStorageTier {#post-documentswccapiv11filesbydiddidstorageupdatestoragetier}

#### Description

Move a document revision to a different OCI Object Storage tier. (OCI_UPDATE_REV_STORAGE_TIER)

Refer to [Using WebCenter Content with Low-Cost Object Storage Tiers](https://docs.oracle.com/en/cloud/paas/webcenter-content/wc-objectstorage-integration/index.html#managing-documents-in-oci-object-storage-tiers) for more information on managing documents in OCI Object Storage tiers.

#### Parameters

::: table-responsive
  Name          Located in   Description                                                                                                               Required   Schema
  ------------- ------------ ------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dID           path         The dID of the document revision.                                                                                         Yes        string
  storageTier   query        The storage tier the document should be moved to. Supported tiers include 'Standard', 'InfrequentAccess' and 'Archive'.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    The document was moved to the specified storage tier                                            
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The document could not be found                                                                 
  409    The document could not be moved to the specified storage tier                                   
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-14}

Move document revision with dID "5000001" to the "Archive" storage tier.

##### Request

> POST .../documents/wcc/api/v1.1/files/.by.dID/5000001/storage/.updateStorageTier?storageTier=Archive

##### Request Body

None.

##### HTTP Response

> Status = 204

## Restore a Document Located in the Archive Storage Tier

#### POST /documents/wcc/api/v1.1/files/{dDocName}/storage/.restoreFromArchive {#post-documentswccapiv11filesddocnamestoragerestorefromarchive}

#### Description

Restore the primary, web and alternate renditions of a document located in the Archive storage tier in OCI Object Storage. (OCI_RESTORE_REV_FROM_ARCHIVE)

Document restoration may take up to an hour. Once the document renditions are restored, they will be available for accessible for up to 240 hours (by default) before returning to the Archived state.

Refer to [Using WebCenter Content with Low-Cost Object Storage Tiers](https://docs.oracle.com/en/cloud/paas/webcenter-content/wc-objectstorage-integration/index.html#managing-documents-in-oci-object-storage-tiers) for more information on managing documents in OCI Object Storage tiers.

#### Parameters

::: table-responsive
  Name       Located in   Description                                                                                                                                                                      Required   Schema
  ---------- ------------ -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocName   path         The dDocName of the document to restore.                                                                                                                                         Yes        string
  version    query        The version of the document to restore. If the version is not specified, the latest version will be restored.                                                                    No         string
  hours      query        The number of hours the document should available for download after restoration. The minimum is 1 hour, the maximum is 240 hours. If not specified, the default is 240 hours.   No         number
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    The document restore operation was started                                                      
  400    Bad Request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The document could not be found                                                                 
  409    The document could not be restored                                                              
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-15}

Restore the latest version of content item "WCCMEETING01" for 48 hours.

##### Request

> POST .../documents/wcc/api/v1.1/files/WCCMEETING01/storage/.restoreFromArchive?hours=48

##### Request Body

None.

##### HTTP Response

> Status = 204

## Restore a Document Located in the Archive Storage Tier by dID

#### POST /documents/wcc/api/v1.1/files/.by.did/{dID}/storage/.restoreFromArchive {#post-documentswccapiv11filesbydiddidstoragerestorefromarchive}

#### Description

Restore the primary, web and alternate renditions of a document located in the Archive storage tier in OCI Object Storage. (OCI_RESTORE_REV_FROM_ARCHIVE)

Document restoration may take up to an hour. Once the document renditions are restored, they will be available for accessible for up to 240 hours (by default) before returning to the Archived state.

Refer to [Using WebCenter Content with Low-Cost Object Storage Tiers](https://docs.oracle.com/en/cloud/paas/webcenter-content/wc-objectstorage-integration/index.html#managing-documents-in-oci-object-storage-tiers) for more information on managing documents in OCI Object Storage tiers.

#### Parameters

::: table-responsive
  Name    Located in   Description                                                                                                                                                                      Required   Schema
  ------- ------------ -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dID     path         The dID of the document revision to restore.                                                                                                                                     Yes        string
  hours   query        The number of hours the document should available for download after restoration. The minimum is 1 hour, the maximum is 240 hours. If not specified, the default is 240 hours.   No         number
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    The document restore operation was started                                                      
  400    Bad Request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The document could not be found                                                                 
  409    The document could not be restored                                                              
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-16}

Restore revision with dID "5000001" for 48 hours

##### Request

> POST .../documents/wcc/api/v1.1/files/.by.did/5000001/storage/.restoreFromArchive?hours=48

##### Request Body

None.

##### HTTP Response

> Status = 204

## Get Work in Progress

#### GET /documents/wcc/api/v1.1/files/workInProgress/items {#get-documentswccapiv11filesworkinprogressitems}

#### Description

Returns a list of items in GENWWW or DONE status and not present in a workflow. (WORK_IN_PROGRESS)

#### Parameters

::: table-responsive
  Name      Located in   Description                                                                                                                                                    Required   Schema
  --------- ------------ -------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- ---------
  fields    query        The names of metadata fields to be returned for each content item. There is a set of fields that is always returned whether or not this parameter is passed.   No         string
  orderBy   query        The sort field and sort order which will be used to arrange the filtered content items.                                                                        No         string
  limit     query        The maximum number of items listed per page.                                                                                                                   No         integer
  offset    query        Specifies the point from which items are listed for the response.                                                                                              No         integer
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------
  200    Successfully returned work in progress.                                                        [WorkInProgressResponse](#workinprogressresponse)
  204    There is no work in progress.                                                                   
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

## Update Document

#### PUT /documents/wcc/api/v1.1/files/{dDocName} {#put-documentswccapiv11filesddocname}

#### Description

Updates the document. Update the document metadata or update a metadate file to a different file (the file can only be updated once from a metadata file). (UPDATE_DOCINFO_BYFORM & UPDATE_DOCINFO)

#### Parameters

::: table-responsive
  Name                      Located in   Description                                                                                                                        Required   Schema
  ------------------------- ------------ ---------------------------------------------------------------------------------------------------------------------------------- ---------- -----------------------------------------------------------------
  dDocName                  path         The dDocName of the document to be updated.                                                                                        Yes        string
  version                   query        The version of the document to be updated. If not provided, the latest version is updated.                                         No         string
  createPrimaryMetaFile     query        When true, a primary metafile is checked in and the `primaryFile` parameter is ignored.                                            No         boolean
  createAlternateMetaFile   query        When true, an alternate metafile is checked in and the `alternateFile` parameter is ignored.                                       No         boolean
  metadataValues            formData     The metadata as JSON to update. Any field set in the JSON set on the document; fields which are not present will not be updated.   Yes        [MetadataChangeObjectParameter](#metadatachangeobjectparameter)
  primaryFile               formData     The new primary file for this document (can only update from a metadata file).                                                     No         file
  alternateFile             formData     The new alternate file for this document (can only update from a metadata file).                                                   No         file
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204                     Document successfully updated.\                                                                 
                          Returns Header `Location` which is a URI to get the metadata of the updated document.          

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-17}

Update the latest released version of the METADATA document to set the field `xComments` to 'my update'.

##### Request

> PUT .../documents/wcc/api/v1.1/files/METADATA

##### Request Body

FormData Parameters

> metadataValues = {"xComments": "my update"}

##### HTTP Response

> Status = 204

Header

> Location = .../documents/wcc/api/v1.1/files/METADATA

## Update Document by dID

#### PUT /documents/wcc/api/v1.1/files/.by.did/{dID} {#put-documentswccapiv11filesbydiddid}

#### Description

Updates the document. Update the metadata or update a metadate file to a different file (the file can only be updated once from a metadata file). (UPDATE_DOCINFO_BYFORM & UPDATE_DOCINFO)

#### Parameters

::: table-responsive
  Name                      Located in   Description                                                                                                                        Required   Schema
  ------------------------- ------------ ---------------------------------------------------------------------------------------------------------------------------------- ---------- -----------------------------------------------------------------
  dID                       path         The dID of the document to be updated.                                                                                             Yes        string
  createPrimaryMetaFile     query        When true, a primary metafile is checked in and the `primaryFile` parameter is ignored.                                            No         boolean
  createAlternateMetaFile   query        When true, an alternate metafile is checked in and the `alternateFile` parameter is ignored.                                       No         boolean
  metadataValues            formData     The metadata as JSON to update. Any field set in the JSON set on the document; fields which are not present will not be updated.   Yes        [MetadataChangeObjectParameter](#metadatachangeobjectparameter)
  primaryFile               formData     The new primary file for this document (can only update from a metadata file).                                                     No         file
  alternateFile             formData     The new alternate file for this document (can only update from a metadata file).                                                   No         file
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204                     Document successfully updated.\                                                                 
                          Returns Header `Location` which is a URI to get the metadata of the updated document.          

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-18}

Assume the file revision 7603 exists with a metadata file. This update will update the file and set the field `xComments` to 'changeFile'.

##### Request

> PUT .../documents/wcc/api/v1.1/files/.by.did/7603

##### Request Body

FormData Parameters

> metadataValues = {"xComments": "changeFile"}
>
> primaryFile = \[filePath\]

##### HTTP Response

> Status = 204

Header

> Location = .../documents/wcc/api/v1.1/files/ID14007635

## Checkout a Document

#### POST /documents/wcc/api/v1.1/files/{dDocName}/.checkout {#post-documentswccapiv11filesddocnamecheckout}

#### Description

Checkout the latest version of a file. (CHECKOUT_BY_NAME)

#### Parameters

::: table-responsive
  Name       Located in   Description                                 Required   Schema
  ---------- ------------ ------------------------------------------- ---------- --------
  dDocName   path         The dDocName of the document to checkout.   Yes        string
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204                     Document successfully checked out.\                                                             
                          Returns Header `Location` which is a URI to get the metadata of the document.                  

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-19}

Checkout content item ID14045102

##### Request

> POST .../documents/wcc/api/v1.1/files/ID14045102/.checkout

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

Header

> Location = .../documents/wcc/api/v1.1/files/ID14045102

## Reverse the Checkout of a Document

#### POST /documents/wcc/api/v1.1/files/{dDocName}/.undocheckout {#post-documentswccapiv11filesddocnameundocheckout}

#### Description

Reverse the checkout of a file. (UNDO_CHECKOUT_BY_NAME)

#### Parameters

::: table-responsive
  Name       Located in   Description                                             Required   Schema
  ---------- ------------ ------------------------------------------------------- ---------- --------
  dDocName   path         The dDocName of the document to reverse the checkout.   Yes        string
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204                     Successfully reversed the checked out.\                                                         
                          Returns Header `Location` which is a URI to get the metadata of the document.                  

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-20}

Reverse the checkout content item ID14045102.

##### Request

> POST .../documents/wcc/api/v1.1/files/ID14045102/.undocheckout

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

Header

> Location = .../documents/wcc/api/v1.1/files/ID14045102

## Get the Capabilities of a Document

#### GET /documents/wcc/api/v1.1/files/{dDocName}/capabilities {#get-documentswccapiv11filesddocnamecapabilities}

#### Description

Get the capabilities the user has on a document. (FLD_TEST_USER_CAPABILITIES)

#### Parameters

::: table-responsive
  Name                 Located in   Description                                                                                                       Required   Schema
  -------------------- ------------ ----------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocName             path         The dDocName of the document to test.                                                                             Yes        string
  testedCapabilities   query        A comma separated list of capabilities to be tested. See the [Supported Capabilities](#supported-capabilities).   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully tested the document.                                                              [CapabilitiesResponse](#capabilitiesresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-21}

Test if the user can MOVE,COPY and DELETE the content item with dDocName ID14010605.

##### Request

> GET .../documents/wcc/api/v1.1/files/ID14010605/capabilities

##### Request Body

query parameter: testedCapabilities = MOVE,COPY,DELETE

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"count": 3,
"items": [
    {
        "capabiltyName": "MOVE",
        "capabilityValue": -1
    },
    {
        "capabiltyName": "COPY",
        "capabilityValue": -1
    },
    {
        "capabiltyName": "DELETE",
        "capabilityValue": -1
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Get the Capabilities of a Document by dID

#### GET /documents/wcc/api/v1.1/files/.by.did/{dID}/capabilities {#get-documentswccapiv11filesbydiddidcapabilities}

#### Description

Get the capabilities the user has on a revision of a document. (FLD_TEST_USER_CAPABILITIES)

#### Parameters

::: table-responsive
  Name                 Located in   Description                                                                                                       Required   Schema
  -------------------- ------------ ----------------------------------------------------------------------------------------------------------------- ---------- --------
  dID                  path         The dID of the revision to test.                                                                                  Yes        string
  testedCapabilities   query        A comma separated list of capabilities to be tested. See the [Supported Capabilities](#supported-capabilities).   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully tested the revision.                                                              [CapabilitiesResponse](#capabilitiesresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-22}

Test if the user can MOVE, COPY, and DELETE the content item with dID 10805.

##### Request

> GET .../documents/wcc/api/v1.1/files/.by.did/10805/capabilities

##### Request Body

query parameter: testedCapabilities = MOVE,COPY,DELETE

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"count": 3,
"items": [
    {
        "capabiltyName": "MOVE",
        "capabilityValue": -1
    },
    {
        "capabiltyName": "COPY",
        "capabilityValue": -1
    },
    {
        "capabiltyName": "DELETE",
        "capabilityValue": -1
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Get Content Item workflow information

#### GET /documents/wcc/api/v1.1/files/{dDocName}/workflow {#get-documentswccapiv11filesddocnameworkflow}

#### Description

Workflow information based on the dDocName of a document in the workflow. (GET_WORKFLOW_INFO_BYNAME)

#### Parameters

::: table-responsive
  Name       Located in   Description                                    Required   Schema
  ---------- ------------ ---------------------------------------------- ---------- --------
  dDocName   path         The name of a document in an active workflow   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Workflow information based on the dDocName of a document in the workflow.                      [WorkflowInfoResponse](#workflowinforesponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-23}

Get information on the workflow state for content item `ID45654`.

##### Request

> GET ...documents/wcc/api/v1.1/files/ID45654/workflow

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"dWfName": "criteriawf1",
"dWfStepID": 4,
"remainingStepUsers": "weblogic",
"doc_info": {
    "dDocName": "ID45654",
    "dID": 8446,
    "dDocType": "Document",
    "dDocTitle": "workflow demo",
    "dRevLabel": "1",
    "dSecurityGroup": "Public",
    "dDocAuthor": "weblogic",
    "dStatus": "REVIEW",
    "dOriginalName": "Desert.jpg",
    "dFormat": "image/jpeg",
    "dFileSize": 845941,
    "dVitalState": "Y",
    "xReportType": null,
    "xRecordSupersededDate": null,
    "xDeleteApproveDate": null,
    "xIsEditable": 1,
    "xHasFixedClone": 0,
    "xIsACLReadOnlyOnUI": 0,
    "dCharacterSet": null,
    "xComments": "criteriawf1",
    "xRelatedContentTriggerDate": null,
    "dRendition1": null,
    "xNewRevisionDate": "5/22/24 9:34 AM",
    "dRendition2": null,
    "xRecordCancelledDate": null,
    "xRecordCutoffDate": null,
    "dLastModifiedDate": "5/22/24 9:34 AM",
    "dRecordState": null,
    "dRmaSegmentID": 3,
    "dMessage": null,
    "dWebExtension": "jpg",
    "dCheckoutUser": null,
    "xClassifiedMarkings": null,
    "xRecordExpirationDate": null,
    "xVitalPeriod": 0,
    "dIsPrimary": false,
    "dExtension": "jpg",
    "xStorageRule": "DispByContentId",
    "dProcessingState": "Y",
    "xIsFrozen": 0,
    "dWorkflowState": "W",
    "dIndexerState": null,
    "xIsRevisionable": 1,
    "xSuperSupersededDate": null,
    "dReleaseDate": null,
    "xRecordActivationDate": null,
    "dOutDate": null,
    "dRevClassID": 7846,
    "xIsVital": 0,
    "dLanguage": null,
    "xRelatedContentList": null,
    "dIsCheckedOut": false,
    "xIsDeletable": 1,
    "xSupersededContent": null,
    "xIdcProfile": null,
    "dIsWebFormat": false,
    "xFreezeID": 0,
    "xIsSubjectToAudit": 0,
    "dRmaProcessState": "Y",
    "xRecordDestroyDate": null,
    "xFreezeReason": null,
    "dPublishType": null,
    "xVitalPeriodUnits": null,
    "xClbraUserList": null,
    "xSource": null,
    "xIsRecord": 0,
    "xRecordFilingDate": "5/22/24 9:32 AM",
    "xFolderID": null,
    "dInDate": "2024-05-22 16:32:00Z",
    "dRevRank": "0",
    "xRecordReviewDate": null,
    "dDocID": 13691,
    "xClbraAliasList": null,
    "dLocation": null,
    "xSupplementalMarkings": null,
    "xCpdIsTemplateEnabled": 0,
    "xPartitionId": null,
    "xWebFlag": null,
    "dReleaseState": "E",
    "xCpdIsLocked": 0,
    "xRecordObsoleteDate": null,
    "xAnnotationDetails": 0,
    "xVitalReviewer": null,
    "dFlag1": null,
    "dCreateDate": "2024-05-22 16:34:00Z",
    "xIsFixedClone": 0,
    "xRecordRescindedDate": null,
    "xExternalDataSet": null,
    "xReportContentType": null,
    "dRevisionID": 1,
    "xLongName": null,
    "xCategoryID": null,
    "dPublishState": null,
    "xAuditPeriod": null,
    "dDocAccount": null,
    "xIsCutoff": 0,
    "xNoLatestRevisionDate": null,
    "xLibraryGUID": null
},
"workflowInfo": {
    "dWfID": 2,
    "dWfName": "criteriawf1",
    "dWfDescription": "Criteria Workflow1",
    "dCompletionDate": null,
    "dSecurityGroup": "Public",
    "dWfStatus": "INPROCESS",
    "dWfType": "Criteria",
    "dIsCollaboration": 0
},
"wf_doc_info": {
    "dWfID": 2,
    "dDocName": "ID45654",
    "dWfDocState": "INPROCESS",
    "dWfComputed": null,
    "dWfCurrentStepID": 4,
    "dWfDirectory": "public"
},
"workflowStep": {
    "dWfStepID": 4,
    "dWfStepName": "step1",
    "dWfID": 2,
    "dWfStepDescription": null,
    "dWfStepType": ":R:C:CE:",
    "dWfStepIsAll": 0,
    "dWfStepWeight": 1,
    "dWfStepIsSignature": 0,
    "dUsers": "weblogic",
    "dHasTokens": "false"
},
"workflowSteps": [
    {
        "dWfStepID": 3,
        "dWfStepName": "contribution",
        "dWfID": 2,
        "dWfStepDescription": null,
        "dWfStepType": ":C:CA:CE:",
        "dWfStepIsAll": 0,
        "dWfStepWeight": 1,
        "dWfStepIsSignature": 0
    },
    {
        "dWfStepID": 4,
        "dWfStepName": "step1",
        "dWfID": 2,
        "dWfStepDescription": null,
        "dWfStepType": ":R:C:CE:",
        "dWfStepIsAll": 0,
        "dWfStepWeight": 1,
        "dWfStepIsSignature": 0
    }
],
"workflowActionHistory": [
    {
        "dWfName": "criteriawf1",
        "dWfStepName": "contribution",
        "wfAction": "CHECKIN",
        "wfActionTs": "2024-05-22 16:34:05Z",
        "wfUsers": "weblogic",
        "wfMessage": null
    },
    {
        "dWfName": "criteriawf1",
        "dWfStepName": "contribution",
        "wfAction": "APPROVE",
        "wfActionTs": "2024-05-22 16:34:05Z",
        "wfUsers": "weblogic",
        "wfMessage": null
    },
    {
        "dWfName": "criteriawf1",
        "dWfStepName": "step1",
        "wfAction": "WORK_NOTIFICATION",
        "wfActionTs": "2024-05-22 16:34:05Z",
        "wfUsers": "weblogic",
        "wfMessage": "Content item 'ID45654' is ready for workflow step 'step1'.\n"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Approve a workflow

#### POST /documents/wcc/api/v1.1/files/{dDocName}/workflow/.approve {#post-documentswccapiv11filesddocnameworkflowapprove}

#### Description

Approve a content item in a workflow. (WORKFLOW_APPROVE)

#### Parameters

::: table-responsive
  Name       Located in   Description                            Required   Schema
  ---------- ------------ -------------------------------------- ---------- --------
  dDocName   path         dDocName of the document in workflow   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    The content item has been approved.                                                             
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-24}

Approve content item `ID9400002010` in its workflow.

##### Request

> POST .../documents/wcc/api/v1.1/files/ID9400002010/workflow/.approve

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Approve a workflow

#### POST /documents/wcc/api/v1.1/files/.by.did/{dID}/workflow/.approve {#post-documentswccapiv11filesbydiddidworkflowapprove}

#### Description

Approve a content item revision in a workflow. (WORKFLOW_APPROVE)

#### Parameters

::: table-responsive
  Name   Located in   Description                      Required   Schema
  ------ ------------ -------------------------------- ---------- --------
  dID    path         dID of a document in workflow.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    The content item has been approved.                                                             
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-25}

Approve item revision `23454323` in its workflow.

##### Request

> POST .../documents/wcc/api/v1.1/files/.by.did/23454323/workflow/.approve

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Reject a workflow

#### POST /documents/wcc/api/v1.1/files/{dDocName}/workflow/.reject {#post-documentswccapiv11filesddocnameworkflowreject}

#### Description

Reject a content item in a workflow. (WORKFLOW_REJECT)

#### Parameters

::: table-responsive
  Name            Located in   Description                            Required   Schema
  --------------- ------------ -------------------------------------- ---------- --------
  dDocName        path         dDocName of the document in workflow   Yes        string
  rejectMessage   query        The rejection message.                 No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    The content item has been rejected.                                                             
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-26}

Reject content item `ID9400002011` in its workflow.

##### Request

> POST .../documents/wcc/api/v1.1/files/ID9400002011/workflow/.reject

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Reject a workflow

#### POST /documents/wcc/api/v1.1/files/.by.did/{dID}/workflow/.reject {#post-documentswccapiv11filesbydiddidworkflowreject}

#### Description

Rejects content item revision in a workflow. (WORKFLOW_REJECT)

#### Parameters

::: table-responsive
  Name            Located in   Description                                            Required   Schema
  --------------- ------------ ------------------------------------------------------ ---------- --------
  dID             path         Revision identifier of the content item in workflow.   Yes        string
  rejectMessage   query        The rejection message.                                 No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    The content item has been rejected.                                                             
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-27}

Reject item revision `24353523` in its workflow.

##### Request

> POST .../documents/wcc/api/v1.1/files/.by.did/24353523/workflow/.reject

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Add an Attachment

#### POST /documents/wcc/api/v1.1/files/{dDocName}/attachments/data {#post-documentswccapiv11filesddocnameattachmentsdata}

#### Description

Upload an attachment to a document. (EDIT_RENDITIONS)

#### Parameters

::: table-responsive
  Name                      Located in   Description                                                                                                         Required   Schema
  ------------------------- ------------ ------------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocName                  path         The dDocName of the document to add the attachment.                                                                 Yes        string
  extRenditionName          formData     The name of the attachment.                                                                                         Yes        string
  extRenditionDescription   formData     The description of the attachment.                                                                                  No         string
  extRenditionFile          formData     The file for this attachment.                                                                                       Yes        file
  version                   formData     The the version of the document to add the attachment. By default, the attachment is added to the latest version.   No         string
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Attachment successfully created.\                                                               
                          Returns Header `Location` which is a URI to get the list of attachments.                       

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-28}

Upload a new attachment to the latest version of `ID14004398`.

##### Request

> POST .../documents/wcc/api/v1.1/files/ID14004398/attachments/data

##### Request Body

FormData Parameters

> extRenditionName = myattachment1
>
> extRenditionDescription = my attachment example
>
> extRenditionFile = \[filePath\]

##### HTTP Response

> Status = 201

Header

> Location = .../documents/wcc/api/v1.1/files/.by.did/8616/attachments/

## Add an Attachment by dID

#### POST /documents/wcc/api/v1.1/files/.by.did/{dID}/attachments/data {#post-documentswccapiv11filesbydiddidattachmentsdata}

#### Description

Upload an attachment to a specified document revision. (EDIT_RENDITIONS)

#### Parameters

::: table-responsive
  Name                      Located in   Description                                      Required   Schema
  ------------------------- ------------ ------------------------------------------------ ---------- --------
  dID                       path         The dID of the revision to add the attachment.   Yes        string
  extRenditionName          formData     The name of the attachment.                      Yes        string
  extRenditionDescription   formData     The description of the attachment.               No         string
  extRenditionFile          formData     The file for this attachment.                    Yes        file
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Attachment successfully created.\                                                               
                          Returns Header `Location` which is a URI to get the list of attachments.                       

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-29}

Upload a new attachment to the revision `4398`.

##### Request

> POST .../documents/wcc/api/v1.1/files/.by.did/4398/attachments/data

##### Request Body

FormData Parameters

> extRenditionName = myattachment2
>
> extRenditionDescription = attachment description
>
> extRenditionFile = \[filePath\]

##### HTTP Response

> Status = 201

Header

> Location = .../documents/wcc/api/v1.1/files/.by.did/4389/attachments/

## Download an Attachment

#### GET /documents/wcc/api/v1.1/files/{dDocName}/attachments/{extRenditionName}/data/ {#get-documentswccapiv11filesddocnameattachmentsextrenditionnamedata}

#### Description

Download the attachment file as a stream. (GET_FILE)

#### Parameters

::: table-responsive
  Name               Located in   Description                                                                                                                 Required   Schema
  ------------------ ------------ --------------------------------------------------------------------------------------------------------------------------- ---------- --------------------------
  Range              header       The byte range of the attachment to download.                                                                               No         [Range](#range-requests)
  dDocName           path         The dDocName of the document the attachment belongs to.                                                                     Yes        string
  extRenditionName   path         The name of the attachment.                                                                                                 Yes        string
  version            query        The version of the document to download the attachment. By default, the attachment is downloaded from the latest version.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the complete stream of the file.                                          
  206    Successfully returned the byte range of the document                                            
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  416    The server cannot return the bytes requested                                                    
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-30}

Download an attachment `myattachment1` from the latest version of `ID14004398`.

##### Request

> GET ...documents/wcc/api/v1.1/files/ID14004398/attachments/myattachment1/data

##### Request Body

None

##### HTTP Response

> Status = 200

Body

> stream of file contents as application/octet-stream

## Download an Attachment by dID

#### GET /documents/wcc/api/v1.1/files/.by.did/{dID}/attachments/{extRenditionName}/data/ {#get-documentswccapiv11filesbydiddidattachmentsextrenditionnamedata}

#### Description

Download the attachment file from a document revision as a stream. (GET_FILE)

#### Parameters

::: table-responsive
  Name               Located in   Description                                          Required   Schema
  ------------------ ------------ ---------------------------------------------------- ---------- --------------------------
  Range              header       The byte range the of the attachment to download.    No         [Range](#range-requests)
  dID                path         The dID of the revision the attachment belongs to.   Yes        string
  extRenditionName   path         The name of the attachment.                          Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the complete stream of the file                                           
  206    Successfully returned the byte range of the file                                                
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  416    The server cannot return the bytes requested                                                    
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-31}

Download an attachment `myattachment1` from revision `4398`.

##### Request

> GET ...documents/wcc/api/v1.1/files/.by.did/4398/attachments/myattachment1/data

##### Request Body

None

##### HTTP Response

> Status = 200

Body

> stream of file contents as application/octet-stream

## Delete an Attachment

#### DELETE /documents/wcc/api/v1.1/files/files/{dDocName}/attachments/{extRenditionName} {#delete-documentswccapiv11filesfilesddocnameattachmentsextrenditionname}

#### Description

Delete an attachment from a document. (DELETE_RENDITIONS)

#### Parameters

::: table-responsive
  Name               Located in   Description                                                                                                            Required   Schema
  ------------------ ------------ ---------------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocName           path         The dDocName of the document to delete the attachment.                                                                 Yes        string
  extRenditionName   path         The name of the attachment.                                                                                            Yes        string
  version            query        The version of the document to delete the attachment. By default, the attachment is deleted from the latest version.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted an attachment                                                              
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-32}

Delete attachment `myattachment1` from the latest version of `ID14004398`.

##### Request

> DELETE .../documents/wcc/api/v1.1/files/ID14004398/attachments/myattachment1

##### Request Body

None.

##### HTTP Response

> Status = 204

## Delete an Attachment by dID

#### DELETE /documents/wcc/api/v1.1/files/files/.by.did/{dID}/attachments/{extRenditionName} {#delete-documentswccapiv11filesfilesbydiddidattachmentsextrenditionname}

#### Description

Delete an attachment a document revision. (DELETE_RENDITIONS)

#### Parameters

::: table-responsive
  Name               Located in   Description                                         Required   Schema
  ------------------ ------------ --------------------------------------------------- ---------- --------
  dID                path         The dID of the revision to delete the attachment.   Yes        string
  extRenditionName   path         The name of the attachment.                         Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted an attachment                                                              
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-33}

Delete attachment `myattachment2` from the revsion `4398`.

##### Request

> DELETE .../documents/wcc/api/v1.1/files/.by.did/4398/attachments/myattachment2

##### Request Body

None.

##### HTTP Response

> Status = 204

## Get List of Attachments

#### GET /documents/wcc/api/v1.1/files/{dDocName}/attachments/ {#get-documentswccapiv11filesddocnameattachments}

#### Description

Get a list of attachements for a document. (DOC_INFO)

#### Parameters

::: table-responsive
  Name       Located in   Description                                                                                                            Required   Schema
  ---------- ------------ ---------------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocName   path         The dDocName of the document to get the list of attachments.                                                           Yes        string
  version    query        The version of the document to get the list attachments. By default, the attachment list is from the latest version.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------
  200    Successfully returned the list of attachments                                                  [AttachmentListResponse](#attachmentlistresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-34}

Get the list of attachments for content item `ID010801`.

##### Request

> GET .../documents/wcc/api/v1.1/files/ID010801/attachments

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"count": 3,
"items": [
    {
    "extRenditionName": "att1",
    "extRenditionDescription": "attachment 1",
    "extRenditionOriginalName": "5pages.DOC",
    "extRenditionFileType": "application/msword",
    "extRenditionFileSize": 19968
    },
    {
    "extRenditionName": "anImage",
    "extRenditionDescription": "cups",
    "extRenditionOriginalName": "Mugs.png",
    "extRenditionFileType": "image/png",
    "extRenditionFileSize": 226620
    },
    {
    "extRenditionName": "mountains",
    "extRenditionDescription": null,
    "extRenditionOriginalName": "Desert.jpg",
    "extRenditionFileType": "image/jpeg",
    "extRenditionFileSize": 845941
    }
    ]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Get List of Attachments by dID

#### GET /documents/wcc/api/v1.1/files/.by.did/{dID}/attachments/ {#get-documentswccapiv11filesbydiddidattachments}

#### Description

Get a list of attachements for a revision of a document. (DOC_INFO)

#### Parameters

::: table-responsive
  Name   Located in   Description                                               Required   Schema
  ------ ------------ --------------------------------------------------------- ---------- --------
  dID    path         The dID of the revision to get the list of attachments.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------
  200    Successfully returned the list of attachments                                                  [AttachmentListResponse](#attachmentlistresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-35}

Get the list of attachments for revision `10601`.

##### Request

> GET .../documents/wcc/api/v1.1/files/.by.did/10601/attachments

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy"count": 3,
"items": [
    {
        "extRenditionName": "att1",
        "extRenditionDescription": "attachment 1",
        "extRenditionOriginalName": "5pages.DOC",
        "extRenditionFileType": "application/msword",
        "extRenditionFileSize": 19968
    },
    {
        "extRenditionName": "anImage",
        "extRenditionDescription": "cups",
        "extRenditionOriginalName": "Mugs.png",
        "extRenditionFileType": "image/png",
        "extRenditionFileSize": 226620
    },
    {
        "extRenditionName": "mountains",
        "extRenditionDescription": null,
        "extRenditionOriginalName": "Desert.jpg",
        "extRenditionFileType": "image/jpeg",
        "extRenditionFileSize": 845941
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Get the Taxonomies of a Document

#### GET /documents/wcc/api/v1.1/files/{dDocName}/taxonomies {#get-documentswccapiv11filesddocnametaxonomies}

#### Description

Get the taxonomies for a document.

#### Parameters

::: table-responsive
  Name       Located in   Description                                       Required   Schema
  ---------- ------------ ------------------------------------------------- ---------- --------
  dDocName   path         The dDocName of the document to get taxonomies.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------
  200    Successfully returned the associated taxonomies                                                An array of [TaxonomyListOnDocumentObject](#taxonomylistondocumentobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

# Workflow

## Create a Workflow

#### POST /documents/wcc/api/v1.1/workflow {#post-documentswccapiv11workflow}

#### Description

Create a workflow. (ADD_WORKFLOW)

#### Parameters

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------
  Name               Located in     Description                                    Required                           Schema
  ------------------ -------------- ---------------------------------------------- ---------------------------------- --------------
  dWfName            formData       The workflow name.                             Yes                                string

  dWfType            formData       The workflow type; supported types include:\   Yes                                string
                                    •Basic\                                                                           
                                    •Criteria\                                                                        
                                    •SubWorkflow                                                                      

  dWfDescription     formData       The workflow description.                      Yes                                string

  dSecurityGroup     formData       The security group the workflow applies to.    Yes                                string

  dWfCriteriaName    formData       The workflow criteria field.                   Yes, if `dWfType` is `Criteria`.   string

  dWfCriteriaValue   formData       The workflow criteria value.                   Yes, if `dWfType` is `Criteria`.   string
  ----------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Successfully created a workflow.\                                                               
                          Returns Header `Location` which is a URI to get the definition of the added workflow.          

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-36}

Create a basic workflow in the Public security group.

##### Request

> POST .../documents/wcc/api/v1.1/workflow

##### Request Body

FormData Parameters

> dWfName = MyBasicWF
>
> dSecurityGroup = Public
>
> dWfType = Basic
>
> dWfDescription = a basic workflow

##### HTTP Response

> Status = 201

Body

None.

Header

> Location = .../documents/wcc/api/v1.1/workflows/MyBasicWF

## Get Workflow Information

#### GET /documents/wcc/api/v1.1/workflows/{dWfName} {#get-documentswccapiv11workflowsdwfname}

#### Description

Get workflow information by its name. (GET_WORKFLOW)

#### Parameters

::: table-responsive
  Name      Located in   Description         Required   Schema
  --------- ------------ ------------------- ---------- --------
  dWfName   path         The workflow name   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -------------------------------------------------------------
  200    Workflow information                                                                           [WorkflowInformationResponse](#workflowinformationresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-37}

Get information on workflow `criteriawf1`.

##### Request

> GET .../documents/wcc/api/v1.1/workflows/criteriawf1

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"workflow": {
    "dWfID": 2,
    "dWfName": "criteriawf1",
    "dWfDescription": "Criteria Workflow1",
    "dCompletionDate": null,
    "dSecurityGroup": "Public",
    "dWfStatus": "INPROCESS",
    "dWfType": "Criteria",
    "dIsCollaboration": 0
},
"workflowSteps": [
    {
        "dWfStepID": 3,
        "dWfStepName": "contribution",
        "dWfID": 2,
        "dWfStepDescription": null,
        "dWfStepType": ":C:CA:CE:",
        "dWfStepIsAll": 0,
        "dWfStepWeight": 1,
        "dWfStepIsSignature": 0
    },
    {
        "dWfStepID": 4,
        "dWfStepName": "step1",
        "dWfID": 2,
        "dWfStepDescription": null,
        "dWfStepType": ":R:C:CE:",
        "dWfStepIsAll": 0,
        "dWfStepWeight": 1,
        "dWfStepIsSignature": 0,
        "dAliases": "weblogic\tuser"
    }
],
"workflowStepEvents": [
    {
        "dWfStepName": "step1",
        "wfEntryScript": null,
        "wfExitScript": null,
        "wfUpdateScript": null
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Edit a Workflow

#### PUT /documents/wcc/api/v1.1/workflows/{dWfName} {#put-documentswccapiv11workflowsdwfname}

#### Description

Update a workflow. (EDIT_WORKFLOW, EDIT_WORKFLOWCRITERIA)

#### Parameters

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------
  Name               Located in     Description                                    Required                           Schema
  ------------------ -------------- ---------------------------------------------- ---------------------------------- --------------
  dWfName            path           The workflow name.                             Yes                                string

  dWfType            formData       The workflow type; supported types include:\   Yes                                string
                                    •Basic\                                                                           
                                    •Criteria\                                                                        
                                    •SubWorkflow                                                                      

  dWfDescription     formData       The workflow description.                      No                                 string

  dSecurityGroup     formData       The security group the workflow applies to.    Yes                                string

  dWfCriteriaName    formData       The workflow criteria field.                   Yes, if `dWfType` is `Criteria`.   string

  dWfCriteriaValue   formData       The workflow criteria value.                   Yes, if `dWfType` is `Criteria`.   string
  ----------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204                     Successfully updated a workflow.\                                                               
                          Returns Header `Location` which is a URI to get the definition of the added workflow.          

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-38}

Update the `dWfDescription` in the `MyBasicWF` workflow.

##### Request

> PUT .../documents/wcc/api/v1.1/workflows/MyBasicWF

##### Request Body

FormData Parameters

> dSecurityGroup = Public
>
> dWfType = Basic
>
> dWfDescription = the basic workflow for Public

##### HTTP Response

> Status = 204

Body

None.

Header

> Location = .../documents/wcc/api/v1.1/workflows/MyBasicWF

## Delete a Workflow

#### DELETE /documents/wcc/api/v1.1/workflows/{dWfName} {#delete-documentswccapiv11workflowsdwfname}

#### Description

Delete a workflow. (DELETE_WORKFLOW, DELETE_WORKFLOWCRITERIA)

#### Parameters

::: table-responsive
  Name      Located in   Description          Required   Schema
  --------- ------------ -------------------- ---------- --------
  dWfName   path         The workflow name.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted a workflow.                                                                
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-39}

Delete the `MyBasicWF` workflow.

##### Request

> DELETE .../documents/wcc/api/v1.1/workflows/MyBasicWF

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Start a Workflow

#### POST /documents/wcc/api/v1.1/workflows/{dWfName}/.start {#post-documentswccapiv11workflowsdwfnamestart}

#### Description

Start a basic workflow or enable a criteria workflow. (WORKFLOW_START, CRITERIAWORKFLOW_ENABLE)

#### Parameters

::: table-responsive
  Name        Located in   Description                                                                                                      Required   Schema
  ----------- ------------ ---------------------------------------------------------------------------------------------------------------- ---------- --------
  dWfName     path         The workflow name.                                                                                               Yes        string
  wfMessage   formData     The message included in the "Workflow Started" notification e-mail. This is only passed for `Basic` workflows.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully started a workflow.                                                                
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-40}

Start the `MyBasicWF` workflow.

##### Request

> POST .../documents/wcc/api/v1.1/workflows/MyBasicWF/.start

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Cancel a Workflow

#### POST /documents/wcc/api/v1.1/workflows/{dWfName}/.cancel {#post-documentswccapiv11workflowsdwfnamecancel}

#### Description

Cancel a basic workflow or disable a criteria workflow. (WORKFLOW_CANCEL, CRITERIAWORKFLOW_DISABLE)

#### Parameters

::: table-responsive
  Name      Located in   Description          Required   Schema
  --------- ------------ -------------------- ---------- --------
  dWfName   path         The workflow name.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully canceled a workflow.                                                               
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-41}

Cancel the `MyBasicWF` workflow.

##### Request

> POST .../documents/wcc/api/v1.1/workflows/MyBasicWF/.cancel

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Create a Workflow Step

#### POST /documents/wcc/api/v1.1/workflows/{dWfName}/steps {#post-documentswccapiv11workflowsdwfnamesteps}

#### Description

Create a workflow step for the workflow. (ADD_WORKFLOWSTEP)

#### Parameters

::: table-responsive
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                 Located in     Description                                                                                                                                   Required       Schema
  -------------------- -------------- --------------------------------------------------------------------------------------------------------------------------------------------- -------------- --------------
  dWfName              path           The workflow name.                                                                                                                            Yes            string

  dWfStepName          formData       The workflow step name.                                                                                                                       Yes            string

  dWfStepDescription   formData       The workflow step description.                                                                                                                Yes            string

  dWfStepType          formData       The workflow step type. The following types are supported:\                                                                                   Yes            string
                                      •Review\                                                                                                                                                     
                                      •Review/Edit Revision\                                                                                                                                       
                                      •Review/New Revision\                                                                                                                                        
                                      •or it can be passed as a string of stepType codes for example `:R:C:CE:(Reviewer/Contributor), :R:(Review)`                                                 

  dWfStepIsAll         formData       When `true`, all users should approve the revision for the next step.\                                                                        No             boolean
                                      When `false`, the `dWfStepWeight` field specifies the number of users required for the next step. The default is `false`.                                    

  dWfStepWeight        formData       The number of reviewers that must approve the revision before the workflow passes to the next step. The default is `1`.                       No             number

  wfEntryScript        formData       The step entry script. It must be placed within \<\$ and \$\> delimiters.                                                                     No             string

  wfExitScript         formData       The step entry script. It must be placed within \<\$ and \$\> delimiters.                                                                     No             string

  wfUpdateScript       formData       The step entry script. It must be placed within \<\$ and \$\> delimiters.                                                                     No             string

  dAliases             formData       A comma delimited list of aliases and users to be used for the step. The format for this parameter is: `alias1,usertype1,alias2,usertype2`.   Yes            string
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201    Successfully created a workflow step.                                                           
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-42}

Add the `RCWF1Step3` step to the `CriteriaWF1` workflow.

##### Request

> POST .../documents/wcc/api/v1.1/workflows/CriteriaWF1/steps

##### Request Body

FormData Parameters

> dWfStepName = RCWF1Step3
>
> dWfStepDescription = Step 3 Description
>
> dWfStepType = Review/New Revision
>
> dWfStepIsAll = 1
>
> dWfStepWeight = 1
>
> dAliases = weblogic,user
>
> dWfStepDescription = Step3

##### HTTP Response

> Status = 201

Body

None.

## Edit a Workflow Step

#### PUT /documents/wcc/api/v1.1/workflows/{dWfName}/steps/{dWfStepName} {#put-documentswccapiv11workflowsdwfnamestepsdwfstepname}

#### Description

Edit a workflow step for the workflow. (EDIT_WORKFLOWSTEP)

#### Parameters

::: table-responsive
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                 Located in     Description                                                                                                                                   Required       Schema
  -------------------- -------------- --------------------------------------------------------------------------------------------------------------------------------------------- -------------- --------------
  dWfName              path           The workflow name.                                                                                                                            Yes            string

  dWfStepName          path           The workflow step name.                                                                                                                       Yes            string

  dWfStepDescription   formData       The workflow step description.                                                                                                                No             string

  dWfStepType          formData       The workflow step type. The following types are supported:\                                                                                   Yes            string
                                      •Review\                                                                                                                                                     
                                      •Review/Edit Revision\                                                                                                                                       
                                      •Review/New Revision\                                                                                                                                        
                                      •or it can be passed as a string of stepType codes for example `:R:C:CE:(Reviewer/Contributor), :R:(Review)`                                                 

  dWfStepIsAll         formData       When `true`, all users should approve the revision for the next step.\                                                                        No             boolean
                                      When `false`, the `dWfStepWeight` field specifies the number of users required for the next step. The default is `false`.                                    

  dWfStepWeight        formData       The number of reviewers that must approve the revision before the workflow passes to the next step. The default is `1`.                       No             number

  wfEntryScript        formData       The step entry script. It must be placed within \<\$ and \$\> delimiters.                                                                     No             string

  wfExitScript         formData       The step entry script. It must be placed within \<\$ and \$\> delimiters.                                                                     No             string

  wfUpdateScript       formData       The step entry script. It must be placed within \<\$ and \$\> delimiters.                                                                     No             string

  dAliases             formData       A comma delimited list of aliases and users to be used for the step. The format for this parameter is: `alias1,usertype1,alias2,usertype2`.   Yes            string
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully edited a workflow step.                                                            
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-43}

Change the type of the `RCWF1Step3` step to the `MyBasicWF` workflow.

##### Request

> POST .../documents/wcc/api/v1.1/workflows/CriteriaWF1/steps/RCWF1Step3

##### Request Body

FormData Parameters

> dWfStepType = Contributor

##### HTTP Response

> Status = 204

Body

None.

## Delete a Workflow Step

#### DELETE /documents/wcc/api/v1.1/workflows/{dWfName}/steps/{dWfStepName} {#delete-documentswccapiv11workflowsdwfnamestepsdwfstepname}

#### Description

Delete a workflow step from the workflow. (DELETE_WORKFLOWSTEP)

#### Parameters

::: table-responsive
  Name          Located in   Description               Required   Schema
  ------------- ------------ ------------------------- ---------- --------
  dWfName       path         The workflow name.        Yes        string
  dWfStepName   path         The workflow step name.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted a workflow step.                                                           
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-44}

Delete the `RCWF1Step3` step from the `MyBasicWF` workflow.

##### Request

> DELETE .../documents/wcc/api/v1.1/workflows/CriteriaWF1/steps/RCWF1Step3

##### Request Body

FormData Parameters

> dWfStepType = Contributor

##### HTTP Response

> Status = 204

Body

None.

## Add a User to a Workflow Step

#### POST /documents/wcc/api/v1.1/workflows/{dWfName}/steps/{dWfStepName}/aliases {#post-documentswccapiv11workflowsdwfnamestepsdwfstepnamealiases}

#### Description

Add an alias or a user to the specified workflow step of a workflow. (ADD_WORKFLOWALIASES)

#### Parameters

::: table-responsive
  Name          Located in   Description                                             Required   Schema
  ------------- ------------ ------------------------------------------------------- ---------- --------
  dWfName       path         The workflow name.                                      Yes        string
  dWfStepName   path         The workflow step name.                                 Yes        string
  dAlias        query        The name of the alias or the user to add to the step.   Yes        string
  dAliasType    query        The type of user must be either `user` or `alias`.      Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully added the user to the workflow step.                                               
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-45}

Add the weblogic user to the `RCWF1Step3` step from the `CriteriaWF1` workflow.

##### Request

> POST .../documents/wcc/api/v1.1/workflows/CriteriaWF1/steps/RCWF1Step3/aliases

##### Request Body

> Query Parameters: dAlias=weblogic & dAliasType=user

##### HTTP Response

> Status = 204

Body

None.

## Delete a User from a Workflow Step

#### POST /documents/wcc/api/v1.1/workflows/{dWfName}/steps/{dWfStepName}/aliases/{dAlias} {#post-documentswccapiv11workflowsdwfnamestepsdwfstepnamealiasesdalias}

#### Description

Delete an alias or a user from the specified workflow step of a workflow. (DELETE_WFCONTRIBUTORS)

#### Parameters

::: table-responsive
  Name          Located in   Description                                                  Required   Schema
  ------------- ------------ ------------------------------------------------------------ ---------- --------
  dWfName       path         The workflow name.                                           Yes        string
  dWfStepName   path         The workflow step name.                                      Yes        string
  dAlias        path         The name of the alias or the user to delete from the step.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted the user from the workflow step.                                           
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-46}

Delete the weblogic user from the `RCWF1Step3` step from the `CriteriaWF1` workflow.

##### Request

> DELETE .../documents/wcc/api/v1.1/workflows/CriteriaWF1/steps/RCWF1Step3/aliases/weblogic

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Add a document to Workflow

#### POST /documents/wcc/api/v1.1/workflows/{dWfName}/files/{dDocName} {#post-documentswccapiv11workflowsdwfnamefilesddocname}

#### Description

Add a document to a basic workflow. Note that criteria workflows are not supported. (ADD_WORKFLOWDOCUMENT)

#### Parameters

::: table-responsive
  Name       Located in   Description                            Required   Schema
  ---------- ------------ -------------------------------------- ---------- --------
  dWfName    path         The workflow name.                     Yes        string
  dDocName   path         The dDocName of the document to add.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully added the document to the workflow.                                                
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-47}

Add the document identified by `ID14006201` to the `MyBasicWF` workflow.

##### Request

> POST .../documents/wcc/api/v1.1/workflows/MyBasicWF/files/ID14006201

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Remove a document from a Workflow

#### DELETE /documents/wcc/api/v1.1/workflows/{dWfName}/files/{dDocName} {#delete-documentswccapiv11workflowsdwfnamefilesddocname}

#### Description

Remove a document from a basic workflow. Note that criteria workflows are not supported. (DELETE_WORKFLOWDOCUMENTS)

#### Parameters

::: table-responsive
  Name       Located in   Description                               Required   Schema
  ---------- ------------ ----------------------------------------- ---------- --------
  dWfName    path         The workflow name.                        Yes        string
  dDocName   path         The dDocName of the document to remove.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully removed the document from the workflow.                                            
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-48}

Remove the document identified by `ID14006201` from the `MyBasicWF` workflow.

##### Request

> DELETE .../documents/wcc/api/v1.1/workflows/MyBasicWF/files/ID14006201

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Get content item revisions in a workflow

#### GET /documents/wcc/api/v1.1/workflows/{dWfName}/docrevisions {#get-documentswccapiv11workflowsdwfnamedocrevisions}

#### Description

Get a list of content item revisions that are in the workflow. (GET_WORKFLOWDOCREVISIONS)

#### Parameters

::: table-responsive
  Name      Located in   Description     Required   Schema
  --------- ------------ --------------- ---------- --------
  dWfName   path         Workflow name   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -------------------------------------------------------------------------------
  200    Workflow information                                                                           [WorkflowInformationRevisionsResponse](#workflowinformationrevisionsresponse)
  204    There are no content items in this workflow.                                                    
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-49}

Get information on revisions in the workflow `criteriawf1`.

##### Request

> GET ...documents/wcc/api/v1.1/workflows/criteriawf1/docrevisions

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"wf_info": {
    "dWfID": 2,
    "dWfName": "criteriawf1",
    "dWfDescription": "Criteria Workflow1",
    "dCompletionDate": null,
    "dSecurityGroup": "Public",
    "dWfStatus": "INPROCESS",
    "dWfType": "Criteria",
    "dIsCollaboration": 0
},
"workflowSteps": [
    {
        "dWfStepID": 3,
        "dWfStepName": "contribution",
        "dWfID": 2,
        "dWfStepDescription": null,
        "dWfStepType": ":C:CA:CE:",
        "dWfStepIsAll": 0,
        "dWfStepWeight": 1,
        "dWfStepIsSignature": 0,
        "dHasTokens": "false"
    },
    {
        "dWfStepID": 4,
        "dWfStepName": "step1",
        "dWfID": 2,
        "dWfStepDescription": null,
        "dWfStepType": ":R:C:CE:",
        "dWfStepIsAll": 0,
        "dWfStepWeight": 1,
        "dWfStepIsSignature": 0,
        "dUsers": "user:weblogic",
        "dHasTokens": "false"
    }
],
"wfDocuments": [
    {
        "dWfID": 2,
        "dDocName": "ID45654",
        "dWfDocState": "INPROCESS",
        "dWfComputed": null,
        "dWfCurrentStepID": 4,
        "dWfDirectory": "public",
        "dVitalState": "Y",
        "xReportType": null,
        "xIsEditable": "1",
        "xHasFixedClone": 0,
        "dCharacterSet": null,
        "xRelatedContentTriggerDate": null,
        "xRecordCutoffDate": null,
        "dLastModifiedDate": "2024-05-22 16:34:18Z",
        "dRecordState": null,
        "dMessage": null,
        "dWebExtension": "jpg",
        "xClassifiedMarkings": null,
        "xStorageRule": "DispByContentId",
        "dIndexerState": null,
        "xSuperSupersededDate": null,
        "dReleaseDate": null,
        "dOutDate": null,
        "dRevClassID": 7846,
        "xIsVital": "0",
        "xRelatedContentList": null,
        "xIsDeletable": "1",
        "dRmaProcessState": "Y",
        "xRecordDestroyDate": null,
        "xVitalPeriodUnits": null,
        "xSource": null,
        "xRecordFilingDate": "2024-05-22 16:32:00Z",
        "dStatus": "REVIEW",
        "dInDate": "2024-05-22 16:32:00Z",
        "xRecordReviewDate": null,
        "xCpdIsTemplateEnabled": 0,
        "xPartitionId": null,
        "dReleaseState": "E",
        "xCpdIsLocked": 0,
        "xIsFixedClone": 0,
        "xReportContentType": null,
        "xCategoryID": null,
        "dPublishState": null,
        "xAuditPeriod": null,
        "dID": 8446,
        "xLibraryGUID": null,
        "xRecordSupersededDate": null,
        "xDeleteApproveDate": null,
        "xIsACLReadOnlyOnUI": 0,
        "xComments": "criteriawf1",
        "dRendition1": null,
        "xNewRevisionDate": "2024-05-22 16:34:05Z",
        "dRendition2": null,
        "xRecordCancelledDate": null,
        "dRmaSegmentID": 3,
        "dDocTitle": "workflow demo",
        "dCheckoutUser": null,
        "dFileSize": 845941,
        "xRecordExpirationDate": null,
        "xVitalPeriod": 0,
        "dIsPrimary": 1,
        "dExtension": "jpg",
        "dProcessingState": "Y",
        "xIsFrozen": "0",
        "dWorkflowState": "W",
        "xIsRevisionable": "1",
        "dDocType": "Document",
        "xRecordActivationDate": null,
        "dLanguage": null,
        "dIsCheckedOut": 0,
        "xSupersededContent": null,
        "xIdcProfile": null,
        "dDocAuthor": "weblogic",
        "dIsWebFormat": 0,
        "xFreezeID": "0",
        "xIsSubjectToAudit": "0",
        "xFreezeReason": null,
        "dPublishType": null,
        "dFormat": "image/jpeg",
        "xClbraUserList": null,
        "xIsRecord": "0",
        "xFolderID": null,
        "dRevRank": 0,
        "dDocID": 13691,
        "xClbraAliasList": null,
        "dLocation": null,
        "xSupplementalMarkings": null,
        "xWebFlag": null,
        "dOriginalName": "Desert.jpg",
        "xRecordObsoleteDate": null,
        "xAnnotationDetails": 0,
        "dSecurityGroup": "Public",
        "xVitalReviewer": null,
        "dFlag1": null,
        "dCreateDate": "2024-05-22 16:34:05Z",
        "xRecordRescindedDate": null,
        "xExternalDataSet": null,
        "dRevisionID": 1,
        "xLongName": null,
        "dRevLabel": "1",
        "dDocAccount": null,
        "xIsCutoff": "0",
        "xNoLatestRevisionDate": null
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Active Workflows

#### GET /documents/wcc/api/v1.1/workflows/active/items {#get-documentswccapiv11workflowsactiveitems}

#### Description

Get active workflows. (GET_ACTIVE_WORKFLOWS)

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------
  200    The list of active workflows.                                                                  [WorkflowActiveResponse](#workflowactiveresponse)
  204    There are no active standard workflows in the system.                                           
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-50}

List the active workflows.

##### Request

> GET .../documents/wcc/api/v1.1/workflows/active/items

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"count": 3,
"items": [
    {
        "dWfID": 2,
        "dWfName": "criteriawf1",
        "dWfDescription": "Criteria Workflow1",
        "dCompletionDate": null,
        "dSecurityGroup": "Public",
        "dWfStatus": "INPROCESS",
        "dWfType": "Criteria",
        "dIsCollaboration": 0
    },
    {
        "dWfID": 201,
        "dWfName": "criteriawf2",
        "dWfDescription": null,
        "dCompletionDate": null,
        "dSecurityGroup": "Public",
        "dWfStatus": "INPROCESS",
        "dWfType": "Criteria",
        "dIsCollaboration": 0
    },
    {
        "dWfID": 402,
        "dWfName": "criteriawf4",
        "dWfDescription": "Criteria 4 on Secure Group",
        "dCompletionDate": null,
        "dSecurityGroup": "Secure",
        "dWfStatus": "INPROCESS",
        "dWfType": "Criteria",
        "dIsCollaboration": 0
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Get Current Workflow Assignments

#### GET .../documents/wcc/api/v1.1/workflows/inqueue/items {#get-documentswccapiv11workflowsinqueueitems}

#### Description

Get the user's current workflow assignments.(GET_WORKFLOW_INQUEUE_LIST,GET_WORKFLOW_INQUEUE_LIST_EX)

#### Parameters

::: table-responsive
  Name               Located in   Description                                                                                                                                                                           Required   Schema
  ------------------ ------------ ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  fields             query        A comma separated list of fields to be returned for each content item. By default, all fields will be returned.                                                                       No         string
  orderBy            query        The sort field and sort order which will be used to arrange the filtered content items. For example: `dwfQueueLastActionTs:Desc` will sort the specified field in descending order.   No         string
  limit              query        The maximum number of items listed per page. If not provided, the limit is calculated from the config setting `WfInqueueMaxRows`. If neither are set, the default is 20.              No         number
  offset             query        Specifies the point from which items are listed for the response.                                                                                                                     No         number
  doMarkSubscribed   query        When `1` adds a `fIsSubscribed` field in the resultset to indicate if the folder is subscribed.                                                                                       No         number
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------------
  200    Workflow in queue for the user.                                                                [WorkflowInQueueResponse](#workflowinqueueresponse)
  204    The workflow in queue is empty.                                                                 
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-51}

The user's workflow assignments.

##### Request

> GET .../documents/wcc/api/v1.1/workflows/inqueue/items

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"count": 1,
"hasMore": false,
"numPages": 1,
"pageNumber": 1,
"totalRows": 1,
"startRow": 1,
"endRow": 1,
"items": [
    {
        "dUser": "weblogic",
        "dDocName": "ID45654",
        "dID": 8446,
        "dWfID": 2,
        "dWfName": "criteriawf1",
        "dWfStepName": "step1",
        "dwfQueueActionState": null,
        "dwfQueueEnterTs": "2024-05-22 16:34:05Z",
        "dwfQueueLastActionTs": "2024-05-22 16:34:05Z",
        "wfMessage": null,
        "dVitalState": "Y",
        "xReportType": null,
        "xIsEditable": "1",
        "xHasFixedClone": 0,
        "dCharacterSet": null,
        "xRelatedContentTriggerDate": null,
        "dwfMessage": null,
        "xRecordCutoffDate": null,
        "dLastModifiedDate": "2024-05-22 16:34:18Z",
        "dWfStepType": ":R:C:CE:",
        "dRecordState": null,
        "dMessage": null,
        "dWebExtension": "jpg",
        "xClassifiedMarkings": null,
        "xStorageRule": "DispByContentId",
        "dIndexerState": null,
        "dWfStepID": 4,
        "xSuperSupersededDate": null,
        "dReleaseDate": null,
        "dOutDate": null,
        "dRevClassID": 7846,
        "xIsVital": "0",
        "xRelatedContentList": null,
        "xIsDeletable": "1",
        "dWfStepDescription": null,
        "wfQueueLastActionTs": "2024-05-22 16:34:05Z",
        "dRmaProcessState": "Y",
        "xRecordDestroyDate": null,
        "xVitalPeriodUnits": null,
        "dWfStepWeight": 1,
        "xSource": null,
        "xRecordFilingDate": "2024-05-22 16:32:00Z",
        "dStatus": "REVIEW",
        "dInDate": "2024-05-22 16:32:00Z",
        "xRecordReviewDate": null,
        "dWfStepIsAll": 0,
        "xCpdIsTemplateEnabled": 0,
        "xPartitionId": null,
        "dReleaseState": "E",
        "xCpdIsLocked": 0,
        "xIsFixedClone": 0,
        "xReportContentType": null,
        "xCategoryID": null,
        "dPublishState": null,
        "xAuditPeriod": null,
        "xLibraryGUID": null,
        "xRecordSupersededDate": null,
        "xDeleteApproveDate": null,
        "xIsACLReadOnlyOnUI": 0,
        "xComments": "criteriawf1",
        "dRendition1": null,
        "xNewRevisionDate": "2024-05-22 16:34:05Z",
        "dRendition2": null,
        "xRecordCancelledDate": null,
        "wfQueueActionState": null,
        "dRmaSegmentID": 3,
        "dDocTitle": "workflow demo",
        "dCheckoutUser": null,
        "dFileSize": 845941,
        "xRecordExpirationDate": null,
        "xVitalPeriod": 0,
        "dIsPrimary": 1,
        "dExtension": "jpg",
        "dProcessingState": "Y",
        "xIsFrozen": "0",
        "dWorkflowState": "W",
        "xIsRevisionable": "1",
        "dDocType": "Document",
        "xRecordActivationDate": null,
        "dLanguage": null,
        "dIsCheckedOut": 0,
        "xSupersededContent": null,
        "xIdcProfile": null,
        "dDocAuthor": "weblogic",
        "dIsWebFormat": 0,
        "xFreezeID": "0",
        "xIsSubjectToAudit": "0",
        "xFreezeReason": null,
        "dPublishType": null,
        "dFormat": "image/jpeg",
        "xClbraUserList": null,
        "xIsRecord": "0",
        "xFolderID": null,
        "dRevRank": 0,
        "dDocID": 13691,
        "xClbraAliasList": null,
        "dLocation": null,
        "xSupplementalMarkings": null,
        "wfQueueEnterTs": "2024-05-22 16:34:05Z",
        "xWebFlag": null,
        "dOriginalName": "Desert.jpg",
        "dWfStepIsSignature": 0,
        "xRecordObsoleteDate": null,
        "xAnnotationDetails": 0,
        "dSecurityGroup": "Public",
        "xVitalReviewer": null,
        "dFlag1": null,
        "dCreateDate": "2024-05-22 16:34:05Z",
        "xRecordRescindedDate": null,
        "xExternalDataSet": null,
        "dRevisionID": 1,
        "xLongName": null,
        "dRevLabel": "1",
        "dDocAccount": null,
        "xIsCutoff": "0",
        "xNoLatestRevisionDate": null
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

# System

## PING Server

#### GET /documents/wcc/api/v1.1/system/ping {#get-documentswccapiv11systemping}

#### Description

Verify the server is running (PING_SERVER).

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    The server is available                                                                         
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-52}

Validate the server is responding.

##### Request

> GET .../documents/wcc/api/v1.1/system/ping

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Get DocProfiles Information

#### GET /documents/wcc/api/v1.1/system/docProfiles {#get-documentswccapiv11systemdocprofiles}

#### Description

List information about all the docProfiles. (GET_DOCPROFILES)

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the docProfiles information.                                             [DocProfilesResponse](#docprofilesresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-53}

Get all the docProfiles.

##### Request

> GET .../documents/wcc/api/v1.1/system/docProfiles

##### Request Body

None

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"defFileExists": true,
"dpDidSetDefaultTriggerField": 1,
"dpTriggerField": "dSecurityGroup",
"count": 2,
"items": [
    {
        "dpName": "Profile_1",
        "dpDescription": "Profile 1",
        "dpTriggerValue": "Secure",
        "dpDisplayLabel": "Profile_1",
        "dDocClass": "Base",
        "isValid": true
    },
    {
        "dpName": "Profile_2",
        "dpDescription": "Profile 2",
        "dpTriggerValue": "Public",
        "dpDisplayLabel": "Profile_2",
        "dDocClass": "Base",
        "isValid": true
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Create a Document Profile

#### POST /documents/wcc/api/v1.1/system/docProfiles {#post-documentswccapiv11systemdocprofiles}

#### Description

Create a new document profile. (ADD_DOCPROFILE)

#### Parameters

::: table-responsive
  Name                Located in   Description                                                                                                                                                                   Required   Schema
  ------------------- ------------ ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dpName              formData     The unique name of the profile.                                                                                                                                               Yes        string
  dpDescription       formData     The description of the profile.                                                                                                                                               Yes        string
  dpTriggerValue      formData     The trigger associated with the profile.                                                                                                                                      Yes        string
  dpDisplayLabel      formData     The display label of the profile.                                                                                                                                             Yes        string
  isValidateTrigger   formData     When set to true, validates that the trigger value exists. If the value does not exist, the service fails. When false, the profile is added without validating the trigger.   No         string
:::

#### Responses

::: table-responsive
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                     Schema
  ----------------------- ----------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Successfully created new profile.\                                                               
                          Returns Header `Location` which is a URI to get the definition of the added document profile.   

  400                     Bad request                                                                                     [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                     

  403                     User is not allowed to take this action                                                          

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request    [GeneralErrorResponse](#generalerrorresponse)
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-54}

Creates new document profile called `MyNewProfile`.

##### Request

> POST .../documents/wcc/api/v1.1/system/docProfiles

##### Request Body

FormData Parameters

> dpName = MyNewProfile
>
> dpDescription = the description
>
> dpTriggerValue = Secure
>
> dpDisplayLabel = display label

##### HTTP Response

> Status = 201

Header

> Location = .../documents/wcc/api/v1.1/system/docProfiles/MyNewProfile

## Update a Document Profile

#### PUT /documents/wcc/api/v1.1/system/docProfiles/{dpName} {#put-documentswccapiv11systemdocprofilesdpname}

#### Description

Edit a document profile. (EDIT_DOCPROFILE)

#### Parameters

Note: At least one of the formData fields must be included in the request.

::: table-responsive
  Name             Located in   Description                                Required   Schema
  ---------------- ------------ ------------------------------------------ ---------- --------
  dpName           path         The unique name of the profile.            Yes        string
  dpDescription    formData     The description of the profile.            Maybe      string
  dpTriggerValue   formData     The trigger associated with the profile.   Maybe      string
  dpDisplayLabel   formData     The display label of the profile.          Maybe      string
:::

#### Responses

::: table-responsive
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                                        Schema
  ----------------------- ------------------------------------------------------------------------------------------------------------------ -----------------------------------------------
  204                     Successfully updated the profile.\                                                                                  
                          Returns Header `Location` which is a URI to get the A URI to get the definition of the updated document profile.   

  400                     Bad request                                                                                                        [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                                        

  403                     User is not allowed to take this action                                                                             

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request                       [GeneralErrorResponse](#generalerrorresponse)
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-55}

Update the document profile called `MyNewProfile` by changing the trigger value.

##### Request

> PUT .../documents/wcc/api/v1.1/system/docProfiles/MyNewProfile

##### Request Body

FormData Parameters

> dpTriggerValue = Public

##### HTTP Response

> Status = 204

Header

> Location = .../documents/wcc/api/v1.1/system/docProfiles/MyNewProfile

## Get a Document Profile

#### GET /documents/wcc/api/v1.1/system/docProfiles/{dpName} {#get-documentswccapiv11systemdocprofilesdpname}

#### Description

Get a document profile. (GET_DOCPROFILE)

#### Parameters

::: table-responsive
  Name     Located in   Description                       Required   Schema
  -------- ------------ --------------------------------- ---------- --------
  dpName   path         The unique name of the profile.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned document profile information.                                            [DocProfileResponse](#docprofileresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-56}

Get the document profile called `MyNewProfile`.

##### Request

> GET .../documents/wcc/api/v1.1/system/docProfiles/MyNewProfile

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

> {
>
> "dpName": "MyNewProfile",
>
> "dpDescription": "new description",
>
> "dpDisplayLabel": "updateDisplay",
>
> "dpTriggerValue": "Public",
>
> "dDocClass": "Base",
>
> "isValid": null,
>
> "defFileExists": true
>
> }

## Delete a Document Profile

#### DELETE /documents/wcc/api/v1.1/system/docProfiles/{dpName} {#delete-documentswccapiv11systemdocprofilesdpname}

#### Description

Delete a document profile. (DELETE_DOCPROFILE)

#### Parameters

::: table-responsive
  Name     Located in   Description                       Required   Schema
  -------- ------------ --------------------------------- ---------- --------
  dpName   path         The unique name of the profile.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted the profile.                                                               
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-57}

Delete the document profile called `MyNewProfile`.

##### Request

> DELETE .../documents/wcc/api/v1.1/system/docProfiles/MyNewProfile

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Query a Data Source

#### GET /documents/wcc/api/v1.1/system/{dataSource}/items {#get-documentswccapiv11systemdatasourceitems}

#### Description

Query the database via a defined data source. (GET_DATARESULTSET)

#### Parameters

::: table-responsive
  Name          Located in   Description                                                                                    Required   Schema
  ------------- ------------ ---------------------------------------------------------------------------------------------- ---------- --------
  dataSource    path         The server data source to query. The server table `DataSources` lists the supported sources.   Yes        string
  whereClause   query        The WHERE clause for the select query.                                                         No         string
  orderClause   query        The ORDER clause for the select query.                                                         No         string
  maxRows       query        Sets the maximum number of rows to be returned.                                                No         number
  startRow      query        The number of rows to skip when the table is returned.                                         No         number
:::

#### Responses

::: table-responsive
  Code   Description                                                                                                                                            Schema
  ------ ------------------------------------------------------------------------------------------------------------------------------------------------------ -----------------------------------------------
  200    The dataSource was successfully queried. The response depends on the data source being queried and are not explicitly defined in this documentation.   [DataSourceResponse](#datasourceresponse)
  400    Bad request                                                                                                                                            [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                                                                            
  403    User is not allowed to take this action                                                                                                                 
  500    The server encountered an unexpected condition that prevented it from fulfilling the request                                                           [GeneralErrorResponse](#generalerrorresponse)
:::

## Get All Document Types

#### GET /documents/wcc/api/v1.1/system/doctypes {#get-documentswccapiv11systemdoctypes}

#### Description

List information about all document types. (GET_DOCTYPES)

#### Parameters

None.

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned document type information.                                               [DocTypeResponse](#doctyperesponse)
  204    There are no document types defined.                                                            
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-58}

Get all the document types.

##### Request

> GET .../documents/wcc/api/v1.1/system/doctypes

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"count": 5,
"items": [
    {
    "dDocType": "Application",
    "dDescription": "wwDocTypeDesc_Application",
    "dGif": "ucm_application_file.png"
    },
    {
    "dDocType": "Binary",
    "dDescription": "wwDocTypeDesc_Binary",
    "dGif": "ucm_binaryfile.png"
    },
    {
    "dDocType": "DigitalMedia",
    "dDescription": "wwDocTypeDesc_DigitalMedia",
    "dGif": "ucm_digital_asset.png"
    },
    {
    "dDocType": "Document",
    "dDescription": "wwDocTypeDesc_Document",
    "dGif": "ucm_document.png"
    },
    {
    "dDocType": "System",
    "dDescription": "wwDocTypeDesc_System",
    "dGif": "ucm_system_file.png"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Create a Document Type

#### POST /documents/wcc/api/v1.1/system/doctypes {#post-documentswccapiv11systemdoctypes}

#### Description

Create a new document type. (ADD_DOCTYPE)

#### Parameters

::: table-responsive
  Name           Located in   Description                                                                                                                              Required   Schema
  -------------- ------------ ---------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocType       formData     The unique identifier of document type.                                                                                                  Yes        string
  dDescription   formData     The description of the document type.                                                                                                    Yes        string
  dGif           formData     The file name of image for the document type. Note that the images are on the server in the `IntradocWebDir`/images/docgifs directory.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201    The document type was created.                                                                  
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-59}

Add a document type called `MyNewType`, with description `new type`, using the image `ucm_doc.png`.

##### Request

> POST .../documents/wcc/api/v1.1/system/doctypes

##### Request Body

FormData Parameters

> dDocType = MyNewType
>
> dDescription = new type
>
> dGif = ucm_doc.png

##### HTTP Response

> Status = 201

Body

None.

## Edit a Document Type

#### PUT /documents/wcc/api/v1.1/system/doctypes/{dDocType} {#put-documentswccapiv11systemdoctypesddoctype}

#### Description

Edit an existing document type. (EDIT_DOCTYPE)

#### Parameters

::: table-responsive
  Name           Located in   Description                                                                                                                              Required   Schema
  -------------- ------------ ---------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dDocType       path         The unique identifier of document type.                                                                                                  Yes        string
  dDescription   formData     The description of the document type.                                                                                                    Yes        string
  dGif           formData     The file name of image for the document type. Note that the images are on the server in the `IntradocWebDir`/images/docgifs directory.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully updated the document type.                                                         
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-60}

Edit the document type called `MyNewType`, update the description to `my new type`, update the image to `ucm_engineering.png`.

##### Request

> PUT .../documents/wcc/api/v1.1/system/doctypes/MyNewType

##### Request Body

FormData Parameters

> dDescription = my new type
>
> dGif = ucm_engineering.png

##### HTTP Response

> Status = 204

Body

None.

## Delete a Document Type

#### DELETE /documents/wcc/api/v1.1/system/doctypes/{dDocType} {#delete-documentswccapiv11systemdoctypesddoctype}

#### Description

Delete a document type. (DELETE_DOCTYPE)

#### Parameters

::: table-responsive
  Name       Located in   Description                               Required   Schema
  ---------- ------------ ----------------------------------------- ---------- --------
  dDocType   path         The unique identifier of document type.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted the document type.                                                         
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-61}

Delete the document type called `MyNewType`.

##### Request

> DELETE .../documents/wcc/api/v1.1/system/doctypes/MyNewType

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Configuration Information

#### GET /documents/wcc/api/v1.1/system/docConfigInfo {#get-documentswccapiv11systemdocconfiginfo}

#### Description

Get the server configuration information. (GET_DOC_CONFIG_INFO)

#### Parameters

::: table-responsive
  Name                Located in   Description                                   Required   Schema
  ------------------- ------------ --------------------------------------------- ---------- --------
  rowLimit            query        Limits the number of rows in all arrays.      No         number
  includeResultSets   query        A comma separated list of arrays to return.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -------------------------------------------------
  200    Successfully returned the configuration information.                                           [DocConfigInfoResponse](#docconfiginforesponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-62}

Get the server configuration information, limiting the arrays to 1 row.

##### Request

> GET .../documents/wcc/api/v1.1/system/docConfigInfo

##### Request Body

> query parameter rowLimit = 1

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"idc_name": "server",
"accessListPrivilegesGrantedWhenEmpty": 1,
"authorDelete": 1,
"foldersIndexParentFolderValues": 0,
"httpAbsoluteCgiUrl": "https://content.company.com/cs/idcplg",
"httpAbsoluteCgiUrlWebdav": "https://content.company.com/cs/idcplg/webdav",
"httpAbsoluteWebdavPath": "https://content.company.com/_dav/cs/idcplg/webdav",
"httpBaseAbsoluteRoot": "https://content.company.com",
"httpCgiPath": "/cs/idcplg",
"httpRelativeWebRoot": "/cs/",
"illegalUrlSegmentCharacters": ";/\\?:@&=+\"#%<>*~|[]iI",
"instanceDescription": "Instance server",
"instanceMenuLabel": "server",
"isAutoNumber": 1,
"isQueryObjectPersistent": null,
"localeDateFormatPattern": "yyyy-MM-dd HH:mm:ss",
"localeTimeZoneID": "America/Chicago",
"localeTimeZoneOffsetMillis": -21600000,
"maxResults": 200,
"memoFieldSize": 2000,
"productVersion": "12.2.1.0.0",
"productVersionInfo": "7.3.5.186",
"renditionListExportedForStaticAccess": null,
"searchIndexerEngineName": "DATABASE.METADATA",
"systemTimeZone": "America/Chicago",
"useAccounts": 0,
"useCollaboration": 0,
"useDatabaseWfInQueue": 1,
"useEntitySecurity": 0,
"acceptsResponseDateFormat": 1,
"dUser": "weblogic",
"webdavbaseurl": "https://content.company.com/_dav/cs/idcplg/webdav/core/tip",
"doc_default_infoCount": 0,
"docFormatsCount": 1,
"docMetaDefinitionCount": 1,
"docTypesCount": 1,
"featuresCount": 1,
"docFormats": [
    {
        "dFormat": "application/msword",
        "dConversion": "Word",
        "dDescription": "apMicrosoftWordDesc",
        "dIsEnabled": 1,
        "overrideStatus": "full",
        "isSystem": 1
    }
],
"docMetaDefinition": [
    {
        "dName": "xComments",
        "dType": "Memo",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "wwComments",
        "dIsOptionList": 0,
        "dOrder": 1,
        "dIsPlaceholderField": 0,
        "dDocMetaSet": "DocMeta"
    }
],
"docTypes": [
    {
        "dDocType": "Application",
        "dDescription": "wwDocTypeDesc_Application",
        "dGif": "ucm_application_file.png"
    }
],
"features": [
    {
        "idcFeatureName": "ConfigurationMigration",
        "idcFeatureVersion": "1.0",
        "idcFeatureLevel": "1.0.1.68",
        "idcFeatureComponent": "ConfigMigrationUtility"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Metadata Fields and DocType Information

#### GET /documents/wcc/api/v1.1/system/docMetaInfo {#get-documentswccapiv11systemdocmetainfo}

#### Description

Get metadata fields information and available docTypes (GET_DOC_METADATA_INFO).

#### Parameters

None.

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the configuration information.                                           [DocMetaInfoResponse](#docmetainforesponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-63}

Get the server metadata information.

##### Request

> GET .../documents/wcc/api/v1.1/system/docMetaInfo

##### Request Body

None

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"isAutoNumber": 1,
"useAccounts": 0,
"docMetaDefinitionCount": 17,
"docTypesCount": 7,
"docMetaDefinition": [
    {
        "dName": "xComments",
        "dType": "Memo",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "wwComments",
        "dIsOptionList": 0,
        "dOrder": 1,
        "dIsPlaceholderField": 0,
        "dDocMetaSet": "DocMeta"
    },
    {
        "dName": "xExternalDataSet",
        "dType": "BigText",
        "dIsRequired": 0,
        "dIsEnabled": 0,
        "dIsSearchable": 0,
        "dCaption": "wwExternalDataSet",
        "dIsOptionList": 0,
        "dOrder": 5,
        "dIsPlaceholderField": 0
    },
    {
        "dName": "xIdcProfile",
        "dType": "Text",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "wwProfile",
        "dIsOptionList": 1,
        "dOptionListKey": "view://ProfileTriggerValues",
        "dOrder": 7,
        "dOptionListType": "choice",
        "dIsPlaceholderField": 0
    },
    {
        "dName": "xTemplateType",
        "dType": "Text",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 0,
        "dCaption": "wwCheckinTemplateTemplateType",
        "dIsOptionList": 1,
        "dOptionListKey": "TemplateTypeList",
        "dOrder": 40,
        "dOptionListType": "choice",
        "dIsPlaceholderField": 0,
        "dComponentName": "DynamicConverter",
        "dOptionListValues": "HC Template,GUI Template,Layout Template,Script Template,Viewer Cache Template"
    },
    {
        "dName": "xAnnotationDetails",
        "dType": "Int",
        "dIsRequired": 0,
        "dIsEnabled": 0,
        "dIsSearchable": 0,
        "dCaption": "apAnnotationDetails",
        "dIsOptionList": 0,
        "dDefaultValue": "0",
        "dOrder": 100,
        "dIsPlaceholderField": 0,
        "dComponentName": "Imaging"
    },
    {
        "dName": "xIsACLReadOnlyOnUI",
        "dType": "Int",
        "dIsRequired": 0,
        "dIsEnabled": 0,
        "dIsSearchable": 0,
        "dCaption": "wwFldIsACLReadOnlyOnUI",
        "dIsOptionList": 0,
        "dDefaultValue": "0",
        "dOrder": 101,
        "dIsPlaceholderField": 0
    },
    {
        "dName": "xLibraryGUID",
        "dType": "BigText",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "wwFldLibrary",
        "dIsOptionList": 0,
        "dOrder": 10000,
        "dIsPlaceholderField": 0,
        "dComponentName": "FrameworkFolders"
    },
    {
        "dName": "xPartitionId",
        "dType": "Text",
        "dIsRequired": 0,
        "dIsEnabled": 0,
        "dIsSearchable": 1,
        "dCaption": "apFsPartitionId",
        "dIsOptionList": 0,
        "dOrder": 10600,
        "dIsPlaceholderField": 0,
        "dComponentName": "FileStoreProvider"
    },
    {
        "dName": "xWebFlag",
        "dType": "Text",
        "dIsRequired": 0,
        "dIsEnabled": 0,
        "dIsSearchable": 1,
        "dCaption": "apFsWebFlag",
        "dIsOptionList": 0,
        "dOrder": 10601,
        "dIsPlaceholderField": 0,
        "dComponentName": "FileStoreProvider"
    },
    {
        "dName": "xStorageRule",
        "dType": "Text",
        "dIsRequired": 0,
        "dIsEnabled": 0,
        "dIsSearchable": 1,
        "dCaption": "apFsStorageRule",
        "dIsOptionList": 1,
        "dOptionListKey": "view://StorageRuleView",
        "dDefaultValue": "DispByContentId",
        "dOrder": 10602,
        "dOptionListType": "choice",
        "dIsPlaceholderField": 0,
        "dComponentName": "FileStoreProvider"
    },
    {
        "dName": "xTest",
        "dType": "Int",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 0,
        "dCaption": "TestInt",
        "dIsOptionList": 0,
        "dOrder": 10603,
        "dIsPlaceholderField": 0,
        "dDecimalScale": 1,
        "dDocMetaSet": "DocMeta"
    },
    {
        "dName": "xReqText",
        "dType": "Text",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "ReqText",
        "dIsOptionList": 0,
        "dOrder": 10604,
        "dIsPlaceholderField": 0,
        "dDecimalScale": 1,
        "dDocMetaSet": "DocMeta"
    },
    {
        "dName": "xLongText",
        "dType": "BigText",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "LongText",
        "dIsOptionList": 0,
        "dOrder": 10605,
        "dIsPlaceholderField": 0,
        "dDecimalScale": 1,
        "dDocMetaSet": "DocMeta"
    },
    {
        "dName": "xMemo",
        "dType": "Memo",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "Memo",
        "dIsOptionList": 0,
        "dDefaultValue": "Test memo",
        "dOrder": 10606,
        "dIsPlaceholderField": 0,
        "dDecimalScale": 1,
        "dDocMetaSet": "DocMeta"
    },
    {
        "dName": "xDate",
        "dType": "Date",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "Date",
        "dIsOptionList": 0,
        "dOrder": 10608,
        "dIsPlaceholderField": 0,
        "dDecimalScale": 1,
        "dDocMetaSet": "DocMeta"
    },
    {
        "dName": "xDecimal",
        "dType": "Decimal",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "Decimal",
        "dIsOptionList": 0,
        "dOrder": 10609,
        "dIsPlaceholderField": 0,
        "dDecimalScale": 2,
        "dDocMetaSet": "DocMeta"
    },
    {
        "dName": "xOptionList",
        "dType": "Text",
        "dIsRequired": 0,
        "dIsEnabled": 1,
        "dIsSearchable": 1,
        "dCaption": "OptionList",
        "dIsOptionList": 1,
        "dOptionListKey": "OptionListList",
        "dOrder": 10610,
        "dOptionListType": "choice",
        "dIsPlaceholderField": 0,
        "dDecimalScale": 1,
        "dDocMetaSet": "DocMeta",
        "dOptionListValues": "yes,no,maybe"
    }
],
"docTypes": [
    {
        "dDocType": "Application",
        "dDescription": "wwDocTypeDesc_Application",
        "dGif": "ucm_application_file.png"
    },
    {
        "dDocType": "Binary",
        "dDescription": "wwDocTypeDesc_Binary",
        "dGif": "ucm_binaryfile.png"
    },
    {
        "dDocType": "DigitalMedia",
        "dDescription": "wwDocTypeDesc_DigitalMedia",
        "dGif": "ucm_digital_asset.png"
    },
    {
        "dDocType": "Document",
        "dDescription": "wwDocTypeDesc_Document",
        "dGif": "ucm_document.png"
    },
    {
        "dDocType": "System",
        "dDescription": "wwDocTypeDesc_System",
        "dGif": "ucm_system_file.png"
    },
    {
        "dDocType": "TEST_DocType3",
        "dDescription": "Test DocType3",
        "dGif": "ucm_document.png"
    },
    {
        "dDocType": "type2",
        "dDescription": "test",
        "dGif": "ucm_doc.png"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Get the Associated Taxonomies for a Security Group

#### GET /documents/wcc/api/v1.1/system/securityGroups/{dSecurityGroup}/taxonomies {#get-documentswccapiv11systemsecuritygroupsdsecuritygrouptaxonomies}

#### Description

List the taxonomies associated with a security group. (TXY_GET_SECURITY_GROUP_TAXONOMIES) User must have R access to the security group.

#### Parameters

::: table-responsive
  Name             Located in   Description                                                                                                                                Required   Schema
  ---------------- ------------ ------------------------------------------------------------------------------------------------------------------------------------------ ---------- --------
  dSecurityGroup   path         The name of a security group.                                                                                                              Yes        string
  sortField        query        The field used to sort the taxonomies; fields can be sorted by `dTaxonomyName` or `dLastModifiedDate`. The default is `dTaxonomyName``.`   No         string
  sortOrder        query        The order to sort the taxonomies; supported values are `Asc` and `Desc`. The default is `Asc`.                                             No         string
  startRow         query        The row number at which to start returning the taxonomies. This is for pagination. The defaults is 0.                                      No         number
  count            query        The number of taxonomies to return. This is for pagination. The dDefault is 20; the maximum is 100.                                        No         number
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------------------
  200    Returned taxonomy assignments successfully.                                                    [TaxonomyListResponseObject](#taxonomylistresponseobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  404    The dSecurityGroup is invalid.                                                                  
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

# Folders

## Browse the root folder

#### GET /documents/wcc/api/v1.1/folders/browse/ {#get-documentswccapiv11foldersbrowse}

#### Description

List the content and structure of the root (FLD_ROOT) folder. (FLD_BROWSE)

#### Parameters

::: table-responsive
  Name                    Located in   Description                                                                                                                                                                                                Required   Schema
  ----------------------- ------------ ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- ---------
  fldapp                  query        Specifies the Folders Application of the location being returned                                                                                                                                           No         string
  doCombinedBrowse        query        When true, data will be returned in a combined pagination mode.                                                                                                                                            No         boolean
  foldersFirst            query        When true, folders are listed before files in the combined pagination mode.                                                                                                                                No         boolean
  folderCount             query        The number of folders to return.                                                                                                                                                                           No         integer
  folderStartRow          query        The row number at which to start returning data. Used for pagination.                                                                                                                                      No         integer
  fileCount               query        The number of files to return.                                                                                                                                                                             No         integer
  fileStartRow            query        The row number at which to start returning data. Used for pagination.                                                                                                                                      No         integer
  combinedCount           query        The number of items (folders+files) to return.                                                                                                                                                             No         integer
  combinedStartRow        query        The row number at which to start returning data. Used for pagination.                                                                                                                                      No         integer
  doRetrieveTargetInfo    query        When true, returns target folder's information for all shortcuts retrieved in a resultset ChildTargetFolders.                                                                                              No         boolean
  doMarkFavorites         query        When true, adds a 'fIsFavorite' field in the resultset to indicate if the folder/file is favorite. It is favorite if it has a shortcut in Favorites folder.                                                No         boolean
  doMarkSubscribed        query        When true, adds a 'fIsSubscribed' field in the resultset to indicate if the folder is subscribed. A folder is subscribed if the Subscription table has an entry for folder and the user calling the API.   No         boolean
  doRetrieveDocumentURL   query        When true, adds a URL field returned data; the web location of the document.                                                                                                                               No         boolean
  doRetrieveUniqueLinks   query        When true, post processes array to return only unique links. For example, if folder's shortcut and folder itself are in the array, only folder itself will be returned.                                    No         boolean
  foldersSortField        query        The field Name from FolderFolders table on which to sort the records.                                                                                                                                      No         string
  foldersSortOrder        query        Sort order on `foldersSortField` field.                                                                                                                                                                    No         string
  filesSortField          query        The field name from FolderFiles table on which to sort the records.                                                                                                                                        No         string
  filesSortOrder          query        Sort order on `filesSortField` field.                                                                                                                                                                      No         string
  combinedSortField       query        The Field name (common to both FolderFiles and FolderFolders tables) on which to sort the items.                                                                                                           No         string
  combinedSortOrder       query        Sort order on `combinedSortField` field.                                                                                                                                                                   No         string
  foldersFilterParams     query        The comma separated list of filter parameters on the browse. For example `foldersFilterParams=fIsContribution&fIsContribution=1` will return folders with fIsContribution=1.                               No         string
  foldersFilterQuery      query        The Standard Query syntax for filtering out the folders. This is in DATABASE engine format on FolderFolders table. For example. `foldersFilterQuery=fFolderName'ent'</code>`                               No         string
  fieldName               query        The value for the specified field in `foldersFilterParams` and `filesFilterParams`                                                                                                                         No         string
  filesFilterParams       query        The comma separated list of filter parameters on the browse. For example, `filesFilterParams=fOwner&fOwner=sysadmin` will return folders with fOwner=sysadmin                                              No         string
  filesFilterQuery        query        The Standard Query syntax for filtering out the files. This is in DATABASE engine format on FolderFiles table. For example, `filesFilterQuery=fFileName'test'`                                             No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    The structure of a folder.                                                                     [FolderBrowseResponse](#folderbrowseresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-64}

Browse the FLD_ROOT folder. This example shows an additional folder and file.

##### Request

> GET .../documents/wcc/api/v1.1/folders/browse/

##### Request Body

None

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"numFolders": 3,
"hasMoreChildFolders": 0,
"numFiles": 1,
"totalChildFoldersCount": 3,
"totalChildFilesCount": 1,
"hasMoreChildFiles": 0,
"hasMoreChildItems": 0,
"folderPath": "/",
"folderInfo": {
    "fFolderGUID": "FLD_ROOT",
    "fParentGUID": "idcnull",
    "fFolderName": "/",
    "fFolderType": "owner",
    "fInhibitPropagation": 0,
    "fPromptForMetadata": 0,
    "fIsContribution": 1,
    "fIsInTrash": 0,
    "fRealItemGUID": null,
    "fLibraryType": null,
    "fIsLibrary": 0,
    "fDocClasses": null,
    "fTargetGUID": null,
    "fApplication": "framework",
    "fOwner": "sysadmin",
    "fCreator": "sysadmin",
    "fLastModifier": "sysadmin",
    "fCreateDate": "2024-03-18 20:15:54Z",
    "fLastModifiedDate": "2024-03-18 20:15:54Z",
    "fSecurityGroup": "Public",
    "fDocAccount": null,
    "fClbraUserList": null,
    "fClbraAliasList": null,
    "fClbraRoleList": null,
    "fFolderDescription": null,
    "fChildFoldersCount": 5,
    "fChildFilesCount": 1,
    "fFolderSize": 0,
    "fAllocatedFolderSize": -1,
    "fAllocatorParentFolderGUID": null,
    "fApplicationGUID": null,
    "fIsReadOnly": 0,
    "fIsACLReadOnlyOnUI": 0,
    "fDisplayName": "/",
    "fDisplayDescription": null,
    "fIsBrokenShortcut": null,
    "isLeaf": "0",
    "itemType": "1",
    "folderPermissions": "RWDA"
},
"childFolders": [
    {
        "fFolderGUID": "FLD_ENTERPRISE_LIBRARY",
        "fParentGUID": "FLD_ROOT",
        "fFolderName": "Enterprise Libraries",
        "fFolderType": "owner",
        "fInhibitPropagation": 0,
        "fPromptForMetadata": 0,
        "fIsContribution": 1,
        "fIsInTrash": 0,
        "fRealItemGUID": null,
        "fLibraryType": null,
        "fIsLibrary": 0,
        "fDocClasses": null,
        "fTargetGUID": null,
        "fApplication": "framework",
        "fOwner": "sysadmin",
        "fCreator": "sysadmin",
        "fLastModifier": "sysadmin",
        "fCreateDate": "2024-03-18 20:15:54Z",
        "fLastModifiedDate": "2024-03-18 20:15:54Z",
        "fSecurityGroup": "Public",
        "fDocAccount": null,
        "fClbraUserList": null,
        "fClbraAliasList": null,
        "fClbraRoleList": null,
        "fFolderDescription": "Enterprise Libraries",
        "fChildFoldersCount": 0,
        "fChildFilesCount": 0,
        "fFolderSize": 0,
        "fAllocatedFolderSize": -1,
        "fAllocatorParentFolderGUID": null,
        "fApplicationGUID": null,
        "fIsReadOnly": 0,
        "fIsACLReadOnlyOnUI": 0,
        "fDisplayName": "Enterprise Libraries",
        "fDisplayDescription": "Enterprise Libraries"
    },
    {
        "fFolderGUID": "D6413BD3FD74C638A5ED5352AEF456C1",
        "fParentGUID": "FLD_ROOT",
        "fFolderName": "Folder",
        "fFolderType": "owner",
        "fInhibitPropagation": 0,
        "fPromptForMetadata": 0,
        "fIsContribution": 1,
        "fIsInTrash": 0,
        "fRealItemGUID": null,
        "fLibraryType": null,
        "fIsLibrary": 0,
        "fDocClasses": null,
        "fTargetGUID": null,
        "fApplication": "framework",
        "fOwner": "weblogic",
        "fCreator": "weblogic",
        "fLastModifier": "weblogic",
        "fCreateDate": "2024-05-22 20:46:39Z",
        "fLastModifiedDate": "2024-05-22 20:46:39Z",
        "fSecurityGroup": "Public",
        "fDocAccount": null,
        "fClbraUserList": null,
        "fClbraAliasList": null,
        "fClbraRoleList": null,
        "fFolderDescription": null,
        "fChildFoldersCount": 0,
        "fChildFilesCount": 0,
        "fFolderSize": 0,
        "fAllocatedFolderSize": -1,
        "fAllocatorParentFolderGUID": null,
        "fApplicationGUID": null,
        "fIsReadOnly": 0,
        "fIsACLReadOnlyOnUI": 0,
        "fDisplayName": "Folder",
        "fDisplayDescription": null
    },
    {
        "fFolderGUID": "FLD_USERS",
        "fParentGUID": "FLD_ROOT",
        "fFolderName": "Users",
        "fFolderType": "owner",
        "fInhibitPropagation": 0,
        "fPromptForMetadata": 0,
        "fIsContribution": 1,
        "fIsInTrash": 0,
        "fRealItemGUID": null,
        "fLibraryType": null,
        "fIsLibrary": 0,
        "fDocClasses": null,
        "fTargetGUID": null,
        "fApplication": "framework",
        "fOwner": "sysadmin",
        "fCreator": "sysadmin",
        "fLastModifier": "sysadmin",
        "fCreateDate": "2024-03-18 20:15:54Z",
        "fLastModifiedDate": "2024-03-18 20:15:54Z",
        "fSecurityGroup": "Public",
        "fDocAccount": null,
        "fClbraUserList": null,
        "fClbraAliasList": null,
        "fClbraRoleList": null,
        "fFolderDescription": null,
        "fChildFoldersCount": 0,
        "fChildFilesCount": 0,
        "fFolderSize": 0,
        "fAllocatedFolderSize": -1,
        "fAllocatorParentFolderGUID": null,
        "fApplicationGUID": null,
        "fIsReadOnly": 0,
        "fIsACLReadOnlyOnUI": 0,
        "fDisplayName": "Users",
        "fDisplayDescription": "Users"
    }
],
"childFiles": [
    {
        "fFileGUID": "D7B229794E0E9A57985D771CD070C750",
        "fParentGUID": "FLD_ROOT",
        "fTargetGUID": null,
        "fFileName": "Mugs.jpg",
        "fPublishedFileName": "Mugs.jpg",
        "fFileType": "owner",
        "fIsInTrash": 0,
        "fRealItemGUID": null,
        "fDocClass": "Base",
        "fInhibitPropagation": 0,
        "dDocName": "ID14006204",
        "dRevisionID": 1,
        "dPublishedRevisionID": 1,
        "fApplication": "framework",
        "fOwner": "weblogic",
        "fCreator": "weblogic",
        "fLastModifier": "weblogic",
        "fCreateDate": "2024-05-22 20:48:40Z",
        "fLastModifiedDate": "2024-05-22 20:48:40Z",
        "fSecurityGroup": "Public",
        "fDocAccount": null,
        "fClbraUserList": null,
        "fClbraAliasList": null,
        "fClbraRoleList": null,
        "fIsACLReadOnlyOnUI": "0",
        "dID": 6207,
        "dDocType": "Document",
        "dDocTitle": "file",
        "dDocAuthor": "weblogic",
        "dRevClassID": 6205,
        "dRevLabel": "1",
        "dIsCheckedOut": 0,
        "dCheckoutUser": null,
        "dSecurityGroup": "Public",
        "dCreateDate": "2024-05-22 20:48:40Z",
        "dInDate": "2024-05-22 20:46:00Z",
        "dOutDate": null,
        "dStatus": "RELEASED",
        "dReleaseState": "Y",
        "dFlag1": null,
        "dWebExtension": "jpg",
        "dProcessingState": "Y",
        "dMessage": null,
        "dDocAccount": null,
        "dReleaseDate": "2024-05-22 20:48:42Z",
        "dRendition1": null,
        "dRendition2": null,
        "dIndexerState": null,
        "dPublishType": null,
        "dPublishState": null,
        "dWorkflowState": null,
        "dRevRank": 0,
        "dDocID": 6413,
        "dIsPrimary": 1,
        "dIsWebFormat": 0,
        "dLocation": null,
        "dOriginalName": "Mugs.jpg",
        "dFormat": "image/jpeg",
        "dExtension": "jpg",
        "dFileSize": 53834,
        "dLanguage": null,
        "dCharacterSet": null,
        "xComments": null,
        "xExternalDataSet": null,
        "xIdcProfile": null,
        "xAnnotationDetails": 0,
        "xPartitionId": null,
        "xWebFlag": null,
        "xStorageRule": "DispByContentId",
        "xTest": 0,
        "xReqText": null,
        "xLongText": null,
        "xMemo": "Test memo",
        "xDate": null,
        "xDecimal": null,
        "xOptionList": null,
        "xLibraryGUID": null,
        "xIsACLReadOnlyOnUI": 0,
        "dIndexedID": 6207,
        "dDocCreatedDate": "2024-05-22 20:48:40Z",
        "dDocCreator": "weblogic",
        "dDocLastModifiedDate": "2024-05-22 20:48:40Z",
        "dDocLastModifier": "weblogic",
        "dDocOwner": "weblogic",
        "dDocFunction": null,
        "dDocClass": null,
        "fDisplayName": "Mugs.jpg"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Browse a folder

#### GET /documents/wcc/api/v1.1/folders/browse/{fFolderGUID} {#get-documentswccapiv11foldersbrowseffolderguid}

#### Description

List the content and structure of the specified folder. (FLD_BROWSE)

#### Parameters

::: table-responsive
  Name                    Located in   Description                                                                                                                                                                                                Required   Schema
  ----------------------- ------------ ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- ---------
  fFolderGUID             path         The folder GUID.                                                                                                                                                                                           Yes        string
  fldapp                  query        Specifies the Folders Application of the location being returned                                                                                                                                           No         string
  doCombinedBrowse        query        When true, data will be returned in a combined pagination mode.                                                                                                                                            No         boolean
  foldersFirst            query        When true, folders are listed before files in the combined pagination mode.                                                                                                                                No         boolean
  folderCount             query        The number of folders to return.                                                                                                                                                                           No         integer
  folderStartRow          query        The row number at which to start returning data. Used for pagination.                                                                                                                                      No         integer
  fileCount               query        The number of files to return.                                                                                                                                                                             No         integer
  fileStartRow            query        The row number at which to start returning data. Used for pagination.                                                                                                                                      No         integer
  combinedCount           query        The number of items (folders+files) to return.                                                                                                                                                             No         integer
  combinedStartRow        query        The row number at which to start returning data. Used for pagination.                                                                                                                                      No         integer
  doRetrieveTargetInfo    query        When true, returns target folder's information for all shortcuts retrieved in a resultset ChildTargetFolders.                                                                                              No         boolean
  doMarkFavorites         query        When true, adds a 'fIsFavorite' field in the resultset to indicate if the folder/file is favorite. It is favorite if it has a shortcut in Favorites folder.                                                No         boolean
  doMarkSubscribed        query        When true, adds a 'fIsSubscribed' field in the resultset to indicate if the folder is subscribed. A folder is subscribed if the Subscription table has an entry for folder and the user calling the API.   No         boolean
  doRetrieveDocumentURL   query        When true, adds a URL field returned data; the web location of the document.                                                                                                                               No         boolean
  doRetrieveUniqueLinks   query        When true, post processes array to return only unique links. For example, if folder's shortcut and folder itself are in the array, only folder itself will be returned.                                    No         boolean
  foldersSortField        query        The field Name from FolderFolders table on which to sort the records.                                                                                                                                      No         string
  foldersSortOrder        query        Sort order on `foldersSortField` field.                                                                                                                                                                    No         string
  filesSortField          query        The field name from FolderFiles table on which to sort the records.                                                                                                                                        No         string
  filesSortOrder          query        Sort order on `filesSortField` field.                                                                                                                                                                      No         string
  combinedSortField       query        The Field name (common to both FolderFiles and FolderFolders tables) on which to sort the items.                                                                                                           No         string
  combinedSortOrder       query        Sort order on `combinedSortField` field.                                                                                                                                                                   No         string
  foldersFilterParams     query        The comma separated list of filter parameters on the browse. For example `foldersFilterParams=fIsContribution&fIsContribution=1` will return folders with fIsContribution=1                                No         string
  foldersFilterQuery      query        The Standard Query syntax for filtering out the folders. This is in DATABASE engine format on FolderFolders table. For example. `foldersFilterQuery=fFolderName'ent'</code>`                               No         string
  fieldName               query        The value for the specified field in `foldersFilterParams` and `filesFilterParams`                                                                                                                         No         string
  filesFilterParams       query        The comma separated list of filter parameters on the browse. For example, `filesFilterParams=fOwner&fOwner=sysadmin` will return folders with fOwner=sysadmin                                              No         string
  filesFilterQuery        query        The Standard Query syntax for filtering out the files. This is in DATABASE engine format on FolderFiles table. For example, `filesFilterQuery=fFileName'test'`                                             No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    The structure of a folder.                                                                     [FolderBrowseResponse](#folderbrowseresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-65}

Browse any folder.

##### Request

> GET .../documents/wcc/api/v1.1/folders/browse/D6413BD3FD74C638A5ED5352AEF456C1

##### Request Body

None

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"numFolders": 1,
"hasMoreChildFolders": 0,
"numFiles": 1,
"totalChildFoldersCount": 1,
"totalChildFilesCount": 1,
"hasMoreChildFiles": 0,
"hasMoreChildItems": 0,
"folderPath": "/Folder",
"folderInfo": {
    "fFolderGUID": "D6413BD3FD74C638A5ED5352AEF456C1",
    "fParentGUID": "FLD_ROOT",
    "fFolderName": "Folder",
    "fFolderType": "owner",
    "fInhibitPropagation": 0,
    "fPromptForMetadata": 0,
    "fIsContribution": 1,
    "fIsInTrash": 0,
    "fRealItemGUID": null,
    "fLibraryType": null,
    "fIsLibrary": 0,
    "fDocClasses": null,
    "fTargetGUID": null,
    "fApplication": "framework",
    "fOwner": "weblogic",
    "fCreator": "weblogic",
    "fLastModifier": "weblogic",
    "fCreateDate": "2024-05-22 20:46:39Z",
    "fLastModifiedDate": "2024-05-22 20:46:39Z",
    "fSecurityGroup": "Public",
    "fDocAccount": null,
    "fClbraUserList": null,
    "fClbraAliasList": null,
    "fClbraRoleList": null,
    "fFolderDescription": null,
    "fChildFoldersCount": 1,
    "fChildFilesCount": 1,
    "fFolderSize": 0,
    "fAllocatedFolderSize": -1,
    "fAllocatorParentFolderGUID": null,
    "fApplicationGUID": null,
    "fIsReadOnly": 0,
    "fIsACLReadOnlyOnUI": 0,
    "fDisplayName": "Folder",
    "fDisplayDescription": null,
    "fIsBrokenShortcut": null,
    "isLeaf": "0",
    "itemType": "1",
    "folderPermissions": "RWDA"
},
"childFolders": [
    {
        "fFolderGUID": "4B4AFD71D5A999DBABB16704FEABCEBD",
        "fParentGUID": "D6413BD3FD74C638A5ED5352AEF456C1",
        "fFolderName": "jpgs",
        "fFolderType": "owner",
        "fInhibitPropagation": 17,
        "fPromptForMetadata": 0,
        "fIsContribution": 0,
        "fIsInTrash": 0,
        "fRealItemGUID": null,
        "fLibraryType": null,
        "fIsLibrary": 0,
        "fDocClasses": null,
        "fTargetGUID": null,
        "fApplication": "query",
        "fOwner": "weblogic",
        "fCreator": "weblogic",
        "fLastModifier": "weblogic",
        "fCreateDate": "2024-05-22 21:01:13Z",
        "fLastModifiedDate": "2024-05-22 21:01:13Z",
        "fSecurityGroup": "Public",
        "fDocAccount": null,
        "fClbraUserList": null,
        "fClbraAliasList": null,
        "fClbraRoleList": null,
        "fFolderDescription": null,
        "fChildFoldersCount": 0,
        "fChildFilesCount": 0,
        "fFolderSize": 0,
        "fAllocatedFolderSize": -1,
        "fAllocatorParentFolderGUID": null,
        "fApplicationGUID": null,
        "fIsReadOnly": 0,
        "fIsACLReadOnlyOnUI": 0,
        "fDisplayName": "jpgs",
        "fDisplayDescription": null
    }
],
"childFiles": [
    {
        "fFileGUID": "2D09591B678E8AF05D8F6B1EB1879EA9",
        "fParentGUID": "D6413BD3FD74C638A5ED5352AEF456C1",
        "fTargetGUID": null,
        "fFileName": "sample.txt",
        "fPublishedFileName": "sample.txt",
        "fFileType": "owner",
        "fIsInTrash": 0,
        "fRealItemGUID": null,
        "fDocClass": "Base",
        "fInhibitPropagation": 0,
        "dDocName": "ID798656784",
        "dRevisionID": 1,
        "dPublishedRevisionID": 1,
        "fApplication": "framework",
        "fOwner": "weblogic",
        "fCreator": "weblogic",
        "fLastModifier": "weblogic",
        "fCreateDate": "2024-05-22 21:03:08Z",
        "fLastModifiedDate": "2024-05-22 21:03:08Z",
        "fSecurityGroup": "Public",
        "fDocAccount": null,
        "fClbraUserList": null,
        "fClbraAliasList": null,
        "fClbraRoleList": null,
        "fIsACLReadOnlyOnUI": "0",
        "dID": 6208,
        "dDocType": "Document",
        "dDocTitle": "sample",
        "dDocAuthor": "weblogic",
        "dRevClassID": 6206,
        "dRevLabel": "1",
        "dIsCheckedOut": 0,
        "dCheckoutUser": null,
        "dSecurityGroup": "Public",
        "dCreateDate": "2024-05-22 21:03:08Z",
        "dInDate": "2024-05-22 21:01:00Z",
        "dOutDate": null,
        "dStatus": "RELEASED",
        "dReleaseState": "Y",
        "dFlag1": null,
        "dWebExtension": "txt",
        "dProcessingState": "Y",
        "dMessage": null,
        "dDocAccount": null,
        "dReleaseDate": "2024-05-22 21:03:09Z",
        "dRendition1": null,
        "dRendition2": null,
        "dIndexerState": null,
        "dPublishType": null,
        "dPublishState": null,
        "dWorkflowState": null,
        "dRevRank": 0,
        "dDocID": 6415,
        "dIsPrimary": 1,
        "dIsWebFormat": 0,
        "dLocation": null,
        "dOriginalName": "sample.txt",
        "dFormat": "text/plain",
        "dExtension": "txt",
        "dFileSize": 6,
        "dLanguage": null,
        "dCharacterSet": null,
        "xComments": null,
        "xExternalDataSet": null,
        "xIdcProfile": null,
        "xAnnotationDetails": 0,
        "xPartitionId": null,
        "xWebFlag": null,
        "xStorageRule": "DispByContentId",
        "xTest": 0,
        "xReqText": null,
        "xLongText": null,
        "xMemo": "Test memo",
        "xDate": null,
        "xDecimal": null,
        "xOptionList": null,
        "xLibraryGUID": null,
        "xIsACLReadOnlyOnUI": 0,
        "dIndexedID": 6208,
        "dDocCreatedDate": "2024-05-22 21:03:08Z",
        "dDocCreator": "weblogic",
        "dDocLastModifiedDate": "2024-05-22 21:03:08Z",
        "dDocLastModifier": "weblogic",
        "dDocOwner": "weblogic",
        "dDocFunction": null,
        "dDocClass": null,
        "fDisplayName": "sample.txt"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Create Folder

#### POST /documents/wcc/api/v1.1/folders {#post-documentswccapiv11folders}

#### Description

Create a folder or a shortcut to a folder. (FLD_CREATE_FOLDER)

#### Parameters

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                                      Located in     Description                                                                                                               Required       Schema
  ----------------------------------------- -------------- ------------------------------------------------------------------------------------------------------------------------- -------------- --------------
  fFolderName                               query          The name of the created folder.                                                                                           Yes            string

  fParentGUID                               query          The GUID of the parent folder where the new folder will be created.                                                       Yes            string

  fSecurityGroup                            query          The security group for the new folder. The default is `Public`.                                                           No             string

  fFolderType                               query          The type of folder to create.                                                                                             No             string

  fOwner                                    query          The owner of the folder. The default is the user making the call.                                                         No             string

  fInhibitPropagation                       query          A bitmap used to determine restrictions on files/folders from propagating values from its parent. The bitmap includes:\   No             number
                                                           0 - inhibit propagation is set to none indicating values can be inherited from parent\                                                   
                                                           1 - inhibit propagation for metadata indicating metadata is not inherited from parent\                                                   
                                                           16 - inhibit propagation for folder security indicating folder security is not inherited from parent\                                    
                                                           17 - inhibit propagation for metadata and folder security                                                                                

  fTargetGUID                               query          The GUID of the target folder when creating a folder shortcut.                                                            No             string

  ConflictResolutionMethod                  query          The method used to resolve folder name conflict:\                                                                         No             string
                                                           •ResolveDuplicates -- The created folder will be given a unique name based on the `fFolderName` parameter\                               
                                                           •SkipDuplicates -- The folder will not be created if the name already exists                                                             

  isForceInheritSecurityForFolderCreation   query          When true, the folder will inherit the security from the parent folder.                                                   No             boolean
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                                 Schema
  ----------------------- ----------------------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Folder successfully created.\                                                                                
                          Returns Header `Location` which is a URI to get the metadata of the new folder.                             

  400                     Bad request                                                                                                 [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                                 

  403                     User is not allowed to take this action                                                                      

  409                     If `ConflictResolutionMethod` is `SkipDuplicates` and the folder name exists, a new folder is not created    

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request                [GeneralErrorResponse](#generalerrorresponse)
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-66}

Create a folder named `RestCreatedFolder` under FLD_ROOT

##### Request

> POST .../documents/wcc/api/v1.1/folders?fParentGUID=FLD_ROOT&fFolderName=RestCreatedFolder

##### Request Parameters

> fParentGUID = FLD_ROOT
>
> fFolderName = RestCreatedFolder

##### Request Body

None.

##### HTTP Response

> Status = 201

Header

> Location = .../documents/wcc/api/v1.1/folders/BE8851AE80FBAFAB3C2BB4AA127402C0

## Get Folder Information

#### GET /documents/wcc/api/v1.1/folders/{fFolderGUID} {#get-documentswccapiv11foldersffolderguid}

#### Description

Get information about the specified folder (FLD_INFO).

#### Parameters

::: table-responsive
  Name          Located in   Description       Required   Schema
  ------------- ------------ ----------------- ---------- --------
  fFolderGUID   path         The folder GUID   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Folder information.                                                                            [FolderInfoResponse](#folderinforesponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-67}

Get folder information for the folder with GUID `D6413BD3FD74C638A5ED5352AEF456C1`.

##### Request

> GET .../documents/wcc/api/v1.1/folders/D6413BD3FD74C638A5ED5352AEF456C1

##### Request Body

None

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"folderPath": "/Folder",
"folderInfo": {
    "fFolderGUID": "D6413BD3FD74C638A5ED5352AEF456C1",
    "fParentGUID": "FLD_ROOT",
    "fFolderName": "Folder",
    "fFolderType": "owner",
    "fInhibitPropagation": 0,
    "fPromptForMetadata": 0,
    "fIsContribution": 1,
    "fIsInTrash": 0,
    "fRealItemGUID": null,
    "fLibraryType": null,
    "fIsLibrary": 0,
    "fDocClasses": null,
    "fTargetGUID": null,
    "fApplication": "framework",
    "fOwner": "weblogic",
    "fCreator": "weblogic",
    "fLastModifier": "weblogic",
    "fCreateDate": "2024-05-22 20:46:39Z",
    "fLastModifiedDate": "2024-05-22 20:46:39Z",
    "fSecurityGroup": "Public",
    "fDocAccount": null,
    "fClbraUserList": null,
    "fClbraAliasList": null,
    "fClbraRoleList": null,
    "fFolderDescription": null,
    "fChildFoldersCount": 1,
    "fChildFilesCount": 1,
    "fFolderSize": 0,
    "fAllocatedFolderSize": -1,
    "fAllocatorParentFolderGUID": null,
    "fApplicationGUID": null,
    "fIsReadOnly": 0,
    "fIsACLReadOnlyOnUI": 0,
    "fDisplayName": "Folder",
    "fDisplayDescription": null,
    "fIsBrokenShortcut": null,
    "isLeaf": "0",
    "itemType": "1",
    "folderPermissions": "RWDA"
}
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Get File Information

#### GET /documents/wcc/api/v1.1/folders/files/{fFileGUID} {#get-documentswccapiv11foldersfilesffileguid}

#### Description

Get information about the specified file in a folder. (FLD_INFO)

#### Parameters

::: table-responsive
  Name        Located in   Description     Required   Schema
  ----------- ------------ --------------- ---------- --------
  fFileGUID   path         The file GUID   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    File information and metadata.                                                                 [FileInfoResponse](#fileinforesponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-68}

Get file information for the file with GUID `D7B229794E0E9A57985D771CD070C750`.

##### Request

> GET .../documents/wcc/api/v1.1/folders/files/D7B229794E0E9A57985D771CD070C750

##### Request Body

None

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"filePath": "/Mugs.jpg",
"fileInfo": {
    "fFileGUID": "D7B229794E0E9A57985D771CD070C750",
    "fParentGUID": "FLD_ROOT",
    "fTargetGUID": null,
    "fFileName": "Mugs.jpg",
    "fPublishedFileName": "Mugs.jpg",
    "fFileType": "owner",
    "fIsInTrash": 0,
    "fRealItemGUID": null,
    "fDocClass": "Base",
    "fInhibitPropagation": 0,
    "dDocName": "ID14006204",
    "dRevisionID": 1,
    "dPublishedRevisionID": 1,
    "fApplication": "framework",
    "fOwner": "weblogic",
    "fCreator": "weblogic",
    "fLastModifier": "weblogic",
    "fCreateDate": "2024-05-22 20:48:40Z",
    "fLastModifiedDate": "2024-05-22 20:48:40Z",
    "fSecurityGroup": "Public",
    "fDocAccount": null,
    "fClbraUserList": null,
    "fClbraAliasList": null,
    "fClbraRoleList": null,
    "fIsACLReadOnlyOnUI": "0",
    "fDisplayName": "Mugs.jpg",
    "fDisplayDescription": null,
    "dCharacterSet": null,
    "dCheckoutUser": null,
    "dCreateDate": "2024-05-22 20:48:40Z",
    "dDocAccount": null,
    "dDocAuthor": "weblogic",
    "dDocCreatedDate": "{ts '2024-05-22 15:48:40.640'}",
    "dDocCreator": "weblogic",
    "dDocID": 6413,
    "dDocLastModifiedDate": "{ts '2024-05-22 15:48:40.640'}",
    "dDocLastModifier": "weblogic",
    "dDocOwner": "weblogic",
    "dDocTitle": "file",
    "dDocType": "Document",
    "dExtension": "jpg",
    "dFileSize": 53834,
    "dFlag1": null,
    "dFormat": "image/jpeg",
    "dID": 6207,
    "dInDate": "2024-05-22 20:46:00Z",
    "dIndexerState": null,
    "dIsCheckedOut": 0,
    "dIsPrimary": 1,
    "dIsWebFormat": 0,
    "dLanguage": null,
    "dLocation": null,
    "dMessage": null,
    "dOriginalName": "Mugs.jpg",
    "dOutDate": null,
    "dProcessingState": "Y",
    "dPublishState": null,
    "dPublishType": null,
    "dReleaseDate": "2024-05-22 20:48:42Z",
    "dReleaseState": "Y",
    "dRendition1": null,
    "dRendition2": null,
    "dRevClassID": 6205,
    "dRevLabel": "1",
    "dRevRank": 0,
    "dSecurityGroup": "Public",
    "dStatus": "RELEASED",
    "dWebExtension": "jpg",
    "dWorkflowState": null,
    "fFolderType": "soft",
    "isLeaf": "1",
    "itemType": "2",
    "xAnnotationDetails": 0,
    "xComments": null,
    "xDate": null,
    "xDecimal": null,
    "xExternalDataSet": null,
    "xIdcProfile": null,
    "xIsACLReadOnlyOnUI": 0,
    "xLibraryGUID": null,
    "xLongText": null,
    "xMemo": "Test memo",
    "xOptionList": null,
    "xPartitionId": null,
    "xReqText": null,
    "xStorageRule": "DispByContentId",
    "xTest": 0,
    "xWebFlag": null,
    "permissions": "RWDA"
}
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Download a File

#### GET /documents/wcc/api/v1.1/folders/files/{fFileGUID}/data {#get-documentswccapiv11foldersfilesffileguiddata}

#### Description

Downloads file content for the specified file in a folder as a stream. (GET_FILE)

#### Parameters

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name           Located in     Description                                                                                                                                    Required       Schema
  -------------- -------------- ---------------------------------------------------------------------------------------------------------------------------------------------- -------------- --------------------------
  Range          header         The bytes range of the document to download.                                                                                                   No             [Range](#range-requests)

  fFileGUID      path           The file GUID to download.                                                                                                                     Yes            string

  version        query          The version of the document to download. If not provided, the latest released version is assumed.                                              No             string

  rendition      query          The rendition of the document to retrieve. If rendition is not specified, primary rendition of the document is assumed. Allowed renditions:\   No             string
                                •primary\                                                                                                                                                     
                                •alternate\                                                                                                                                                   
                                •web\                                                                                                                                                         
                                •rendition:T\                                                                                                                                                 
                                •custom attachments by name                                                                                                                                   
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the complete stream of the file.                                          
  206    Successfully returned the byte range of the document                                            
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  416    The server cannot return the bytes requested                                                    
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-69}

Download a document.

##### Request

> GET ...documents/wcc/api/v1.1/folders/files/B068546C1ED509D905EA455D0E03F70D/data

##### Request Body

None

##### HTTP Response

> Status = 200

Body

> stream of file contents as application/octet-stream

## Delete a File

#### DELETE /documents/wcc/api/v1.1/folders/files/{fFileGUID} {#delete-documentswccapiv11foldersfilesffileguid}

#### Description

Delete the specified file in a folder. (FLD_DELETE)

#### Parameters

::: table-responsive
  Name              Located in   Description                                                                                        Required   Schema
  ----------------- ------------ -------------------------------------------------------------------------------------------------- ---------- ---------
  fFileGUID         path         The file GUID                                                                                      Yes        string
  permanentDelete   query        The default is false and the file is moved to trash. When true, the file is permanently deleted.   No         boolean
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Document successfully deleted.                                                                  
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                        [GeneralErrorResponse](#generalerrorresponse)
  404    File not found                                                                                 [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-70}

Delete the file identified by GUID `3C0A71FAA57BFA57D8CAB45097BA8D85`

##### Request

> DELETE .../documents/wcc/api/v1.1/folders/files/3C0A71FAA57BFA57D8CAB45097BA8D85

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Delete a Folder

#### DELETE /documents/wcc/api/v1.1/folders/{fFolderGUID} {#delete-documentswccapiv11foldersffolderguid}

#### Description

Delete the specified folder. (FLD_DELETE)

#### Parameters

::: table-responsive
  Name              Located in   Description                                                                                        Required   Schema
  ----------------- ------------ -------------------------------------------------------------------------------------------------- ---------- ---------
  fFolderGUID       path         The folder GUID                                                                                    Yes        string
  permanentDelete   query        The default is false and the file is moved to trash. When true, the file is permanently deleted.   No         boolean
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Folder successfully deleted.                                                                    
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                        [GeneralErrorResponse](#generalerrorresponse)
  404    Folder not found                                                                               [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-71}

Delete the folder identified by GUID `1BC11C2415A60641882C48EAF488A1E6`

##### Request

> DELETE .../documents/wcc/api/v1.1/folders/1BC11C2415A60641882C48EAF488A1E6

##### Request Body

None.

##### HTTP Response

> Status = 204

Body

None.

## Search Files and Folders

#### GET /documents/wcc/api/v1.1/folders/search/items {#get-documentswccapiv11folderssearchitems}

#### Description

Search for files and folders in a folder structure. (FLD_FOLDER_SEARCH)

#### Parameters

::: table-responsive
  Name      Located in   Description                                                                                                            Required   Schema
  --------- ------------ ---------------------------------------------------------------------------------------------------------------------- ---------- ---------
  q         query        The search query to search for content items. See [Using Universal Query Format](#using-the-universal-query-format).   No         string
  fields    query        A comma separated list of metadata fields to return. By default, metadata fields are returned.                         No         string
  orderBy   query        The sort field and sort order which will be used to arrange the filtered content items.                                No         string
  limit     query        The maximum number of items listed per page.                                                                           No         integer
  offset    query        Specifies the point from which items are listed for the response.                                                      No         integer
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------------------
  200    Successfully returned the search results                                                       [FoldersSearchResultsResponse](#folderssearchresultsresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                        [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-72}

Search for 4 folders in the `Secure` security group and only return 3 specific fields.

##### Request

> GET .../documents/wcc/api/v1.1/folders/search/items

##### Request Body

> Query Parameters: itemType=Folder & q=fSecurityGroup \<matches\> \`Secure\` & fields=fFolderGUID, fSecurityGroup, fCreateDate & limit=4

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"itemType": "Folder",
"q": " fSecurityGroup LIKE 'Secure'   AND  ((fLibraryType != '2' OR fLibraryType IS NULL))   AND  ((fLibraryType != '3' OR fLibraryType IS NULL))   AND  ((UPPER(fFolderName) != UPPER('Trash') OR (UPPER(fFolderName) IS NULL)))   AND  ((UPPER(fFolderName) != UPPER('_REAL_ITEMS') OR (UPPER(fFolderName) IS NULL)))   AND  ((fIsInTrash != '1' OR fIsInTrash IS NULL)) ",
"count": 4,
"hasMore": true,
"totalCount": 16,
"orderBy": "fFolderName:Asc",
"offset": 0,
"limit": 4,
"startRow": 0,
"nextRow": 4,
"dataSource": "FldFolders",
"searchEngineName": "DATABASE.METADATA.FOLDERS",
"items": [
    {
        "fFolderGUID": "802062EE2F5EC5A3E2DAAA27B72D14CD",
        "fCreateDate": "2024-07-07 12:08:50Z",
        "fSecurityGroup": "Secure"
    },
    {
        "fFolderGUID": "81D398B26CCA513DD66AC0315F3949FD",
        "fCreateDate": "2024-07-07 12:08:52Z",
        "fSecurityGroup": "Secure"
    },
    {
        "fFolderGUID": "4B8EB5A37E29FA7752A58EAE2D2C0236",
        "fCreateDate": "2024-07-24 06:39:40Z",
        "fSecurityGroup": "Secure"
    },
    {
        "fFolderGUID": "8FD8B4C33F4C5ACB3A7B25820AA6CFBE",
        "fCreateDate": "2024-07-24 06:39:50Z",
        "fSecurityGroup": "Secure"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Create File Link

#### POST /documents/wcc/api/v1.1/folders/{fFolderGUID}/{dDocName}/filelinks {#post-documentswccapiv11foldersffolderguidddocnamefilelinks}

#### Description

Create a link to a file. (FLD_CREATE_FILE)

#### Parameters

::: table-responsive
  --------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                       Located in     Description                                                                      Required       Schema
  -------------------------- -------------- -------------------------------------------------------------------------------- -------------- --------------
  fFolderGUID                path           The GUID of the parent folder where the new link will be created.                Yes            string

  dDocName                   path           The dDocName of this file to be linked.                                          Yes            string

  fFileType                  query          The type of file link to create.\                                                No             string
                                            Two types are supported:\                                                                       
                                            •owner -- the folder owner who owns the file\                                                   
                                            •soft -- the folder contains a shortcut to the file\                                            
                                            The default is `soft`.                                                                          

  ConflictResolutionMethod   query          The method used to resolve name conflict:\                                       No             string
                                            •ResolveDuplicates -- The created link will be given a unique name.\                            
                                            •SkipDuplicates -- The request will fail with 409 if the name already exists.\                  
                                            •FailDuplicates -- The request will fail with 400 if the name already exists.                   
  --------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                              Schema
  ----------------------- -------------------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Link successfully created.\                                                                               
                          Returns Header `Location` which is a URI to get the metadata of the new file.                            

  400                     Bad request                                                                                              [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                              

  403                     User is not allowed to take this action                                                                   

  409                     If `ConflictResolutionMethod` is `SkipDuplicates` and the file name exists, a new link is not created.    

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request             [GeneralErrorResponse](#generalerrorresponse)
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-73}

Create a soft link to a file identified by PUB under FLD_ROOT, but fail if the link exists.

##### Request

> POST .../documents/wcc/api/v1.1/folders/FLD_ROOT/PUB/filelinks

##### Request Body

> Query Parameters: fFileType=soft & ConflictResolutionMethod=FailDuplicates

##### HTTP Response

> Status = 201

Body

None.

Header

> Location = .../documents/wcc/api/v1.1/folders/files/D9A148D824005417997C9E39C716CB4F

## Get the Capabilities of a Folder

#### GET /documents/wcc/api/v1.1/folders/{fFolderGUID}/capabilities {#get-documentswccapiv11foldersffolderguidcapabilities}

#### Description

Get the capabilities the user has on a folder. (FLD_TEST_USER_CAPABILITIES)

#### Parameters

::: table-responsive
  Name                 Located in   Description                                                                                                       Required   Schema
  -------------------- ------------ ----------------------------------------------------------------------------------------------------------------- ---------- --------
  fFolderGUID          path         The folder GUID to test.                                                                                          Yes        string
  testedCapabilities   query        A comma separated list of capabilities to be tested. See the [Supported Capabilities](#supported-capabilities).   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully tested the folder.                                                                [CapabilitiesResponse](#capabilitiesresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-74}

Test if the user can CREATE_CHILD_FOLDER, CREATE_CHILD_DOCUMENT and COPY the `Users:weblogic` folder.

##### Request

> GET .../documents/wcc/api/v1.1/folders/FLD_USER:weblogic/capabilities

##### Request Body

query parameter: testedCapabilities = CREATE_CHILD_FOLDER,CREATE_CHILD_DOCUMENT,COPY

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"count": 3,
"items": [
    {
        "capabilityName": "CREATE_CHILD_FOLDER",
        "capabilityValue": 1
    },
    {
        "capabilityName": "CREATE_CHILD_DOCUMENT",
        "capabilityValue": 1
    },
    {
        "capabilityName": "COPY",
        "capabilityValue": 1
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Get the Capabilities of a File

#### GET /documents/wcc/api/v1.1/folders/files/{fFileGUID}/capabilities {#get-documentswccapiv11foldersfilesffileguidcapabilities}

#### Description

Get the capabilities the user has on a file. (FLD_TEST_USER_CAPABILITIES)

#### Parameters

::: table-responsive
  Name                 Located in   Description                                                                                            Required   Schema
  -------------------- ------------ ------------------------------------------------------------------------------------------------------ ---------- --------
  fFileGUID            path         The file GUID to test.                                                                                 Yes        string
  testedCapabilities   query        A comma separated list of capabilities to be tested. See the [Supported Capabilities](#capabilities)   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully tested the file.                                                                  [CapabilitiesResponse](#capabilitiesresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-75}

Test if the user can MOVE, COPY, and DELETE the file.

##### Request

> GET ...documents/wcc/api/v1.1/folders/files/B068546C1ED509D905EA455D0E03F70D/capabilities

##### Request Body

query parameter: testedCapabilities = MOVE,COPY,DELETE

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"count": 3,
"items": [
    {
        "capabilityName": "MOVE",
        "capabilityValue": 1
    },
    {
        "capabilityName": "COPY",
        "capabilityValue": 1
    },
    {
        "capabilityName": "DELETE",
        "capabilityValue": 1
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

# Users

## Get Permission Information

#### GET /documents/wcc/api/v1.1/users/permissions {#get-documentswccapiv11userspermissions}

#### Description

Get a list of permissions that the current user has for the securityGroups and documentAccounts. (GET_USER_PERMISSIONS)

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------
  200    Successfully returned the user permissions info.                                               [UserPermissionResponse](#userpermissionresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-76}

List all the permissions a user has.

##### Request

> GET .../documents/wcc/api/v1.1/users/permissions

##### Request Body

None

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"dName": "weblogic",
"securityGroups": [
    {
        "dGroupName": "Public",
        "privilege": "15"
    },
    {
        "dGroupName": "RecordsGroup",
        "privilege": "15"
    },
    {
        "dGroupName": "Reservation",
        "privilege": "15"
    },
    {
        "dGroupName": "Secure",
        "privilege": "15"
    }
],
"userSecurityFlags": [
    {
        "flag": "IsAdmin",
        "value": "1"
    },
    {
        "flag": "AdminAtLeastOneGroup",
        "value": "1"
    },
    {
        "flag": "IsSubAdmin",
        "value": "1"
    },
    {
        "flag": "IsSysManager",
        "value": "1"
    },
    {
        "flag": "IsContributor",
        "value": "1"
    },
    {
        "flag": "ActAsAnonymous",
        "value": "0"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Obtain the OAuth Token

#### POST /documents/wcc/api/v1.1/users/token {#post-documentswccapiv11userstoken}

#### Description

Generates the OAuth for the user. (GENERATE_OAUTH_TOKEN)

#### Parameters

None.

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the token.                                                               [OAuthResponse](#oauthresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

# Pages

## Get Custom Fields

#### GET /documents/wcc/api/v1.1/pages/displayFields {#get-documentswccapiv11pagesdisplayfields}

#### Description

Get information about custom metadata fields for different content server pages. (GET_DISPLAY_FIELDS)

#### Parameters

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------
  Name             Located in     Description                                              Required       Schema
  ---------------- -------------- -------------------------------------------------------- -------------- --------------
  dpAction         query          The type of action; must be one of:\                     Yes            string
                                  •CheckinNew\                                                            
                                  •CheckinSel\                                                            
                                  •CheckinSimilar\                                                        
                                  •Info\                                                                  
                                  •Update\                                                                
                                  •Search\                                                                
                                  •FLDMetadataUpdate\                                                     
                                  •FLDMetadataInfo                                                        

  dID              query          Required when `dpAction` is:\                            Sometimes      integer
                                  •CheckinSel\                                                            
                                  •CheckinSimilar\                                                        
                                  •Info\                                                                  
                                  •Update.                                                                

  fFolderGUID      query          Required when `dpAction` is:\                            Sometimes      String
                                  •FLDMetadataUpdate\                                                     
                                  •FLDMetadataInfo                                                        

  dpTriggerValue   query          The trigger value that will be used to load a profile.   No             string
  ----------------------------------------------------------------------------------------------------------------------
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the DisplayFields information.                                           [CustomFieldsResponse](#customfieldsresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-77}

List display fields for the `Search` action.

##### Request

> GET .../documents/wcc/api/v1.1/pages/displayFields

##### Request Body

> Query Parameters: dpAction = Search

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"displayFieldInfo": [
    {
        "fieldName": "dOriginalName",
        "fieldType": "Text",
        "fieldLabel": "Name",
        "fieldLength": "255",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "1",
        "requiredMsg": "Please specify a file name.",
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "5",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "dDocName",
        "fieldType": "Text",
        "fieldLabel": "Content ID",
        "fieldLength": "100",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "10",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "dDocType",
        "fieldType": "Text",
        "fieldLabel": "Type",
        "fieldLength": "30",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "1",
        "requiredMsg": "Please specify a type.",
        "defaultValue": "Document",
        "displayValue": "Document - Any generic document",
        "isOptionList": "1",
        "isTreeOptionList": null,
        "optionList": "dDocType.options",
        "optionListType": "choice",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "20",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "dDocTitle",
        "fieldType": "Text",
        "fieldLabel": "Title",
        "fieldLength": "255",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "1",
        "requiredMsg": "Please specify a title.",
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "30",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "dDocAuthor",
        "fieldType": "Text",
        "fieldLabel": "Author",
        "fieldLength": "200",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "1",
        "requiredMsg": "Please specify an author.",
        "defaultValue": "weblogic",
        "displayValue": "weblogic",
        "isOptionList": "1",
        "isTreeOptionList": null,
        "optionList": "dDocAuthor.options",
        "optionListType": "choice",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "40",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "1"
    },
    {
        "fieldName": "dSecurityGroup",
        "fieldType": "Text",
        "fieldLabel": "Security Group",
        "fieldLength": "30",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "1",
        "requiredMsg": "Please specify a security group.",
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": "1",
        "isTreeOptionList": null,
        "optionList": "dSecurityGroup.options",
        "optionListType": "choice",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "50",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "primaryFile",
        "fieldType": "File",
        "fieldLabel": "Primary File",
        "fieldLength": null,
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "1",
        "requiredMsg": "Please specify a primary file.",
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "100",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "alternateFile",
        "fieldType": "File",
        "fieldLabel": "Alternate File",
        "fieldLength": null,
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "110",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "dRevLabel",
        "fieldType": "Text",
        "fieldLabel": "Revision",
        "fieldLength": "10",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "1",
        "requiredMsg": "Please specify a revision label.",
        "defaultValue": "1",
        "displayValue": "1",
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "200",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "dInDate",
        "fieldType": "Date",
        "fieldLabel": "Release Date",
        "fieldLength": "20",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "1",
        "requiredMsg": "Please specify a release date.",
        "defaultValue": "5/21/24 11:45 PM",
        "displayValue": "5/21/24 11:45 PM",
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "1000",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "dOutDate",
        "fieldType": "Date",
        "fieldLabel": "Expiration Date",
        "fieldLength": "20",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "1010",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "xComments",
        "fieldType": "Memo",
        "fieldLabel": "Comments",
        "fieldLength": "2000",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "5001",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "xIdcProfile",
        "fieldType": "Text",
        "fieldLabel": "Profile",
        "fieldLength": "30",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": "1",
        "isTreeOptionList": null,
        "optionList": "xIdcProfile.options",
        "optionListType": "choice",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "5006",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "xTemplateType",
        "fieldType": "Text",
        "fieldLabel": "Template Type",
        "fieldLength": "30",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": "1",
        "isTreeOptionList": null,
        "optionList": "xTemplateType.options",
        "optionListType": "choice",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "5040",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "xParentFolders",
        "fieldType": "Memo",
        "fieldLabel": "Parent Folder",
        "fieldLength": null,
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": null,
        "isTreeOptionList": null,
        "optionList": null,
        "optionListType": null,
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "5100",
        "decimalScale": null,
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "xTestField2",
        "fieldType": "Text",
        "fieldLabel": "TestField2",
        "fieldLength": "30",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": "1",
        "isTreeOptionList": "1",
        "optionList": "xTestField2.options",
        "optionListType": "choice",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": "/",
        "treeNodeStorageSeparator": "/",
        "order": "20010",
        "decimalScale": "1",
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "xTestMetadata",
        "fieldType": "Text",
        "fieldLabel": "TestMetadata",
        "fieldLength": "30",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": "1",
        "isTreeOptionList": null,
        "optionList": "xTestMetadata.options",
        "optionListType": "combo",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "20011",
        "decimalScale": "1",
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "xTestRelField",
        "fieldType": "Text",
        "fieldLabel": "TestRelField",
        "fieldLength": "30",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": "1",
        "isTreeOptionList": "1",
        "optionList": "xTestRelField.options",
        "optionListType": "choice",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": "/",
        "treeNodeStorageSeparator": "/",
        "order": "20012",
        "decimalScale": "1",
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "xTestUseView",
        "fieldType": "Text",
        "fieldLabel": "TestUseView",
        "fieldLength": "30",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": "1",
        "isTreeOptionList": "1",
        "optionList": "xTestUseView.options",
        "optionListType": "choice",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": "/",
        "treeNodeStorageSeparator": "/",
        "order": "20013",
        "decimalScale": "1",
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    },
    {
        "fieldName": "xtestUseView1",
        "fieldType": "Text",
        "fieldLabel": "testUseView1",
        "fieldLength": "30",
        "isHidden": "0",
        "isReadOnly": "0",
        "isRequired": "0",
        "requiredMsg": null,
        "defaultValue": null,
        "displayValue": null,
        "isOptionList": "1",
        "isTreeOptionList": null,
        "optionList": "xtestUseView1.options",
        "optionListType": "choice",
        "isDependent": "0",
        "dependentOnField": null,
        "isPadMultiselectStorage": "0",
        "multiselectStorageSeparator": null,
        "multiselectDisplaySeparator": null,
        "isShowSelectionPath": "0",
        "isStoreSelectionPath": "0",
        "treeNodeDisplaySeparator": null,
        "treeNodeStorageSeparator": null,
        "order": "20014",
        "decimalScale": "1",
        "isError": "0",
        "errorMsg": null,
        "isUserName": "0"
    }
],
"dDocType.options": [
    {
        "dOption": "Application",
        "dDescription": "Application - Files owned by Content Server applications"
    },
    {
        "dOption": "Binary",
        "dDescription": "Binary - Executables, zip files, etc"
    },
    {
        "dOption": "DigitalMedia",
        "dDescription": "DigitalMedia - Audio, video, and images"
    },
    {
        "dOption": "Document",
        "dDescription": "Document - Any generic document"
    },
    {
        "dOption": "RetentionCategory",
        "dDescription": "wwDocTypeDesc_RetentionCategory"
    },
    {
        "dOption": "System",
        "dDescription": "System - System configuration, templates, settings"
    }
],
"xTemplateType.options": [
    {
        "dOption": "HC Template",
        "dDescription": "HTML Conversion Template"
    },
    {
        "dOption": "GUI Template",
        "dDescription": "Classic HTML Conversion Template"
    },
    {
        "dOption": "Layout Template",
        "dDescription": "Classic HTML Conversion Layout"
    },
    {
        "dOption": "Script Template",
        "dDescription": "Script Template"
    },
    {
        "dOption": "Viewer Cache Template",
        "dDescription": "Viewer Cache Template"
    }
],
"dDocAuthor.options": [
    {
        "dOption": "LCMUser",
        "dDescription": "LCMUser"
    },
    {
        "dOption": "OracleSystemUser",
        "dDescription": "OracleSystemUser"
    },
    {
        "dOption": "sysadmin",
        "dDescription": "sysadmin"
    },
    {
        "dOption": "weblogic",
        "dDescription": "weblogic"
    }
],
"dSecurityGroup.options": [
    {
        "dOption": "Public",
        "dDescription": "Public"
    },
    {
        "dOption": "RecordsGroup",
        "dDescription": "RecordsGroup"
    },
    {
        "dOption": "Reservation",
        "dDescription": "Reservation"
    },
    {
        "dOption": "Secure",
        "dDescription": "Secure"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

# Public Links

## List Public Links for a File

#### GET /documents/wcc/api/v1.1/publiclinks/.by.file/{fFileGUID} {#get-documentswccapiv11publiclinksbyfileffileguid}

#### Description

List public links on a file. User needs to have R permission on the file. (GET_SHARED_LINKS_FOR_OBJECT)

#### Parameters

::: table-responsive
  Name        Located in   Description                                                                                                                                                                                                      Required   Schema
  ----------- ------------ ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  fFileGUID   path         The GUID of a file for which to list public links.                                                                                                                                                               Yes        string
  offset      query        The starting index from which public links are returned in the response. It defaults to 0 if not specified.                                                                                                      No         number
  limit       query        The maximum number of public links that can be returned per request. It defaults to 20 if not specified. It cannot exceed 100.                                                                                   No         number
  orderBy     query        The sort field and order for the public links. The sortable fields are `dLinkName`, `dLastModifiedDate`, and `dExpirationDate`. The supported sort order are `Asc` and `Desc`. The default is `dLinkName:Asc`.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------
  200    Successfully returned a list of public links.                                                  [PublicLinkListResponse](#publiclinklistresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User doesn't have sufficient privilege to list public links                                    [GeneralErrorResponse](#generalerrorresponse)
  404    The specified fFileGUID is invalid                                                             [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-78}

List public links for file with GUID 'A7A055176A59B08B3067DFF63C58EC94'.

##### Request

> GET .../documents/wcc/api/v1.1/publiclinks/.by.file/A7A055176A59B08B3067DFF63C58EC94

##### Request Body

> Query Parameters: offset = 0 & limit = 10 & orderBy = dLastModifiedDate:Desc

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"offset": 0,
"limit": 10,
"count": 3,
"orderBy": "dLastModifiedDate:Desc",
"hasMore": false,
"totalResults": 3,
"items": [
    {
        "dLinkID": "LDCF346562A4A661ED4FB20F4C5A2B873A6BAA8B8036",
        "dLinkType": "fFileGUID",
        "dLinkName": "filelink_3",
        "dLinkDesc": "file link for external review",
        "fItemGUID": "A7A055176A59B08B3067DFF63C58EC94",
        "fOwner": "adminUser",
        "dLinkPrivilege": "R",
        "dAssignedUsers": "@everybody",
        "dAccessCode": "myaccesscode3",
        "dCreatedDate": "2024-06-21 00:02:35Z",
        "dLastModifiedDate": "2024-07-19 04:03:24Z",
        "dExpirationDate": "2025-07-31 00:07:00Z",
        "dCreator": "adminUser"
    },
    {
        "dLinkID": "LDB8D7C2E1710EFE7FF05560057D14DE1C8D2155903D",
        "dLinkType": "fFileGUID",
        "dLinkName": "filelink_2",
        "dLinkDesc": "another file link",
        "fItemGUID": "A7A055176A59B08B3067DFF63C58EC94",
        "fOwner": "adminUser",
        "dLinkPrivilege": "R",
        "dAssignedUsers": "@everybody",
        "dAccessCode": "myaccesscode2",
        "dCreatedDate": "2024-06-21 19:41:44Z",
        "dLastModifiedDate": "2024-06-21 19:41:44Z",
        "dExpirationDate": "2025-06-21 19:41:00Z",
        "dCreator": "adminUser"
    },
    {
        "dLinkID": "LD2BCF193359D93AA03456C70D676DD14CBAC502D2BE",
        "dLinkType": "fFileGUID",
        "dLinkName": "filelink_1",
        "dLinkDesc": "file link",
        "fItemGUID": "A7A055176A59B08B3067DFF63C58EC94",
        "fOwner": "adminUser",
        "dLinkPrivilege": "R",
        "dAssignedUsers": "@everybody",
        "dAccessCode": "myaccesscode1",
        "dCreatedDate": "2024-06-20 23:32:33Z",
        "dLastModifiedDate": "2024-06-20 23:32:33Z",
        "dExpirationDate": "2025-06-20 23:32:00Z",
        "dCreator": "adminUser"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## List Public Links for a Folder

#### GET /documents/wcc/api/v1.1/publiclinks/.by.folder/{fFolderGUID} {#get-documentswccapiv11publiclinksbyfolderffolderguid}

#### Description

List public links on a folder. User needs to have R permission on the folder. (GET_SHARED_LINKS_FOR_OBJECT)

#### Parameters

::: table-responsive
  Name          Located in   Description                                                                                                                                                                                                      Required   Schema
  ------------- ------------ ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  fFolderGUID   path         The GUID of a folder for which to list public links.                                                                                                                                                             Yes        string
  offset        query        The starting index from which public links are returned in the response. It defaults to 0 if not specified.                                                                                                      No         number
  limit         query        The maximum number of public links that can be returned per request. It defaults to 20 if not specified. It cannot exceed 100.                                                                                   No         number
  orderBy       query        The sort field and order for the public links, The sortable fields are `dLinkName`, `dLastModifiedDate`, and `dExpirationDate`. The supported sort order are `Asc` and `Desc`. The default is `dLinkName:Asc`.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------
  200    Successfully returned a list of public links.                                                  [PublicLinkListResponse](#publiclinklistresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User doesn't have sufficient privilege to list public links                                    [GeneralErrorResponse](#generalerrorresponse)
  404    The specified fFolderGUID is invalid                                                           [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-79}

List public links for folder with GUID '5FE2C336245D8E730C39176A3D76CB2E'.

##### Request

> GET .../documents/wcc/api/v1.1/publiclinks/.by.folder/5FE2C336245D8E730C39176A3D76CB2E

##### Request Body

> Query Parameters: offset = 0 & limit = 3 & orderBy = dExpirationDate:Asc

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"offset": 0,
"limit": 3,
"count": 3,
"orderBy": "dExpirationDate:Asc",
"hasMore": true,
"totalResults": 13,
"items": [
    {
        "dLinkID": "LF90A58364AD3433C89721DDE0ED373F871C2C14B3A0",
        "dLinkType": "fFolderGUID",
        "dLinkName": "folder_link_1",
        "dLinkDesc": "",
        "fItemGUID": "5FE2C336245D8E730C39176A3D76CB2E",
        "fOwner": "adminUser",
        "dLinkPrivilege": "R",
        "dAssignedUsers": "@everybody",
        "dAccessCode": "myaccesscode1",
        "dCreatedDate": "2024-07-25 07:01:37Z",
        "dLastModifiedDate": "2024-07-25 07:01:37Z",
        "dExpirationDate": "2024-07-25 07:02:11Z",
        "dCreator": "adminUser"
    },
    {
        "dLinkID": "LF7F0ADC33DA869AF8890EF876ABC0755573CE05CA48",
        "dLinkType": "fFolderGUID",
        "dLinkName": "folder_link_2",
        "dLinkDesc": "",
        "fItemGUID": "5FE2C336245D8E730C39176A3D76CB2E",
        "fOwner": "adminUser",
        "dLinkPrivilege": "R",
        "dAssignedUsers": "@everybody",
        "dAccessCode": "myaccesscode2",
        "dCreatedDate": "2024-07-24 23:18:19Z",
        "dLastModifiedDate": "2024-07-24 23:18:19Z",
        "dExpirationDate": "2024-08-09 23:59:00Z",
        "dCreator": "adminUser"
    },
    {
        "dLinkID": "LFA75D3D741E410F0DD98704D2B59819C56BEAC314CF",
        "dLinkType": "fFolderGUID",
        "dLinkName": "folder_link_3",
        "dLinkDesc": "",
        "fItemGUID": "5FE2C336245D8E730C39176A3D76CB2E",
        "fOwner": "adminUser",
        "dLinkPrivilege": "R",
        "dAssignedUsers": "@everybody",
        "dAccessCode": "myaccesscode3",
        "dCreatedDate": "2024-07-24 23:13:09Z",
        "dLastModifiedDate": "2024-07-24 23:13:09Z",
        "dExpirationDate": "2024-08-24 23:59:00Z",
        "dCreator": "adminUser"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Create Public Link to a File

#### POST /documents/wcc/api/v1.1/publiclinks/.by.file/{fFileGUID} {#post-documentswccapiv11publiclinksbyfileffileguid}

#### Description

Create a public link to a file. (CREATE_SHARED_LINK)

#### Parameters

::: table-responsive
  Name        Located in   Description                                                                                 Required   Schema
  ----------- ------------ ------------------------------------------------------------------------------------------- ---------- ---------------------------------------
  fFileGUID   path         The GUID of a file for which to create a public link.                                       Yes        string
  body        body         The link properties bean defining the new public link. The `dLinkName` field is required.   Yes        [PublicLinkObject](#publiclinkobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Public link successfully created.\                                                              
                          Returns Header `Location` which is a URI to get the link info for the new link.                

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                        [GeneralErrorResponse](#generalerrorresponse)

  404                     The file does not exist                                                                        [GeneralErrorResponse](#generalerrorresponse)

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-80}

Create a read only public link to a file with GUID 29504F74B07E6AC3F5B01002007ACE90. The caller must have read/write access to the file.

##### Request

> POST .../documents/wcc/api/v1.1/publiclinks/.by.file/29504F74B07E6AC3F5B01002007ACE90

##### Request Body

> {
>
> "dLinkName": "external_review",
>
> "dLinkDesc": "a link for external review",
>
> "dLinkPrivilege": "R",
>
> "dAccessCode": "myaccesscode",
>
> "dExpirationDate": "2024-10-01 07:00:00Z"
>
> }

##### HTTP Response

> Status = 201

Body

None.

Header

> Location = .../documents/wcc/api/v1.1/publiclinks/LDC52C7174798C6814B965C92EE76E200DEE6410B78B

## Create Public Link to a Folder

#### POST /documents/wcc/api/v1.1/publiclinks/.by.folder/{fFolderGUID} {#post-documentswccapiv11publiclinksbyfolderffolderguid}

#### Description

Create a public link to a folder. (CREATE_SHARED_LINK)

#### Parameters

::: table-responsive
  Name          Located in   Description                                                                                 Required   Schema
  ------------- ------------ ------------------------------------------------------------------------------------------- ---------- ---------------------------------------
  fFolderGUID   path         The GUID of a folder for which to create a public link.                                     Yes        string
  body          body         The link properties bean defining the new public link. The `dLinkName` field is required.   Yes        [PublicLinkObject](#publiclinkobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Public link successfully created.\                                                              
                          Returns Header `Location` which is a URI to get the link info for the new link.                

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                        [GeneralErrorResponse](#generalerrorresponse)

  404                     The folder does not exist.                                                                     [GeneralErrorResponse](#generalerrorresponse)

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-81}

Create a read only public link to a folder with GUID F08E631C39EC3839268C6D87A307721D. The caller must have read/write access to the folder.

##### Request

> POST .../documents/wcc/api/v1.1/publiclinks/.by.folder/F08E631C39EC3839268C6D87A307721D

##### Request Body

> {
>
> "dLinkName": "external_review",
>
> "dLinkDesc": "link for external review on files under this folder",
>
> "dLinkPrivilege": "R",
>
> "dAccessCode": "myaccesscode",
>
> "dExpirationDate": "2024-10-01 07:00:00Z"
>
> }

##### HTTP Response

> Status = 201

Body

None.

Header

> Location = .../documents/wcc/api/v1.1/publiclinks/LFFAFDED274FFC50478707A0C1238F94114052E9EF9F

## Get Public Link Information

#### GET /documents/wcc/api/v1.1/publiclinks/{dLinkID} {#get-documentswccapiv11publiclinksdlinkid}

#### Description

Get the information of a public link. (GET_SHARED_LINK_INFO)

#### Parameters

::: table-responsive
  Name      Located in   Description                Required   Schema
  --------- ------------ -------------------------- ---------- --------
  dLinkID   path         The ID of a public link.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Public link information.                                                                       [PublicLinkResponse](#publiclinkresponse)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                        [GeneralErrorResponse](#generalerrorresponse)
  404    The link was not found                                                                         [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-82}

Get the public link information for the link with GUID LDC52C7174798C6814B965C92EE76E200DEE6410B78B.

##### Request

> GET .../documents/wcc/api/v1.1/publiclinks/LDC52C7174798C6814B965C92EE76E200DEE6410B78B

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

> {
>
> "dLinkID": "LDC52C7174798C6814B965C92EE76E200DEE6410B78B",
>
> "dLinkType": "fFileGUID",
>
> "dLinkName": "external_review",
>
> "dLinkDesc": "a link for external review",
>
> "fItemGUID": "29504F74B07E6AC3F5B01002007ACE90",
>
> "fOwner": "adminUser",
>
> "dLinkPrivilege": "R",
>
> "dAssignedUsers": "@everybody",
>
> "dAccessCode": "myaccesscode",
>
> "dCreatedDate": "2024-07-11 21:40:27Z",
>
> "dLastModifiedDate": "2024-07-11 23:09:30Z",
>
> "dExpirationDate": "2024-10-01 07:00:00Z",
>
> "dCreator": "adminUser"
>
> }

## Update Public Link

#### PUT /documents/wcc/api/v1.1/publiclinks/{dLinkID} {#put-documentswccapiv11publiclinksdlinkid}

#### Description

Update the information of a public link. (EDIT_SHARED_LINK)

#### Parameters

::: table-responsive
  Name      Located in   Description                                           Required   Schema
  --------- ------------ ----------------------------------------------------- ---------- ---------------------------------------
  dLinkID   path         The ID of a public link.                              Yes        string
  body      body         The link properties bean to update the public link.   Yes        [PublicLinkObject](#publiclinkobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204                     Public link successfully updated.\                                                              
                          Returns Header `Location` which is a URI to get the link info for the new link.                

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                        [GeneralErrorResponse](#generalerrorresponse)

  404                     The link ID was not found                                                                      [GeneralErrorResponse](#generalerrorresponse)

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-83}

Update public link information for the link with GUID LDC52C7174798C6814B965C92EE76E200DEE6410B78B

##### Request

> PUT .../documents/wcc/api/v1.1/publiclinks/LDC52C7174798C6814B965C92EE76E200DEE6410B78B

##### Request Body

> { "dLinkName": "external_review",
>
> "dLinkDesc": "a link for external review - extend to end of 2024",
>
> "dLinkPrivilege": "R",
>
> "dAccessCode": "myaccesscode",
>
> "dExpirationDate": "2025-01-01 07:59:00Z" }

##### HTTP Response

> Status = 204

Body

None.

Header

> Location = .../documents/wcc/api/v1.1/publiclinks/LDC52C7174798C6814B965C92EE76E200DEE6410B78B

## Delete Public Link

#### DELETE /documents/wcc/api/v1.1/publiclinks/{dLinkID} {#delete-documentswccapiv11publiclinksdlinkid}

#### Description

Delete a public link. (DELETE_SHARED_LINK)

#### Parameters

::: table-responsive
  Name      Located in   Description                Required   Schema
  --------- ------------ -------------------------- ---------- --------
  dLinkID   path         The ID of a public link.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Public link successfully deleted.                                                               
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                        [GeneralErrorResponse](#generalerrorresponse)
  404    The link ID was not found                                                                      [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-84}

Delete the public link with GUID LDC52C7174798C6814B965C92EE76E200DEE6410B78B.

##### Request

> DELETE .../documents/wcc/api/v1.1/publiclinks/LDC52C7174798C6814B965C92EE76E200DEE6410B78B

##### Request Body

None.

##### HTTP Response

> Status = 204

# Applinks

## Create an Applink to a File

#### POST /documents/wcc/api/v1.1/applinks/.by.file/{fFileGUID} {#post-documentswccapiv11applinksbyfileffileguid}

#### Description

Create an applink to a file. (CREATE_APP_LINK)

#### Parameters

::: table-responsive
  Name        Located in   Description                                                   Required   Schema
  ----------- ------------ ------------------------------------------------------------- ---------- -----------------------------------------------------------
  fFileGUID   path         The GUID of a file for which you need to create an applink.   Yes        string
  body        body         The properties defining the new applink.                      Yes        [AppLinkCreateRequestObject](#applinkcreaterequestobject)
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -------------------------------------------------------------
  201    The applink was successfully created.                                                          [AppLinkCreateResponseObject](#applinkcreateresponseobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User doesn't have sufficient privilege to create an applink                                    [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-85}

Create an applink with R privilege for a file with GUID E4B2D14C25ED0387A7FCC58F8BBE36A6 on behalf of Application1. The caller should have R permissions on the file.

##### Request

> POST .../documents/wcc/api/v1.1/applinks/.by.file/E4B2D14C25ED0387A7FCC58F8BBE36A6

##### Request Body

> {
>
> ::: {.language-plaintext .highlighter-rouge}
> ::: highlight
> ::: {role="separator" aria-label="Code Block start"}
> :::
>
> ``` highlight
> Copy  "dAssignedUser": "Application1",
>
>   "dLinkPrivilege": "R",
>
>   "dUserLocale": "fr-CA",
>
>   "dUserTimeZone": "America/Montreal"
> ```
>
> ::: {role="separator" aria-label="Code Block end"}
> :::
> :::
> :::
>
> }

##### HTTP Response

> Status = 201

Body

> {
>
> ::: {.language-plaintext .highlighter-rouge}
> ::: highlight
> ::: {role="separator" aria-label="Code Block start"}
> :::
>
> ``` highlight
> Copy  "dAppLinkID": "LDyqaygFMt0IsLr4cfmfNoykhwbjXQ6otUPVRMNx35I_08JDVoxjAEOqKmMhYVWNCuhnBqyCG7zbKfs2YhnbhVWg==",
>
>   "dAppLinkAccessToken": "tL5Ot18ZC5X92S8IyBcGB8AO1eWlONYz51MQu2hyRwXJuxB9Nuz0qq-05m0FP3wl",
>
>   "dAppLinkRefreshToken": "hvqLCBjCCtREw463ucOeU_oZdpG1qe0Z_oHbpvvaVEU8PYvjTO6ByYpA_L5sbxjd",
>
>   "dEmbedUrl": "http://hostname/cs/idcplg?IdcService=REDWOODUI_VIEWER&dDocName=MHONG1SUBN6522017014&RevisionSelectionMethod=LatestReleased&dAppLinkID=LDyqaygFMt0IsLr4cfmfNoykhwbjXQ6otUPVRMNx35I_08JDVoxjAEOqKmMhYVWNCuhnBqyCG7zbKfs2YhnbhVWg=="
> ```
>
> ::: {role="separator" aria-label="Code Block end"}
> :::
> :::
> :::
>
> }

## Create an Applink to a Folder

#### POST /documents/wcc/api/v1.1/applinks/.by.folder/{fFolderGUID} {#post-documentswccapiv11applinksbyfolderffolderguid}

#### Description

Create an applink to a folder. (CREATE_APP_LINK)

#### Parameters

::: table-responsive
  Name          Located in   Description                                            Required   Schema
  ------------- ------------ ------------------------------------------------------ ---------- -----------------------------------------------------------
  fFolderGUID   path         The GUID of a folder for which to create an applink.   Yes        string
  body          body         The properties defining the new applink.               Yes        [AppLinkCreateRequestObject](#applinkcreaterequestobject)
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -------------------------------------------------------------
  201    The applink was successfully created.                                                          [AppLinkCreateResponseObject](#applinkcreateresponseobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User doesn't have sufficient privilege to create an applink                                    [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-86}

Create an applink with RWD privilege for a folder with GUID 2290B1F709557ED1FAE557E51C4C7EA8 on behalf of Application2. The caller should have RWD permissions on the folder.

##### Request

> POST .../documents/wcc/api/v1.1/applinks/.by.folder/2290B1F709557ED1FAE557E51C4C7EA8

##### Request Body

> {
>
> ::: {.language-plaintext .highlighter-rouge}
> ::: highlight
> ::: {role="separator" aria-label="Code Block start"}
> :::
>
> ``` highlight
> Copy  "dAssignedUser": "Application2",
>
>   "dLinkPrivilege": "RWD",
>
>   "dUserLocale": "en-US",
>
>   "dUserTimeZone": "US/Pacific"
> ```
>
> ::: {role="separator" aria-label="Code Block end"}
> :::
> :::
> :::
>
> }

##### HTTP Response

> Status = 201

Body

> {
>
> ::: {.language-plaintext .highlighter-rouge}
> ::: highlight
> ::: {role="separator" aria-label="Code Block start"}
> :::
>
> ``` highlight
> Copy  "dAppLinkID": "LFWCFa-ATW6L4V9wpCA9Rf76Zu8A8D5yM_2xvM2yrUJ_LsVBf_-gGZvP388Lyu2LMSQ8MOFJioiG33r_W-Zbawlg==",
>
>   "dAppLinkAccessToken": "h3g6HLblvJtaO2Ih3hzz-84YFI4nHBWR_PyEzbMxj-pgTD7IYbO5zUve7m74wg-bDgVKqo7QPgwG_11E2tMudA==",
>
>   "dAppLinkRefreshToken": "sfuwd_7UazorI2oOx73g44JprCEmol-rTImgrEps3iV_BRTbpDd9TBp_gmevzV8l",
>
>   "dEmbedUrl": "http://hostname/cs/idcplg?IdcService=REDWOODUI_FOLDER_VIEWER&fFolderGUID=2290B1F709557ED1FAE557E51C4C7EA8&dAppLinkID=LFWCFa-ATW6L4V9wpCA9Rf76Zu8A8D5yM_2xvM2yrUJ_LsVBf_-gGZvP388Lyu2LMSQ8MOFJioiG33r_W-Zbawlg=="
> ```
>
> ::: {role="separator" aria-label="Code Block end"}
> :::
> :::
> :::
>
> }

## Refresh Applink Token

#### POST /documents/wcc/api/v1.1/applinks/{dAppLinkID}/.refreshAccessToken {#post-documentswccapiv11applinksdapplinkidrefreshaccesstoken}

#### Description

Refresh an applink access token. (GET_APP_LINK_ACCESS_TOKEN)

#### Parameters

::: table-responsive
  Name         Located in   Description                                          Required   Schema
  ------------ ------------ ---------------------------------------------------- ---------- -----------------------------------------------
  dAppLinkID   path         The applink GUID to be refreshed.                    Yes        string
  body         body         The properties defining the tokens of the applink.   Yes        [AppLinkRefreshObject](#applinkrefreshobject)
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201    The applink access token was successfully refreshed.                                           [AppLinkRefreshObject](#applinkrefreshobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                        [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-87}

Get a new access token for an applink with ID `LFWCFa-ATW6L4V9wpCA9Rf76Zu8A8D5yM_2xvM2yrUJ_LsVBf_-gGZvP388Lyu2LMSQ8MOFJioiG33r_W-Zbawlg==`. The caller can be any user, authenticated or anonymous.

##### Request

> POST .../documents/wcc/api/v1.1/applinks/LFWCFa-ATW6L4V9wpCA9Rf76Zu8A8D5yM_2xvM2yrUJ_LsVBf\_-gGZvP388Lyu2LMSQ8MOFJioiG33r_W-Zbawlg==/.refreshAccessToken

##### Request Body

> {
>
> ::: {.language-plaintext .highlighter-rouge}
> ::: highlight
> ::: {role="separator" aria-label="Code Block start"}
> :::
>
> ``` highlight
> Copy  "dAppLinkAccessToken": "h3g6HLblvJtaO2Ih3hzz-84YFI4nHBWR_PyEzbMxj-pgTD7IYbO5zUve7m74wg-bDgVKqo7QPgwG_11E2tMudA==",
>
>   "dAppLinkRefreshToken": "sfuwd_7UazorI2oOx73g44JprCEmol-rTImgrEps3iV_BRTbpDd9TBp_gmevzV8l"
> ```
>
> ::: {role="separator" aria-label="Code Block end"}
> :::
> :::
> :::
>
> }

##### HTTP Response

> Status = 201

Body

> {
>
> ::: {.language-plaintext .highlighter-rouge}
> ::: highlight
> ::: {role="separator" aria-label="Code Block start"}
> :::
>
> ``` highlight
> Copy  "dAppLinkAccessToken": "25FpDgtUx0GnDJbk3O4kwZHr5CBaWrFFD64_HofREpmzaHR0ys96h0FTfwSd9BfQnNZ8n-55311wnLeXxHXfZw==",
>
>   "dAppLinkRefreshToken": "sfuwd_7UazorI2oOx73g44JprCEmol-rTImgrEps3iV_BRTbpDd9TBp_gmevzV8l"
> ```
>
> ::: {role="separator" aria-label="Code Block end"}
> :::
> :::
> :::
>
> }

# Background Jobs

## Start Background Delete Job

#### POST /documents/wcc/api/v1.1/.bulk/.delete {#post-documentswccapiv11bulkdelete}

#### Description

Start a bulk delete job. Note that jobs are limited to deleting 1000 files per job. (INIT_BACKGROUND_JOB)

#### Parameters

::: table-responsive
  Name   Located in   Description                                        Required   Schema
  ------ ------------ -------------------------------------------------- ---------- -----------------------------------------------------------
  body   body         The list of IDs to be deleted in the background.   Yes        [BackgroundJobRequestObject](#backgroundjobrequestobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  202                     The bulk delete job was started.\                                                               
                          Returns Header `Location` which is a URI to get the status of the job.                         

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-88}

Start a bulk delete job for 3 IDs: dDocName:ID1, dID:2402, and fFileGUID:45BCCE664ADF7074FE227C8A61F0A188

##### Request

> POST .../documents/wcc/api/v1.1/.bulk/.delete

##### Request Body

> {
>
> "itemIds": "dDocName:ID1,dID:2402,fFileGUID:45BCCE664ADF7074FE227C8A61F0A188"
>
> }

##### HTTP Response

> Status = 202

Body

None

Header

> Location = .../documents/wcc/api/v1.1/.bulk/9488D4CF0448AE509B6E5637502858EF1733172082092

## Start Background Download Job

#### POST /documents/wcc/api/v1.1/.bulk/.download {#post-documentswccapiv11bulkdownload}

#### Description

Start a bulk download job where the server will download up to 1000 files (or 2gb) and store them in a zip file. When the job is complete, the job owner can download the zip file. (INIT_BACKGROUND_JOB)

#### Parameters

::: table-responsive
  Name   Located in   Description                                                      Required   Schema
  ------ ------------ ---------------------------------------------------------------- ---------- -----------------------------------------------------------
  body   body         The list of IDs to be downloaded and zipped in the background.   Yes        [BackgroundJobRequestObject](#backgroundjobrequestobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  202                     The bulk download job was started.\                                                             
                          Returns Header `Location` which is a URI to get the status of the job.                         

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-89}

Start a bulk download job for 3 IDs: dDocName:ID45, dID:4302, and fFileGUID:68BCB34664ADF7074FE227C8A61F0A199

##### Request

> POST .../documents/wcc/api/v1.1/.bulk/.download

##### Request Body

> {
>
> "itemIds": "dDocName:ID45,dID:4302,fFileGUID:68BCB34664ADF7074FE227C8A61F0A199"
>
> }

##### HTTP Response

> Status = 202

Body

None

Header

> Location = .../documents/wcc/api/v1.1/.bulk/9933F209EB6380150CBB59B43E8DF1871733333625519

## Start Background Add Catgory Job

#### POST /documents/wcc/api/v1.1/.bulk/categories/.add {#post-documentswccapiv11bulkcategoriesadd}

#### Description

Start a bulk add category job. (INIT_BACKGROUND_JOB)

#### Parameters

::: table-responsive
  Name   Located in   Description                                        Required   Schema
  ------ ------------ -------------------------------------------------- ---------- ---------------------------------------------------------------------------------
  body   body         A list of items to add categories in a taxonomy.   Yes        [BackgroundCategoryActionRequestObject](#backgroundcategoryactionrequestobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  202                     The bulk add categories job was started.\                                                       
                          Returns Header `Location` which is a URI to get the status of the job.                         

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-90}

Start a job to add the categories 47A8176EE9682AACBBE15C5DCDB11144 and 80406453BB6C1FE72740D8DA05F34E50 from taxonomy A746D984C500F280923C36033F14D63D for 3 IDs: dDocName:ID1, dID:2402, and fFileGUID:45BCCE664ADF7074FE227C8A61F0A188

##### Request

> POST .../documents/wcc/api/v1.1/.bulk/categories/.add

##### Request Body

> {
>
> "itemIds": "dDocName:ID1,dID:2402,fFileGUID:45BCCE664ADF7074FE227C8A61F0A188"
>
> "taxonomyGUID": "A746D984C500F280923C36033F14D63D",
>
> "categoryGUIDs": "47A8176EE9682AACBBE15C5DCDB11144,80406453BB6C1FE72740D8DA05F34E50"
>
> }

##### HTTP Response

> Status = 202

Body

None

Header

> Location = .../documents/wcc/api/v1.1/.bulk/9488D4CF0448AE509B6E5637502858EF1733172082092

## Start Background Remove Catgory Job

#### POST /documents/wcc/api/v1.1/.bulk/categories/.remove {#post-documentswccapiv11bulkcategoriesremove}

#### Description

Start a bulk remove category job. (INIT_BACKGROUND_JOB)

#### Parameters

::: table-responsive
  Name   Located in   Description                                               Required   Schema
  ------ ------------ --------------------------------------------------------- ---------- ---------------------------------------------------------------------------------
  body   body         A list of items to remove the categories in a taxonomy.   Yes        [BackgroundCategoryActionRequestObject](#backgroundcategoryactionrequestobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  202                     The bulk remove categories job was started.\                                                    
                          Returns Header `Location` which is a URI to get the status of the job.                         

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-91}

Start a job to remove the categories 80406453BB6C1FE72740D8DA05F34E50 from taxonomy A746D984C500F280923C36033F14D63D from 3 IDs: dDocName:ID1, dID:2402, and fFileGUID:45BCCE664ADF7074FE227C8A61F0A188

##### Request

> POST .../documents/wcc/api/v1.1/.bulk/categories/.remove

##### Request Body

> {
>
> "itemIds": "dDocName:ID1,dID:2402,fFileGUID:45BCCE664ADF7074FE227C8A61F0A188"
>
> "taxonomyGUID": "A746D984C500F280923C36033F14D63D",
>
> "categoryGUIDs": "80406453BB6C1FE72740D8DA05F34E50"
>
> }

##### HTTP Response

> Status = 202

Body

None

Header

> Location = .../documents/wcc/api/v1.1/.bulk/9488D4CF0448AE509B6E5637502858EF1733172082092

## Start Background Copy Catgory Job

#### POST /documents/wcc/api/v1.1/.bulk/categories/.copy {#post-documentswccapiv11bulkcategoriescopy}

#### Description

Start a bulk copy category job. (INIT_BACKGROUND_JOB)

#### Parameters

::: table-responsive
  Name   Located in   Description                         Required   Schema
  ------ ------------ ----------------------------------- ---------- -----------------------------------------------------------------------------
  body   body         A category to copy in a taxonomy.   Yes        [BackgroundCategoryCopyRequestObject](#backgroundcategorycopyrequestobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  202                     The bulk copy category job was started.\                                                        
                          Returns Header `Location` which is a URI to get the status of the job.                         

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-92}

Start a job to copy the category E39DEEC7AAB687B50EE7F18505BDB65B in the taxonomy A746D984C500F280923C36033F14D63D under parent category 4DB547640114D6861E36DF9B0B095D34.

##### Request

> POST .../documents/wcc/api/v1.1/.bulk/categories/.copy

##### Request Body

> {
>
> "taxonomyGUID": "A746D984C500F280923C36033F14D63D",
>
> "sourceCategoryGUID": "E39DEEC7AAB687B50EE7F18505BDB65B",
>
> "targetParentGUID": "4DB547640114D6861E36DF9B0B095D34",
>
> "targetPosition": 0
>
> }

##### HTTP Response

> Status = 202

Body

None

Header

> Location = .../documents/wcc/api/v1.1/.bulk/28BE891AE8FBDA22D902038CC02154A71748570803916

## Cancel a Background Job

#### POST /documents/wcc/api/v1.1/.bulk/{dJobID}/.cancel {#post-documentswccapiv11bulkdjobidcancel}

#### Description

Request that the server cancel (terminate) a background job. It is up to the job being cancelled to wind down and stop. (CANCEL_BACKGROUND_JOB)

#### Parameters

::: table-responsive
  Name     Located in   Description                                              Required   Schema
  -------- ------------ -------------------------------------------------------- ---------- --------
  dJobID   path         The GUID assigned by the server to the background job.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                                                             Schema
  ------ --------------------------------------------------------------------------------------------------------------------------------------- -----------------------------------------------
  202    The background job was told to terminate.                                                                                                
  401    Unauthorized                                                                                                                             
  404    Bad job ID. Only the creator of the job or a server admin can cancel the job. Only jobs in `PROCESSING` or `PENDING` can be canceled.   [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request                                            [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-93}

Cancel background job with ID `9488D4CF0448AE509B6E5637502858EF1733172082092`

##### Request

> POST .../documents/wcc/api/v1.1/.bulk/9488D4CF0448AE509B6E5637502858EF1733172082092/.cancel

##### Request Body

None.

##### HTTP Response

> Status = 202

Body

None.

## Get Status on a Background Job

#### GET /documents/wcc/api/v1.1/.bulk/{dJobID} {#get-documentswccapiv11bulkdjobid}

#### Description

Get the current status of a background job. (STATUS_BACKGROUND_JOB)

#### Parameters

::: table-responsive
  Name     Located in   Description                                              Required   Schema
  -------- ------------ -------------------------------------------------------- ---------- --------
  dJobID   path         The GUID assigned by the server to the background job.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned background job status.                                                   [BackgroundJobStatus](#backgroundjobstatus)
  401    Unauthorized                                                                                    
  404    Bad job ID. Only the creator of the job or a server admin can get the status of a job.         [GeneralErrorResponse](#generalerrorresponse)
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-94}

Get the status of the background job with ID `9488D4CF0448AE509B6E5637502858EF1733172082092`; in this example, 3 items were submitted to be deleted, but 1 failed.

##### Request

> POST .../documents/wcc/api/v1.1/.bulk/9488D4CF0448AE509B6E5637502858EF1733172082092/

##### Request Body

None.

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"completed": true,
"completedPercentage": 100,
"state": "COMPLETED",
"message": null,
"itemsProcessed": 3,
"totalItemsCount": 3,
"failedItemsCount": 1,
"failedItems": [
    {
        "id": "TEST1",
        
        "message": "Unable to retrieve information for 'TEST1'. Unable to find latest revision for item 'TEST1'."
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Download Background Job File

#### GET /documents/wcc/api/v1.1/.bulk/{dJobID}/package {#get-documentswccapiv11bulkdjobidpackage}

#### Description

Downloads file content produced by a background job. (REST_GET_PACKAGE_BACKGROUND_JOB)

#### Parameters

::: table-responsive
  Name     Located in   Description                                              Required   Schema
  -------- ------------ -------------------------------------------------------- ---------- --------------------------
  Range    header       The byte range of the zip file to download.              No         [Range](#range-requests)
  dJobID   path         The GUID assigned by the server to the background job.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the complete stream of the file                                           
  206    Successfully returned the byte range of the file                                                
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  416    The server cannot return the bytes requested                                                    
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example {#example-95}

Get the file produced by the background job with ID `60AB088BA61691456925B7607FC3C1331733415582598`. Note that the URI would come from the job status.

##### Request

> GET .../documents/wcc/api/v1.1/.bulk/60AB088BA61691456925B7607FC3C1331733415582598/.download/package

##### Request Body

None

##### HTTP Response

> Status = 200

Body

> stream of file contents as application/octet-stream

# Taxonomies

## Create a Taxonomy

#### POST /documents/wcc/api/v1.1/taxonomies {#post-documentswccapiv11taxonomies}

#### Description

Create a taxonomy. (TXY_CREATE_TAXONOMY) User needs to have admin role or have RWDA permissions on the security group and the account (if enabled) that are to be set on the new taxonomy.

#### Parameters

::: table-responsive
  Name   Located in   Description                                                                                                                   Required   Schema
  ------ ------------ ----------------------------------------------------------------------------------------------------------------------------- ---------- -----------------------------------------------
  body   body         The taxonomy properties object defining the taxonomy. The `dTaxonomyName` and `dTaxonomySecurityGroup` fields are required.   Yes        [TaxonomyCreateObject](#taxonomycreateobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Taxonomy successfully created.\                                                                 
                          Returns Header `Location` which is a URI to get the info for the new taxonomy.                 

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

## Update a Taxonomy

#### PUT /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID} {#put-documentswccapiv11taxonomiesdtaxonomyguid}

#### Description

Update a taxonomy. (TXY_EDIT_TAXONOMY) User needs to have the admin role or RWDA permissions on all security groups and accounts (if enabled) referenced.

#### Parameters

::: table-responsive
  Name            Located in   Description                                             Required   Schema
  --------------- ------------ ------------------------------------------------------- ---------- -----------------------------------------------
  dTaxonomyGUID   path         The taxonomy GUID.                                      Yes        string
  body            body         The taxonomy properties object defining the taxonomy.   Yes        [TaxonomyUpdateObject](#taxonomyupdateobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204                     Taxonomy successfully updated.\                                                                 
                          Returns Header `Location` which is a URI to get the info for the taxonomy.                     

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  404                     The dTaxonomyGUID was invalid                                                                   

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

## Get a Taxonomy

#### GET /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID} {#get-documentswccapiv11taxonomiesdtaxonomyguid}

#### Description

Get a taxonomy. (TXY_GET_TAXONOMY_INFO) User needs to have admin role or have R permission on the security group and the account (if Accounts is enabled) that are set on the taxonomy or have R permission on any security group mapped with the taxonomy.

#### Parameters

::: table-responsive
  Name            Located in   Description          Required   Schema
  --------------- ------------ -------------------- ---------- --------
  dTaxonomyGUID   path         The taxonomy GUID.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Taxonomy information                                                                           [TaxonomyObject](#taxonomyobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The dTaxonomyGUID was invalid                                                                   
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

## Delete a Taxonomy

#### DELETE /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID} {#delete-documentswccapiv11taxonomiesdtaxonomyguid}

#### Description

Delete a taxonomy. (TXY_DELETE_TAXONOMY) User needs to have admin role or have RWDA permissions on the security group and the account that are set on the taxonomy. If the taxonomy is in use, it cannot be deleted.

#### Parameters

::: table-responsive
  Name            Located in   Description          Required   Schema
  --------------- ------------ -------------------- ---------- --------
  dTaxonomyGUID   path         The taxonomy GUID.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted the Taxonomy                                                               
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The dTaxonomyGUID was invalid                                                                   
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

## List All Taxonomies

#### GET /documents/wcc/api/v1.1/taxonomies/ {#get-documentswccapiv11taxonomies}

#### Description

List all taxonomies a user can access. (TXY_GET_TAXONOMIES) If user has admin role, user can access all the taxonomies. Otherwise, user can access the taxonomies where user has R permission on the security groups and accounts (if Accounts is enabled) that are set on those taxonomies or has R permission on any security group mapped with those taxonomies.

#### Parameters

::: table-responsive
  Name        Located in   Description                                                                                                                                Required   Schema
  ----------- ------------ ------------------------------------------------------------------------------------------------------------------------------------------ ---------- --------
  sortField   query        The field used to sort the taxonomies; fields can be sorted by `dTaxonomyName` or `dLastModifiedDate`. The default is `dTaxonomyName``.`   No         string
  sortOrder   query        The order to sort the taxonomies; supported values are `Asc` and `Desc`. The default is `Asc`.                                             No         string
  startRow    query        The row number at which to start returning the taxonomies. This is for pagination. The defaults is 0.                                      No         number
  count       query        The number of taxonomies to return. This is for pagination. The dDefault is 20; the maximum is 100.                                        No         number
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------------------
  200    List of taxonomy information                                                                   [TaxonomyListResponseObject](#taxonomylistresponseobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

## Associate a Taxonomy to a Security Group

#### POST /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID}/securityGroups/.add {#post-documentswccapiv11taxonomiesdtaxonomyguidsecuritygroupsadd}

#### Description

Associate a taxonomy to a security group. (TXY_ADD_TAXONOMY_TO_SECURITY_GROUP) User needs to have either the admin role or RWDA permissions on the security group and the account (if Accounts is enabled) that is set on the taxonomy and have RWDA permissions on the security group the taxonomy is added to.

#### Parameters

::: table-responsive
  Name            Located in   Description                                                                            Required   Schema
  --------------- ------------ -------------------------------------------------------------------------------------- ---------- ----------------------------------------------------
  dTaxonomyGUID   path         The taxonomy GUID.                                                                     Yes        string
  body            body         The security group to associate with the taxonomy. The `dSecurityGroup` is required.   Yes        [TaxonomyAssociateObject](#taxonomyassociatebject)
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Taxonomy associated successfully.                                                               
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The dTaxonomyGUID was invalid                                                                   
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

## Remove Taxonomy from Associated Security Group

#### POST /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID}/securityGroups/.remove {#post-documentswccapiv11taxonomiesdtaxonomyguidsecuritygroupsremove}

#### Description

Remove a taxonomy from an associated security group. (TXY_REMOVE_TAXONOMY_FROM_SECURITY_GROUP) User needs to have either the admin role or RWDA permissions on the security group and the account (if Accounts is enabled) that is set on the taxonomy and have RWDA permissions on the security group the taxonomy is removed from.

#### Parameters

::: table-responsive
  Name            Located in   Description                                                                                                                                                                                Required   Schema
  --------------- ------------ ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------- -----------------------------------------------------
  dTaxonomyGUID   path         The taxonomy GUID.                                                                                                                                                                         Yes        string
  forceRemove     query        When true the taxonomy will be unassociated from the security group even if a content item in the security group is categorized with categories from the taxonomy. The default is false.   No         boolean
  body            body         The security group to remove the taxonomy from. The `dSecurityGroup` is required.                                                                                                          Yes        [TaxonomyAssociateObject](#taxonomyassociateobject)
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Taxonomy unassociated successfully.                                                             
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The dTaxonomyGUID was invalid                                                                   
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

## Get the Security Groups Associated with a Taxonomy

#### GET /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID}/securityGroups/ {#get-documentswccapiv11taxonomiesdtaxonomyguidsecuritygroups}

#### Description

List the security groups associated with a taxonomy. (TXY_GET_TAXONOMY_SECURITY_GROUPS) User needs to have admin role or R permission on the security group and the account (if Accounts is enabled) that are set on the taxonomy or have R permission on any security group mapped with the taxonomy

#### Parameters

::: table-responsive
  Name            Located in   Description                                                                                                                Required   Schema
  --------------- ------------ -------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dTaxonomyGUID   path         The taxonomy GUID.                                                                                                         Yes        string
  sortField       query        The field used to sort the security groups; fields can be sorted by `dSecurityGroup`. The default is `dSecurityGroup``.`   No         string
  sortOrder       query        The order to sort the security groups; supported values are `Asc` and `Desc`. The default is `Asc`.                        No         string
  startRow        query        The row number at which to start returning the security groups. This is for pagination. The defaults is 0.                 No         number
  count           query        The number of security groups to return. This is for pagination. The dDefault is 20; the maximum is 100.                   No         number
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- ---------------------------------------------------------------------
  200    Returned associated security groups successfully.                                              [TaxonomyAssociateResponseObject](#taxonomyassociateresponseobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The dTaxonomyGUID was invalid                                                                   
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

## Create a Category

#### POST /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID}/categories {#post-documentswccapiv11taxonomiesdtaxonomyguidcategories}

#### Description

Create a category. (TXY_CREATE_CATEGORY) User needs to have the admin role or RWDA permissions on all security groups and accounts (if enabled) referenced.

#### Parameters

::: table-responsive
  Name            Located in   Description                                                                                                        Required   Schema
  --------------- ------------ ------------------------------------------------------------------------------------------------------------------ ---------- -----------------------------------------------
  dTaxonomyGUID   path         The taxonomy GUID.                                                                                                 Yes        string
  body            body         The category properties object defining the category. The `dCategoryName` and `dParentGUID` fields are required.   Yes        [CategoryCreateObject](#categorycreateobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Category successfully created.\                                                                 
                          Returns Header `Location` which is a URI to get the info for the new category.                 

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  404                     The dTaxonomyGUID or dCategoryGUID was invalid                                                  

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

## Update a Category

#### PUT /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID}/categories/{dCategoryGUID}/ {#put-documentswccapiv11taxonomiesdtaxonomyguidcategoriesdcategoryguid}

#### Description

Update a category. (TXY_EDIT_CATEGORY) User needs to have admin role or RWDA permissions on the security group and the account (if enabled) that are set on the taxonomy.

#### Parameters

::: table-responsive
  Name            Located in   Description                                             Required   Schema
  --------------- ------------ ------------------------------------------------------- ---------- -----------------------------------------------
  dTaxonomyGUID   path         The taxonomy GUID.                                      Yes        string
  dCategoryGUID   path         The category GUID.                                      Yes        string
  body            body         The category properties object defining the category.   Yes        [CategoryUpdateObject](#categoryupdateobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204                     Category successfully updated.\                                                                 
                          Returns Header `Location` which is a URI to get the info for the updated category.             

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  403                     User is not allowed to take this action                                                         

  404                     The dTaxonomyGUID or dCategoryGUID was invalid                                                  

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

## Read a Category

#### GET /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID}/categories/{dCategoryGUID}/ {#get-documentswccapiv11taxonomiesdtaxonomyguidcategoriesdcategoryguid}

#### Description

Read a category. (TXY_GET_CATEGORY_INFO) User needs to have admin role or R permission on the security group and the account (if Accounts is enabled) that are set on the taxonomy or have R permission on any security group mapped with the taxonomy.

#### Parameters

::: table-responsive
  Name            Located in   Description                                                                                                          Required   Schema
  --------------- ------------ -------------------------------------------------------------------------------------------------------------------- ---------- ---------
  dTaxonomyGUID   path         The taxonomy GUID.                                                                                                   Yes        string
  dCategoryGUID   path         The category GUID.                                                                                                   Yes        string
  fields          query        Additional fields to return. Only `dChildren` is supported; when specified, children of the category are returned.   No         string
  limit           query        The maximum number of child categories returned per request, when `fields=dChildren` is specified.                   No         integer
  offset          query        The starting index from which child categories are returned in the response, when `fields=dChildren` is specified.   No         integer
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Category information                                                                           [CategoryObject](#categoryobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The dTaxonomyGUID or dCategoryGUID was invalid                                                  
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

## Delete a Category

#### DELETE /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID}/categories/{dCategoryGUID}/ {#delete-documentswccapiv11taxonomiesdtaxonomyguidcategoriesdcategoryguid}

#### Description

Delete an unreferenced category. (TXY_DELETE_CATEGORY) User needs to have admin role or RWDA permissions on the security group and the account (if enabled) that are set on the taxonomy.

#### Parameters

::: table-responsive
  Name            Located in   Description          Required   Schema
  --------------- ------------ -------------------- ---------- --------
  dTaxonomyGUID   path         The taxonomy GUID.   Yes        string
  dCategoryGUID   path         The category GUID.   Yes        string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  204    Successfully deleted the category                                                               
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The dTaxonomyGUID or dCategoryGUID was invalid                                                  
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

## Copy a Category

#### POST /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID}/categories/{dCategoryGUID}/.copy {#post-documentswccapiv11taxonomiesdtaxonomyguidcategoriesdcategoryguidcopy}

#### Description

Copy a category and descendants. (TXY_COPY_CATEGORY) User needs to have the admin role or RWDA permissions on the security groups and accounts (if enabled) set on the taxonomy.

#### Parameters

::: table-responsive
  Name            Located in   Description                                                              Required   Schema
  --------------- ------------ ------------------------------------------------------------------------ ---------- -------------------------------------------
  dTaxonomyGUID   path         The taxonomy GUID.                                                       Yes        string
  dCategoryGUID   path         The source category GUID of the category being copied.                   Yes        string
  body            body         The target category details. The `targetParentGUID` field is required.   Yes        [CategoryCopyObject](#categorycopyobject)
:::

#### Responses

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Code                    Description                                                                                    Schema
  ----------------------- ---------------------------------------------------------------------------------------------- -----------------------------------------------
  201                     Category and descendants successfully copied.\                                                  
                          Returns Header `Location` which is a URI to get the info for the copied category.              

  400                     Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)

  401                     Unauthorized                                                                                    

  404                     The dTaxonomyGUID or dCategoryGUID was invalid                                                  

  403                     User is not allowed to take this action                                                         

  500                     The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### Example {#example-96}

Copy category EAA0DFC5F9E0095A225334D21CD62361 under taxonomy A746D984C500F280923C36033F14D63D to target parent E39DEEC7AAB687B50EE7F18505BDB65B as its first child.

##### Request

> POST .../documents/wcc/api/v1.1/taxonomies/A746D984C500F280923C36033F14D63D/categories/EAA0DFC5F9E0095A225334D21CD62361/.copy

##### Request Body

> {
>
> "targetParentGUID": "E39DEEC7AAB687B50EE7F18505BDB65B",
>
> "targetPosition": "0"
>
> }

##### HTTP Response

> Status = 201

Body

None

Header

> Location = .../documents/wcc/api/v1.1/taxonomies/A746D984C500F280923C36033F14D63D/categories/96692109F7DB439CEE70FE49B63DDF0A

## Search categories

#### GET /documents/wcc/api/v1.1/taxonomies/{dTaxonomyGUID}/categories/ {#get-documentswccapiv11taxonomiesdtaxonomyguidcategories}

#### Description

Get information all the categories within the specified taxonomy. (TXY_GET_TAXONOMY_CATEGORIES) User needs to have admin role or R permission on the security group and the account (if Accounts is enabled) that are set on the taxonomy or have R permission on any security group mapped with the taxonomy.

#### Parameters

::: table-responsive
  Name            Located in   Description                                                                                                                                                                                                                                                               Required   Schema
  --------------- ------------ ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------- --------
  dTaxonomyGUID   path         The taxonomy GUID.                                                                                                                                                                                                                                                        Yes        string
  sortField       query        The field used to sort the categories; fields can be sorted by `dCategoryName` or `dPosition`. The default is `dCategoryName``.`                                                                                                                                          No         string
  sortOrder       query        The order to sort the categories; supported values are `Asc` and `Desc`. The default is `Asc`.                                                                                                                                                                            No         string
  startRow        query        The row number at which to start returning the taxonomies. This is for pagination. The defaults is 0.                                                                                                                                                                     No         number
  count           query        The number of categories to return. This is for pagination. The default is 20; the maximum is 100.                                                                                                                                                                        No         number
  QueryText       query        The search query to search within categories. See [Using Universal Query Format](#using-the-universal-query-format%22). The field `dCategoryName` supports both case-sensitive contains and exact matches. The fields `dParentGUID` and `dAPIName` support exact match.   No         string
:::

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------------------
  200    Category information                                                                           [CategoryListResponseObject](#categorylistresponseobject)
  400    Bad request                                                                                    [GeneralErrorResponse](#generalerrorresponse)
  401    Unauthorized                                                                                    
  403    User is not allowed to take this action                                                         
  404    The dTaxonomyGUID was invalid                                                                   
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

# Generic REST

## Generic GET API

#### GET /documents/wcc/api/v1.1/generic/invoke {#get-documentswccapiv11genericinvoke}

#### Description

Allows user to call any IdcService via a GET REST-like api.

#### Parameters

::: table-responsive
  Name                  Located in   Description                                                                                                                               Required   Schema
  --------------------- ------------ ----------------------------------------------------------------------------------------------------------------------------------------- ---------- ---------
  IdcService            query        The IdcService to invoke.                                                                                                                 Yes        string
  idcToken              query        The unique token required when invoking non-scriptable IdcServices                                                                        No         string
  OnErrorReturnBinder   query        When false, any error is returned as a [GeneralErrorResponse](#generalerrorresponse) ; By default, the service binder is returned.        No         boolean
  IsJson                query        When true, the response is returned in a HDA-JSON format; By default, the response is created by converting the service binder to JSON.   No         boolean
:::

Many other parameters are supported depending on the IdcService being invoked.

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- -----------------------------------------------
  200    Successfully returned the data or completed the action                                          
  400    Bad request                                                                                     
  401    Unauthorized or invalid `idcToken`                                                              
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request   [GeneralErrorResponse](#generalerrorresponse)
:::

#### Example 1 {#example-1}

Invoke PING_SERVER to verify the server is up.

##### Request

> GET .../documents/wcc/api/v1.1/generic/invoke?IdcService=PING_SERVER

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"IdcService": "PING_SERVER",
"refreshSubjects": "",
"dUser": "weblogic",
"changedSubjects": "",
"StatusMessageKey": "!csUserLoggedIn,weblogic",
"StatusMessage": "!csUserLoggedIn,weblogic",
"idcToken": "1736280901320:0B26876ED70B73F6CE0FFEEF504EBE94",
"UserAttribInfo": [
    {
        "dUserName": "weblogic",
        "AttributeInfo": "account,#all,15,account,#none,15,role,Administrators,15,role,admin,15,role,refineryadmin,15,role,rmaadmin,15,role,pcmadmin,15,role,ermadmin,15,role,sysmanager,15,role,guest,15,role,authenticated,15"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

#### Example 2

Invoke PING_SERVER to verify the server is up using HDA-JSON.

##### Request

> GET .../documents/wcc/api/v1.1/generic/invoke?IdcService=PING_SERVER&IsJson=1

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"LocalData": {
    "IdcService": "PING_SERVER",
    "IsJson": "1",
    "StatusMessage": "!csUserLoggedIn,weblogic",
    "StatusMessageKey": "!csUserLoggedIn,weblogic",
    "changedSubjects": "",
    "dUser": "weblogic",
    "idcToken": "1736281499183:9B031AD110220D0A1122B4A7B49C4C52",
    "refreshSubjects": ""
},
"ResultSets": {
    "UserAttribInfo": {
        "currentRow": 0,
        "fields": [
            {
                "name": "dUserName"
            },
            {
                "name": "AttributeInfo"
            }
        ],
        "rows": [
            [
                "weblogic",
                "account,#all,15,account,#none,15,role,Administrators,15,role,admin,15,role,refineryadmin,15,role,rmaadmin,15,role,pcmadmin,15,role,ermadmin,15,role,sysmanager,15,role,guest,15,role,authenticated,15"
            ]
        ]
    }
}
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

#### Example 3 {#example-3}

Invoke DOC_INFO on a document with dID 6402 with a specific timezone and date format.

##### Request

> GET .../documents/wcc/api/v1.1/generic/invoke?IdcService=DOC_INFO&dID=6402&UserTimeZone=UTC&UserDateFormat=yyyy-MM-dd'T'HH:mm:ssZ!tUTC

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"dDocName": "PUB2",
"encodeDocUrl": "",
"refreshSubjects": "",
"hasAnnotations": "0",
"hasStickyNotes": "0",
"dComprehensivePrivilege": "15",
"ProxyNativeFormat": "",
"dpTriggerField": "xIdcProfile",
"UserDateFormat": "yyyy-MM-dd'T'HH:mm:ssZ!tUTC",
"UserTimeZone": "UTC",
"dSubscriptionAlias": "weblogic",
"hasRestrictedRedactions": "0",
"dStatus": "RELEASED",
"dSubscriptionType": "Folder",
"dDocTitle": "test1",
"dSubscriptionID": " ",
"dpAction": "Info",
"DocUrl": "http://wcchost.com:16200/cs/groups/public/documents/document/mdaw/mdbw/~edisp/pub2.jpg",
"hasStandardRedactions": "0",
"dUser": "weblogic",
"dDocFormats": "image/jpeg",
"UseForwardOnlyCursor": "",
"isDocProfileUsed": "true",
"IdcService": "DOC_INFO",
"dPrivilege": "15",
"IsQueryObjectPersistent": "",
"isDocProfileDone": "1",
"changedSubjects": "",
"dID": "6402",
"dpEvent": "OnRequest",
"idcToken": "1736282245205:3DB05D77481F2DE1CB4CC055F44AF120",
"DOC_INFO": [
    {
        "dID": "6402",
        "dDocName": "PUB2",
        "dDocType": "Document",
        "dDocTitle": "test1",
        "dDocAuthor": "weblogic",
        "dRevClassID": "6602",
        "dRevisionID": "1",
        "dRevLabel": "1",
        "dIsCheckedOut": "0",
        "dCheckoutUser": "",
        "dSecurityGroup": "Public",
        "dCreateDate": "2025-01-07T20:02:39Z",
        "dInDate": "2025-01-07T20:02:00Z",
        "dOutDate": "",
        "dStatus": "RELEASED",
        "dReleaseState": "Y",
        "dFlag1": "",
        "dWebExtension": "jpg",
        "dProcessingState": "Y",
        "dMessage": "",
        "dDocAccount": "",
        "dReleaseDate": "2025-01-07T20:02:46Z",
        "dRendition1": "",
        "dRendition2": "",
        "dIndexerState": "",
        "dPublishType": "",
        "dPublishState": "",
        "dWorkflowState": "",
        "dRevRank": "0",
        "dDocID": "6803",
        "dIsPrimary": "1",
        "dIsWebFormat": "0",
        "dLocation": "",
        "dOriginalName": "primary.jpg",
        "dFormat": "image/jpeg",
        "dExtension": "jpg",
        "dFileSize": "53834",
        "dLanguage": "",
        "dCharacterSet": "",
        "xComments": "",
        "xExternalDataSet": "",
        "xIdcProfile": "",
        "xTemplateType": "",
        "xAnnotationDetails": "0",
        "xLibraryGUID": "",
        "xPartitionId": "",
        "xWebFlag": "",
        "xStorageRule": "DispByContentId",
        "xIsACLReadOnlyOnUI": "0",
        "dIndexedID": "6402",
        "dDocCreatedDate": "2025-01-07T20:02:39Z",
        "dDocCreator": "weblogic",
        "dDocLastModifiedDate": "2025-01-07T20:02:39Z",
        "dDocLastModifier": "weblogic",
        "dDocOwner": "weblogic",
        "dDocFunction": "",
        "dDocClass": ""
    }
],
"REVISION_HISTORY": [
    {
        "dFormat": "image/jpeg",
        "dInDate": "2025-01-07T20:02:00Z",
        "dOutDate": "",
        "dStatus": "RELEASED",
        "dProcessingState": "Y",
        "dRevLabel": "1",
        "dID": "6402",
        "dDocName": "PUB2",
        "dRevisionID": "1",
        "dDocAuthor": "weblogic"
    }
],
"UserAttribInfo": [
    {
        "dUserName": "weblogic",
        "AttributeInfo": "account,#all,15,account,#none,15,role,Administrators,15,role,admin,15,role,refineryadmin,15,role,rmaadmin,15,role,pcmadmin,15,role,ermadmin,15,role,sysmanager,15,role,guest,15,role,authenticated,15"
    }
],
"AssociatedTopFields": null,
"RestrictedLists": null
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

## Generic POST API

#### POST /documents/wcc/api/v1.1/generic/invoke {#post-documentswccapiv11genericinvoke}

#### Description

Allows user to call any IdcService via a POST REST-like api.

#### Parameters

::: table-responsive
  Name                  Located in   Description                                                                                                                               Required   Schema
  --------------------- ------------ ----------------------------------------------------------------------------------------------------------------------------------------- ---------- ---------
  IdcService            formData     The IdcService to invoke.                                                                                                                 Yes        string
  primaryFile           formData     The primary file for this document.                                                                                                       No         file
  alternateFile         formData     The alternate file for this document.                                                                                                     No         file
  idcToken              formData     The unique token required when invoking non-scriptable IdcServices.                                                                       No         string
  OnErrorReturnBinder   formData     When false, any error is returned as a [GeneralErrorResponse](#generalerrorresponse) ; By default, the service binder is returned.        No         boolean
  IsJson                formData     When true, the response is returned in a HDA-JSON format; By default, the response is created by converting the service binder to JSON.   No         boolean
:::

Many other parameters are supported depending on the IdcService being invoked.

#### Responses

::: table-responsive
  Code   Description                                                                                    Schema
  ------ ---------------------------------------------------------------------------------------------- --------
  200    Successfully returned the data or completed the action                                          
  400    Bad request                                                                                     
  401    Unauthorized or invalid `idcToken`                                                              
  403    User is not allowed to take this action                                                         
  500    The server encountered an unexpected condition that prevented it from fulfilling the request    
:::

#### Example 1 {#example-1-1}

Invoke PING_SERVER to verify the server is up.

##### Request

> POST .../documents/wcc/api/v1.1/generic/invoke

##### Request Body

FormData Parameters

> IdcService = PING_SERVER

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"IdcService": "PING_SERVER",
"refreshSubjects": "",
"dUser": "weblogic",
"changedSubjects": "",
"StatusMessageKey": "!csUserLoggedIn,weblogic",
"StatusMessage": "!csUserLoggedIn,weblogic",
"idcToken": "1736280901320:0B26876ED70B73F6CE0FFEEF504EBE94",
"UserAttribInfo": [
    {
        "dUserName": "weblogic",
        "AttributeInfo": "account,#all,15,account,#none,15,role,Administrators,15,role,admin,15,role,refineryadmin,15,role,rmaadmin,15,role,pcmadmin,15,role,ermadmin,15,role,sysmanager,15,role,guest,15,role,authenticated,15"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

#### Example 2

Invoke CHECKIN_UNIVERSAL to checkin a file.

##### Request

> POST .../documents/wcc/api/v1.1/generic/invoke

##### Request Body

FormData Parameters

> IdcService = CHECKIN_UNIVERSAL
>
> dDocTitle = GENERIC_REST_UPLOAD
>
> dSecurityGroup = Public
>
> dDocType = Document
>
> idcToken = 1736283993479:7E39ED06BAEDC0AE4476ABA5B21EABC7
>
> primaryFile = \[filePath\]

##### HTTP Response

> Status = 200

Body

::: {.language-plaintext .highlighter-rouge}
::: highlight
::: {role="separator" aria-label="Code Block start"}
:::

``` highlight
Copy{
"dDocOwner": "weblogic",
"dWebOriginalName": "MP006606~1.jpg",
"xAnnotationDetails:isSetDefault": "1",
"dActionDate": "1/7/25 2:43 PM",
"xDocumentTags": "",
"dpTriggerField": "xIdcProfile",
"dClbraName": "",
"isCheckin": "1",
"dWebExtension": "jpg",
"scriptableActionType": "3",
"LockedContents1": "dDocName:MP006606",
"xStorageRule": "DispByContentId",
"isEditMode": "1",
"VaultfilePath": "/Oracle12/Middleware/Oracle_Home/user_projects/domains/mp/ucm/cs/vault/document/mg1w/mda2/6406.jpg",
"IdcService": "CHECKIN_UNIVERSAL",
"scriptableActionFlags": "12",
"dOutDate": "",
"dRevClassID": "6606",
"dAction": "Checkin",
"idcToken": "1736284437645:95165A90739D0046AD06FEC37B9BCF7B",
"dDocCreator": "weblogic",
"dDocLastModifier": "weblogic",
"dStatus": "DONE",
"dInDate": "{ts '2025-01-07 14:43:57.664'}",
"xIdcProfile:isSetDefault": "1",
"scriptableActionErr": "",
"xPartitionId": "",
"isStatusChanged": "1",
"dReleaseState": "N",
"xTemplateType": "",
"UseForwardOnlyCursor": "",
"dConversion": "PassThru",
"scriptableActionFunction": "determineCheckin",
"dDocLastModifiedDate": "{ts '2025-01-07 14:43:57.664'}",
"RenditionId": "webViewableFile",
"xExternalDataSet:isSetDefault": "1",
"dPublishState": "",
"dID": "6406",
"xLibraryGUID": "",
"dpEvent": "OnImport",
"xIsACLReadOnlyOnUI": "0",
"xComments": "",
"StatusCode": "0",
"xLibraryGUID:isSetDefault": "1",
"reserveLocation": "false",
"isInfoOnly": "",
"dRawDocID": "6811",
"dDocTitle": "GENERIC_REST_UPLOAD",
"dFileSize": "53834",
"StatusMessageKey": "!csServiceStatusMessage_checkin,MP006606",
"dpAction": "CheckinNew",
"scriptableActionParams": "",
"dActionMillis": "384417675",
"dIsPrimary": "1",
"dExtension": "jpg",
"dProcessingState": "Y",
"dWorkflowState": "",
"isDocProfileUsed": "true",
"dDocType": "Document",
"noDocLock": "1",
"dDocCreatedDate": "{ts '2025-01-07 14:43:57.664'}",
"IsQueryObjectPersistent": "",
"xWebFlag:isSetDefault": "1",
"changedSubjects": "documents,1736197393848",
"xIsACLReadOnlyOnUI:isSetDefault": "1",
"xTemplateType:isSetDefault": "1",
"xPartitionId:isSetDefault": "1",
"dDocName": "MP006606",
"xIdcProfile": "",
"dDocAuthor": "weblogic",
"dIsWebFormat": "0",
"refreshSubjects": "",
"dPublishType": "",
"xComments:isSetDefault": "1",
"xClbraUserList": "",
"dFormat": "image/jpeg",
"xStorageRule:isSetDefault": "1",
"xParentFolders:isSetDefault": "1",
"xDocumentTags:isSetDefault": "1",
"dRevRank": "0",
"dDocID": "6812",
"xClbraAliasList": "",
"xParentFolders": "",
"dLocation": "",
"primaryFile": "primary.jpg",
"StorageRule": "DispByContentId",
"xWebFlag": "",
"dOriginalName": "primary.jpg",
"dUser": "weblogic",
"isNew": "1",
"xAnnotationDetails": "0",
"dSecurityGroup": "Public",
"StatusMessage": "!csServiceStatusMessage_checkin,MP006606",
"dFlag1": "",
"dCreateDate": "{ts '2025-01-07 14:43:57.664'}",
"xExternalDataSet": "",
"prevReleaseState": "",
"DocExists": "",
"dRevisionID": "1",
"isDocProfileDone": "1",
"dRevLabel": "1",
"dDocAccount": "",
"WebfilePath": "/Oracle/Middleware/Oracle_Home/user_projects/domains/mp/ucm/cs/weblayout/groups/public/documents/document/mg1w/mda2/~edisp/mp006606~1.jpg",
"UserAttribInfo": [
    {
        "dUserName": "weblogic",
        "AttributeInfo": "account,#all,15,account,#none,15,role,Administrators,15,role,admin,15,role,refineryadmin,15,role,rmaadmin,15,role,pcmadmin,15,role,ermadmin,15,role,sysmanager,15,role,guest,15,role,authenticated,15"
    }
]
}
```

::: {role="separator" aria-label="Code Block end"}
:::
:::
:::

# Models

#### AppLinkCreateRequestObject

The following fields describe an applink request body for a folder or a file.

::: table-responsive
  ------------------------------------------------------------------------------------------------------------
  Name              Type              Description                                            Required
  ----------------- ----------------- ------------------------------------------------------ -----------------
  dAssignedUser     string            The assigned user of an applink.                       Yes

  dLinkPrivilege    string            The privilege of an applink. The available options:\   No
                                      •R\                                                    
                                      •RW\                                                   
                                      •RWD\                                                  
                                      •RWDA\                                                 
                                      The default is 'R'                                     

  dUserLocale       string            The locale of an applink.                              No

  dUserTimeZone     string            The time zone of an applink.                           No
  ------------------------------------------------------------------------------------------------------------
:::

#### AppLinkCreateResponseObject

The following fields describe an applink response when a new applink is created for a folder or a file.

::: table-responsive
  Name                   Type     Description                  Required
  ---------------------- -------- ---------------------------- ----------
  dAppLinkAccessToken    string   The applink access token.    No
  dAppLinkID             string   The applink ID.              No
  dAppLinkRefreshToken   string   The applink refresh token.   No
  dEmbedUrl              string   The applink embed URL.       No
:::

#### AppLinkRefreshObject

The following fields describe an applink access token.

::: table-responsive
  Name                   Type     Description                  Required
  ---------------------- -------- ---------------------------- ----------
  dAppLinkAccessToken    string   The applink access token.    Yes
  dAppLinkRefreshToken   string   The applink refresh token.   Yes
:::

#### AttachmentListResponse

The following fields describe a list of attachments.

::: table-responsive
  Name    Type                                             Description                              Required
  ------- ------------------------------------------------ ---------------------------------------- ----------
  count   number                                           The number of attachments in the list.   No
  items   array of [AttachmentObject](#attachmentobject)   An array of attachment objects.          No
:::

#### AttachmentObject

The following fields describe an attachment.

::: table-responsive
  Name                       Type     Description                           Required
  -------------------------- -------- ------------------------------------- ----------
  extRenditionName           string   The name of the attachment.           No
  extRenditionDescription    string   The description of the attachment.    No
  extRenditionOriginalName   string   The file name of the attachment.      No
  extRenditionType           string   The (MIME) type of the attachment.    No
  extRenditionFileSize       number   The size of the rendition in bytes.   No
:::

#### BackgroundJobRequestObject

The following fields describe how a background job gets the file IDs to act on.

::: table-responsive
  Name      Type     Description                                                                                         Required
  --------- -------- --------------------------------------------------------------------------------------------------- ----------
  itemIds   string   A comma-separated list of IDs in the form: `dID:[DID],dDocName:[DDOCNAME],fFileGUID:[fFileGUID]`.   Yes
:::

#### BackgroundCategoryActionRequestObject

The following fields describe how to add or remove categories.

::: table-responsive
  Name            Type     Description                                                                                         Required
  --------------- -------- --------------------------------------------------------------------------------------------------- ----------
  itemIds         string   A comma-separated list of IDs in the form: `dID:[DID],dDocName:[DDOCNAME],fFileGUID:[fFileGUID]`.   Yes
  taxonomyGUID    string   The GUID of a taxonomy.                                                                             Yes
  categoryGUIDs   string   A comma-separated list of category GUIDs under the taxonomy to be added or removed.                 Yes
:::

#### BackgroundCategoryCopyRequestObject

The following fields describe how to copy a category.

::: table-responsive
  Name                 Type     Description                                                                                                                                                                                                  Required
  -------------------- -------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ----------
  taxonomyGUID         string   The GUID of a taxonomy.                                                                                                                                                                                      Yes
  sourceCategoryGUID   string   The source category GUID in the taxonomy to copy.                                                                                                                                                            Yes
  targetParentGUID     string   The category GUID of a target parent category in the same taxonomy.                                                                                                                                          Yes
  targetPosition       number   The position of the copied category under the parent target category. The position is a 0-based positive integer. If not provided, the copied category is the last child under the parent target category.   No
:::

#### BackgroundJobStatus

The following fields describe the status of a background job. It's possible for jobs to have additional fields.

::: table-responsive
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                  Type                                                                        Description                                                                                                                                                                             Required
  --------------------- --------------------------------------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -----------------
  completed             boolean                                                                     A boolean indicating if the job is complete.                                                                                                                                            No

  completedPercentage   number                                                                      A number between 0 and 100 capturing how much of the process has been completed.                                                                                                        No

  state                 string                                                                      A word indicating the state of the job. The supported states are:\                                                                                                                      No
                                                                                                    •COMPLETED -- Job Completed Successfully. Note that this state means that at least a part of the background job was successful. The entire job may have failures even in this state.\   
                                                                                                    •FAILED -- The job failed.\                                                                                                                                                             
                                                                                                    •PROCESSING -- The job is currently being worked on.\                                                                                                                                   
                                                                                                    •PENDING -- The job is queued for the background thread pool.\                                                                                                                          
                                                                                                    •CANCELED -- The job was issued a cancel request.                                                                                                                                       

  message               string                                                                      A brief message about the status of the job.                                                                                                                                            No

  itemsProcessed        number                                                                      The number of files processed by the job.                                                                                                                                               No

  totalItemsCount       number                                                                      The total number of items expected to be processed.                                                                                                                                     No

  failedItemsCount      number                                                                      The number of items that have failed to be processed.                                                                                                                                   No

  failedItems           An array of [BackgroundJobFailedItems](#backgroundjobfaileditems)           An array of items that failed to be processed.                                                                                                                                          No

  downloadDetails       [BackgroundJobDownloadDetailsObject](#backgroundjobdownloaddetailsobject)   An object describing the file the server created for the job.                                                                                                                           No
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### BackgroundJobFailedItems

The following fields describe a background job's failed items.

::: table-responsive
  Name      Type     Description                          Required
  --------- -------- ------------------------------------ ----------
  id        string   The item id that failed.             No
  message   string   The error message for the failure.   No
:::

#### BackgroundJobDownloadDetailsObject

The following fields describe a file created by the background job.

::: table-responsive
  Name       Type                                              Description                                     Required
  ---------- ------------------------------------------------- ----------------------------------------------- ----------
  filename   string                                            The file name of the file created by the job.   No
  filesize   number                                            The size of the file in bytes.                  No
  links      [BackgroundJobDownload](#backgroundjobdownload)   The URI information to access the file.         No
:::

#### BackgroundJobDownload

The following fields describe the URI to access a file created by the background job.

::: table-responsive
  Name        Type     Description                                 Required
  ----------- -------- ------------------------------------------- ----------
  href        string   The URI to access the download.             No
  rel         string   The type of link.                           No
  method      string   The HTTP method used to access the URI.     No
  mediaType   string   The MIME type of the file being returned.   No
:::

#### CategoryCopyObject

The following fields decribe the target category.

::: table-responsive
  Name               Type     Description                                                                                                                                                                                                  Required
  ------------------ -------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ----------
  targetParentGUID   string   The GUID of the parent target category.                                                                                                                                                                      Yes
  targetPosition     number   The position of the copied category under the parent target category. The position is a 0-based positive integer. If not provided, the copied category is the last child under the parent target category.   No
:::

#### CategoryCreateObject

The following fields describe a category.

::: table-responsive
  Name            Type     Description                                                                                                                                                                           Required
  --------------- -------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  dCategoryName   string   The name of the category.                                                                                                                                                             Yes
  dParentGUID     string   The parent category GUID. A top level category uses the taxonomy GUID as the parent.                                                                                                  Yes
  dDescription    string   The decription of the category.                                                                                                                                                       No
  dAPIName        string   An API name of the category. If not provided, the server will generate an API name.                                                                                                   No
  dPosition       number   The position of the category under the parent. The position is a 0-based positive integer. If not provided, the server will assign the position based on the the previous category.   No
:::

#### CategorySimpleObject

The following fields describe a category.

::: table-responsive
  Name                Type     Description                                                                                        Required
  ------------------- -------- -------------------------------------------------------------------------------------------------- ----------
  dCategoryGUID       string   The category GUID.                                                                                 No
  dCategoryName       string   The name of the category.                                                                          No
  dDescription        string   The description of the category.                                                                   No
  dParentGUID         string   The parent taxonomy or category GUID. A top level category uses the taxonomy GUID as the parent.   No
  dAPIName            string   An API name of the category.                                                                       No
  dPosition           number   The position of the category under the parent.                                                     No
  dPath               string   The full name path from the root category to this category separated by the `/`` character.`       No
  dGUIDPath           string   The full GUID path from the root category to this category separated by the `/` character.         No
  dAPINamePath        string   The full API name path from the root category to this category separated by the `/` character.     No
  dLastModifiedDate   string   The last modified date of the category.                                                            No
:::

#### CategoryObject

The following fields describe a category.

::: table-responsive
  Name                Type                                                                              Description                                                                                        Required
  ------------------- --------------------------------------------------------------------------------- -------------------------------------------------------------------------------------------------- ----------
  dCategoryGUID       string                                                                            The category GUID.                                                                                 No
  dCategoryName       string                                                                            The name of the category.                                                                          No
  dDescription        string                                                                            The description of the category.                                                                   No
  dParentGUID         string                                                                            The parent taxonomy or category GUID. A top level category uses the taxonomy GUID as the parent.   No
  dAPIName            string                                                                            An API name of the category.                                                                       No
  dPosition           number                                                                            The position of the category under the parent.                                                     No
  dPath               string                                                                            The full name path from the root category to this category separated by the `/`` character.`       No
  dGUIDPath           string                                                                            The full GUID path from the root category to this category separated by the `/` character.         No
  dAPINamePath        string                                                                            The full API name path from the root category to this category separated by the `/` character.     No
  dLastModifiedDate   string                                                                            The last modified date of the category.                                                            No
  dChildren           array [CategoryChildrenListResponseObject](#categorychildrenlistresponseobject)   Information about the child categories.                                                            No
:::

#### CategoryChildrenListResponseObject

The following describes a list of child categories.

::: table-responsive
  Name            Type                                                            Description                                                                                                                                                                            Required
  --------------- --------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  offset          number                                                          The starting index from which the child categories are returned.                                                                                                                       No
  limit           number                                                          The number of child categories.                                                                                                                                                        No
  count           number                                                          The number of child categories returned.                                                                                                                                               No
  hasMore         boolean                                                         This is `true` if the request did not return all of the child categories. This occurs when limit is reached and there are additional child categories that could have been returned.   No
  toltalResults   number                                                          The total number of child categories available.                                                                                                                                        No
  categories      An array of [CategoryChildrenObject](#categorychildrenobject)   List of child categories information.                                                                                                                                                  No
:::

#### CategoryChildrenObject

The following represents a child category.

::: table-responsive
  Name            Type     Description                                            Required
  --------------- -------- ------------------------------------------------------ ----------
  dCategoryGUID   string   The child category GUID.                               No
  dCategoryName   string   The name of the child category.                        No
  dDescription    string   The description of the child category.                 No
  dAPIName        string   An API name of the child category.                     No
  dPosition       number   The position of the child category under the parent.   No
:::

#### CategoryListResponseObject

The following describes a list of categories.

::: table-responsive
  Name            Type                                                  Description                                                                                                                                                                Required
  --------------- ----------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  offset          number                                                The starting index from which the categories are returned.                                                                                                                 No
  limit           number                                                The number of categories.                                                                                                                                                  No
  count           number                                                The number of categories returned.                                                                                                                                         No
  orderBy         string                                                The sort field and order for the list.                                                                                                                                     No
  hasMore         boolean                                               This is `true` if the request did not return all of the categories. This occurs when limit is reached and there are additional vategories that could have been returned.   No
  toltalResults   number                                                The total number of categories available.                                                                                                                                  No
  categories      An array of [CategoryObject](#categorysimpleobject)   List of categories information.                                                                                                                                            No
:::

#### CategoryUpdateObject

The following fields can be updated for a category. If optional values are not supplied, the current values will be kept.

::: table-responsive
  Name            Type     Description                                                                                                                                               Required
  --------------- -------- --------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  dCategoryName   string   The name of the category.                                                                                                                                 No
  dDescription    string   The decription of the category.                                                                                                                           No
  dParentGUID     string   The parent category GUID. This field is updatable if the category and all of its descendants are not assigned to any content item.                        No
  dAPIName        string   An API name of the category. If not provided, the server will generate an API name.                                                                       No
  dPosition       number   The position of the category under the parent. This field is updatable if the category and all of its descendants are not assigned to any content item.   No
:::

#### GeneralErrorResponse

General error response

::: table-responsive
  Name          Type     Description                                                                   Required
  ------------- -------- ----------------------------------------------------------------------------- ----------
  type          string   A link that describes the type of error.                                      No
  title         string   A brief summary error message.                                                No
  detail        string   Details about the error from the server. The service `StatusMessage`.         No
  errorKey      string   When the error comes from the service layer, the service `StatusMessageKey`   No
  o:errorCode   number   When the error comes from the service layer, the service `StatusCode`         No
:::

#### ResubmitConflictErrorResponse

Resubmit Conflict 409 Response

::: table-responsive
  Name          Type     Description                                                                              Required
  ------------- -------- ---------------------------------------------------------------------------------------- ----------
  type          string   https://www.rfc-editor.org/rfc/rfc9110.html#name-409-conflict                            No
  title         string   Unable to resubmit.                                                                      No
  detail        string   Unable to resubmit content item. The content item is not in a failed conversion state.   No
  errorKey      string   !csUnableToResubmitItem2!csResubmitNotFailed                                             No
  o:errorCode   number   -1                                                                                       No
:::

#### MetadataChangeObjectParameter

The following fields are examples of what can be changed as metadata for a content item. Other fields may be allowed.

::: table-responsive
  Name             Type     Description                          Required
  ---------------- -------- ------------------------------------ ----------
  dDocType         string   The type of document                 No
  dDocTitle        string   The title of the document            No
  dRevLabel        string   The revision of the document         No
  dSecurityGroup   string   The security group of the document   No
  dDocName         string   The id of the document               No
  dDocAuthor       string   The author of the document           No
  xComments        string   String comments about the document   No
:::

#### MetadataObjectResponse

The following fields are examples of what can be returned as metadata for a content item.

::: table-responsive
  Name                   Type     Description                                                                            Required
  ---------------------- -------- -------------------------------------------------------------------------------------- ----------
  dDocType               string   The type of document                                                                   No
  dDocTitle              string   The title of the document                                                              No
  dRevLabel              string   The revision of the document                                                           No
  dSecurityGroup         string   The security group of the document                                                     No
  dDocName               string   The id of the document                                                                 No
  dDocAuthor             string   The author of the document                                                             No
  dStatus                string   This field is a summary field of computations from other status fields.                No
  dOriginalName          string   The document file name                                                                 No
  dFormat                string   The document mime type                                                                 No
  dFileSize              number   The document filesize (in bytes)                                                       No
  dDocCreatedDate        string   The date and time the document was first uploaded into the server in ISO-8601 format   No
  dDocCreator            string   The user who originally created the document                                           No
  dDocLastModifiedDate   string   The date and time any revision of the document was last modified in ISO-8601 format    No
  dDocLastModifier       string   The last user to modify the document                                                   No
:::

#### MetadataVersionsResponse

The response returning the metadata for all versions of a document.

::: table-responsive
  Name              Type                                                      Description                                      Required
  ----------------- --------------------------------------------------------- ------------------------------------------------ ----------
  dDocName          string                                                    The id of the document                           No
  count             number                                                    The number of versions returned                  No
  dRevLabelLatest   string                                                    The latest revision of this document             No
  items             array [MetadataObjectResponse](#metadataobjectresponse)   An array of metadata objects for each revision   No
:::

#### SearchResultsResponse

The response returning search results.

::: table-responsive
  Name           Type                                                      Description                                                              Required
  -------------- --------------------------------------------------------- ------------------------------------------------------------------------ ----------
  count          number                                                    The number of documents returned.                                        No
  hasMore        boolean                                                   When false, all the results for the search are returned.                 No
  offset         number                                                    The offset when used to return a limited number of search results.       No
  totalResults   number                                                    The total number of documents that satisfy the search.                   No
  limit          number                                                    The maximum number of items listed.                                      No
  q              string                                                    The search query used to generate this response.                         No
  orderBy        string                                                    The sort field and sort order used to generate this response.            No
  pageNumber     number                                                    The page number of items in the response (useful in pagination).         No
  numPages       number                                                    The total number of pages returned.                                      No
  totalRows      number                                                    The total number of rows returned.                                       No
  startRow       number                                                    The start row number of `items` returned.                                No
  endRow         number                                                    The end row number of `items` returned.                                  No
  items          array [MetadataObjectResponse](#metadataobjectresponse)   An array of metadata objects for documents that meet the search query.   No
:::

#### WorkInProgressResponse

The response returning work in progress.

::: table-responsive
  Name                       Type                                                      Description                                                                Required
  -------------------------- --------------------------------------------------------- -------------------------------------------------------------------------- ----------
  count                      number                                                    The number of documents returned.                                          No
  hasMore                    boolean                                                   When false, all the results are returned.                                  No
  offset                     number                                                    The offset when used to return a limited number of search results.         No
  totalResults               number                                                    The total number of documents that satisfy the search.                     No
  limit                      number                                                    The maximum number of items listed.                                        No
  orderBy                    string                                                    The sort field and sort order used to generate this response.              No
  pageNumber                 number                                                    The page number of items in the response (useful in pagination).           No
  repository                 string                                                    The name of the repository.                                                No
  computedSearchEngineName   string                                                    The name of the search engine.                                             No
  items                      array [MetadataObjectResponse](#metadataobjectresponse)   An array of metadata objects for documents that are in work in progress.   No
:::

#### FolderBrowseResponse

The response when browsing a folder.

::: table-responsive
  Name                     Type                                            Description                                                                                                                                                                         Required
  ------------------------ ----------------------------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  numFolders               number                                          The number of folders in the folder.                                                                                                                                                No
  hasMoreChildFolders      number                                          This is `1` if the request did not return all of the child folders. This occurs when folderCount is reached and there are additional folders that could have been returned.         No
  numFiles                 number                                          The number of files in the folder.                                                                                                                                                  No
  totalChildFoldersCount   number                                          The total number of folders in the parent folder.                                                                                                                                   No
  totalChildFilesCount     number                                          The total number of files in the parent folder.                                                                                                                                     No
  hasMoreChildFiles        number                                          This is `1` if the request did not return all of the child files. This occurs when fileCount is reached and there are additional documents that could have been returned.           No
  hasMoreChildItems        number                                          This is `1` if the request did not return all of the child items (folders+files). This occurs when count is reached and there are additional items that could have been returned.   No
  folderPath               string                                          The path for folder.                                                                                                                                                                No
  folderInfo               [FileInfoObject](#fileinfoobject)               Information about the folder.                                                                                                                                                       No
  childFolders             An array of [FileInfoObject](#fileinfoobject)   An array of information about all folders that exist in the folder.                                                                                                                 No
  childTargetFolders       An array of [FileInfoObject](#fileinfoobject)   An array of information about all folder targets that exist in the folder.                                                                                                          No
  childFiles               [FileInfoObject](#fileinfoobject)               Information about all files in the folder.                                                                                                                                          No
:::

#### FolderInfoResponse

The response for folder information.

::: table-responsive
  Name         Type                                    Description                                                           Required
  ------------ --------------------------------------- --------------------------------------------------------------------- ----------
  folderPath   string                                  The path to the folder.                                               No
  targetPath   string                                  If the path is a shortcut, the path to the target folder.             No
  folderInfo   [FolderInfoObject](#folderinfoobject)   The information about the folder.                                     No
  targetInfo   [FolderInfoObject](#folderinfoobject)   If the path is a shortcut, the information about the target folder.   No
:::

#### FileInfoResponse

The response for file information.

::: table-responsive
  Name       Type                                  Description                       Required
  ---------- ------------------------------------- --------------------------------- ----------
  filePath   string                                The path to the file.             No
  fileInfo   [FileInfoObject](#folderinfoobject)   The information about the file.   No
:::

#### FolderInfoObject

Folder information.

::: table-responsive
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                         Type              Description                                                                                                               Required
  ---------------------------- ----------------- ------------------------------------------------------------------------------------------------------------------------- -----------------
  fFolderGUID                  string            The folder GUID.                                                                                                          No

  fParentGUID                  string            The GUID of the parent folder                                                                                             No

  fFolderName                  string            The folder name.                                                                                                          No

  fFolderType                  string            The folder type. Can be `owner` for created folders or `soft` for shortcuts.                                              No

  fInhibitPropagation          number            A bitmap used to determine restrictions on files/folders from propagating values from its parent. The bitmap includes:\   No
                                                 0 - inhibit propagation is set to none indicating values can be inherited from parent\                                    
                                                 1 - inhibit propagation for metadata indicating metadata is not inherited from parent\                                    
                                                 16 - inhibit propagation for folder security indicating folder security is not inherited from parent\                     
                                                 17 - inhibit propagation for metadata and folder security                                                                 

  fPromptForMetadata           number            Indicates how to prompt for metadata; if `0` then do not need to prompt for metadata.                                     No

  fIsContribution              number            When `1`, items can be contributed to the folder.                                                                         No

  fIsInTrash                   number            When `1` the folder is in the trash.                                                                                      No

  fRealItemGUID                string            The folder actual GUID.                                                                                                   No

  fLibraryType                 number            The library type. accepted values are:\                                                                                   No
                                                 0 - not a library folder\                                                                                                 
                                                 1 - enterprise library folder\                                                                                            
                                                 2 - application library folder\                                                                                           
                                                 3 - system library folder                                                                                                 

  fIsLibrary                   number            When `1` the folder is a Library Folder.                                                                                  No

  fDocClasses                  string            The list folder classes.                                                                                                  No

  fTargetGUID                  string            If the folder is a shortcut, the target GUID                                                                              No

  fApplication                 string            The folder application.                                                                                                   No

  fOwner                       string            The owner of the folder.                                                                                                  No

  fCreator                     string            The creator of the folder.                                                                                                No

  fLastModifier                string            The last modifier on the folder                                                                                           No

  fCreateDate                  string            The date the folder was created in ISO-8601 format.                                                                       No

  fLastModifiedDate            string            The date the folder was modified in ISO-8601 format.                                                                      No

  fSecurityGroup               string            The folder security group.                                                                                                No

  fDocAccount                  string            The folder account.                                                                                                       No

  fClbraUserList               string            The folder collaboration user list.                                                                                       No

  fClbraAliasList              string            The folder collaboration alias list.                                                                                      No

  fClbraRoleList               string            The folder collaboration role list.                                                                                       No

  fFolderDescription           string            The folder description.                                                                                                   No

  fChildFoldersCount           number            The number of folders in the folder.                                                                                      No

  fChildFilesCount             number            The number of files in the folder.                                                                                        No

  fFolderSize                  number            The number of bytes the folder uses.                                                                                      No

  fAllocatedFolderSize         number            The number of bytes allocated to the folder. If the limit was not set, `-1` is used.                                      No

  fAllocatorParentFolderGUID   string            The GUID of the parent folder the allocation effects.                                                                     No

  fApplicationGUID             string            The folder application GUID                                                                                               No

  fIsReadOnly                  number            When `1` the folder is read only.                                                                                         No

  fIsACLReadOnlyOnUI           number            When `1` the folder should be displayed as read only.                                                                     No

  fDisplayName                 string            The folder display name.                                                                                                  No

  fDisplayDescription          string            The folder display description.                                                                                           No

  fIsBrokenShortcut            number            When `1` the shortcut broken.                                                                                             No

  isLeaf                       string            When `1` the folder is a leaf.                                                                                            No

  itemType                     string            The folder type; the value `1` is a folder, `2` is a file, and `3` is a document.                                         No

  folderPermissions            string            The user permissions on the folder; can be none, or a combination of `R`ead, `W`rite, `D`elete or `A`ccess.               No
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### FileInfoObject

File information and the metadata associated with the file, fields from [MetadataObjectResponse](#metadataobjectresponse) can be included in this object.

::: table-responsive
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                  Type              Description                                                                                                               Required
  --------------------- ----------------- ------------------------------------------------------------------------------------------------------------------------- -----------------
  fFileGUID             string            The file GUID.                                                                                                            No

  fParentGUID           string            The parent GUID of the file.                                                                                              No

  fTargetGUID           string            The GUID of the target file.                                                                                              No

  fFileName             string            The file name of the file.                                                                                                No

  fPublishedFileName    string            The file name when published.                                                                                             No

  fFolderType           string            The folder type. Can be `owner` for created folders or `soft` for shortcuts.                                              No

  fIsInTrash            number            When `1` the file is in the trash.                                                                                        No

  fRealItemGUID         string            The actual file GUID.                                                                                                     No

  fDocClass             string            The file class.                                                                                                           No

  fInhibitPropagation   number            A bitmap used to determine restrictions on files/folders from propagating values from its parent. The bitmap includes:\   No
                                          0 - inhibit propagation is set to none indicating values can be inherited from parent\                                    
                                          1 - inhibit propagation for metadata indicating metadata is not inherited from parent\                                    
                                          16 - inhibit propagation for folder security indicating folder security is not inherited from parent\                     
                                          17 - inhibit propagation for metadata and folder security                                                                 

  fApplication          string            The file application.                                                                                                     No

  fOwner                string            The owner of the file.                                                                                                    No

  fCreator              string            The creator of the file.                                                                                                  No

  fLastModifier         string            The last modifier of the file                                                                                             No

  fCreateDate           string            The date the file was created in ISO-8601 format.                                                                         No

  fLastModifiedDate     string            The date the file was modified in ISO-8601 format.                                                                        No

  fSecurityGroup        string            The file security group.                                                                                                  No

  fDocAccount           string            The file account.                                                                                                         No

  fClbraUserList        string            The file collaboration user list.                                                                                         No

  fClbraAliasList       string            The file collaboration alias list.                                                                                        No

  fClbraRoleList        string            The file collaboration role list.                                                                                         No

  fIsACLReadOnlyOnUI    number            When `1` the file should be displayed as read only.                                                                       No

  fDisplayName          string            The file display name.                                                                                                    No

  fDisplayDescription   string            The file display description.                                                                                             No

  fIsBrokenShortcut     number            When `1` the shortcut broken.                                                                                             No

  isLeaf                string            When `1` the folder is a leaf.                                                                                            No

  itemType              string            The folder type; the value `1` is a folder, `2` is a file, and `3` is a document.                                         No

  folderPermissions     string            The user permissions on the folder; can be none, or a combination of `R`ead, `W`rite, `D`elete or `A`ccess.               No
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### TaxonomyAssociateObject

The following fields associate a taxonomy to a security group.

::: table-responsive
  Name             Type     Description                                          Required
  ---------------- -------- ---------------------------------------------------- ----------
  dSecurityGroup   string   The security group to associate with the taxonomy.   Yes
:::

#### TaxonomyAssociateResponseObject

The following describes a list of associated security groups.

::: table-responsive
  Name            Type                                                              Description                                                                                                                                                                 Required
  --------------- ----------------------------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  offset          number                                                            The starting index from which the security groups are returned.                                                                                                             No
  limit           number                                                            The number of security groups returned.                                                                                                                                     No
  count           number                                                            The number of security groups returned in the current response.                                                                                                             No
  orderBy         string                                                            The sort field and order for the list.                                                                                                                                      No
  hasMore         boolean                                                           This is `true` if the request did not return all of the security groups. This occurs when limit is reached and there are additional groups that could have been returned.   No
  toltalResults   number                                                            The total number of security groups available.                                                                                                                              No
  items           An array of [TaxonomyAssociateObject](#taxonomyassociateobject)   List of security groups.                                                                                                                                                    No
:::

#### TaxonomyCreateObject

The following fields describe a taxonomy.

::: table-responsive
  Name                     Type     Description                                                                                                       Required
  ------------------------ -------- ----------------------------------------------------------------------------------------------------------------- ----------
  dTaxonomyName            string   The name of the taxonomy.                                                                                         Yes
  dDescription             string   The decription of the taxonomy.                                                                                   No
  dAbbreviation            string   A unique three letter abbreviation for the taxonomy. If not provided, the server will generate an abbreviation.   No
  dTaxonomySecurityGroup   string   The security group the taxonomy belongs to.                                                                       Yes
  dTaxonomyAccount         string   The optional account the taxonomy belongs to.                                                                     No
:::

#### TaxonomyListResponseObject

The followimng describes a list of taxonomies.

::: table-responsive
  Name            Type                                            Description                                                                                                                                                                Required
  --------------- ----------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  offset          number                                          The starting index from which the taxonomies are returned.                                                                                                                 No
  limit           number                                          The number of taxonomies returned.                                                                                                                                         No
  count           number                                          The number of taxonomies returned in the current response.                                                                                                                 No
  orderBy         string                                          The sort field and order for the list.                                                                                                                                     No
  hasMore         boolean                                         This is `true` if the request did not return all of the taxonomies. This occurs when limit is reached and there are additional taxonomies that could have been returned.   No
  toltalResults   number                                          The total number of taxonomies available.                                                                                                                                  No
  items           An array of [TaxonomyObject](#taxonomyobject)   List of taxonomy information for the current response.                                                                                                                     No
:::

#### TaxonomyUpdateObject

The following fields can update a taxonomy. If optional values are not supplied, the current values will be kept. The caller must have the admin role or RWDA permissions on all security groups and accounts (if enabled) referenced.

::: table-responsive
  Name                     Type     Description                                                                                                       Required
  ------------------------ -------- ----------------------------------------------------------------------------------------------------------------- ----------
  dTaxonomyName            string   The name of the taxonomy.                                                                                         No
  dDescription             string   The decription of the taxonomy.                                                                                   No
  dAbbreviation            string   A unique three letter abbreviation for the taxonomy. If not provided, the server will generate an abbreviation.   No
  dTaxonomySecurityGroup   string   The security group the taxonomy belongs to.                                                                       No
  dTaxonomyAccount         string   The optional account the taxonomy belongs to.                                                                     No
:::

#### TaxonomyObject

The following fields describe a taxonomy.

::: table-responsive
  Name                     Type     Description                                                                                                       Required
  ------------------------ -------- ----------------------------------------------------------------------------------------------------------------- ----------
  dTaxonomyName            string   The name of the taxonomy.                                                                                         No
  dDescription             string   The decription of the taxonomy.                                                                                   No
  dAbbreviation            string   A unique three letter abbreviation for the taxonomy. If not provided, the server will generate an abbreviation.   No
  dTaxonomySecurityGroup   string   The security group the taxonomy belongs to.                                                                       No
  dTaxonomyAccount         string   The optional account the taxonomy belongs to.                                                                     No
  dCreatedDate             string   The created date of the taxonomy.                                                                                 No
  dCreatedBy               string   The user that created the taxonomy.                                                                               No
  dLastModifiedDate        string   The last modified date of the taxonomy.                                                                           No
  dLastModifiedBy          string   The user that last modified the taxonomy.                                                                         No
:::

#### TaxonomyListOnDocumentObject

The response of taxonomies on a content item.

::: table-responsive
  Name            Type                                                  Description                                                                                                Required
  --------------- ----------------------------------------------------- ---------------------------------------------------------------------------------------------------------- ----------
  dTaxonomyGUID   string                                                The taxonomy GUID.                                                                                         No
  dTaxonomyName   string                                                The name of the taxonomy.                                                                                  No
  dAbbreviation   string                                                A three letter abbreviation for the taxonomy. If not provided, the server will generate an abbreviation.   No
  categories      An array of [CategoryObject](#categorysimpleobject)   List of categories information.                                                                            No
:::

#### WorkflowInfoResponse

The response for a content item in a workflow.

::: table-responsive
  Name                    Type                                                    Description                                                 Required
  ----------------------- ------------------------------------------------------- ----------------------------------------------------------- ----------
  dWfName                 string                                                  The name of the workflow.                                   No
  dWfStepID               number                                                  The unique id of a step within the workflow.                No
  remainingStepUsers      string                                                  A list of usernames that need to act on the workflow step   No
  dName                   string                                                  The author of the document in a workflow.                   No
  authorAddress           string                                                  The E-mail address of the author.                           No
  doc_info                [MetadataObjectResponse](#metadataobjectresponse)       The metadata of the document.                               No
  workflowInfo            [WorkflowObject](#workflowobject)                       The metadata of the workflow.                               No
  wf_doc_info             [WorkflowDocumentObject](#workflowdocumentobject)       Additional workflow metadata.                               No
  workflowStep            [WorkflowStepObject](#workflowstepobject)               The current workflow step.                                  No
  workflowSteps           An array of [WorkflowStepObject](#workflowstepobject)   An array of all workflow steps for this document.           No
  workflowStepEvents      [WorkflowStepEventObject](#workflowstepeventobject)     The workflow events                                         No
  workflowActionHistory   [WorkflowActionObject](#workflowactionobject)           The workflow action history.                                No
:::

#### WorkflowInformationResponse

The response for workflow information.

::: table-responsive
  Name                 Type                                                            Description                                         Required
  -------------------- --------------------------------------------------------------- --------------------------------------------------- ----------
  workflow             [WorkflowObject](#workflowobject)                               The metadata of the workflow.                       No
  workflowSteps        An array of [WorkflowStepObject](#workflowstepobject)           An array of all workflow steps for this workflow.   No
  workflowStepEvents   [WorkflowStepEventObject](#workflowstepeventobject)             The workflow events.                                No
  wfDocuments          An array of [WorkflowDocumentObject](#workflowdocumentobject)   The content items associated with this workflow.    No
:::

#### WorkflowInformationRevisionsResponse

The response for content item revisions that are in the workflow.

::: table-responsive
  Name             Type                                                                    Description                                                 Required
  ---------------- ----------------------------------------------------------------------- ----------------------------------------------------------- ----------
  wf_info          [WorkflowObject](#workflowobject)                                       The workflow metadata.                                      No
  workflowSteps    An array of [WorkflowStepObject](#workflowstepobject)                   An array of all workflow steps for this document.           No
  workflowStates   An array of [WorkflowStateObject](#workflowstateobject)                 An array of workflow states.                                No
  wfDocuments      An array of [WorkflowMetaDocumentObject](#workflowmetadocumentobject)   An array of content item metadata objects in the workflow   No
:::

#### WorkflowActiveResponse

The response for active workflows.

::: table-responsive
  Name    Type                                            Description                      Required
  ------- ----------------------------------------------- -------------------------------- ----------
  count   number                                          The number of active workflows   No
  items   An array of [WorkflowObject](#workflowobject)   An array of active workflows.    No
:::

#### WorkflowObject

The following fields describe a workflow.

::: table-responsive
  ----------------------------------------------------------------------------------------------------------------------------------------------------
  Name               Type              Description                                                                                   Required
  ------------------ ----------------- --------------------------------------------------------------------------------------------- -----------------
  dWfID              number            Unique counter id of the workflow.                                                            No

  dWfName            string            User assigned name of the workflow.                                                           No

  dWfDescription     string            Description of the Workflow.                                                                  No

  dCompletionDate    string            Date workflow last reached completion and all docs were released to web in ISO-8601 format.   No

  dSecurityGroup     string            Security Group the workflow belongs to (dSecurityGroup).                                      No

  dWfStatus          string            State of the workflow:\                                                                       No
                                       INIT = not started or inactive\                                                               
                                       INPROCESS = active                                                                            

  dWfType            string            Type of the workflow:\                                                                        No
                                       Basic\                                                                                        
                                       Criteria\                                                                                     
                                       Sub-workflow                                                                                  

  dProjectID         string            If set this workflow is a staging workflow.                                                   No

  dIsCollaboration   number            When `1` the workflow is related to Collaboration Project.                                    No
  ----------------------------------------------------------------------------------------------------------------------------------------------------
:::

#### WorkflowStepObject

The following fields describe steps in workflows.

::: table-responsive
  Name                 Type     Description                                                                                                                                                                          Required
  -------------------- -------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ----------
  dWfStepID            number   Incrementing counter used to uniquely identify step within the workflow.                                                                                                             No
  dWfStepName          string   The name of workflow step assigned by user.                                                                                                                                          No
  dWfID                number   Unique counter id of the workflow.                                                                                                                                                   No
  dWfStepDescription   string   Description of the Workflow step.                                                                                                                                                    No
  dWfStepType          string   Workflow Step type (AutoContribution, Contribution, Reviewer/Contribution, Reviewer).                                                                                                No
  dWfStepIsAll         number   If requires all users to approve (1 or 0).                                                                                                                                           No
  dWfStepHasWeight     number   Enables the limited reviewer option.                                                                                                                                                 No
  dWfStepWeight        number   The number of users to approve if dWfStepIsAll is false(0).                                                                                                                          No
  dWfStepIsSignature   number   Determines whether signature is required for the current workflow step. This is "1" if signature is required (which requires "ElectronicSignatures" to be enabled), "0" otherwise.   No
  dAliases             string   The list of alias users.                                                                                                                                                             No
:::

#### WorkflowStepEventObject

The following fields describe steps in workflows.

::: table-responsive
  Name             Type     Description                                                               Required
  ---------------- -------- ------------------------------------------------------------------------- ----------
  dWfStepName      string   Name of workflow step assigned by user.                                   No
  wfEntryScript    string   The step entry script.                                                    No
  wfExitScript     string   The step exit script. Must be placed within \<\$ and \$\> delimiters.     No
  wfUpdateScript   string   The step update script. Must be placed within \<\$ and \$\> delimiters.   No
:::

#### WorkflowDocumentObject

The following fields describe documents in a workflow.

::: table-responsive
  ------------------------------------------------------------------------------------------------------------
  Name               Type              Description                                           Required
  ------------------ ----------------- ----------------------------------------------------- -----------------
  dWfID              number            Unique counter id of the workflow.                    No

  dDocName           string            dDocName of the content item part of the workflow.    No

  dWfDocState        string            Current state of the workflow:\                       No
                                       INIT = not started or inactive\                       
                                       INPROCESS = active                                    

  dWfComputed        string            Derived workflow information.                         No

  dWfCurrentStepID   number            Unique identifier of step that the workflow is in.    No

  dWfDirectory       string            Location of the workflow document's companion file.   No

  dClbraName         string            Collaboration Project Name.                           No
  ------------------------------------------------------------------------------------------------------------
:::

#### WorkflowMetaDocumentObject

The following fields describe documents in a workflow.

::: table-responsive
  Name                   Type     Description                                                                                    Required
  ---------------------- -------- ---------------------------------------------------------------------------------------------- ----------
  dWfID                  number   Unique counter id of the workflow.                                                             No
  dDocName               string   dDocName of the content item part of the workflow.                                             No
  dWfDocState            string   Current workflow state of document (INIT = workflow not active, INPROCESS = workflow active)   No
  dWfComputed            string   Derived workflow information.                                                                  No
  dWfCurrentStepID       number   Unique identifier of step that the workflow is in.                                             No
  dWfDirectory           string   Location of the workflow document's companion file.                                            No
  dClbraName             string   Collaboration Project Name.                                                                    No
  dDocType               string   The type of document                                                                           No
  dDocTitle              string   The title of the document                                                                      No
  dRevLabel              string   The revision of the document                                                                   No
  dSecurityGroup         string   The security group of the document                                                             No
  dDocName               string   The id of the document                                                                         No
  dDocAuthor             string   The author of the document                                                                     No
  dStatus                string   This field is a summary field of computations from other status fields.                        No
  dOriginalName          string   The document file name                                                                         No
  dFormat                string   The document mime type                                                                         No
  dFileSize              number   The document filesize (in bytes)                                                               No
  dDocCreatedDate        string   The date and time the document was first uploaded into the server in ISO-8601 format           No
  dDocCreator            string   The user who originally created the document                                                   No
  dDocLastModifiedDate   string   The date and time any revision of the document was last modified in ISO-8601 format            No
  dDocLastModifier       string   The last user to modify the document                                                           No
:::

#### WorkflowStateObject

The following fields describe workflow states of document.

::: table-responsive
  Name         Type     Description                                                                                  Required
  ------------ -------- -------------------------------------------------------------------------------------------- ----------
  dID          number   Revision Identifier of the document.                                                         No
  dDocName     string   dDocName of the document part of the workflow.                                               No
  dWfID        number   Unique counter id of the workflow.                                                           No
  dUserName    string   User that has approved document.                                                             No
  dWfEntryTs   number   The entry timestamp(in ISO-8601 format) for a user into a particular step in the workflow.   No
:::

#### WorkflowActionObject

The following fields describe workflow action on content from corresponding HDA file in `data\workflow\states`{.language-plaintext .highlighter-rouge} directory.

::: table-responsive
  Name          Type     Description                       Required
  ------------- -------- --------------------------------- ----------
  dWfName       string   Workflow name.                    No
  dWfStepName   string   Workflow Step Name.               No
  wfAction      string   Action performed at the Step.     No
  wfActionTs    string   Action Time in ISO-8601 format.   No
  wfUsers       string   Workflow Step Users.              No
  wfMessage     string   Workflow Message at the step.     No
:::

#### WorkflowInQueueResponse

The response for the workflow in queue.

::: table-responsive
  Name         Type      Description                                                                                                                                                                                  Required
  ------------ --------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  count        number    The number of items returned.                                                                                                                                                                No
  hasMore      boolean   If `false` all items have been returned.                                                                                                                                                     No
  numPages     number    The number of pages the queue items can be returned.                                                                                                                                         No
  pageNumber   number    The page number of the search results returned.                                                                                                                                              No
  totalRows    number    The total number of workflow items in queue for the user.                                                                                                                                    No
  startRow     number    The start row in the workflow queue.                                                                                                                                                         No
  endRow       number    The last row in the workflow queue."                                                                                                                                                         No
  items        array     Each entry is a composite of fields from [WorkflowInQueueObject](#workflowinqueueobject), [WorkflowStepObject](#workflowstepobject), and [MetadataObjectResponse](#metadataobjectresponse)   No
:::

#### WorkflowInQueueObject

The following fields describe a workflow queue.

::: table-responsive
  Name                   Type                                  Description                                                                                 Required
  ---------------------- ------------------------------------- ------------------------------------------------------------------------------------------- ----------
  dUser                  string                                The user to which the document assigned in workflow.                                        No
  dDocName               string                                The dDocName of the document in workflow.                                                   No
  dID                    number                                The revision identifier of the document in workflow.                                        No
  dWfID                  number                                Unique counter id of the workflow.                                                          No
  dWfName                string                                User assigned name of the workflow.                                                         No
  dWfStepName            string                                The name of workflow step assigned by user.                                                 No
  dwfQueueActionState    string } The workflow action state.   No                                                                                           
  dwfQueueEnterTs        string                                The workflow entry time in ISO-8601 format.                                                 No
  dwfQueueLastActionTs   string                                The timestamp when the last workflow action performed on the document in ISO-8601 format.   No
  wfMessage              string                                The workflow message for the step.                                                          No
  fIsFavorite            number                                Indicates if the item is a favorite item. Applicable only when Frameworkfolders enabled.    No
:::

#### DocProfilesResponse

The response for DocProfiles information.

::: table-responsive
  Name                          Type                                                Description                                                    Required
  ----------------------------- --------------------------------------------------- -------------------------------------------------------------- ----------
  defFileExists                 boolean                                             True if the definition file exists.                            No
  dpDidSetDefaultTriggerField   number                                              When `1` the default trigger field is set or `0` if not set.   No
  dpTriggerField                string                                              The trigger field for the profile.                             No
  count                         number                                              The number of docProfiles returned.                            No
  items                         An array of [DocProfileObject](#docprofileobject)   An array DocProfile objects.                                   No
:::

#### DocProfileObject

The following fields describe a DocProfile

::: table-responsive
  Name             Type      Description                                                                     Required
  ---------------- --------- ------------------------------------------------------------------------------- ----------
  dpName           string    The name of the profile.                                                        No
  dpDescription    string    The description for the profile.                                                No
  dpTriggerValue   string    The trigger associated with the profile.                                        No
  dpDisplayLabel   string    The display label of the profile.                                               No
  dDocClass        string    The class the profile is assigned.                                              No
  isValid          boolean   True when dpTriggerValue is in optionlist of dpTriggerField; otherwise false.   No
:::

#### UserPermissionResponse

The response is used for user permissions information.

::: table-responsive
  Name                Type                                                            Description                                   Required
  ------------------- --------------------------------------------------------------- --------------------------------------------- ----------
  dName               string                                                          The name of user.                             No
  documentAccounts    An array of [UserAccountObject](#useraccountobject)             An array of accounts and privileges.          No
  securityGroups      An array of [UserGroupObject](#usergroupobject)                 An array of security groups and privileges.   No
  userSecurityFlags   An array of [UserSecurityFlagObject](#usersecurityflagobject)   An array of security flags.                   No
:::

#### UserAccountObject

The following fields describe an account and the privilege for the account.

::: table-responsive
  Name          Type     Description                                          Required
  ------------- -------- ---------------------------------------------------- ----------
  dDocAccount   string   The account the user has access to.                  No
  privilege     string   The privilege level corresponding for the account.   No
:::

#### UserGroupObject

The following fields describe a group and the privilege for the security group.

::: table-responsive
  Name         Type     Description                                                 Required
  ------------ -------- ----------------------------------------------------------- ----------
  dGroupName   string   The security group the user has access to.                  No
  privilege    string   The privilege level corresponding for the security group.   No
:::

#### UserSecurityFlagObject

The following fields describe security group and values.

::: table-responsive
  Name    Type     Description                                Required
  ------- -------- ------------------------------------------ ----------
  flag    string   The flags related to the security group.   No
  value   string   The value set for of this flag.            No
:::

#### CustomFieldsResponse

The response is used for display fields information.

::: table-responsive
  Name                 Type                                                                    Description                                                                                                                                                                                        Required
  -------------------- ----------------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  displayFieldInfo     An array of [DisplayFieldInfoObject](#displayfieldinfoobject)           An array of custom metadata fields information.                                                                                                                                                    No
  displayGroupInfo     An array of [DisplayFieldGroupObject](displayfieldgroupobject)          An array showing how metadata fields can be grouped together.                                                                                                                                      No
  xFieldName.options   An array of [DisplayFieldDropDownObject](#displayfielddropdownobject)   An array showing how options can be used to create drop-down lists for specific fields. Note that `xFieldName` is the name custom metadata field, and the responses may include multiple arrays.   No
:::

#### DisplayFieldInfoObject

The following fields describe a custom metadata field.

::: table-responsive
  Name                          Type                                                   Description                                                                                                                   Required
  ----------------------------- ------------------------------------------------------ ----------------------------------------------------------------------------------------------------------------------------- ----------
  fieldName                     string                                                 The name of the field.                                                                                                        No
  fieldType                     string                                                 The type of field.                                                                                                            No
  fieldLabel                    string                                                 The display label for the field.                                                                                              No
  fieldLength                   string                                                 The length of the field allowed.                                                                                              No
  isHidden                      string                                                 When `1` the field should be a hidden form field.                                                                             No
  isReadOnly                    string                                                 When `1` the field should be a non-input form field.                                                                          No
  isRequired                    string                                                 When `1` the field should be a required form field.                                                                           No
  requiredMsg                   string                                                 The message displayed if required field is not populated.                                                                     No
  defaultValue                  string                                                 The default value for the field.                                                                                              No
  displayValue                  string                                                 The display value to be used for the `defaultValue`.                                                                          No
  isOptionList                  string                                                 When `1` the field is a drop-down option list.                                                                                No
  optionList                    string                                                 If `isOptionList` is set to `1`; the name of the additional table containing the options used to create the drop-down list.   No
  isTreeOptionList              string                                                 When `1` the option list is using a tree.                                                                                     No
  optionListType                string                                                 The type of option list.                                                                                                      No
  isDependent                   string                                                 When `1` the field is part of DCL. The value of this field depends on another field.                                          No
  dependentOnField              string                                                 When `1` the name of the field that will derive the value for this field.                                                     No
  isPadMultiselectStorage       string                                                 Used by option list of type multi\*.                                                                                          No
  multiselectDisplaySeparator   string                                                 Used by option list of type multi\*.                                                                                          No
  multiselectStorageSeparator   string                                                 Used by option list of type multi\*.                                                                                          No
  isShowSelectionPath           string                                                 When `1` show the full path. The full path appears on the Info Page.                                                          No
  isStoreSelectionPath          string When `1` store the full path in the database.   No                                                                                                                             
  treeNodeDisplaySeparator      string                                                 The character that must be used to display the separators.                                                                    No
  treeNodeStorageSeparator      string                                                 The character that must be used as a separator when the path is saved in the database.                                        No
  order                         string                                                 The order of the field.                                                                                                       No
  decimalScale                  string                                                 The decimalScale if the field allows decimal values.                                                                          No
  isError                       string                                                 When `1` there was an error when retrieving information about a field.                                                        No
  errorMsg                      string                                                 The field's error message.                                                                                                    No
:::

#### DisplayFieldGroupObject

The following fields describe a custom metadata field grouping.

::: table-responsive
  Name             Type     Description                                                             Required
  ---------------- -------- ----------------------------------------------------------------------- ----------
  parentField      string   The name of the field that should appear first in the group.            No
  groupFieldList   string   The list of fields that should appear together with the parent field.   No
  groupHeader      string   The name of the group.                                                  No
  defaultHide      string   When `1` the group should be collapsed by default.                      No
:::

#### DisplayFieldDropDownObject

The following fields describe the options that a custom metadata uses to create a drop-down list.

::: table-responsive
  Name           Type     Description                         Required
  -------------- -------- ----------------------------------- ----------
  dOption        string   The internal value of the option.   No
  dDescription   string   The display value of the option.    No
:::

#### PublicLinkObject

The following fields describe the public link options.

::: table-responsive
  Name              Type     Description                                           Required
  ----------------- -------- ----------------------------------------------------- ----------
  dLinkName         string   The name of a public link.                            Yes
  dLinkDesc         string   The description of a public link.                     No
  dLinkPrivilege    string   The privilege of a public link. Default value: "R".   No
  dAccessCode       string   The access code to use the public link.               No
  dExpirationDate   string   The expiration date of a public link.                 No
:::

#### PublicLinkResponse

The following fields describe a public link for a folder or a file. Guest users could access a folder or a file via its public links without authentication.

::: table-responsive
  Name                Type     Description                                                                           Required
  ------------------- -------- ------------------------------------------------------------------------------------- ----------
  dLinkID             string   The ID of a public link.                                                              No
  dLinkType           string   The type of a public link. The possible values are 'fFolderGUID' and 'fFileGUID'.     No
  dLinkName           string   The name of a public link.                                                            No
  dLinkDesc           string   The description of a public link.                                                     No
  fItemGUID           string   The GUID of the linked folder or file.                                                No
  fOwner              string   The owner user of the linked folder or file.                                          No
  dLinkPrivilege      string   The privilege of a public link.                                                       No
  dAssignedUsers      string   The assigned users of a public link. A public link is always assigned to all users.   No
  dAccessCode         string   The access code of a public link.                                                     No
  dCreatedDate        string   The created date of a public link.                                                    No
  dLastModifiedDate   string   The last modified date of a public link.                                              No
  dExpirationDate     string   The expiration date of a public link.                                                 No
:::

#### PublicLinkListResponse

The response when listing public links for a folder or a file.

::: table-responsive
  Name            Type                                                    Description                                                                                                                                                                    Required
  --------------- ------------------------------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ----------
  offset          number                                                  The starting index from which the public links are returned. Defaults to 0 if not specified in the request.                                                                    No
  limit           number                                                  The number of public links up to which a request can return. Defaults to 20 if not specified in the request. Maximum allowed value is 100.                                     No
  count           number                                                  The number of public links returned in the current response.                                                                                                                   No
  orderBy         string                                                  The sort field and order for the public links.                                                                                                                                 No
  hasMore         boolean                                                 This is `true` if the request did not return all of the public links. This occurs when limit is reached and there are additional public links that could have been returned.   No
  toltalResults   number                                                  The total number of public links available on the folder or file.                                                                                                              No
  items           An array of [PublicLinkResponse](#publiclinkresponse)   List of public link information for all the public links returned in the current response.                                                                                     No
:::

#### DataSourceResponse

The following fields describe the query of a data source.

::: table-responsive
  Name          Type     Description                                                              Required
  ------------- -------- ------------------------------------------------------------------------ ----------
  copyAborted   number   When true, the request does not return all rows from the query.          No
  nextRow       number   The number of the next row in the query.                                 No
  count         number   The number of rows returned.                                             No
  items         array    The query results. The fields will vary based on the data source used.   No
:::

#### FoldersSearchResultsResponse

The response returning search results for items in folders.

::: table-responsive
  Name               Type                                                                                         Description                                                                       Required
  ------------------ -------------------------------------------------------------------------------------------- --------------------------------------------------------------------------------- ----------
  count              number                                                                                       The number of documents returned.                                                 No
  hasMore            boolean                                                                                      When false, all the results for the search were returned.                         No
  offset             number                                                                                       The offset when used to return a limited number of search results.                No
  totalResults       number                                                                                       The total number of documents that satisfy the search.                            No
  limit              number                                                                                       The maximum number of items listed.                                               No
  q                  string                                                                                       The search query used to generate this response.                                  No
  orderBy            string                                                                                       The sort field and sort order used to generate this response.                     No
  startRow           number                                                                                       The start row number of `items` returned.                                         No
  endRow             number                                                                                       The end row number of `items` returned.                                           No
  nextRow            number                                                                                       The number of the next row in the query.                                          No
  itemType           string                                                                                       The item type; the value `1` is a folder, `2` is a file, and `3` is a document.   No
  dataSource         string                                                                                       The dataSource used for generating search results.                                No
  searchEngineName   string                                                                                       The name of search engine used.                                                   No
  items              array of either [FolderInfoObject](#folderinfoobject) or [FileInfoObject](#fileinfoobject)   An array of metadata objects for documents that meet the search query.            No
:::

#### DocTypeResponse

The response for returning all document type information.

::: table-responsive
  Name    Type                                       Description                              Required
  ------- ------------------------------------------ ---------------------------------------- ----------
  count   number                                     The number of document types returned.   No
  items   array of [DocTypeObject](#doctypeobject)   An array of DocType objects.             No
:::

#### DocTypeObject

The following fields describe a document type.

::: table-responsive
  Name           Type     Description                                                                                                                              Required
  -------------- -------- ---------------------------------------------------------------------------------------------------------------------------------------- ----------
  dDocType       string   The unique identifier of document type.                                                                                                  No
  dDescription   string   The description of the document type.                                                                                                    No
  dGif           string   The file name of image for the document type. Note that the images are on the server in the `IntradocWebDir`/images/docgifs directory.   No
:::

#### DocConfigInfoResponse

The response for returning configuration information. Note that servers may return different fields depending on the server configuration.

::: table-responsive
  Name                                   Type                                                                              Description                                                                                                Required
  -------------------------------------- --------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------------- ----------
  accessListPrivilegesGrantedWhenEmpty   number                                                                            When `1`, the server allows access to everyone when the access list is empty.                              No
  httpAbsoluteCgiUrl                     string                                                                            The absolute HTTP CGI URL.                                                                                 No
  httpAbsoluteCgiUrlWebdav               string                                                                            The absolute HTTP CGI URL for WebDAV                                                                       No
  httpAbsoluteWebdavPath                 string                                                                            The absolute WebDAV URL.                                                                                   No
  httpBaseAbsoluteRoot                   string                                                                            The absolute root URL.                                                                                     No
  httpCgiPath                            string                                                                            The relative CGI URL.                                                                                      No
  httpRelativeWebRoot                    string                                                                            The relative webroot.                                                                                      No
  idc_name                               string                                                                            The server name.                                                                                           No
  illegalUrlSegmentCharacters            string                                                                            The illegal URL segment characters.                                                                        No
  instanceDescription                    string                                                                            The description of the instance.                                                                           No
  instanceMenuLabel                      string                                                                            The menu label of the instance.                                                                            No
  isAutoNumber                           number                                                                            When `1`, the server automatically assigns content ID (dDocName) on checkin.                               No
  isQueryObjectPersistent                number                                                                            When `1`, queried resultsets are persisted.                                                                No
  localeDateFormatPattern                string                                                                            The locale date format.                                                                                    No
  localeTimeZoneID                       string                                                                            The ID of the locale time zone.                                                                            No
  localeTimeZoneOffsetMillis             number                                                                            The locale time zone offset in milliseconds.                                                               No
  maxResults                             number                                                                            The maximum search results returned.                                                                       No
  memoFieldSize                          number                                                                            The maximum allowed size of a memo field.                                                                  No
  productVersion                         string                                                                            The server product version.                                                                                No
  productVersionInfo                     string                                                                            The server product version information.                                                                    No
  searchIndexerEngineName                string                                                                            The search engine name used for indexing.                                                                  No
  specialAuthGroups                      string                                                                            The list of special Auth Groups (ACL-related).                                                             No
  systemTimeZone                         string                                                                            The server time zone.                                                                                      No
  useAccounts                            number                                                                            When `1`, accounts are enabled.                                                                            No
  useCollaboration                       number                                                                            When `1`, collaboration is enabled.                                                                        No
  useDatabaseWfInQueue                   number                                                                            When `1`, server workflow in-queues are fetched using database query.                                      No
  useEntitySecurity                      number                                                                            When `1`, entity security is enabled.                                                                      No
  useForwardOnlyCursor                   number                                                                            When `1`, forward only cursor is used if the parameters object is reused for subsequent calls of search.   No
  zonedSecurityFields                    string                                                                            A comma separated list of zoned security fields.                                                           No
  acceptsResponseDateFormat              number                                                                            When `1`, the response date format is accepted.                                                            No
  dUser                                  string                                                                            The caller's user name.                                                                                    No
  defaultAccount                         string                                                                            The default account if accounts are enabled.                                                               No
  webdavbaseurl                          string                                                                            The absolute WebDAV base URL.                                                                              No
  doc_default_infoCount                  number                                                                            The number of items in the `doc_default_info` array.                                                       No
  docFormatsCount                        number                                                                            The number of items in the `docFormats` array.                                                             No
  docMetaDefinitionCount                 number                                                                            The number of items in the `docMetaDefinitions` array.                                                     No
  docTypesCount                          number                                                                            The number of items in the `docTypes` array.                                                               No
  featuresCount                          number                                                                            The number of items in the `features` array.                                                               No
  doc_default_info                       an array of fields similar to [MetadataObjectResponse](#metadataobjectresponse)   An array of default metadata values.                                                                       No
  docFormats                             an array of [DocFormatObject](#docformatobject)                                   An array of document formats                                                                               No
  docMetaDefinitions                     an array of [DocMetaDefinitionObject](#docmetadefinitionobject)                   An array of metadata fields.                                                                               No
  docTypes                               array of [DocTypeObject](#doctypeobject)                                          An array of DocType objects.                                                                               No
  features                               array of [FeaturesObject](#featuresobject)                                        An array of features the server supports.                                                                  No
:::

#### DocMetaInfoResponse

The response for returning configuration information. Note that servers may return different fields depending on the server configuration.

::: table-responsive
  Name                     Type                                                              Description                                                                    Required
  ------------------------ ----------------------------------------------------------------- ------------------------------------------------------------------------------ ----------
  isAutoNumber             number                                                            When `1`, the server automatically assigns content ID (dDocName) on checkin.   No
  useAccounts              number                                                            When `1`, accounts are enabled.                                                No
  docMetaDefinitionCount   number                                                            The number of items in the `docMetaDefinitions` array.                         No
  docTypesCount            number                                                            The number of items in the `docTypes` array.                                   No
  docMetaDefinitions       an array of [DocMetaDefinitionObject](#docmetadefinitionobject)   An array of metadata fields.                                                   No
  docTypes                 array of [DocTypeObject](#doctypeobject)                          An array of DocType objects.                                                   No
:::

#### DocFormatObject

The following fields describe a document format.

::: table-responsive
  Name             Type     Description                                                       Required
  ---------------- -------- ----------------------------------------------------------------- ----------
  dFormat          string   The server MIME type of the file.                                 No
  dConversion      string   The name of the conversion mapped to this format or `PassThru`.   No
  dDescription     string   The dDescription of the format.                                   No
  dIsEnabled       number   When `1`, the format is enabled.                                  No
  overrideStatus   string   When `full`, the format is already present in database.           No
  isSystem         number   When `1`, the format is a system format.                          No
:::

#### DocMetaDefinitionObject

The following fields describe a document metadata field.

::: table-responsive
  Name                  Type      Description                                            Required
  --------------------- --------- ------------------------------------------------------ ----------
  dName                 string    The field name.                                        No
  dType                 string    The field type.                                        No
  dIsRequired           number    When `1`, the field is required.                       No
  dIsEnabled            number    When `1`, the field is enabled.                        No
  dIsSearchable         number    When `1`, the field is searchable.                     No
  dCaption              string    The caption when the field is displayed.               No
  dIsOptionList         number    When `1`, the option list is available.                No
  dOptionListKey        string    The option list key.                                   No
  dDefaultValue         string    The field default value.                               No
  dOrder                number    The load order of the field.                           No
  dOptionList           string    The option list type.                                  No
  dIsPlaceholderField   number    When `1`, the field is a placeholder.                  No
  dCategory             string    The category information of the field.                 No
  dExtraDefinition      string    Additional information for the field.                  No
  dDecimalScale         integer   The supported decimal values of the field.             No
  dDocMetaSet           string    When `DocMeta`the field is defined in DocMeta table.   No
:::

#### FeaturesObject

The following fields describe a server feature.

::: table-responsive
  Name                  Type     Description                            Required
  --------------------- -------- -------------------------------------- ----------
  idcFeatureName        string   The feature name.                      No
  idcFeatureVersion     string   The feature version.                   No
  idcFeatureLevel       string   The feature level or version.          No
  idcFeatureComponent   string   The component providing the feature.   No
:::

#### CapabilitiesResponse

The following fields describe a capabilities response.

::: table-responsive
  Name    Type                                                 Description                            Required
  ------- ---------------------------------------------------- -------------------------------------- ----------
  count   number                                               The number of capabilities returned.   No
  items   array of [CapabilitiesObject](#capabilitiesobject)   An array of capabilities objects.      No
:::

#### CapabilitiesObject

The following fields describe a capability.

::: table-responsive
  Name              Type     Description                                                                                                                                                                                 Required
  ----------------- -------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------
  capabilityName    string   The name of the capability tested.                                                                                                                                                          No
  capabilityValus   number   The number indicating the capability: `-1` means the capability does not apply to the object; `0` means the object does not have the capability; `1` means the object has the capability.   No
:::

#### OAuthResponse

The following fields describe an OAuth token response.

::: table-responsive
  Name           Type     Description                                              Required
  -------------- -------- -------------------------------------------------------- ----------
  accessToken    string   The access token used to access the scopes.              No
  refreshToken   string   The refresh token used to regenerate the access token.   No
  tokenType      string   The access token type, for example: `Bearer`.            No
  expiresIn      number   The expiry time of the access token in seconds.          No
:::

#### DocProfileResponse

The following fields describe a document profile response.

::: table-responsive
  Name                          Type      Description                                                                                                           Required
  ----------------------------- --------- --------------------------------------------------------------------------------------------------------------------- ----------
  dpName                        string    The name of the document profile.                                                                                     No
  dpDescription                 string    The description of the document profile.                                                                              No
  dpDisplayLabel                string    The label which is used on the UI for the document profile.                                                           No
  dpTriggerField                string    The profile trigger field.                                                                                            No
  dpTriggerValue                string    The trigger value for the document profile.                                                                           No
  dDocClass                     string    The document class.                                                                                                   No
  isValid                       boolean   True if the trigger value is valid.                                                                                   No
  defFileExists                 boolean   True if profile file exists,                                                                                          No
  dpExludeNonRuleFields         number    When `1`, indicates the non-rule field is to be excluded. *Non-rule fields* are fields that are not part of a rule.   No
  dpHasLinkScripts              number    When `1`, indicates the profile has personalization links.                                                            No
  dpHasCheckinLinkScript        number    When `1`, indicates the profile has script for the check-in link.                                                     No
  dpCheckinLinkScriptIsCustom   number    When `1`, indicates the script for the check-in link is a custom script.                                              No
  dpCheckinLinkCustomScript     string    The custom script for the check-in link.                                                                              No
  dpCheckinLinkScriptSummary    string    The script summary for the check-in link.                                                                             No
  dpHasSearchLinkScript         number    When `1`, indicates the profile has script for the search link.                                                       No
  dpSearchLinkScriptIsCustom    number    When `1`, indicates the script for the search link is a custom script.                                                No
  dpSearchLinkCustomScript      string    The custom script for the search link.                                                                                No
  dpSearchLinkScriptSummary     string    The script summary for the search link                                                                                No
:::

</div>
