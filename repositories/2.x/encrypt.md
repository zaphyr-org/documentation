# Encrypt

A convenient interface for encrypting and decrypting text via OpenSSL using `AES-128` and `AES-256` encryption.

## Installation

To get started, install the encrypt repository via the [Composer](https://getcomposer.org/) package manager:

```bash
composer require zaphyr-org/encrypt
```

## Configuration

Before we can start encrypting, we need to make a few configurations. For encryption and decryption we can use a
16 character private key with `AES-128-CBC` cipher or a 32 character private key with the `AES-256-CBC` cipher.

### AES-128-CBC cipher

For an `AES-128-CBC` cipher, simply pass a 16 character string to the constructor of the `Encryt` instance:

```php
new Zaphyr\Encrypt\Encrypt('diw84lfnd74jdms6');
```

### AES-256-CBC cipher

If you use an `AES-256-CBC` cipher with a 32 character string, a second parameter must be passed to
the `Encrypt` instance:

```php
new Zaphyr\Encrypt\Encrypt('OOQPAgC4tA7NanCiVCa1QN5BiRDpdQZR', 'AES-256-CBC');
```

> [!WARNING]
> Please do not use the keys shown in the examples in your application!

## Encrypting

To encrypt values simply use the `encrypt()` method. If the value can not be properly encrypted, an `EncryptException`
will be thrown:

```php
try {
    $encryptor = new Zaphyr\Encrypt\Encrypt('diw84lfnd74jdms6');
    $encrypted = $encryptor->encrypt('My deepest secrets');
} catch(Zaphyr\Encrypt\Exception\EncryptException $exception) {
    //
}
```

### Encrypting without serialization

Encrypted values are passed via serialize during encryption, which enables the encryption of objects and arrays.
If you want to encrypt values without serialization, you can use the `encryptString()` method:

```php
$encryptor->encryptString('My deepest secrets');
```

## Decrypting

You may also want to decrypt values using the decrypt method. If the variable cannot be decrypted or the `MAC`
is invalid, a `DecryptException` is thrown:

```php
try {
    $encryptor = new Zaphyr\Encrypt\Encrypt('diw84lfnd74jdms6');
    $decrypted = $encryptor->decrypt('eyJpdiI[…]0=');
} catch(Zaphyr\Encrypt\Exception\DecryptException $exception) {
    //
}
```

### Decrypting without serialization

If you want to decrypt values without serialization, you can use the `decryptString()` method:

```php
$encryptor->decryptString('eyJpdiI[…]0=');
```
