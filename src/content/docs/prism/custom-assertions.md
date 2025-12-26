---
title: Custom Assertions
description: Create and use custom assertion logic beyond strict equality.
---

Prism supports custom assertion logic through a pluggable assertion interface, enabling complex validation rules beyond simple pass/fail checks.

## AssertionInterface

Custom assertions implement the `AssertionInterface`:

```php
<?php

namespace Cline\Prism\Contracts;

interface AssertionInterface
{
    /**
     * Execute the assertion against test data.
     *
     * @param  mixed $data          Data to validate
     * @param  mixed $expectedValid Expected validation result
     * @param  mixed $actualValid   Actual validation result from validator
     * @return bool  True if assertion passes, false otherwise
     */
    public function assert(mixed $data, mixed $expectedValid, mixed $actualValid): bool;

    /**
     * Get human-readable name for this assertion.
     */
    public function getName(): string;

    /**
     * Get failure message when assertion fails.
     *
     * @param  mixed  $data          Data that failed assertion
     * @param  mixed  $expectedValid Expected validation result
     * @param  mixed  $actualValid   Actual validation result
     * @return string Descriptive failure message
     */
    public function getFailureMessage(mixed $data, mixed $expectedValid, mixed $actualValid): string;
}
```

## Built-in Assertions

### StrictEqualityAssertion

Default assertion - validates exact equality:

```php
use Cline\Prism\Assertions\StrictEqualityAssertion;

$assertion = new StrictEqualityAssertion();

// Passes if expectedValid === actualValid
$assertion->assert($data, true, true);   // true
$assertion->assert($data, false, false); // true
$assertion->assert($data, true, false);  // false
```

**Use when**: Exact pass/fail validation required.

### AnyOfAssertion

Validates if actual result matches any of multiple expected values:

```php
use Cline\Prism\Assertions\AnyOfAssertion;

$assertion = new AnyOfAssertion();

// Passes if actualValid matches any value in expectedValid array
$assertion->assert($data, [true, false], true);  // true
$assertion->assert($data, [true, false], false); // true
$assertion->assert($data, [true], false);        // false
```

**Use when**: Multiple outcomes are acceptable (implementation-dependent edge cases).

## Creating Custom Assertions

### 1. Implement Interface

```php
<?php

namespace App\Assertions;

use Cline\Prism\Contracts\AssertionInterface;

final readonly class LenientBooleanAssertion implements AssertionInterface
{
    public function assert(mixed $data, mixed $expectedValid, mixed $actualValid): bool
    {
        // Treat any truthy/falsy value as matching boolean expectation
        $expectedBool = (bool) $expectedValid;
        $actualBool = (bool) $actualValid;

        return $expectedBool === $actualBool;
    }

    public function getName(): string
    {
        return 'LenientBoolean';
    }

    public function getFailureMessage(mixed $data, mixed $expectedValid, mixed $actualValid): string
    {
        return sprintf(
            'Expected boolean %s, but got %s',
            $expectedValid ? 'true' : 'false',
            $actualValid ? 'true' : 'false'
        );
    }
}
```

### 2. Register Assertion

Configure in `prism.php` by passing to `PrismRunner`:

```php
<?php

use Cline\Prism\Services\CustomAssertionService;
use App\Assertions\LenientBooleanAssertion;

$assertionService = new CustomAssertionService([
    'lenient-boolean' => new LenientBooleanAssertion(),
]);

// Pass to PrismRunner constructor
$runner = new PrismRunner($filterService, $assertionService);
```

### 3. Use in Tests

Reference assertion by name in test files:

```json
{
  "description": "Boolean validation",
  "schema": { "type": "boolean" },
  "tests": [
    {
      "description": "truthy value",
      "data": 1,
      "valid": true,
      "assertion": "lenient-boolean"
    },
    {
      "description": "falsy value",
      "data": 0,
      "valid": false,
      "assertion": "lenient-boolean"
    }
  ]
}
```

## Advanced Assertion Examples

### Range Tolerance Assertion

Allow validation results within a tolerance range:

```php
<?php

final readonly class RangeToleranceAssertion implements AssertionInterface
{
    public function __construct(
        private float $tolerance = 0.1
    ) {}

    public function assert(mixed $data, mixed $expectedValid, mixed $actualValid): bool
    {
        // For numeric comparisons, allow small differences
        if (is_numeric($expectedValid) && is_numeric($actualValid)) {
            return abs($expectedValid - $actualValid) <= $this->tolerance;
        }

        // Fall back to strict equality for non-numeric
        return $expectedValid === $actualValid;
    }

    public function getName(): string
    {
        return 'RangeTolerance';
    }

    public function getFailureMessage(mixed $data, mixed $expectedValid, mixed $actualValid): string
    {
        return sprintf(
            'Expected %s ±%s, but got %s',
            $expectedValid,
            $this->tolerance,
            $actualValid
        );
    }
}
```

### Error Type Assertion

Validate specific error types are thrown:

```php
<?php

final readonly class ErrorTypeAssertion implements AssertionInterface
{
    public function assert(mixed $data, mixed $expectedValid, mixed $actualValid): bool
    {
        // expectedValid contains expected error type
        // actualValid contains actual error type from validator

        if ($expectedValid === null && $actualValid === null) {
            return true; // Both succeeded
        }

        if ($expectedValid === null || $actualValid === null) {
            return false; // One succeeded, one failed
        }

        // Compare error types
        return $expectedValid === $actualValid;
    }

    public function getName(): string
    {
        return 'ErrorType';
    }

    public function getFailureMessage(mixed $data, mixed $expectedValid, mixed $actualValid): string
    {
        return sprintf(
            'Expected error type "%s", but got "%s"',
            $expectedValid ?? 'success',
            $actualValid ?? 'success'
        );
    }
}
```

### Schema Version Assertion

Different behavior for different schema versions:

```php
<?php

final readonly class SchemaVersionAssertion implements AssertionInterface
{
    public function assert(mixed $data, mixed $expectedValid, mixed $actualValid): bool
    {
        // expectedValid is array: ['draft-7' => true, 'draft-2020' => false]
        // actualValid is validation result for current schema version

        if (!is_array($expectedValid)) {
            return $expectedValid === $actualValid;
        }

        // Get current schema version from test context
        $currentVersion = $this->getCurrentSchemaVersion();

        $expected = $expectedValid[$currentVersion] ?? $expectedValid['default'] ?? true;

        return $expected === $actualValid;
    }

    public function getName(): string
    {
        return 'SchemaVersion';
    }

    public function getFailureMessage(mixed $data, mixed $expectedValid, mixed $actualValid): string
    {
        $version = $this->getCurrentSchemaVersion();

        return sprintf(
            'For schema version %s, expected %s but got %s',
            $version,
            $expectedValid[$version] ?? 'default',
            $actualValid ? 'valid' : 'invalid'
        );
    }

    private function getCurrentSchemaVersion(): string
    {
        // Implementation to detect current schema version
        return 'draft-7';
    }
}
```

## Using Assertions in Tests

### Default Assertion

Without specifying assertion field, uses `StrictEqualityAssertion`:

```json
{
  "description": "valid string",
  "data": "hello",
  "valid": true
}
```

### Named Assertion

Specify custom assertion by name:

```json
{
  "description": "lenient validation",
  "data": 1,
  "valid": true,
  "assertion": "lenient-boolean"
}
```

### Multiple Expected Values

Use `AnyOfAssertion` for multiple acceptable outcomes:

```json
{
  "description": "implementation dependent",
  "data": {"foo": "bar"},
  "valid": [true, false],
  "assertion": "any-of"
}
```

## Assertion Service

### Configuration

Create and configure assertion service:

```php
<?php

use Cline\Prism\Services\CustomAssertionService;
use App\Assertions\LenientBooleanAssertion;
use App\Assertions\RangeToleranceAssertion;

$assertionService = new CustomAssertionService([
    'lenient-boolean' => new LenientBooleanAssertion(),
    'range-tolerance' => new RangeToleranceAssertion(0.05),
    'error-type' => new ErrorTypeAssertion(),
]);
```

### Checking Available Assertions

```php
// Check if assertion registered
$assertionService->hasAssertion('lenient-boolean'); // true

// Get all assertion names
$names = $assertionService->getAssertionNames();
// ['lenient-boolean', 'range-tolerance', 'error-type']
```

### Listing Assertions

List registered assertions from CLI:

```bash
vendor/bin/prism test --list-assertions
```

Output:

```
Available Custom Assertions

Total: 3

  • lenient-boolean
  • range-tolerance
  • error-type

Use these assertions in your test files with the 'assertion' field.
```

## Best Practices

### 1. Descriptive Names

Use clear, descriptive assertion names:

```php
// Good
'lenient-boolean'
'range-tolerance'
'error-type-match'

// Bad
'custom1'
'my-assertion'
'test'
```

### 2. Meaningful Error Messages

Provide detailed failure messages:

```php
public function getFailureMessage(mixed $data, mixed $expectedValid, mixed $actualValid): string
{
    return sprintf(
        'Expected %s to validate as %s, but validator returned %s. Data: %s',
        $this->getName(),
        $expectedValid ? 'valid' : 'invalid',
        $actualValid ? 'valid' : 'invalid',
        json_encode($data)
    );
}
```

### 3. Type Safety

Use type hints and validation:

```php
public function assert(mixed $data, mixed $expectedValid, mixed $actualValid): bool
{
    if (!is_numeric($expectedValid) || !is_numeric($actualValid)) {
        throw new InvalidArgumentException('RangeTolerance requires numeric values');
    }

    return abs($expectedValid - $actualValid) <= $this->tolerance;
}
```

### 4. Fallback Behavior

Provide sensible fallbacks:

```php
public function assert(mixed $data, mixed $expectedValid, mixed $actualValid): bool
{
    // Try custom logic first
    if ($this->canUseCustomLogic($expectedValid, $actualValid)) {
        return $this->customAssert($expectedValid, $actualValid);
    }

    // Fall back to strict equality
    return $expectedValid === $actualValid;
}
```

### 5. Documentation

Document assertion behavior in code and docs:

```php
/**
 * Lenient boolean assertion allowing truthy/falsy value comparison.
 *
 * Treats any truthy value (1, "yes", true, [1]) as boolean true,
 * and any falsy value (0, "", false, null, []) as boolean false.
 *
 * Use for validators with flexible boolean interpretation.
 */
final readonly class LenientBooleanAssertion implements AssertionInterface
{
    // ...
}
```

## Next Steps

- Explore [filtering options](/prism/filtering/)
- Learn about [performance features](/prism/performance/)
- See [advanced features](/prism/advanced-features/)
