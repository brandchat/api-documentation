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

## Event types

The following event types (in the `type` property present in every callback) are currently available:

* `subscribe`
* `unsubscribe`
* `message`
* `profile`
* `location`

## Payloads

### subscribe event

The request payload will have the following structure:

```json
{
  "type": "subscribe",
  "timestamp": int,
  "subscribe": {
    "userId": int
  }
}
```

