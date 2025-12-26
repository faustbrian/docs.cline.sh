---
title: Getting Started
description: Install and start using Assert for validation in PHP 8.4+.
---

Assert is a comprehensive assertion library for PHP 8.4+ that enables robust validation and preconditions throughout your code.

## Installation

```bash
composer require cline/assert
```

## Basic Usage

The library provides four main ways to perform assertions:

### 1. Static Method Calls

Use `Assertion` class directly for immediate validation:

```php
use Cline\Assert\Assertions\Assertion;

Assertion::string($value);
Assertion::notEmpty($value);
Assertion::minLength($value, 3);
```

### 2. Assertion Chains

Use `Assert::that()` for fluent, chainable assertions:

```php
use Cline\Assert\Assert;

Assert::that($email)
    ->notEmpty()
    ->email();

Assert::that($password)
    ->string()
    ->minLength(8)
    ->maxLength(100);
```

### 3. Expect API (Jest/Pest-style)

Use `expect()` for test-friendly, chainable expectations:

```php
use function Cline\Assert\expect;

expect($value)->toBeString();
expect($count)->toBeInt();
expect($result)->toBe(42);
expect($items)->toHaveCount(3);
expect($value)->not->toBeNull();
```

### 4. Lazy Assertions

Collect multiple validation errors before throwing:

```php
use Cline\Assert\Assert;

Assert::lazy()
    ->that($email, 'email')->email()
    ->that($age, 'age')->integer()->min(18)
    ->that($name, 'name')->notEmpty()->minLength(2)
    ->verifyNow(); // Throws exception with all errors
```

## Exception Handling

All assertions throw `AssertionFailedException` when validation fails:

```php
use Cline\Assert\Assertions\Assertion;
use Cline\Assert\Exceptions\AssertionFailedException;

try {
    Assertion::integer($value);
} catch (AssertionFailedException $e) {
    echo $e->getMessage();
    echo $e->getPropertyPath();
    echo $e->getValue();
    echo $e->getConstraints();
}
```

## Custom Messages

All assertions accept optional custom error messages:

```php
Assertion::notEmpty($username, 'Username is required');
Assertion::email($email, 'Please provide a valid email address');

Assert::that($age)
    ->integer('Age must be a number')
    ->min(18, 'You must be at least 18 years old');
```

### Message Placeholders

Custom messages support sprintf-style placeholders:

```php
Assertion::minLength($username, 3, 'Username must be at least %2$s characters. Got: %s');
// => "Username must be at least 3 characters. Got: ab"

Assertion::range($age, 18, 65, 'Age must be between %2$s and %3$s. Got: %s');
// => "Age must be between 18 and 65. Got: 17"
```

## Property Paths

Use property paths to identify which field failed validation:

```php
Assertion::string($user->email, null, 'user.email');

Assert::that($user->email, null, 'user.email')
    ->notEmpty()
    ->email();
```

## Next Steps

- [Expect API](/assert/expect-api/) - Jest/Pest-style fluent expectations
- [Assertion Chains](/assert/assertion-chains/) - Fluent API usage
- [Lazy Assertions](/assert/lazy-assertions/) - Batch validation patterns
- [String Assertions](/assert/string-assertions/) - String validation
