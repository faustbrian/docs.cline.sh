---
title: Assertion Chains
description: Fluent interface for validating values with multiple assertions.
---

Assertion chains provide a fluent interface for validating values with multiple assertions.

## Basic Chain Usage

Instead of multiple static calls:

```php
Assertion::string($email);
Assertion::notEmpty($email);
Assertion::email($email);
```

Use a fluent chain:

```php
Assert::that($email)
    ->string()
    ->notEmpty()
    ->email();
```

## Creating Chains

### Basic Chain

```php
use Cline\Assert\Assert;

Assert::that($value)
    ->assertion1()
    ->assertion2()
    ->assertion3();
```

### With Custom Message

```php
Assert::that($age, 'Age must be valid')
    ->integer()
    ->greaterOrEqualThan(18);
```

### With Property Path

```php
Assert::that($user->email, null, 'user.email')
    ->notEmpty()
    ->email();
```

## Available Modifiers

### nullOr() - Allow Null Values

Skip all subsequent assertions if the value is null:

```php
Assert::thatNullOr($middleName)
    ->string()
    ->minLength(2)
    ->maxLength(50);

// Equivalent to:
if ($middleName !== null) {
    Assert::that($middleName)
        ->string()
        ->minLength(2);
}
```

### all() - Validate Array Elements

Validate every element in an array:

```php
Assert::thatAll($emailList)->email();

Assert::thatAll($userIds)
    ->integer()
    ->greaterThan(0);
```

## Common Patterns

### String Validation

```php
Assert::that($username)
    ->string()
    ->notEmpty()
    ->minLength(3)
    ->maxLength(20)
    ->regex('/^[a-z0-9_]+$/');
```

### Number Validation

```php
Assert::that($price)
    ->float()
    ->greaterThan(0, 'Price must be positive')
    ->lessThan(1000000, 'Price too high');
```

### Email Validation

```php
Assert::that($email)
    ->string()
    ->notEmpty('Email is required')
    ->email('Invalid email format')
    ->maxLength(255, 'Email too long');
```

### Password Validation

```php
Assert::that($password)
    ->string()
    ->minLength(8, 'Password must be at least 8 characters')
    ->regex('/[A-Z]/', 'Must contain uppercase letter')
    ->regex('/[a-z]/', 'Must contain lowercase letter')
    ->regex('/[0-9]/', 'Must contain number');
```

### Object Validation

```php
Assert::that($user)
    ->notNull('User not found')
    ->isObject()
    ->isInstanceOf(User::class)
    ->propertyExists('email');
```

### Array Validation

```php
Assert::that($items)
    ->isArray()
    ->notEmpty('Items cannot be empty')
    ->minCount(1)
    ->maxCount(100);
```

## Using nullOr()

### Optional Fields

```php
// Required field
Assert::that($email)->notEmpty()->email();

// Optional field (can be null)
Assert::thatNullOr($phoneNumber)
    ->string()
    ->e164('Invalid phone format');
```

### Configuration Values

```php
Assert::thatNullOr($config['timeout'])
    ->integer()
    ->greaterThan(0);
```

## Using all()

### Validate Array of Values

```php
Assert::thatAll($recipientEmails)
    ->email('All recipients must have valid emails');

Assert::thatAll($quantities)
    ->integer()
    ->greaterThan(0);
```

### With Type Checks

```php
Assert::thatAll($tags)
    ->string()
    ->notEmpty()
    ->minLength(2)
    ->maxLength(30);
```

## Error Messages

### Default Messages

```php
Assert::that($email)
    ->notEmpty()  // "Value is required"
    ->email();    // "Value is not a valid email"
```

### Custom Messages per Assertion

```php
Assert::that($password)
    ->notEmpty('Password is required')
    ->minLength(8, 'Password must be at least 8 characters');
```

### Default Message for Chain

```php
Assert::that($username, 'Username is invalid')
    ->notEmpty()
    ->minLength(3);
```

## Best Practices

### Order Assertions Logically

```php
// Good order: type → null/empty → format → constraints
Assert::that($email)
    ->string()           // 1. Type check first
    ->notEmpty()         // 2. Then null/empty
    ->email()            // 3. Then format
    ->maxLength(255);    // 4. Finally constraints
```

### Fail Fast with Type Checks

```php
Assert::that($count)
    ->integer()          // Ensures numeric operations work
    ->greaterThan(0);
```

### Group Related Validations

```php
Assert::that($name)
    ->string()
    ->notEmpty()
    ->minLength(2)
    ->maxLength(100)
    ->regex('/^[\p{L}\s]+$/u', 'Name can only contain letters');
```

## Common Mistakes

### Wrong Order

```php
// Bad - length check before type check
Assert::that($name)
    ->minLength(3)
    ->string();

// Good - type check first
Assert::that($name)
    ->string()
    ->minLength(3);
```

## Next Steps

- [Lazy Assertions](/assert/lazy-assertions/) - Validate multiple fields
- [Custom Assertions](/assert/custom-assertions/) - Add custom rules
- [Getting Started](/assert/getting-started/) - Core concepts
