---
title: Type Assertions
description: Verify that values match expected PHP types.
---

Type assertions verify that values match expected PHP types.

## Available Assertions

### integer()

Assert that a value is a PHP integer.

```php
use Cline\Assert\Assertions\Assertion;

Assertion::integer($count);
Assertion::integer($id, 'ID must be an integer');
```

### float()

Assert that a value is a PHP float.

```php
Assertion::float($price);
Assertion::float($temperature, 'Temperature must be a float');
```

### string()

Assert that a value is a string.

```php
Assertion::string($name);
Assertion::string($email, 'Email must be a string');
```

### boolean()

Assert that a value is a PHP boolean.

```php
Assertion::boolean($isActive);
Assertion::boolean($flag, 'Flag must be a boolean');
```

### numeric()

Assert that a value is numeric (int, float, or numeric string).

```php
Assertion::numeric($value);
Assertion::numeric($amount, 'Amount must be numeric');
```

### integerish()

Assert that a value is an integer or can be cast to an integer.

```php
Assertion::integerish('123');  // Passes
Assertion::integerish(123);    // Passes
Assertion::integerish('123.0'); // Fails
```

### scalar()

Assert that a value is a PHP scalar.

```php
Assertion::scalar($value);
Assertion::scalar($input, 'Input must be scalar');
```

### isArray()

Assert that a value is an array.

```php
Assertion::isArray($items);
Assertion::isArray($config, 'Config must be an array');
```

### isObject()

Assert that a value is an object.

```php
Assertion::isObject($instance);
Assertion::isObject($model, 'Model must be an object');
```

### isResource()

Assert that a value is a resource.

```php
$file = fopen('file.txt', 'r');
Assertion::isResource($file);
```

### isCallable()

Assert that a value is callable.

```php
Assertion::isCallable($callback);
Assertion::isCallable(fn() => true);
Assertion::isCallable('strlen');
Assertion::isCallable([$object, 'method']);
```

## Chaining Type Assertions

```php
use Cline\Assert\Assert;

Assert::that($age)
    ->integer()
    ->greaterThan(0);

Assert::that($name)
    ->string()
    ->notEmpty();
```

## Common Patterns

### Input Validation

```php
Assert::that($userId)
    ->integer('User ID must be an integer')
    ->greaterThan(0, 'User ID must be positive');
```

### Numeric Type Checking

```php
// Strict integer check
Assert::that($count)->integer();

// Flexible numeric check
Assert::that($amount)->numeric();

// Integer-like check
Assert::that($id)->integerish();
```

## Type Coercion vs Strict Types

### Strict Type Checking

```php
Assertion::integer(123);     // Pass
Assertion::integer('123');   // Fail
Assertion::integer(123.0);   // Fail
```

### Flexible Type Checking

```php
Assertion::numeric(123);     // Pass
Assertion::numeric('123');   // Pass
Assertion::numeric(123.45);  // Pass

Assertion::integerish(123);      // Pass
Assertion::integerish('123');    // Pass
Assertion::integerish(123.0);    // Fail
```

## Best Practices

### Be Specific

```php
// Too loose
Assertion::scalar($age);

// Specific
Assertion::integer($age);
```

### Chain Related Assertions

```php
Assert::that($email)
    ->string()
    ->notEmpty()
    ->email();
```

### Use Integerish for Flexible Input

```php
Assert::that($_POST['quantity'])
    ->integerish()
    ->greaterThan(0);
```

## Next Steps

- [Numeric Assertions](/assert/numeric-assertions/) - Number validation
- [Object Assertions](/assert/object-assertions/) - Object validation
- [Array Assertions](/assert/array-assertions/) - Array validation
