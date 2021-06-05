# Cloudnado S3 Client Application Programming Interface

This document describes the application programming interface (API) used by the Cloudnado platform. The platform attempts to emulate Amazon's AWS S3 as much as possible, but only a small subset of S3's functionality is supported.

Table of Contents
=================

  * [Glossary](#glossary)
  * [Quick-Start](#quick-start)
  * [Authentication](#authentication)
  * [Reference](#reference)
     * [CreateAccessKey](#createaccesskey)
     * [CreateBucket](#createbucket)
     * [DeleteBucket](#deletebucket)
     * [DeleteObject](#deleteobject)
     * [GetBucketAcl](#getbucketacl)
     * [GetBucketLocation](#getbucketlocation)
     * [GetBucketPolicy](#getbucketpolicy)
     * [GetObject](#getobject)
     * [GetObjectAcl](#getobjectacl)
     * [HeadBucket](#headbucket)
     * [HeadObject](#headobject)
     * [ListBuckets](#listbuckets)
     * [ListObjects](#listobjects)
     * [ListObjectsV2](#listobjectsv2)
     * [PutBucketAcl](#putbucketacl)
     * [PutBucketPolicy](#putbucketpolicy)
     * [PutObject](#putobject)
     * [PutObjectAcl](#putobjectacl)

<!--- Created with https://github.com/ekalinin/github-markdown-toc -->

## Glossary

**bucket**: A top-level directory.

**object key**: The object's path, excluding the bucket.

**API key id**: A 24-character string uniquely identifying an account or sub-account. Also called a _token_ or _access key id_.

## Quick-Start

**Upload an object**

To upload an object, use a PUT request while appending your API token to the end of the URL. The PUT request should look like this:

    PUT /sample-bucket/other/lorem-ipsum.txt?token=<api_token> HTTP/1.1
    Host: s3.cloudnado.com
    Content-Type: text/plain
    Content-Length: 446

    Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
    
For example, linux users can use the curl command like this:

    curl -T files/books/dracula.txt 'https://s3.cloudnado.com/sample-bucket/dracula.txt?token=<api_token>'

As long as the API token has write permission for sample-bucket, the upload will be successful.

**Download an object**

To download a object, use a GET request. If the object is set to private, you must append your API token at the end of the URL. The GET request should look like this:

    GET /sample-bucket/other/lorem-ipsum.txt?token=<api_token> HTTP/1.1
    Host: s3.cloudnado.com

For example, linux users can use the curl command like this:

    curl https://s3.cloudnado.com/sample-bucket/dracula.txt?token=<api_token>

As long as the API token has read permission for the object, the download should be successful.

## Authentication

To authenticate requests, you must use Cloudnado API keys, which come in two types: an access key (which has a public "access key ID" component and a private "secret access key" component), and an API token (which is a private secret, and functions as both an "access key ID" and a "secret access key").

Cloudnado is S3-compatible, and therefore follows S3-style authentication. As a result, to authenticate using access keys, add an HTTP authentication header with the following format:

    Authorization: AWS4-HMAC-SHA256 Credential=<access_key_id>/<YYYYMMDD>/us-west-1/s3/aws4_request, SignedHeaders=host;range;x-amz-date, Signature=<signature>
 
See Amazon's [AWS S3 documentation](https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html) for full details.
 
To authenticate using API tokens, add an HTTP authentication header with the following format instead:

    Authorization: AWS4-HMAC-SHA256 Credential=<api_token>/<YYYYMMDD>/us-west-1/s3/aws4_request, SignedHeaders=host;range;x-amz-date, Signature=

Note that using API tokens is less secure than access keys, but provides an easier way to implement authentication as the client does not need to compute the signature. API tokens are designed for rapid development and testing, and is not meant for production use.

In addition to S3-style authentication, Cloudnado also accepts authentication by appending an API token to the URL query "token". For example,

    GET https://s3.cloudnado.com/sample-bucket/sample-object-key.txt?token=<api_token>
    
Appending ?token= to the URL to authenticate works for all endpoints. Note that in this method, only API tokens can be used, and access keys will fail to authenticate.

**This URL query method of authentication is insecure and potentially dangerous** because the URL contains a secret (the API token), yet URLs are treated as public information in most applications. So the API token may be unintentionally leaked or logged somewhere. However, due to convenience, this method is enabled for faster development and rapid debugging.

## Reference

Requests and responses may be formatted for readability. Clients should not depend on the formatting shown in this document.

The actions listed below are the only AWS operations currently available. Not all options described in AWS's documentation are supported.

### CreateAccessKey

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateAccessKey.html_

### CreateBucket

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateBucket.html_

### DeleteBucket

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteBucket.html_

### DeleteObject

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteObject.html_

### GetBucketAcl

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetBucketAcl.html_

### GetBucketLocation

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetBucketLocation.html_

### GetBucketPolicy

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetBucketPolicy.html_

### GetObject

#### Request Syntax

```http
GET /<bucket>/<key> HTTP/1.1

```

#### Response Syntax

```http
HTTP/2.0 200 OK
Content-Length: <body length>
Content-Type: <media type>

<body>
```

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html_

### HeadBucket

### HeadObject

### ListBuckets

#### Request Syntax

```http
GET / HTTP/1.1
Authorization: <auth string>
```

#### Response Syntax

```http
HTTP/1.1 200 OK
Content-Type: text/xml
Content-Length: <body length>
```
```xml
<ListBucketsOutput>
  <Buckets>
    <Bucket>
      <CreationDate>YYYY-MM-DDTHH:MM:SS-HH:MM</CreationDate>
      <Name>bucket_name</Name>
    </Bucket>
  </Buckets>
  <Owner>
    <DisplayName>owner_name</DisplayName>
    <ID>acct_id</ID>
  </Owner>
</ListBucketsOutput>
```

_Reference: (https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListBuckets.html_

### ListObjects

This API call as been deprecated by AWS. Use ListObjectsV2 instead.

### ListObjectsV2

```http
GET /<bucket>?list-type=2&prefix=<string>&delimiter=<string>&encoding-type=url HTTP/1.1
Authorization: <auth string>
```

#### Response Syntax

```http
HTTP/1.1 200 OK
Content-Type: text/xml
Content-Length: <body length>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ListObjectsV2Output>
   <IsTruncated>boolean</IsTruncated>
   <Contents>
      <ETag>string</ETag>
      <Key>string</Key>
      <LastModified>timestamp</LastModified>
      <Owner>
         <DisplayName>string</DisplayName>
         <ID>string</ID>
      </Owner>
      <Size>integer</Size>
      <StorageClass>string</StorageClass>
   </Contents>
   ...
   <Name>string</Name>
   <Prefix>string</Prefix>
   <Delimiter>string</Delimiter>
   <MaxKeys>integer</MaxKeys>
   <CommonPrefixes>
      <Prefix>string</Prefix>
   </CommonPrefixes>
   ...
   <EncodingType>string</EncodingType>
   <KeyCount>integer</KeyCount>
</ListObjectsV2Output>

```

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html_

### PutBucketAcl

### PutBucketPolicy

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketPolicy.html_

### PutObject

#### Request Syntax

```http
PUT /<bucket>/<key> HTTP/1.1
Content-Length: <body length>
Content-Type: <media type>
Authorization: <auth string>

<body>
```

#### Response Syntax

```http
HTTP/1.1 200 OK
```

_Reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html_

### PutObjectAcl
_Reference: lhttps://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObjectAcl.html_
