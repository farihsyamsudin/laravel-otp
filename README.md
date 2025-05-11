# Laravel OTP ▲

[![Latest Stable Version](https://poser.pugx.org/ichtrojan/laravel-otp/v/stable)](https://packagist.org/packages/ichtrojan/laravel-otp) [![Total Downloads](https://poser.pugx.org/ichtrojan/laravel-otp/downloads)](https://packagist.org/packages/ichtrojan/laravel-otp) [![License](https://poser.pugx.org/ichtrojan/laravel-otp/license)](https://packagist.org/packages/ichtrojan/laravel-otp)

## Introduction 🖖

This is a simple package to generate and validate OTPs (One Time Passwords). This can be implemented mostly in Authentication.

## Installation 💽

Install via composer

```bash
composer require ichtrojan/laravel-otp
```

Run Migrations

```bash
php artisan migrate
```

## Usage 🧨

>**NOTE**</br>
>Response are returned as objects. You can access its attributes with the arrow operator (`->`)

### Generate OTP

```php
<?php

use Ichtrojan\Otp\Otp;

(new Otp)->generate(string $identifier, string $type, int $length = 4, int $validity = 10, bool $allowMultiple);
```

* `$identifier`: The identity that will be tied to the OTP.
* `$type`: The type of token to be generated, supported types are `numeric` and `alpha_numeric`
* `$length (optional | default = 4)`: The length of token to be generated.
* `$validity (optional | default = 10)`: The validity period of the OTP in minutes.
* `$allowMultiple (optional | default = false)`: If set to true, multiple OTPs can exist for the same identifier at the same time. If false, existing OTPs for the identifier will be deleted before generating a new one.

### Why Use $allowMultiple = true?

The $allowMultiple parameter is especially useful when your mail server might experience delays or when there is a concern that a user may request OTPs multiple times before the first OTP arrives. In situations like this, if the user makes two OTP requests, they might receive the first OTP (which could still be valid), even though they requested the second one. By setting $allowMultiple = true, you ensure that both OTPs will remain active, allowing the user to successfully authenticate with either token.

This approach prevents situations where the first OTP expires before the user has a chance to use it, ensuring a better user experience in case of delays.

#### Sample

```php
<?php

use Ichtrojan\Otp\Otp;

(new Otp)->generate('michael@okoh.co.uk', 'numeric', 6, 15);
```

This will generate a six digit OTP that will be valid for 15 minutes and the success response will be:

```object
{
  "status": true,
  "token": "282581",
  "message": "OTP generated"
}
```

### Validate OTP

```php
<?php

use Ichtrojan\Otp\Otp;

(new Otp)->validate(string $identifier, string $token)
```

* `$identifier`: The identity that is tied to the OTP.
* `$token`: The token tied to the identity.

#### Sample

```php
<?php

use Ichtrojan\Otp\Otp;

(new Otp)->validate('michael@okoh.co.uk', '282581');
```

#### Responses

**On Success**

```object
{
  "status": true,
  "message": "OTP is valid"
}
```

**Does not exist**

```object
{
  "status": false,
  "message": "OTP does not exist"
}
```

**Not Valid***

```object
{
  "status": false,
  "message": "OTP is not valid"
}
```

**Expired**

```object
{
  "status": false,
  "message": "OTP Expired"
}
```

### Check The validity of an OTP

To verify the validity of an OTP without marking it as used, you can use the `isValid` method:

```php
<?php
use Ichtrojan\Otp\Otp;

(new Otp)->isValid(string $identifier, string $token);
```

This will return a boolean value of the validity of the OTP.

### Delete expired tokens
You can delete expired tokens by running the following artisan command:
```bash
php artisan otp:clean
```
You can also add this artisan command to `app/Console/Kernel.php` to automatically clean on scheduled 
```php
<?php

protected function schedule(Schedule $schedule)
{
    $schedule->command('otp:clean')->daily();
}
```

## Contribution

If you find an issue with this package or you have any suggestion please help out. I am not perfect.
