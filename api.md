# BrandChat API methods

BrandChat offers several API methods that your can call from your server to do the following:

* Send messages
* Lookup profile information
* Upload files (for use in image, voice or video messages)

## API base URL

The base URL for the BrandChat API is:

`https://api.brandchat.social/api/`

All the individual API calls are relative to that URL.

## API signature

Every API call must include a signature as described in the [signature documentation](signature.md), in the `X-Chat-Signature` request header. Please note that any call with a missing or invalid signature will be rejected.

## Response status codes

In general, an HTTP status code of `200 OK` means that everything went well. Any other status code means that something went wrong and the call was not actioned successfully. The payload for unsuccessful will generally include a `reason` property. It is provided purely for informational and debugging purposes and should not be relied on for your business logic. An example of an unsuccessful call would be the following response:

*Status code:* `400 Bad Request`

*Response body:*

```json
{
  "reason": "userId missing or empty"
}
```

The following status codes are used:

* `400 Bad Request`: Something in the request payload was invalid
* `401 Unauthorized`: Bot is not enabled for API access or API signature is missing
* `403 Forbidden`: API signature mismatch or trying to access a resource that falls outside your API key's entitlements
* `500 Internal Server Error`: Something went wrong on the BrandChat system

As is usual for HTTP status codes, the `4xx` range broadly means "something went wrong on your end" and `5xx` means "something went wrong on the other end".

Notes:

* Additional status codes may be added to better differentiate between different error conditions. Please handle any and all status codes gracefully.
* While we attempt to include the `reason` property in all error responses, please do not rely on its presence or absence (or specific content) in your business logic.

## API methods

### Retrieve user profile

*Relative route:* `user/profile`

*Request method:* `POST` with `Content-Type: application/json`

*Request payload:*

```json
{
  "userId": 1337
}
```

The `userId` parameter is an integer representing the unique identifier of the user, which you would generally have obtained from the payload of a preceding [event callback](callbacks.md).

*Response body:*

```json
{
  "userId": 1337,
  "platform": "wechat|kik|facebook|telegram",
  "displayName": "A. Nonymous",
  "profileImageUrl": "http://example.com/path/to/profile.jpg",
  "gender": 0,
  "country": "ZA",
  "province": "Western Cape",
  "city": "Stellenbosch",
  "isSubscribed": true,
  "lastSubscribeTimestamp": 123456789
}
```

The response body properties are identical to the properties in the profile event callback, and are listed here for completeness:

* `userId`: an integer identifying the user uniquely on BrandChat
* `platform`: a string literal representing the platform through which the user interacted
* `displayName`: the display name of the user (empty string indicates *no information*)
* `profileImageUrl`: the URL with a profile picture of the user (empty string indicates *no information*)
* `gender`: an integer representing the user's gender (`0`: unspecified, `1`: man, `2`: woman)
* `country`: the user's country in [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) format (empty string indicates *no information*)
* `province`: a string representing the user's province
* `city`: a string representing the user's city
* `isSubscribed`: a boolean indicating whether the user is subscribed to your bot
* `lastSubscribeTimestamp`: an integer representing the UNIX timestamp (seconds since epoch) when the user last subscribed to your bot (if at all; may be 0) 

Please note that all of the above properties with the exception of the `userId` are retrieved from the user's messaging platform and hence may not necessarily be correct, consistent or complete.

### Send messages

*Relative route:* `send/messages`

*Request method:* `POST` with `Content-Type: application/json`

*Request payload:*

The following is the basic structure of the payload for sending messages to a user:

```json
[
  {
    "type": "text|image|voice|video|audio|list",
    "userId": 1337
  }
]
```

In other words, the payload is a *list* (array) of message objects, each with a `type`, a `userId` (representing the recipient of the message) and some additional type-specific properties. The message objects for each available type are described next.

* Your payload may contain up to 10 messages.
* The request will be rejected if you're attempting to send more than 10 messages at once.
* If any message in a batch fails to validate, **none** of the messages in the batch will be sent.

*Response:*

The response will be an HTTP status code (with `200 OK` indicating success, and any other code indicating failure). The response body will be empty on success (but may contain a JSON object with a `reason` property on failure).

#### text message

The structure for message objects of type `text` is:

```json
{
  "type": "text",
  "userId": 1337,
  "text": "message text that will be sent to the user"
}
```

The string in the `text` property has a maximum character length of 1000 characters.

#### image, voice and video messages

The message objects for `image`, `voice`, and `video` messages can have either of the following two structures, depending mostly on your preference on how to send the file.

1. The file you're sending is on a publicly exposed web route on your server:

```json
{
  "type": "image|voice|video",
  "userId": 1337,
  "url": "http://example.com/path/to/file"
}
```

2. You've previously uploaded the file to BrandChat's servers to obtain a re-usable `fileId`:

```json
{
  "type": "image|voice|video",
  "userId": 1337,
  "fileId": 123456789
}
```

Notes:

* `type`: string literal indicating the type of message (`image` *or* `voice` *or* `video`)
* `userId`: integer used in identifying the intended recipient of the message 
* `url`: string containing the public URL containing the file you're sending
* `fileId`: the integer file ID obtained from a prior file upload to BrandChat's servers (refer to the File upload API call for more details)
* A message object **must** contain *either* a `url` *or* a `fileId`, but *not both*.
* If you are using the `url` method for sending multimedia messages, please ensure that your server returns valid `Content-Length` and `Content-Type` headers for `HEAD` requests. BrandChat validates the file size and content type using a `HEAD` request.
* Which method (`url` or `fileId`) you want to use depends mostly on your preference. However, if you want to send the same file to multiple times, we strongly recommend that you use the `fileId` method for efficiency.

Allowed file sizes and content types are as follows (1 kB = 1024 bytes, 1 MB = 1024 kB):

* For `image` messages:
    * Max file size: 128 kB
    * Allowed content types: `image/jpg`, `image/jpeg`, `image/png`
* For `voice` messages:
    * Max file size: 256 kB
    * Allowed content type: `audio/amr`
* For `video` messages:
    * Max file size: 1 MB
    * Allow content type: `video/mp4`

#### audio messages

TBC

#### list messages

TBC

### Upload files

*Relative route:* `file/upload`

*Request method:* `POST` with `Content-Type: form/multi-part`

*Request payload:*

The request should be a standard multi-part form POST containing a **single** file. (Requests containing zero files, or multiple files, will be rejected.)

Also keep in mind that the [signature](signature.md) for multi-part form POSTs is calculated based on the file contents and not the request body.

The same file size restrictions as described above 

*Response:*

```json
{
  "fileId": 123456789
}
```

The obtained `fileId` may then be used when sending `image`, `voice` or `video` messages in future. At this stage, uploaded files do *not* expire, but we may limit the amount of files that may be stored.

Please ensure that you can handle 64-bit integers for `fileId`. 
