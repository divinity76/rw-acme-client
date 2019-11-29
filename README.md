# Let’s Encrypt ACME client written in PHP

[![Latest Version on Packagist](https://img.shields.io/packagist/v/rogierw/letsencrypt-client.svg?style=flat-square)](https://packagist.org/packages/rogierw/letsencrypt-client)
[![StyleCI](https://github.styleci.io/repos/224902862/shield?branch=master)](https://github.styleci.io/repos/224902862)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/RogierW/letsencrypt-client/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/RogierW/letsencrypt-client/?branch=master)

This library allows you to request, renew and revoke SSL certificates provided by Let's Encrypt.

## Requirements
- PHP ^7.1
- cURL extension

## Installation
You can install the package via composer:

`composer require rogierw/letsencrypt-client`

## Usage

You can create an instance of `Rogierw\Letsencrypt\Api` client.

```php
$client = new Api('test@example.com', __DIR__ . '/__account');
```

### Creating an account
```php
if (!$client->account()->exists()) {
    $account = $client->account()->create();
}

// Or get an existing account.
$account = $client->account()->get();
```

### Creating an order
```php
$order = $client->order()->new($account, ['example.com']);
```

### Getting the order
```php
$order = $client->order()->get($order->id);
```

### Getting the DCV status
```php
$domainValidationStatus = $client->domainValidation()->status($order);

// Get the first element in the array. Usually there is only one element.
$domainValidation = $domainValidationStatus[0];
```

### Start HTTP challenge
```php
if ($domainValidation->isPending()) {
    // Get the data for the HTTP challenge; filename and content.
    $validationData = $client->domainValidation()->getFileValidationData($domainValidation);

    $client->domainValidation()->start($account, $domainValidation);

    $domainValidationStatus = $client->domainValidation()->status($order);
    $domainValidation = $domainValidationStatus[0];
}
```

### Generating a CSR
```php
$privateKey = \Rogierw\Letsencrypt\Support\OpenSsl::generatePrivateKey();
$csr = \Rogierw\Letsencrypt\Support\OpenSsl::generateCsr(['example.com'], $privateKey);
```

### Finalizing order
```php
if ($order->isReady() && $domainValidation->isValid() && $order->isNotFinalized()) {
    $client->order()->finalize($order, $csr);
}
```

### Getting the actual certificate
```php
if ($order->isFinalized()) {
    $certificateBundle = $client->certificate()->getBundle($order);
}
```

### Revoke a certificate
```php
if ($order->isValid()) {
    $client->certificate()->revoke($certificateBundle->fullchain);
}
```