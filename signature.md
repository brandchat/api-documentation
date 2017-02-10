# BrandChat message signature

The API signs every call to your server, and requires you to sign every call to our servers.
 
In order to generate the signature, you need to have an API key. You can find your API key by logging in to https://cms.brandchat.social and enabling API access from the menu.

## Calculating the signature

### JSON POST calls

For calls that are JSON POSTs, the signature is calculated as:

* **HMAC** of the JSON-serialised payload,
* using your **API key** as the secret key, and
* with **SHA1** as the hashing algorithm.
 
Example (PHP):

```php
<?php
$payload = json_encode([
    [
        'type' => 'text',
        'text' => 'Hello world!'
    ],
    [
        'type' => 'text',
        'text' => 'A follow-up message.'
    ]
]);
$key = 'MY_API_KEY_GOES_HERE';
$signature = hash_hmac('sha1', $payload, $key);
```

### File uploads (multipart/form-data)

When uploading files using a POST of type `multipart/form-data`, the signature is calculated based on the file contents:

* **HMAC** of the binary file contents,
* using your **API key** as the secret key, and
* with **SHA1** as the hashing algorithm.

Example (PHP):

```php
<?php
$filename = '/path/to/image.png';
$key = 'MY_API_KEY_GOES_HERE';
$signature = hash_hmac_file('sha1', $filename, $key);
```

### Important note about case sensitivity

Signatures in requests *to* BrandChat will be compared case insensitively.

*However*, it's important to note that the signatures that BrandChat includes in calls to your webhook will use **lowercase** hex digits (hexits). Some HMAC libraries (especially on Windows) seem to generate uppercase hexits. So please ensure that when verifying the signature, you do so in a case-insensitive manner (or convert the reference and calculated signatures to a known case before comparing them).

## Sending the signature

When making requests *to* BrandChat, you need to send the signature in the `X-Chat-Signature` header.

Example (PHP, using [GuzzleHttp](http://docs.guzzlephp.org/en/latest/)):

```php
<?php
$client = new GuzzleHttp\Client();
$response = $client->post('http://api.brandchat.social/api/user/profile?bot=MY_BOT_IDENTIFIER', [
    'body' => '{"userId": 1337}',
    'headers' => [
        'Content-Type' => 'application/json',
        'X-Chat-Signature' => $signature,
    ],
]);
```

## Validating the signature

When receiving events *from* BrandChat, we'll also pass through the signature in the `X-Chat-Signature` header.

You should then calculate your reference signature as the HMAC of the payload (before de-serialisation!) and your API key using SHA1. The reference signature should match the signature in the request.

**We *strongly* recommend that you do not proceed to process an event if the signature fails to validate. Instead, it's advisable to respond with a status of `401 Forbidden`.**

Example (PHP):

```php
<?php
$payload = file_get_contents('php://input');    // read the request body
$signature = $_SERVER['HTTP_X_CHAT_SIGNATURE']; // get the X-Chat-Signature header
$key = 'MY_API_KEY_GOES_HERE';                  // remember to plug in your own API key
$referenceSignature = hash_hmac('sha1', $payload, $key);
if ($signature != $referenceSignature) {
    die("Signature mismatch");
} else {
    // todo: implement the good stuff
}
```

## Next steps

Learn more about [event callbacks](callbacks.md), or go back to the [README](README.md)