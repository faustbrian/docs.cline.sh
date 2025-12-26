---
title: Getting Started
description: Install and start using Prism for validation testing in PHP 8.5+.
---

Prism is a comprehensive testing CLI for PHP 8.5+ that provides a beautiful, feature-rich interface for running validation test suites with Termwind-powered output.

## Installation

```bash
composer require cline/prism
```

## Basic Usage

Prism executes validation tests defined in JSON test files and provides detailed reporting of results.

### 1. Create a Configuration File

Create a `prism.php` file in your project root:

```php
<?php

use Cline\Prism\Contracts\PrismTestInterface;

return [
    new class implements PrismTestInterface {
        public function getName(): string
        {
            return 'my-validator';
        }

        public function getTestDirectory(): string
        {
            return __DIR__ . '/tests/validation';
        }

        public function shouldIncludeFile(string $filePath): bool
        {
            return true; // Include all JSON files
        }

        public function decodeJson(string $json): mixed
        {
            return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
        }

        public function validate(mixed $data, mixed $schema = null): object
        {
            // Your validation logic here
            $isValid = /* your validation */;

            return new class($isValid) {
                public function __construct(private bool $isValid) {}
                public function isValid(): bool { return $this->isValid; }
            };
        }
    },
];
```

### 2. Create Test Files

Create JSON test files in your test directory:

```json
[
  {
    "description": "String validation tests",
    "schema": {
      "type": "string"
    },
    "tests": [
      {
        "description": "valid string",
        "data": "hello",
        "valid": true,
        "tags": ["string", "basic"]
      },
      {
        "description": "invalid type",
        "data": 123,
        "valid": false,
        "tags": ["string", "type-error"]
      }
    ]
  }
]
```

### 3. Run Tests

```bash
vendor/bin/prism test
```

## Output Modes

### Enhanced Terminal Output (Default)

Beautiful, color-coded output with Termwind:

```bash
vendor/bin/prism test
```

### CI-Friendly Output

Plain text output suitable for continuous integration:

```bash
vendor/bin/prism test --ci
```

### Detailed Failures

Show detailed information with syntax-highlighted JSON diff:

```bash
vendor/bin/prism test --failures
```

### Verbose Mode

See each test as it runs with real-time progress:

```bash
vendor/bin/prism test --verbose
# or shorthand
vendor/bin/prism test -v
```

Displays:
- ✓/✗ icon for pass/fail
- Test group and description
- Execution time per test
- Error messages in red

### Version Information

Display the Prism version:

```bash
vendor/bin/prism --version
```

## Examples

Check the `examples/` directory in the Prism repository for complete working examples:

- **`basic/`** - JSON schema validation with type checking, required fields, and format validation
- **`filtering/`** - Tag-based filtering and test organization (coming soon)
- **`custom-assertions/`** - Domain-specific validation rules (coming soon)
- **`comparison/`** - Multi-validator comparison (coming soon)

Each example includes:
- Complete `prism.php` configuration
- Test cases with expected results
- README with usage instructions

## Quick Examples

### Filter Tests by Name Pattern

```bash
vendor/bin/prism test --filter "string.*"
```

### Run Tests for Specific Draft

```bash
vendor/bin/prism test --draft draft-7
```

### Run Tests in Parallel

```bash
vendor/bin/prism test --parallel 4
```

### Watch Mode

Automatically re-run tests when files change:

```bash
vendor/bin/prism test --watch
```

## Next Steps

- Learn about [configuration options](/prism/configuration/)
- Explore [filtering capabilities](/prism/filtering/)
- Discover [performance features](/prism/performance/)
- See [advanced features](/prism/advanced-features/)
