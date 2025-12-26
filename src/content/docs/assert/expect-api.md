---
title: Expect API
description: Jest/Pest-style fluent expectations for testing and validation.
---

The `expect()` API provides a Jest/Pest-style fluent interface for assertions, ideal for testing and validation with a natural, readable syntax.

## Installation

```php
use function Cline\Assert\expect;
```

## Core Expectations

### Value Equality

```php
expect(42)->toBe(42);
expect('hello')->toBe('hello');
expect($result)->toEqual(42);
expect(42)->toStrictEqual(42);
```

### Null Checks

```php
expect(null)->toBeNull();
expect($value)->not->toBeNull();
expect('hello')->toBeDefined();
expect(null)->toBeUndefined();
expect(null)->toBeNullable('string');
```

### Boolean Checks

```php
expect(true)->toBeTrue();
expect(false)->toBeFalse();
expect(1)->toBeTruthy();
expect(0)->toBeFalsy();
```

### Empty Checks

```php
expect('')->toBeEmpty();
expect([])->toBeEmpty();
expect('hello')->not->toBeEmpty();
```

## Type Expectations

```php
expect($value)->toBeString();
expect($count)->toBeInt();
expect($price)->toBeFloat();
expect($active)->toBeBool();
expect($items)->toBeArray();
expect($user)->toBeObject();
expect($callback)->toBeCallable();
expect($collection)->toBeIterable();
expect($number)->toBeNumeric();
```

## Numeric Comparisons

```php
expect(10)->toBeGreaterThan(5);
expect(10)->toBeGreaterThanOrEqual(10);
expect(5)->toBeLessThan(10);
expect(5)->toBeLessThanOrEqual(5);
expect(5)->toBeBetween(1, 10);
expect(3.14159)->toEqualWithDelta(3.14, 0.01);
expect(42)->toBePositive();
expect(-10)->toBeNegative();
expect(4)->toBeEven();
expect(7)->toBeOdd();
expect(10)->toBeDivisibleBy(5);
```

## String Expectations

```php
expect('hello world')->toStartWith('hello');
expect('hello world')->toEndWith('world');
expect('hello world')->toContain('lo wo');
expect('test@example.com')->toMatch('/^.+@.+\..+$/');
expect('hello')->toHaveLength(5);
expect('abc')->toBeAlpha();
expect('abc123')->toBeAlphaNumeric();
expect('hello_world')->toBeSnakeCase();
expect('hello-world')->toBeKebabCase();
expect('helloWorld')->toBeCamelCase();
```

## Collection Expectations

```php
expect([1, 2, 3])->toHaveCount(3);
expect(['name' => 'John'])->toHaveKey('name');
expect(['a' => 1, 'b' => 2])->toHaveKeys(['a', 'b']);
expect([1, 2, 3])->toContain(2);
expect(2)->toBeIn([1, 2, 3]);
expect([3, 2, 1])->toEqualCanonicalizing([1, 2, 3]);
expect([1, 2])->toBeSubsetOf([1, 2, 3, 4, 5]);
expect([1, 2, 3])->toHaveUniqueValues();
expect([1, 2, 3])->toBeSorted();
```

## Object Expectations

```php
expect($user)->toHaveProperty('name');
expect($user)->toHaveProperties(['name', 'email']);
expect($user)->toHaveMethod('save');
expect($iterator)->toBeInstanceOf(ArrayIterator::class);
expect($obj)->toMatchObject(['name' => 'John']);
```

## Format Validations

```php
expect('test@example.com')->toBeEmail();
expect('https://example.com')->toBeUrl();
expect('550e8400-e29b-41d4-a716-446655440000')->toBeUuid();
expect('{"key": "value"}')->toBeJson();
expect('2024-01-15')->toBeValidDate('Y-m-d');
```

## File System Expectations

```php
expect('/path/to/file.txt')->toBeFile();
expect('/path/to/file.txt')->toBeReadableFile();
expect('/path/to/directory')->toBeReadableDirectory();
expect('/path/to/directory')->toBeWritableDirectory();
```

## Exception Expectations

```php
expect(fn() => throw new RuntimeException())->toThrow();
expect(fn() => throw new InvalidArgumentException())
    ->toThrow(InvalidArgumentException::class);
expect(fn() => throw new Exception('Custom error'))
    ->toThrow(Exception::class, 'Custom error');
expect(fn() => $user->save())->not->toThrow();
```

## Date/Time Expectations

```php
expect('2024-01-01')->toBeBefore('2024-12-31');
expect('2024-12-31')->toBeAfter('2024-01-01');
expect(date('Y-m-d'))->toBeToday();
```

## Performance Expectations

```php
$fastOperation = fn () => 1 + 1;
expect($fastOperation)->toCompleteWithin(100); // milliseconds
```

## Negation Modifier

Use `->not` to negate any expectation:

```php
expect(42)->not->toBe(43);
expect('hello')->not->toBeNull();
expect([])->not->toBeEmpty();
expect(42)->not->toBeString();
```

## Each Modifier

Apply expectations to each element in a collection:

```php
expect([1, 2, 3])->each->toBeInt();
expect(['a', 'b', 'c'])->each->toBeString();
expect([1, 2, 3])->each->not->toBeString();

expect(['a' => 1, 'b' => 2])->each(function($expectation, $key) {
    $expectation->toBeInt();
    expect($key)->toBeString();
});
```

## And Modifier

Chain multiple values in a single expression:

```php
expect($user)->toHaveProperty('email')
    ->and($admin)->toHaveProperty('email')
    ->and($guest)->not->toHaveProperty('email');
```

## Conditional Modifiers

### When Modifier

```php
expect($user)->when(
    $isAdmin,
    fn($exp) => $exp->toHaveProperty('adminLevel')
);
```

### Unless Modifier

```php
expect($user)->unless(
    $isGuest,
    fn($exp) => $exp->toHaveProperty('subscription')
);
```

## Sequence Modifier

Apply different expectations to each element in order:

```php
expect([1, 'test', 3.14])->sequence(
    fn($e) => $e->toBeInt(),
    fn($e) => $e->toBeString(),
    fn($e) => $e->toBeFloat()
);
```

## JSON Modifier

Parse JSON and continue chaining:

```php
expect('{"name":"John","age":30}')
    ->json()
    ->toHaveKey('name')
    ->toHaveKey('age');
```

## Custom Validation

### toSatisfy()

```php
expect($age)->toSatisfy(fn($v) => $v > 18);
expect($user)->toSatisfy(function($u) {
    return $u->age > 18 && $u->verified === true;
});
```

### toMatchSchema()

```php
expect($user)->toMatchSchema([
    'type' => 'object',
    'properties' => [
        'name' => ['type' => 'string'],
        'age' => ['type' => 'integer', 'minimum' => 0],
    ],
    'required' => ['name', 'email'],
]);
```

## Asymmetric Matchers

Match partial patterns without requiring exact equality:

```php
expect(['name' => 'John', 'age' => 30])->toEqual([
    'name' => expect()->any('string'),
    'age' => expect()->any('int'),
]);

expect(['id' => 123, 'data' => 'test'])->toEqual([
    'id' => expect()->anything(),
    'data' => expect()->anything(),
]);

expect(['message' => 'Error: Invalid input'])->toEqual([
    'message' => expect()->stringContaining('Error'),
]);

expect(['a' => 1, 'b' => 2, 'c' => 3])->toEqual(
    expect()->arrayContaining(['a' => 1, 'c' => 3])
);
```

## Soft Assertions

Collect multiple assertion failures before throwing:

```php
expect(5)->soft->toBeGreaterThan(10);
expect('hello')->soft->toBeInt();
expect([])->soft->toHaveCount(5);

Expectation::assertSoft(); // Throws with all 3 errors
```

## OR Operator

Value must match at least one group:

```php
expect($value)
    ->or
    ->toBeString()
    ->or
    ->toBeInt()
    ->or
    ->toBeNull();

expect($input)
    ->or
    ->toBeString()
    ->toHaveLength(10)
    ->or
    ->toBeInt()
    ->toBePositive();
```

## XOR Operator

Exactly one group must pass:

```php
expect($configValue)
    ->xor
    ->toBeBoolean()
    ->xor
    ->toBeString()
    ->xor
    ->toBeNumeric();
```

## Snapshot Testing

```php
use Cline\Assert\Snapshots\SnapshotManager;

SnapshotManager::setSnapshotDirectory('__snapshots__');

expect($data)->toMatchSnapshot('user-data');
expect($data)->toMatchInlineSnapshot($expected);
```

## Debugging Helpers

```php
expect($data)->dd();
expect($data)->ddWhen($isDebug);
expect($data)->ddUnless($isProduction);
expect($data)->ray();
```

## Next Steps

- [Getting Started](/assert/getting-started/) - Basic assertion concepts
- [Assertion Chains](/assert/assertion-chains/) - Fluent API usage
- [Custom Assertions](/assert/custom-assertions/) - Create custom rules
