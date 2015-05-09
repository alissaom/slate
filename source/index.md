---
title: Clarifai Developer Documentation

language_tabs:
  - shell
  - python
  - nodejs

toc_footers:
  - <a href='http://developer.clarifai.com/accounts/signup/'>Get a Developer Account</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors
  - vision

search: true
---

# Introduction

The Clarifai API is an image and video recognition service.

The best way to understand the power of our API is to see it in action with our
[demo](http://clarifai.com) where you can
upload your own photos and video to see what's in them.


# Getting Started

First [create a developer account](http://developer.clarifai.com/accounts/signup/).

Once you're signed in, create an
[Application](https://developer.clarifai.com/docs/applications). The application keys
(Client ID and Client Secret) created during this step will later by copied into your client code.

Time to start coding! The easiest way to get started is by using the clients available on the
[Clarifai Github account](https://github.com/Clarifai). These clients wrap the basic REST API,
and also provide utilities such as image resizing. Currently available for Python and node.js,
other languages will be added soon.

After setting up the client, check out our quick
[Tutorial](https://developer.clarifai.com/docs/tutorial).

# API Clients

While the API can be accessed directly through the RESTful HTTP endpoints, several
language-specific clients are available for easier integration.  These include:

* [Python](https://github.com/Clarifai/Clarifai_py) (also our reference client)
* [Java](https://github.com/Clarifai/clarifai-api-java)
* [Nodejs](https://github.com/Clarifai/Clarifai_nodejs)
* [Swift](https://github.com/yazoglr/Claire) (Draft in development)

Client-specific documentation is available in each repo, but the
overall concepts regarding accessing the API and creating applications remains the same as if you
were using REST endpoints.  Most clients take care of authentication, refreshing access tokens, and
support image validation and resizing locally.


# Applications

The first step to using our API to is create an application.
Each application has its own client credentials for [authentication](#authentication).

You can create as many applications as you like.  All applications under a single account share
the account's usage quota.


## Creating Applications

In order to create applications you must first register for your Clarifai
account. Once registered, login to your account. From the Account menu, you can now access the
Application menu which shows a list of the current applications in your account.

Simply click "Create a new Application" to do so and provide a name for the application. That's it!
You're done creating an application.

## Using Applications

Now that you have created an application, click on the application name to view
more details about the application. On the application details page you'll find your Client Id and
Client Secret which is needed to gain access to Clarifai's API. As a convenience, we provide a quick
way to generate an access token for the API on this page as well which can be used for direct
authorization to the API. Please see the Authentication documentation for more details about using
your credentials to access the API.

If you would like to change the name of you application at any time, you may also do so on this
detail page by selecting the edit button, typing in a new name, and saving the changes.

## Deleting Applications

Deleting an application is just as simple as creating one. From your list of
applications first click on the application you would like to delete. At the bottom of the details
page for the application, select the Delete button and it's gone!



# Authentication

API requests are authenticated using OAuth2 Client Credentials.  The Client ID and Client Secret
associated with your Application are used to generate temporary access tokens, which in turn
are sent with each HTTP request to the API.

The authentication parameters are:

Parameter | Description
--------- | -----------
client_id |  identifies which application is trying to access the service. This is unique and generated once for each Application in your account.  
client_secret |  provides security when authorizing with the service. This is unique and generated once for each Application in your account.  
access_token | the client_id and client_secret are needed to generate a unique access token that is then used to authorize your access to the service. This token expires regularly and can be renewed by generating a new access token periodically using the client_id and client_secret as described in the example below.

For more information regarding OAuth2, please see the [spec](http://tools.ietf.org/html/rfc6749).


## Access Tokens

> Get an access token using client keys.

```shell
curl -X POST -d "grant_type=client_credentials" -d "client_id=<client_id>" -d "client_secret=<client_secret>" https://api.clarifai.com/v1/token/
```

> The client_id and client_secret can also be passed as header parameters:

```shell
curl -X POST -d "grant_type=client_credentials" https://<client_id>:<client_secret>@api.clarifai.com/v1/token/
```

> Response:

```json
{
  "access_token": "U5j3rECLLZ86pWRjK35g489QJ4zrQI",
  "expires_in": 36000,
  "scope": "api_access",
  "token_type": "Bearer"
}
```

> The access_token is used in subsequent API calls by sending it in the HTTP Authorization header:

```
"Authorization: Bearer <access_token>"
````

> For example, a tag request passing the access token looks like:

```shell
curl -H "Authorization: Bearer <access_token>" https://api.clarifai.com/v1/tag/?url=http://www.clarifai.com/img/metro-north.jpg
```

### HTTP Request

`GET https://api.clarifai.com/v1/token/`

To generate access tokens, first create an [Application](#applications) in your account, if you
haven't already.  This generates client credentials for authentication.

The token endpoint is used to exchange your client credentials (`client_id` and `client_secret`)
for an access token.

### Request parameters

Parameter | Description
--------- | -----------
grant_type | Required.  Authentication token type. Allowable values: "client_credentials".


## Token expiration and refresh

Each access token is valid for a limited time window, which is reported in the
`expires_in` field of the initial token response.

Requests received with an expired token will generate an HTTP 401 error code, and the
[status_code](#status-codes) `TOKEN_EXPIRED`.


# Requests and Responses

GET
Some of the endpoints support GET requests for convenient access to the API. Most methods however support POST, so it is recommended to utilize POST methods whenever possible.

POST
Other endpoints can accept POST requests which can be in either JSON or multipart-form formats. When sending image bytes, it is highly recommended to use the multipart-form POST as it is much more efficient than json which requires base64 encoding of the images.

## Headers

When using either GET or POST requests, the Authorization header must be provided:

	  "Authorization: Bearer <access_token>"

# Data Formats

The Clarifai API supports multiple input formats for data as described below.

## url

If you would like to process an image that is hosted at an existing publically accessible url,
then you can simply provide that url as an image input format. This url is downloaded on our servers
and then processed, therefore it must contain valid image data and be accessible to public servers.
If there in an error downloading the image or the format is not correct, an error response will be
returned. Note: the url must be url-encoded in many cases. This is to ensure all the necessary url
parameters of the original url are passed correctly to the server so that the data can be
downloaded.

## encoded_data

This format is for passing the raw bytes of a compressed image (such as JPEG or PNG) with the
request. This is convenient for processing local files with the API by transferring image bytes
directly in the request. This input is decoded as a string of bytes on the server side for several
supported image formats. This input format is ONLY supported by POST methods because the raw bytes
of an image may not be encodable in a GET method. The encoded_data should be send via a
multipart-form POST for efficiency reasons.

<aside class="warning">
Warning: Using encoded_data with JSON

If you use the encoded_data image format with a JSON request, you MUST base64 encode the image
before sending the bytes. Otherwise the JSON request will not be parsed properly. Sending the
encoded_data via JSON is therefore NOT RECOMMENDED because base64 encoding adds additional overhead
on both the client/server as well as additional network traffic. Therefore, we strongly recommend
sending encoded_data as a multipart-form as described above. We may not support JSON encoded_data
uploads in the future.
</aside>

Parameter | Description
--------- | -----------

# Responses

Responses from the API are always in JSON format. Each response has the following top-level structure:


```json
{
    "status_code": "OK",
    "status_msg": "All images in request have completed. ",
    "results": { ...something... }
}
```

The top-level fields are:

Parameter | Description
--------- | -----------
status_code | a string summarizing request status. In addition HTTP status code, provides more detail about the response. See [Status Codes](#status-codes) for details.
status_msg | a description of the status code for this request. This can provide more detail when a request has an error.
meta | extra information such as the type of model used to process a request is returned in the meta field. This must be a url safe string.
results | the data sent back with the response. This can be a dictionary of information for information endpoints or an array of results, one for each image in the request. The value here depends on the endpoint and is described in more detail in the docs for each endpoint.

# Usage Quota and Throttling

Each plan has a defined usage quota for access to the API. Once you usage exceeds this maximum,
additional charges may apply. The usage of the API is monitored on a per operation and per image
basis. For example: if you provide a single image for tagging, that counts as one transaction. If
you request multiple operations for a single image, then multiple transactions occur, one for each
operation. Similarly, if multiple images are provided in a batch, then one transaction per image is
accumulated.

## Rate Limits

In addition to your overall usage, there are rate limits in place to ensure fair use of our service.
You can view these on your Account Usage Page. If you exceed the limits defined there you will be
throttled to prevent overuse of the API and allow others to benefit from the service.

