---
title: Exceptions
description: Understanding error handling in the Immutable package
---

## Exception Hierarchy

All exceptions extend from `ImmutableException`, which itself extends PHP's `RuntimeException`:

```
RuntimeException
└── ImmutableException (abstract)
    ├── PropertyDoesNotExistException
    ├── ReadOnlyPropertyException
    ├── InvalidTypeException
    └── UnsupportedTypeException
```

## ImmutableException

The abstract base class for all package exceptions. Use this for catch-all handling:

```php
use Cline\Immutable\Exceptions\ImmutableException;

try {
    $updated = $user->mutate(['invalid' => 'data']);
} catch (ImmutableException $e) {
    // Handles any mutation error
    log_error($e->getMessage());
}
```

## PropertyDoesNotExistException

Thrown when attempting to mutate a property that doesn't exist on the object.

### Factory Method

```php
PropertyDoesNotExistException::forProperty(
    string $property,
    string $class
): self
```

### Example

```php
use Cline\Immutable\Exceptions\PropertyDoesNotExistException;

$user = new UserData('John', 'john@example.com', 30);

try {
    $user->mutate(['nonexistent' => 'value']);
} catch (PropertyDoesNotExistException $e) {
    $e->getMessage();
    // 'Property "nonexistent" does not exist on class "UserData".'
}
```

### When It's Thrown

- Passing a property name that doesn't exist on the object
- Typos in property names
- Attempting to set properties from a parent class that don't exist

## ReadOnlyPropertyException

Thrown when attempting to mutate a property that is marked as read-only and cannot be changed.

### Factory Method

```php
ReadOnlyPropertyException::forProperty(
    string $property,
    string $class
): self
```

### Example

```php
use Cline\Immutable\Exceptions\ReadOnlyPropertyException;

try {
    $user->mutate(['id' => 'new-id']);
} catch (ReadOnlyPropertyException $e) {
    $e->getMessage();
    // 'Property "id" on class "UserData" is read-only and cannot be mutated.'
}
```

### When It's Thrown

- Attempting to mutate a property that has been explicitly marked as immutable
- Trying to change properties that should never change after construction

## InvalidTypeException

Thrown when the value type doesn't match the property's declared type.

### Factory Method

```php
InvalidTypeException::mismatch(
    string $property,
    string $expectedType,
    mixed $actualValue
): self
```

### Example

```php
use Cline\Immutable\Exceptions\InvalidTypeException;

$user = new UserData('John', 'john@example.com', 30);

try {
    $user->mutate(['age' => 'thirty']);
} catch (InvalidTypeException $e) {
    $e->getMessage();
    // 'Property "age" expects type "int", got "string".'
}
```

### When It's Thrown

- Passing a string where int is expected
- Passing wrong class type for object properties
- Passing wrong enum case type
- Null for non-nullable properties

### Type Validation Rules

| Property Type | Valid Values |
|--------------|--------------|
| `int` | Integers only |
| `float` | Floats and integers |
| `string` | Strings only |
| `bool` | Booleans only |
| `array` | Arrays only |
| `object` | Any object |
| `mixed` | Any value |
| `?type` | Type or null |
| Class name | Instance of that class |

Union and intersection types skip validation.

## UnsupportedTypeException

Thrown when attempting to mutate an unsupported value type.

### Factory Method

```php
UnsupportedTypeException::forValue(mixed $value): self
```

### Example

```php
use Cline\Immutable\Exceptions\UnsupportedTypeException;
use Cline\Immutable\Mutator;

try {
    // This would fail if passed a non-object/non-array
    Mutator::mutate('string', ['key' => 'value']);
} catch (UnsupportedTypeException $e) {
    $e->getMessage();
    // 'Cannot mutate value of type "string". Supported types: object, array.'
}
```

### Supported Types

The Mutator supports:
- Arrays
- Objects (any type)
- stdClass

It does not support scalar types (string, int, float, bool) or resources.

## Error Handling Patterns

### Specific Exception Handling

```php
use Cline\Immutable\Exceptions\PropertyDoesNotExistException;
use Cline\Immutable\Exceptions\InvalidTypeException;

try {
    $updated = $user->mutate($changes);
} catch (PropertyDoesNotExistException $e) {
    // Handle unknown property
    throw new ValidationException("Unknown field: " . $e->getMessage());
} catch (InvalidTypeException $e) {
    // Handle type error
    throw new ValidationException("Invalid type: " . $e->getMessage());
}
```

### Validation Before Mutation

For user input, validate before mutating:

```php
$validated = $request->validate([
    'name' => 'string|max:255',
    'age' => 'integer|min:0',
]);

// Now safe to mutate
$updated = $user->mutate($validated);
```

### Logging Errors

```php
use Cline\Immutable\Exceptions\ImmutableException;

try {
    $updated = $user->mutate($data);
} catch (ImmutableException $e) {
    Log::warning('Mutation failed', [
        'class' => get_class($user),
        'data' => $data,
        'error' => $e->getMessage(),
    ]);
    throw $e;
}
```
