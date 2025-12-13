---
title: Null and Empty Assertions
description: Validate null values, empty values, and blank strings.
---

Assertions for validating null values, empty values, and blank strings.

## Available Assertions

### null()

Assert that a value is null.

```php
use Cline\Assert\Assertions\Assertion;

Assertion::null($value);
Assertion::null($optionalField, 'Field must be null');
```

### notNull()

Assert that a value is not null.

```php
Assertion::notNull($value);
Assertion::notNull($user, 'User is required');
```

### notEmpty()

Assert that a value is not empty (using `empty()` check).

```php
Assertion::notEmpty($value);
Assertion::notEmpty($name, 'Name is required');
```

### noContent()

Assert that a value is empty (using `empty()` check).

```php
Assertion::noContent($value);
Assertion::noContent($deletedField, 'Field must be empty');
```

### notBlank()

Assert that a value is not blank (not empty string or whitespace-only).

```php
Assertion::notBlank($value);
Assertion::notBlank($description, 'Description cannot be blank');
```

## Understanding Empty Checks

### What `empty()` Considers Empty

```php
Assertion::noContent('');       // empty string
Assertion::noContent(0);        // integer zero
Assertion::noContent(null);     // null
Assertion::noContent(false);    // boolean false
Assertion::noContent([]);       // empty array
```

## Null vs Empty vs Blank

### null() - Strict Null Check

```php
Assertion::null(null);      // Pass
Assertion::null('');        // Fail - empty string, not null
```

### noContent() - PHP Empty Check

```php
Assertion::noContent(null);     // Pass
Assertion::noContent('');       // Pass
Assertion::noContent(0);        // Pass
```

### notBlank() - Non-Whitespace String

```php
Assertion::notBlank('hello');   // Pass
Assertion::notBlank(' ');       // Fail - whitespace only
Assertion::notBlank('');        // Fail - empty string
```

## Chaining Null/Empty Assertions

```php
use Cline\Assert\Assert;

Assert::that($username)
    ->notNull('Username is required')
    ->notEmpty('Username cannot be empty')
    ->string();

Assert::that($description)
    ->string()
    ->notBlank('Description cannot be blank');
```

## Common Patterns

### Required Field Validation

```php
Assert::that($email)
    ->notNull('Email is required')
    ->notEmpty('Email cannot be empty')
    ->notBlank('Email cannot be blank')
    ->email('Invalid email format');
```

### Optional Field Validation

```php
Assert::thatNullOr($phoneNumber)
    ->string()
    ->e164('Invalid phone format');
```

### Form Input Validation

```php
Assert::lazy()
    ->that($form['name'] ?? null, 'name')->notNull()->notBlank()
    ->that($form['email'] ?? null, 'email')->notNull()->notBlank()->email()
    ->verifyNow();
```

## Using nullOr()

```php
// Allow null OR validate if not null
Assert::thatNullOr($middleName)
    ->string()
    ->minLength(2);
```

## String Whitespace Handling

```php
// notEmpty allows whitespace
Assertion::notEmpty(' ');      // Pass

// notBlank rejects whitespace
Assertion::notBlank(' ');      // Fail
```

## Best Practices

### Check Null First

```php
Assert::that($user)
    ->notNull('User not found')
    ->isObject();
```

### Use notBlank for User Input

```php
Assert::that($comment)
    ->notBlank('Comment cannot be empty');
```

### Combine with Type Checks

```php
Assert::that($name)
    ->notNull()
    ->string()
    ->notBlank();
```

## Next Steps

- [Type Assertions](/assert/type-assertions/) - Type validation
- [String Assertions](/assert/string-assertions/) - String validation
- [Assertion Chains](/assert/assertion-chains/) - Using nullOr()
