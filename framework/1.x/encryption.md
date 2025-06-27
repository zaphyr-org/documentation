# Encryption

ZAPHYR offers a simple and secure solution for encrypting and decrypting data through
its [Encrypt Repository](/docs/repositories/latest/encrypt). Built on
top of [OpenSSL](https://www.openssl.org/), it supports both `AES-256-CBC` and `AES-128-CBC` encryption ciphers. This
package allows you to safely encrypt strings, arrays, and objects, making it ideal for protecting sensitive information
within your application.

## Configuration

As outlined in the [configuration documentation](/docs/framework/latest/configuration#application-key), a unique
application key is automatically generated during the installation of a fresh ZAPHYR application. This key is stored in
your `.env` file and is used for encrypting and decrypting sensitive data:

```bash
APP_KEY=base64:randomlyGeneratedKeyHere
```

The encryption cipher used is defined by the `APP_CIPHER` environment variable. By default, ZAPHYR uses the
`AES-256-CBC` cipher:

```bash
APP_CIPHER=AES-256-CBC
```

If you prefer to use `AES-128-CBC`, simply update the `APP_CIPHER` variable in your `.env` file and generate a new key
with a length of **16 characters**:

```bash
APP_CIPHER=AES-128-CBC
```

ZAPHYR also supports session encryption via the `SESSION_ENCRYPT` variable. This is enabled by default, ensuring all
session data is encrypted:

```bash
SESSION_ENCRYPT=true
```

To disable session encryption, you can set the value to false. However, for optimal security, we strongly recommend
using `AES-256-CBC` with a 32-character key and keeping session encryption enabled.

## Usage

To use the encryption features, you can inject the `EncryptInterface` into your classes or retrieve it directly from
the service container. This interface provides convenient methods for securely encrypting and decrypting data:

```php
$encryptor = $this->container->get(Zaphyr\Encrypt\Contracts\EncryptInterface::class);
```

### Encrypting

To encrypt data, use the `encrypt()` method for arrays, objects, and other serializable types. If you're working with
plain strings, you can use `encryptString()`, which avoids serialization overhead:

```php
try {
    $encryptedData = $encryptor->encrypt(['foo' => 'bar']);
    $encryptedString = $encryptor->encryptString('Hello World!');
} catch (Zaphyr\Encrypt\Exceptions\EncryptException $e) {
    // Handle exception
}
```

### Decrypting

To decrypt encrypted values, use the `decrypt()` method for general data types or `decryptString()` for strings. If the
data cannot be decrypted or the message authentication code (MAC) is invalid, a `DecryptException` will be thrown:

```php
try {
    $decryptedData = $encryptor->decrypt($encryptedData);
    $decryptedString = $encryptor->decryptString($encryptedString);
} catch (Zaphyr\Encrypt\Exceptions\DecryptException $e) {
    // Handle exception
}
```
