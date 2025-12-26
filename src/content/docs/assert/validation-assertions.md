---
title: Validation Assertions
description: Check common data formats like emails, URLs, UUIDs, and IP addresses.
---

Validation assertions check common data formats like emails, URLs, UUIDs, and IP addresses.

## Available Assertions

### email()

Assert that a value is a valid email address.

```php
use Cline\Assert\Assertions\Assertion;

Assertion::email('user@example.com');
Assertion::email($emailAddress, 'Invalid email address');
```

### url()

Assert that a value is a valid URL.

```php
Assertion::url('https://example.com');
Assertion::url($websiteUrl, 'Invalid URL format');
```

### uuid()

Assert that a value is a valid UUID.

```php
Assertion::uuid('550e8400-e29b-41d4-a716-446655440000');
Assertion::uuid($id, 'Invalid UUID format');
```

### ip()

Assert that a value is a valid IPv4 or IPv6 address.

```php
Assertion::ip('192.168.1.1');
Assertion::ip('2001:0db8:85a3:0000:0000:8a2e:0370:7334');
```

### ipv4()

Assert that a value is a valid IPv4 address.

```php
Assertion::ipv4('192.168.1.1');
```

### ipv6()

Assert that a value is a valid IPv6 address.

```php
Assertion::ipv6('2001:0db8:85a3:0000:0000:8a2e:0370:7334');
```

### e164()

Assert that a value is a valid E.164 phone number format.

```php
Assertion::e164('+14155552671');
Assertion::e164($phoneNumber, 'Invalid phone number format');
```

### base64()

Assert that a value is valid base64 encoded data.

```php
Assertion::base64('SGVsbG8gV29ybGQ=');
```

### isJsonString()

Assert that a value is a valid JSON string.

```php
Assertion::isJsonString('{"key":"value"}');
Assertion::isJsonString($jsonData, 'Invalid JSON format');
```

### date()

Assert that a date string matches a specific format.

```php
Assertion::date('2024-01-15', 'Y-m-d');
Assertion::date($dateString, 'Y-m-d H:i:s', 'Invalid date format');
```

## Chaining Validation Assertions

```php
use Cline\Assert\Assert;

Assert::that($email)
    ->string()
    ->notEmpty()
    ->email('Please provide a valid email address');

Assert::that($website)
    ->string()
    ->url('Invalid website URL')
    ->startsWith('https://', 'Website must use HTTPS');
```

## Common Patterns

### User Registration Validation

```php
Assert::lazy()
    ->that($data['email'], 'email')
        ->notEmpty('Email is required')
        ->email('Invalid email address')
    ->that($data['phone'] ?? null, 'phone')
        ->nullOr()->e164('Invalid phone number format')
    ->verifyNow();
```

### URL Validation with Protocol

```php
Assert::that($redirectUrl)
    ->url('Invalid redirect URL')
    ->regex('/^https:\/\//', 'Only HTTPS URLs are allowed');
```

### UUID Primary Key Validation

```php
Assert::that($userId)
    ->notEmpty('User ID is required')
    ->uuid('Invalid user ID format');
```

### Date Range Validation

```php
Assert::that($startDate)
    ->date('Y-m-d', 'Invalid start date format');

Assert::that($endDate)
    ->date('Y-m-d', 'Invalid end date format');
```

## Email Validation

### Email with Domain Check

```php
Assert::that($email)
    ->email('Invalid email format')
    ->endsWith('@company.com', 'Must use company email');
```

### Multiple Email Validation

```php
Assert::thatAll($recipients)
    ->email('All recipients must have valid email addresses');
```

## URL Validation

### HTTPS Only

```php
Assert::that($url)
    ->url('Invalid URL')
    ->startsWith('https://', 'Only HTTPS URLs allowed');
```

### Domain Restriction

```php
Assert::that($callbackUrl)
    ->url()
    ->contains('example.com', 'Callback must be on example.com');
```

## IP Address Validation

### Private IP Range

```php
Assert::that($internalIp)
    ->ipv4()
    ->satisfy(function($ip) {
        return filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE) === false;
    }, 'Must be private IP address');
```

## Best Practices

### Validate Format Before Processing

```php
Assert::that($email)->email();
$user = User::where('email', $email)->first();
```

### Use Specific Validators

```php
// Use built-in validator
Assertion::email($email);

// Instead of generic regex
Assertion::regex($email, '/^[^@]+@[^@]+\.[^@]+$/');
```

## Next Steps

- [String Assertions](/assert/string-assertions/) - String format validation
- [Custom Assertions](/assert/custom-assertions/) - Create custom validators
- [Lazy Assertions](/assert/lazy-assertions/) - Validate multiple fields
