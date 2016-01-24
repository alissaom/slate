# Status codes

Clarifai Vision API responses contain status codes at the top-level request, in addition
 to per-image status codes for multi-image batch requests or video requests.

## Request-level status codes

Status Code | HTTP Status Code | Meaning
---------- | ------- | ----------
OK | 200 | Request succeeded.
ALL_ERROR | 400 | All images in the request failed.
PARTIAL_ERROR | 400 | Some images in request have failed. Please review the error messages per image.
TOKEN_EXPIRED | 401 | Token has expired, you must generate a new access token. See [Access Tokens](#access-tokens).
TOKEN_APP_INVALID | 401 | Application for this token is not valid. Please ensure that you are using ID and SECRET from same application.
TOKEN_INVALID | 401 | Token is not valid. Please use valid tokens for a application in your account.
TOKEN_NONE | 401 | Authentication credentials were not provided in request.
TOKEN_NO_SCOPE | 401 | Token does not have the required scope to access resources.


## Image-level status codes

For multi-image batch requests and video requests.

Status Code |  Meaning
---------- | ----------
OK | Image loading failed, see results for details.
CLIENT_ERROR | Image loading failed, see results for details.
SERVER_ERROR | Image failed to process, see results for details. 
