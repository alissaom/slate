# Tags

> Tag a single image by URL:

```shell
curl -H "Authorization: Bearer <access_token>" https://api.clarifai.com/v1/tag/?url=http://www.clarifai.com/img/metro-north.jpg

```

> Or using a POST request:

```shell
curl -H "Authorization: Bearer <access_token>" --data-urlencode "url=http://www.clarifai.com/img/metro-north.jpg"  https://api.clarifai.com/v1/tag/
````

> Tag multiple images in a batch request, sending URLs:

```shell
curl -H "Authorization: Bearer <access_token>" --data-urlencode "url=http://www.clarifai.com/img/metro-north.jpg" -d "url=http://www.clarifai.com/img/metro-north.jpg"  https://api.clarifai.com/v1/tag/
````

> Tag multiple images in a batch request, uploading image files from local disk:

```shell
curl -H "Authorization: Bearer <access_token>" -F "encoded_data=@/path/to/example.jpeg" -F "encoded_data=@/path/to/another-img.jpeg" https://api.clarifai.com/v1/tag/
```

`GET https://api.clarifai.com/v1/tag/`

Tag an image with recognized objects and descriptive tags.  Returns tags and associated 
confidences (probabilities).

## Request parameters

Parameter | Description
--------- | -----------
url | URL of the content to be recognized.
model | Optional. Specify the type of model to process the images with. Currently must be 'default'.
local_id | User-supplied identifier, returned with the result for each image for convenience when associating results with requests. If not supplied, the request URL is returned when available.
select_classes | Optional. Comma-separated list of class names (string).  
Used to select a specific set of recognized tags to be returned in the response.  See [Selecting tags](#selecting-tags) for details.

## Selecting tags

> Get predictions for a specific set of tags using the `select_classes` request param:

```shell
curl -H "Authorization: Bearer <access_token>" --data-urlencode "url=http://www.clarifai.com/img/metro-north.jpg"  -d "select_classes=smile,frowning,laughing" https://api.clarifai.com/v1/tag/
```

Normally we return the top recognized tags by predicted confidence.
For some applications, the top tags are less useful than getting predictions for a specific set
of tags.  We support this using the `select_classes` request parameter.

When `select_classes` is provided in the request, only results for the requested classes are
returned.  


## Response

> Example response with a single image.

```json
{"meta": {
   "tag": {
       "config": "cf0267260d65c1312cac67ab2474dbdc",
       "model": "default",
       "status_code": "OK",
       "timestamp": 1414364476.483851}},
 "results": [
     {"docid_str": "31fdb2316ff87fb5d747554ba5267313",
      "local_id": "http://www.clarifai.com/img/metro-north.jpg",
      "result": {
          "tag": {
              "classes": [
                  "train",
                  "subway",
                  "railroad",
                  "railway",
                  "station",
                  "rail",
                  "transportation",
                  "urban",
                  "underground",
                  "night",
                  "traffic",
                  "commuter",
                  "bridge",
                  "city",
                  "road",
                  "national park",
                  "public",
                  "platform",
                  "street",
                  "blur"],
              "probs": [
                  0.9994779825210571,
                  0.9983398914337158,
                  0.9967546463012695,
                  0.9949972033500671,
                  0.9938348531723022,
                  0.9938316345214844,
                  0.989970326423645,
                  0.9786827564239502,
                  0.9764524698257446,
                  0.9714784026145935,
                  0.9520049095153809,
                  0.9501165151596069,
                  0.9497720003128052,
                  0.9494657516479492,
                  0.9481779336929321,
                  0.9384146928787231,
                  0.938301682472229,
                  0.9342441558837891,
                  0.9231494665145874,
                  0.9211686849594116]
              }
          },
      "status_code": "OK",
      "status_msg": "OK"}
 ],
 "status_code": "OK",
 "status_msg": "All images in request have completed sucessfully."
}
```

Image and video recognition results are returned in the `results` array,
one result per image. 

Each result
has its own `status_code` and `status_msg` which allows for individual images in a batch to succeed
even if some fail. 

Notable response parameters:

Parameter | Description
--------- | -----------
docid_str | A unique string identifying each result. Used when sending [feedback](#feedback).
local_id | User-supplied identifier, echoed back in each result if supplied in the request, or the URL of the image is returned if available.
classes | An array of tags recognized from this image or video segment.
probs | An array of confidences (probabilities) associated with the tags returned in `classes`.


# Feedback

## HTTP Request

`GET|POST https://api.clarifai.com/v1/feedback/`

This endpoint provides the ability to provide feedback to the API about images that were previously processed, for example to correct errors made by the recognition engine.

We highly encouraged integrating feedback into your application as this dramatically improves the system over time. You or your users can correct errors made by the recognizer. For example, if the system returned "cat" for an image of a dog, you can send a message with add_tags="dog", remove_tags="cat".

Requests to the feedback endpoint can be made with either http GET or POST methods.

<aside class="notice">
Feedback requests are always free. Vote early, vote often!
</aside>

<aside class="warning">
Abuse by providing poor quality or incorrect feedback, may result in account suspension or
termination, as it can affect the service for other users.
</aside>

## Request parameters

Parameter | Description
--------- | -----------
docids | Comma-separated list of docids (strings) that identifies which images/video the feedback applies to. Use the value of the `docid_str` field in the tag response.
url |  Url that identifies the image/video the feedback applies to. Multiple urls can be sent, but since URLs can contain commas, each must be sent as a separate HTTP request param.
add_tags | comma-separated list of tags for positive feedback. UTF-8 encoded.
remove_tags | comma-separated list of tags for negative feedback. UTF-8 encoded.
similar_docids | comma-separated list of docids that are similar to the input docids.
dissimilar_docids | comma-separated list of docids that are dissimilar to the input docids.
search_click | comma-separated list of search terms that generated clicks on a set of docids.

One and only one parameter in the feedback request must identify the image: either `url` or
`docids`.


## Positive feedback

> Add tags or give positive feedback for tags to a docid:

```shell
curl -H "Authorization: Bearer <access_token>" -d "docids=78c742b9dee940c8cf2a06f860025141" -d "add_tags=car,dashboard,driving"  https://api.clarifai.com/v1/feedback/
```

> Add tags, specifying the image by URL instead of docid:

```shell
curl -H "Authorization: Bearer <access_token>" --data-urlencode "url=http://www.clarifai.com/img/metro-north.jpg" -d "add_tags=train station,railway station"  https://api.clarifai.com/v1/feedback/
```

> Suggest tags for multiple images in a single request, with a comma-separated list of docids:

```shell
curl -H "Authorization: Bearer <access_token>" -d "docids=78c742b9dee940c8cf2a06f860025141,5849206sdee940c8cf2a06f8792384059"  -d "search_click=cat,hat,mat" https://api.clarifai.com/v1/feedback/
```
 
Confirm correct and useful tags, or suggest missing tags, using the `add_tags` field.

If the user believes additional tags are relevant to the given image, they can be provided in the
`add_tags` argument. `add_tags` accepts a comma-separated list of tags in UTF-8 encoding.

## Negative feedback on errors

> Remove tags or give negative feedback for tags to a docid:

```shell
curl -H "Authorization: Bearer <access_token>" -d "docids=78c742b9dee940c8cf2a06f860025141" -d "remove_tags=sky,clean,red"  https://api.clarifai.com/v1/feedback/
```

If the user believes tags were are not relevant to the given image, they can be provided in the
`remove_tags` argument. `remove_tags` accepts a comma-separated list of tags in UTF-8 encoding.


## Suggest similar items

> Add similar docids for a given docid:

```shell
curl -H "Authorization: Bearer <access_token>" -d "docids=78c742b9dee940c8cf2a06f860025141" -d "similar_docids=fc957ec10abcc0f4507475827626200a" https://api.clarifai.com/v1/feedback/
```

If there is a notion of similarity between images, this can be fed back to the system by providing
an input set of `docids` and `similar_docids`, a comma-separated list of docids that are similar
to the input docids.


## Suggest dissimilar items

> Add dissimilar docids for a given docid:

```shell
curl -H "Authorization: Bearer <access_token>" -d "docids=78c742b9dee940c8cf2a06f860025141" -d "dissimilar_docids=acd57ec10abcc0f4507475827626785f" https://api.clarifai.com/v1/feedback/
```

If there is a notion of similarity between images, this can be fed back to the system by providing
an input set of `docids` and `dissimilar_docids`, a comma-separated list of docids that are
dissimilar to the input docids.


## Search result click feedback

> Provide feedback on which items were clicked after searching for the specified tags.

```shell
curl -H "Authorization: Bearer <access_token>" -d "docids=78c742b9dee940c8cf2a06f860025141"  -d "search_click=cat" https://api.clarifai.com/v1/feedback/
```

This is useful when showing search results in response to a query, to provide feedback that the
search result was relevant to the query. The `docids` parameter identifies the image that was
clicked, and the `search_click` parameter is a comma-separated list of search terms that
generated the search result.


## Feedback response

> Example feedback response:
	    
```json
{
  "status_code": "OK",
  "status_msg": "Feedback sucessfully recorded. "
}
```

The Feedback response contains `status_code` and `status_msg`.

# Info and API limits

The API enforces certain limits on requests.

For example, the API enforces limits on the size of images that can be processed.
Too small of an image would result in poor recognition results due to low image resolution, and
too large of an image slows down processing unnecessarily due to network transfer and resizing
operations on the server.

## Info

> Example response

```json
{"embed_allowed": true,
 "max_video_batch_size": 1,
 "max_image_size": 1024,
 "max_batch_size": 128,
 "min_image_size": 224,
 "max_image_bytes": 10485760,
 "max_video_size": 1024,
 "min_video_size": 224,
 "max_video_bytes": 104857600,
 "api_version": 0.1,
 "max_video_duration": 1800}
```

The `/info` endpoint returns current API details including limits and thresholds on requests.

The response includes:

Field | Description
--------- | -----------
max_batch_size | The maximum number of images allowed to process in one bach request.
max_image_size | The maximum allowed image size (on the minimum dimension).
min_image_size | The minimum allowed image size (on the minimum dimension).

