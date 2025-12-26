---
title: Array Assertions
description: Validate arrays, collections, and array-accessible objects.
---

Array assertions validate arrays, collections, and array-accessible objects.

## Available Assertions

### isCountable()

Assert that a value is countable.

```php
use Cline\Assert\Assertions\Assertion;

Assertion::isCountable($array);
Assertion::isCountable($collection, 'Value must be countable');
```

### count()

Assert exact number of elements.

```php
Assertion::count($array, 5);
Assertion::count($items, 3, 'Must have exactly 3 items');
```

### minCount()

Assert minimum number of elements.

```php
Assertion::minCount($array, 1);
Assertion::minCount($options, 2, 'Must provide at least 2 options');
```

### maxCount()

Assert maximum number of elements.

```php
Assertion::maxCount($array, 10);
Assertion::maxCount($tags, 5, 'Maximum 5 tags allowed');
```

### keyExists()

Assert that a key exists.

```php
Assertion::keyExists($array, 'name');
Assertion::keyExists($config, 'database', 'Database configuration is required');
```

### keyNotExists()

Assert that a key does NOT exist.

```php
Assertion::keyNotExists($array, 'legacy_field');
Assertion::keyNotExists($data, 'password', 'Password should not be included');
```

### notEmptyKey()

Assert that a key exists and its value is not empty.

```php
Assertion::notEmptyKey($data, 'name');
Assertion::notEmptyKey($form, 'email', 'Email field cannot be empty');
```

### uniqueValues()

Assert all values are unique.

```php
Assertion::uniqueValues($array);
Assertion::uniqueValues($ids, 'All IDs must be unique');
```

### inArray()

Assert that a value exists in an array.

```php
Assertion::inArray($status, ['draft', 'published', 'archived']);
Assertion::inArray($role, $validRoles, 'Invalid role selected');
```

### notInArray()

Assert that a value does NOT exist in an array.

```php
Assertion::notInArray($username, $bannedNames);
Assertion::notInArray($value, $blacklist, 'This value is not allowed');
```

### choice()

Alias for `inArray()`.

```php
Assertion::choice($color, ['red', 'green', 'blue']);
```

### eqArraySubset()

Assert that an array contains a subset.

```php
$expected = ['name' => 'John', 'active' => true];
Assertion::eqArraySubset($user, $expected);
```

## Chaining Array Assertions

```php
use Cline\Assert\Assert;

Assert::that($array)
    ->isArray()
    ->notEmpty()
    ->minCount(1)
    ->maxCount(10);

Assert::that($status)
    ->string()
    ->inArray(['pending', 'approved', 'rejected']);
```

## Common Patterns

### Required Array Keys

```php
Assert::that($config)
    ->isArray()
    ->keyExists('host')
    ->keyExists('port')
    ->keyExists('database');
```

### Form Validation

```php
Assert::lazy()
    ->that($form, 'form')->isArray()
    ->that($form, 'form')->keyExists('name')
    ->that($form, 'form')->notEmptyKey('name')
    ->verifyNow();
```

### Collection Size Validation

```php
Assert::that($items)
    ->isArray()
    ->minCount(1, 'At least one item is required')
    ->maxCount(100, 'Maximum 100 items allowed');
```

## Validating All Elements

Use the `all()` modifier to validate every element:

```php
Assert::thatAll($tags)->string();
Assert::thatAll($quantities)
    ->integer()
    ->greaterThan(0);
Assert::thatAll($emailList)->email();
```

## Nested Arrays

```php
Assert::that($data)
    ->isArray()
    ->keyExists('user')
    ->keyExists('settings');

Assert::that($data['user'])
    ->isArray()
    ->notEmptyKey('id')
    ->notEmptyKey('name');
```

## Next Steps

- [Type Assertions](/assert/type-assertions/) - Type checking
- [Null and Empty Assertions](/assert/null-empty-assertions/) - Empty array checks
- [Lazy Assertions](/assert/lazy-assertions/) - Validate multiple conditions
