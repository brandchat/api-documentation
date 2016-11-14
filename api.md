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

In general, an HTTP status code of 200 means that everything went well. Any other status code means that something went wrong and the call was not actioned successfully. The payload for unsuccessful will generally include a `reason` property. It is provided purely for informational and debugging purposes and should not be relied on for your business logic. An example of an unsuccessful call would be the following response:

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

## Payloads

### User profile

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
    "type": "text|image|voice|video|audio|list"
    // + additional properties for specific type ...
  }
]
```

