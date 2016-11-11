# BrandChat API documentation

BrandChat is a multi-platform service that allows you to create and manage bots on WeChat, Facebook Messenger, Telegram, and Kik. (More messengers, like WhatsApp, will be added as they open up API access to commercial accounts.)

Apart from a feature-rich Content Management System which offers out-of-the-box messaging widgets like quizzes and polls, BrandChat also offers an API to developers. With this API, developers can do things like:

* Be notified when a user subscribes to your bot
* Send and receive user messages for all supported messengers with one simple API
* Look up user profile information like nicknames and profile pictures

## Getting started

If you haven't done so yet, please register an account at https://cms.brandchat.social and enable API access from the menu.

The BrandChat CMS will issue you with:

* An **API key**, which you will use for signing requests *to* BrandChat (like when you send a message) and validating requests *from* BrandChat (like when we deliver a user message to you).
* A **bot identifier**, which you will use on every request to BrandChat so that we know which bot you're interacting with. This identifier is also included in every event that BrandChat delivers to your system, allowing you to distinguish between bots in the case where you have multiple bots running on BrandChat.

You can also configure a **callback URL** which points to *your* server. If configured, BrandChat will deliver events like user messages and new subscriber notifications to this endpoint. While this URL is optional, you'll have to provide it in cases where you want to receive inbound messages.

## Basic flow

### Receiving event messages

When users do things on your bot (like subscribing to it or sending your bot a message), the flow is:

1. BrandChat receives the event.
2. BrandChat does a call to your callback URL. The payload will include details of the event (like the text of the user's message), and also a signature.
3. You should validate the signature by hashing the event payload with your API key and comparing it against the signature. If the signature matches your calculation, you can trust that the event is from BrandChat and not a malicious third party (provided the key was not compromised).
4. You can then run your business logic on the event (like processing the message text to formulate a response message to the user).
4. You can then respond with a list of messages to send back to the user.

Or more succinctly from your server's perspective: Receive, validate, process, optionally respond.  

### Sending messages

If at any time (like on a scheduled process), you want to send a message *to* a user that is not necessarily tied to a message *from* that user, the flow is as follows:

1. Generate payload, and use it in conjunction with your API key to sign the message.
2. Do a call to BrandChat, including your payload and the calculated signature.

Or more succinctly: Create message, send.

## Basics of the API

* With a single exception, every call (both to and from us), is an HTTP **POST** where the request body is the JSON-serialised payload.
* In general, an HTTP response status of **200** indicates *success*; everything else is treated as a *failure*.
* Any callback events to your server that fail (i.e. don't respond with an HTTP 200), will be retried several times, spaced a couple of minutes apart.
* In calls from us to your server, the only requirement is that you respond with an HTTP 200. (I.e., the response body is allowed to be blank.)
* If the call to you is a subscription event (new user subscribed to your bot), or a message from a user, you may optionally include a JSON-serialised list of messages to send back to the user.

## Next steps

Learn more about:

1. [Message signature](signature.md)
2. Callback events (BrandChat to you)
3. BrandChat API (you to BrandChat)
4. API limits
5. Tips and tricks

## Reference implementations

We have open-sourced our [PHP reference implementation](https://github.com/brandchat/api-php) of the API. By including our [composer](https://getcomposer.org) package, you can be up and running in minutes.

More reference implementations to follow...
