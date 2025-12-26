---
title: Lazy Assertions
description: Collect multiple validation errors before throwing.
---

Lazy assertions collect multiple validation errors before throwing, allowing you to report all validation failures at once.

## Why Lazy Assertions?

Traditional assertions fail on the first error:

```php
Assertion::notEmpty($name);     // Fails here
Assertion::email($email);       // Never checked
Assertion::integer($age);       // Never checked
```

Lazy assertions collect all errors:

```php
Assert::lazy()
    ->that($name, 'name')->notEmpty()
    ->that($email, 'email')->email()
    ->that($age, 'age')->integer()
    ->verifyNow(); // Throws with all errors
```

## Basic Usage

```php
use Cline\Assert\Assert;

Assert::lazy()
    ->that($email, 'email')->notEmpty()->email()
    ->that($name, 'name')->notEmpty()->minLength(2)
    ->that($age, 'age')->integer()->greaterOrEqualThan(18)
    ->verifyNow();
```

## Form Validation

### User Registration

```php
$errors = [];

try {
    Assert::lazy()
        ->that($data['username'] ?? null, 'username')
            ->notNull('Username is required')
            ->notEmpty('Username cannot be empty')
            ->minLength(3, 'Username must be at least 3 characters')
        ->that($data['email'] ?? null, 'email')
            ->notNull('Email is required')
            ->email('Invalid email format')
        ->that($data['password'] ?? null, 'password')
            ->notNull('Password is required')
            ->minLength(8, 'Password must be at least 8 characters')
        ->verifyNow();
} catch (LazyAssertionException $e) {
    foreach ($e->getErrorExceptions() as $error) {
        $errors[$error->getPropertyPath()] = $error->getMessage();
    }
}
```

## API Request Validation

```php
public function validateRequest(array $data): void
{
    Assert::lazy()
        ->that($data['action'] ?? null, 'action')
            ->notNull('Action is required')
            ->inArray(['create', 'update', 'delete'], 'Invalid action')
        ->that($data['resource_id'] ?? null, 'resource_id')
            ->notNull('Resource ID is required')
            ->uuid('Invalid resource ID format')
        ->verifyNow();
}
```

## Configuration Validation

```php
Assert::lazy()
    ->that($config['app_name'], 'app_name')
        ->notEmpty('App name is required')
    ->that($config['environment'], 'environment')
        ->inArray(['development', 'staging', 'production'])
    ->that($config['debug'], 'debug')
        ->boolean()
    ->verifyNow();
```

## tryAll() Mode

By default, lazy assertions stop validating a field after the first failure. Use `tryAll()` to validate all assertions:

### Default Behavior

```php
Assert::lazy()
    ->that($age, 'age')
        ->integer()        // Fails here for "abc"
        ->greaterThan(0)   // Not checked
    ->verifyNow();
```

### tryAll() Mode

```php
Assert::lazy()
    ->tryAll()
    ->that($password, 'password')
        ->string()
        ->minLength(8)          // Check all length requirements
        ->regex('/[A-Z]/')      // Check all complexity requirements
        ->regex('/[a-z]/')
        ->regex('/[0-9]/')
    ->verifyNow(); // Reports ALL failures
```

## Error Handling

### Catching Errors

```php
use Cline\Assert\LazyAssertionException;

try {
    Assert::lazy()
        ->that($email, 'email')->email()
        ->that($age, 'age')->integer()
        ->verifyNow();
} catch (LazyAssertionException $e) {
    foreach ($e->getErrorExceptions() as $error) {
        echo $error->getPropertyPath() . ': ' . $error->getMessage() . "\n";
    }
}
```

### JSON API Error Response

```php
try {
    Assert::lazy()
        ->that($request['email'], 'email')->email()
        ->verifyNow();
} catch (LazyAssertionException $e) {
    $errors = array_map(function($error) {
        return [
            'field' => $error->getPropertyPath(),
            'message' => $error->getMessage(),
        ];
    }, $e->getErrorExceptions());

    return response()->json(['errors' => $errors], 422);
}
```

## Nested Property Paths

```php
Assert::lazy()
    ->that($user['email'], 'user.email')->email()
    ->that($user['address']['city'], 'user.address.city')->notEmpty()
    ->that($user['address']['zip'], 'user.address.zip')->regex('/^\d{5}$/')
    ->verifyNow();
```

## Best Practices

### Always Use Property Paths

```php
Assert::lazy()
    ->that($email, 'email')->email()
    ->that($age, 'age')->integer()
    ->verifyNow();
```

### Use Meaningful Property Paths

```php
Assert::lazy()
    ->that($email, 'contact_email')->email()
    ->that($phone, 'contact_phone')->e164()
    ->verifyNow();
```

### Separate Validation from Business Logic

```php
public function createUser(array $data): User
{
    $this->validateUserData($data);
    return User::create($data);
}

private function validateUserData(array $data): void
{
    Assert::lazy()
        ->that($data['email'], 'email')->email()
        ->verifyNow();
}
```

## When to Use

Use lazy assertions when:
- Validating user input (forms, APIs)
- Need to report all errors at once
- Better UX is important

Avoid when:
- Performance-critical code
- Internal validation
- Only one field to validate

## Next Steps

- [Assertion Chains](/assert/assertion-chains/) - Single-field validation
- [Getting Started](/assert/getting-started/) - Basic concepts
- [Custom Assertions](/assert/custom-assertions/) - Custom validation rules
