# Web integration between you and BrandChat

For messenger platforms that support web-views (currently Facebook Messenger, WeChat and Kik), you can configure the system to pass through a *one-time code* to a web-page you can configure on the BrandChat CMS. In this case, the flow is as follows:

1. Configure a content item of type *External web page* under *Content* on the BrandChat CMS. The URL for the external web page should contain a substitution for `:code`, e.g. the URL could be something like http://example.com/my/route?code=:code
2. Link that content to a menu or a keyword on the BrandChat CMS.
3. When the user opens that content, BrandChat will authenticate the user.
4. If we managed to authenticate the user, we will substitute `:code` in your configured URL with a one-time code. After the substitution, we will then redirect to the resulting URL.
5. Your web-page can grab the code from the query string (and pass that code to your back-end, where relevant).
6. Your back-end should then call BrandChat to claim the code.
7. If the one-time code is valid, BrandChat will then respond with a user profile object.

Claiming the one-time code in order to receive the user profile is an API call similar to the *Retrieve user profile* call described in the [API documentation](api.md). It uses the same base URL, and requires an API signature header and a bot identifier in the query string.

Please also familiarise yourself the [API documentation](api.md) for more details regarding the API base URL, bot identifier, API signature, and response status codes.

## Claim one-time code for a user profile

*Relative route:* `code/claim`

*Request method:* `POST` with `Content-Type: application/json`

*Request payload:*

```json
{
  "code": "1337-4PTrl14pslYNm2DZiTl2DfmkEChptq4L"
}
```

The `code` property value is a string representing the unique one-time code passed to your web-page.

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
  "lastSubscribeTimestamp": 123456789,
  "platformIdentifier": "a_unique_identifier_from_the_platform"
}
```

The response body properties are identical to the properties in the retrieve user profile request, and are listed here for completeness:

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
* `platformIdentifier`: a string representing a unique identifier of the user in the context of your bot, provided by the messaging platform. On WeChat, for example, this represents the user's *Open ID*.

Please note that all of the above properties with the exception of the `userId` are retrieved from the user's messaging platform and hence may not necessarily be correct, consistent or complete.

Notes:

* The one-time code is good for a single lookup only.
* As soon as you have claimed the code, it is no longer valid. Any future request to claim it will result in an error. (Hence why we call this a *one-time* code.)
