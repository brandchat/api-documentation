# Event callbacks

Whenever an event happens that should be of interest to you, we do a *callback* to the URL that you configured on the [BrandChat CMS](https://cms.brandchat.social).

The following events currently result in callbacks to you:

* A user subscribes to your bot
* A user unsubscribes from your bot
* A user sends a message to your bot
* A user's profile was updated
* We received a location for a user

## Basic information

* Whenever a callback happens, BrandChat will do an HTTP POST to your callback URL.
* The request body will be a JSON-serialised representation of the event.
* Every callback will be include the [signature](signature.md) which you should validate with your API key to ensure that the message came from BrandChat.
* Every event payload object, irrespective of event type, contains at least the following two properties:
  * `type`: a string literal indicating the event type; you can then parse the remaining payload based on the event type 
  * `timestamp`: an integer indicating the UNIX timestamp (seconds since epoch) of when the message was first received by BrandChat
* Note that in addition to the properties for each event object specified below, we may in future add additional properties. Please ensure that your code will handle additional, currently undocumented properties gracefully (= ignores them).

## Event types

The following event types (in the `type` property present in every callback) are currently available:

* `subscribe`
* `unsubscribe`
* `message`
* `profile`
* `location`

Please note that more event types may be added in future, and you should implement your system to gracefully handle (= ignore) unknown event types.

## Payloads

### subscribe event

`subscribe` events are emitted when a user subscribes to your bot.

The request payload will have the following structure:

```json
{
  "type": "subscribe",
  "timestamp": 123456789,
  "subscribe": {
    "userId": 1337
  }
}
```

The `userId` property contains a unique numeric identifier for this user on the BrandChat platform. Please ensure that you can accomodate 64-bit integers for this field. Also, please note that events for the *same* user will always contain the same `userId`.

### unsubscribe event

`unsubscribe` events are emitted when a user unsubscribes from your bot.

The request payload will have the following structure:

```json
{
  "type": "unsubscribe",
  "timestamp": 123456789,
  "subscribe": {
    "userId": 1337
  }
}
```

The `userId` here is as defined for the `subscribe` event.

### message event

`message` events have the following basic structure:

```json
{
  "type": "message",
  "timestamp": 123456789,
  "message": {
    "type": "text|image|voice|video|location"
  }
}
```

The `message.type` property will indicate what type of message it is. This will allow you to parse the payload for the specific message type.

Please note that additional message types may be added in future. Hence your system should gracefully handle (= ignore) unknown message types.

#### text message

`text` messages have the following structure:

```json
{
  "type": "message",
  "timestamp": 123456789,
  "message": {
    "type": "text",
    "text": "the user's message"
  }
}
```

The `text` property will contain the user's message.

#### image message

`image` messages have the following structure:

```json
{
  "type": "message",
  "timestamp": 123456789,
  "message": {
    "type": "image",
    "image": {
      "url": "http://example.com/path/to/image.jpg",
      "contentType": "image/jpeg",
      "size": 12345
    }
  }
}
```

* `url`: the URL where you can fetch the image which the user sent to your bot
* `contentType`: the MIME type of the image; typically `image/jpg`, `image/jpeg` or `image/png`
* `size`: the size of the image in bytes

#### voice message

`voice` messages have the following structure:

```json
{
  "type": "message",
  "timestamp": 123456789,
  "message": {
    "type": "voice",
    "voice": {
      "url": "http://example.com/path/to/voice.amr",
      "contentType": "audio/amr",
      "size": 12345
    }
  }
}
```

* `url`: the URL where you can fetch the voice recording which the user sent to your bot
* `contentType`: the MIME type of the voice recording; typically `audio/amr`
* `size`: the size of the recording in bytes

#### video message

`video` messages have the following structure:

```json
{
  "type": "message",
  "timestamp": 123456789,
  "message": {
    "type": "video",
    "video": {
      "url": "http://example.com/path/to/video.mp4",
      "contentType": "video/mp4",
      "size": 12345
    }
  }
}
```

* `url`: the URL where you can fetch the video which the user sent to your bot
* `contentType`: the MIME type of the video; typically `video/mp4`
* `size`: the size of the video in bytes

#### location message

`location` messages have the following structure:

```json
{
  "type": "message",
  "timestamp": 123456789,
  "message": {
    "type": "location",
    "location": {
      "latitude": 12.3456,
      "longitude": -65.4321
    }
  }
}
```

* `latitude`: float representing the latitude of the location which the user shared
* `longitude`: float representing the longitude of the location which the user shared

### profile event

`profile` events are emitted when a user's profile is updated. Please note that this event is not necessarily tied to any specific action which the user has taken.

The request payload will have the following structure:

```json
{
  "type": "profile",
  "timestamp": 123456789,
  "profile": {
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
}
```

The properties of the profile object are:

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

### location event

`location` events are emitted when the messaging platform sends us a user's location. Generally, users need to have locations enabled on their device, allow the messenger to locate them, and allow the messenger to pass their location to your bot. Not all platforms currently support locations. If you need user locations, please ensure that you have fallback methods in place (like asking the user to reply with their address).

The request payload has the following structure:
 
 ```json
 {
   "type": "location",
   "timestamp": 123456789,
   "location": {
     "userId": 1337,
     "latitude": 12.3456,
     "longitude": -65.4321
   }
 }
 ```

The properties are:

* `userId`: an integer uniquely identifying the user on BrandChat
* `latitude`: float representing the user's latitude
* `longitude`: float representing the user's longitude

