---
title: Configuration
description: Configure Prism test suites and execution options.
---

Prism uses a `prism.php` configuration file to define test suites and validation logic.

## Configuration File Location

By default, Prism looks for `prism.php` in the current directory. You can specify a different path:

```bash
vendor/bin/prism test path/to/custom-prism.php
```

## PrismTestInterface

Each test suite implements `PrismTestInterface`:

```php
<?php

use Cline\Prism\Contracts\PrismTestInterface;

return [
    new class implements PrismTestInterface {
        public function getName(): string
        {
            return 'validator-name';
        }

        public function getTestDirectory(): string
        {
            return __DIR__ . '/tests/validation';
        }

        public function shouldIncludeFile(string $filePath): bool
        {
            // Filter which test files to include
            return !str_contains($filePath, 'optional');
        }

        public function decodeJson(string $json): mixed
        {
            return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
        }

        public function validate(mixed $data, mixed $schema = null): object
        {
            // Implement your validation logic
            $validator = new MyValidator($schema);
            $isValid = $validator->validate($data);

            return new class($isValid) {
                public function __construct(private bool $isValid) {}
                public function isValid(): bool { return $this->isValid; }
            };
        }
    },
];
```

## Required Methods

### getName(): string

Returns the name of this test suite. Used in test identifiers and output.

```php
public function getName(): string
{
    return 'json-schema-draft-7';
}
```

### getTestDirectory(): string

Returns the absolute path to the directory containing JSON test files.

```php
public function getTestDirectory(): string
{
    return __DIR__ . '/tests/json-schema/draft-7';
}
```

### shouldIncludeFile(string $filePath): bool

Filters which test files to include from the test directory.

```php
public function shouldIncludeFile(string $filePath): bool
{
    // Exclude optional tests
    if (str_contains($filePath, '/optional/')) {
        return false;
    }

    // Only include specific subdirectories
    return str_contains($filePath, '/required/');
}
```

### decodeJson(string $json): mixed

Decodes JSON test file contents. Allows custom JSON parsing logic.

```php
public function decodeJson(string $json): mixed
{
    return json_decode(
        json: $json,
        associative: true,
        depth: 512,
        flags: JSON_THROW_ON_ERROR
    );
}
```

### validate(mixed $data, mixed $schema = null): object

Executes validation logic and returns a result object with `isValid(): bool` method.

```php
public function validate(mixed $data, mixed $schema = null): object
{
    $validator = new JsonSchema\Validator();
    $validator->validate($data, $schema);

    return new class($validator->isValid()) {
        public function __construct(private bool $isValid) {}
        public function isValid(): bool { return $this->isValid; }
    };
}
```

## Multiple Test Suites

Configure multiple validators in one file:

```php
<?php

return [
    new Draft7Validator(),
    new Draft2020Validator(),
    new CustomValidator(),
];
```

Run tests for all suites:

```bash
vendor/bin/prism test
```

Or filter by suite name:

```bash
vendor/bin/prism test --draft draft-7
```

## Test File Structure

JSON test files follow this structure:

```json
[
  {
    "description": "Test group description",
    "schema": { "type": "string" },
    "tests": [
      {
        "description": "Test case description",
        "data": "test data",
        "valid": true,
        "tags": ["optional", "tag", "array"],
        "assertion": "custom-assertion-name"
      }
    ]
  }
]
```

### Test File Fields

- **description** (string, required): Description of the test group
- **schema** (mixed, optional): Schema to validate against
- **tests** (array, required): Array of individual test cases

### Test Case Fields

- **description** (string, required): Description of this test case
- **data** (mixed, required): Data to validate
- **valid** (boolean, required): Expected validation result
- **tags** (array, optional): Tags for filtering and organization
- **assertion** (string, optional): Custom assertion name to use

## Example: JSON Schema Validator

```php
<?php

use Cline\Prism\Contracts\PrismTestInterface;
use Opis\JsonSchema\Validator;

return [
    new class implements PrismTestInterface {
        public function getName(): string
        {
            return 'opis-json-schema';
        }

        public function getTestDirectory(): string
        {
            return __DIR__ . '/tests/JSON-Schema-Test-Suite/tests/draft7';
        }

        public function shouldIncludeFile(string $filePath): bool
        {
            // Exclude optional tests and format tests
            return !str_contains($filePath, '/optional/')
                && !str_contains($filePath, '/format/');
        }

        public function decodeJson(string $json): mixed
        {
            return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
        }

        public function validate(mixed $data, mixed $schema = null): object
        {
            $validator = new Validator();
            $result = $validator->validate($data, $schema);

            return new class($result->isValid()) {
                public function __construct(private bool $isValid) {}
                public function isValid(): bool { return $this->isValid; }
            };
        }
    },
];
```

## Next Steps

- Learn about [filtering options](/prism/filtering/)
- Explore [performance features](/prism/performance/)
- See [output formats](/prism/output-formats/)
