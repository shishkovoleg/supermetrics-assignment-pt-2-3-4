```php
$tokenInfo = file_get_contents('https://api.supermetrics.com/assignment/register?client_id=ju16a6m81mhid5ue1z3v2g0uh&email=my@name.com&name=My%20Name');
```

## Unfortunately this snippet will not work as expected. Below is explanation.

1. By default, file_get_contents() sends a GET HTTP request, but we should send a POST request, since we are calling a POST API endpoint.
To send POST request we should provide a context object as third argument:
   
```php
$data = http_build_query(
    array(
        'client_id' => 'ju16a6m81mhid5ue1z3v2g0uh',
        'email' => 'my@name.com',
        'name' => 'My Name',
    )
);
$ctx = stream_context_create(
array(
  'http'=>array(
    'header'=> 'Content-type: application/x-www-form-urlencoded',
    'method'=>'POST',
    'content'=> $data
  )
)
);
$tokenInfo = file_get_contents($url, false, $ctx);
```

2. Even if we provide a context object to make a POST request it still can fail because we send request to https and https wrapper requires to have php_openssl extension enabled and allow_url_fopen must be set to on:

```php
extension=php_openssl.dll

allow_url_fopen = On
```

As a workaround we can omit SSL checks with providing in context following key-value pairs

```php
    "ssl"=>array(
        "verify_peer" => false,
        "verify_peer_name" => false,
    ),
```

but I even don't want to test it since it doesn't look good.

## Solutions

### If we need only one HTTP call in our very small application where we don't have any libraries for working with HTTP requests

I propose to use CURL functions instead. Sequence is `curl_init, curl_setopt, curl_exec, curl_close`.

### If need more than one HTTP request in our application

- probably we use some framework, in this case we should check what our framework has out of the box for working with HTTP requests.
- if we don't have solutions for HTTP requests out of the box in our framework we can install guzzle with following command (supposed that we use composer)
```
composer require guzzlehttp/guzzle
```



