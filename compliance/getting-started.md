# Getting Started

A beautiful compliance testing CLI for PHP projects with Termwind-powered output.

## Installation

Install via Composer:

```bash
composer require cline/compliance
```

The package includes a global `compliance` binary that can be used from any project.

## Requirements

- PHP 8.5+
- Composer

## Quick Example

Create a `compliance.php` configuration file in your project root:

```php
<?php

use Cline\Compliance\Contracts\ComplianceTestInterface;
use Cline\Compliance\Contracts\ValidationResult;

return [
    new class implements ComplianceTestInterface {
        public function getName(): string
        {
            return 'My Compliance Test';
        }

        public function getValidatorClass(): string
        {
            return MyValidator::class;
        }

        public function getTestDirectory(): string
        {
            return __DIR__.'/tests/compliance';
        }

        public function validate(mixed $data, mixed $schema): ValidationResult
        {
            $validator = new MyValidator();

            return $validator->validate($data, $schema);
        }

        public function getTestFilePatterns(): array
        {
            return ['*.json'];
        }

        public function decodeJson(string $json): mixed
        {
            return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
        }
    },
];
```

Run your compliance tests:

```bash
vendor/bin/compliance test
```

## CLI Output

The compliance tool provides beautiful terminal output powered by Termwind:

```
┌─────────────────────────────────────────────┐
│ Compliance Test Results                     │
├─────────────────────────────────────────────┤
│ My Compliance Test                          │
│   Total: 150   Passed: 148   Failed: 2     │
└─────────────────────────────────────────────┘
```

### Show Failures

Display detailed failure information:

```bash
vendor/bin/compliance test --failures
```

### CI Mode

Use CI-friendly output without Termwind formatting:

```bash
vendor/bin/compliance test --ci
```

### Filter by Draft

Run tests for a specific draft/version only:

```bash
vendor/bin/compliance test --draft="Draft 04"
```

## Basic Usage

### Creating Test Adapters

Implement the `ComplianceTestInterface` to create test adapters:

```php
<?php

declare(strict_types=1);

use Cline\Compliance\Contracts\ComplianceTestInterface;
use Cline\Compliance\Contracts\ValidationResult;

final readonly class MyComplianceAdapter implements ComplianceTestInterface
{
    public function __construct(
        private string $name,
        private string $validatorClass,
        private string $testDirectory,
    ) {}

    public function getName(): string
    {
        return $this->name;
    }

    public function getValidatorClass(): string
    {
        return $this->validatorClass;
    }

    public function getTestDirectory(): string
    {
        return $this->testDirectory;
    }

    public function validate(mixed $data, mixed $schema): ValidationResult
    {
        $validator = new ($this->validatorClass)();

        return $validator->validate($data, $schema);
    }

    public function getTestFilePatterns(): array
    {
        return ['*.json'];
    }

    public function decodeJson(string $json): mixed
    {
        return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
    }
}
```

### Test File Format

Compliance tests use JSON files with the following structure:

```json
[
  {
    "description": "Test suite description",
    "schema": {
      "type": "object",
      "properties": {
        "name": { "type": "string" }
      }
    },
    "tests": [
      {
        "description": "valid data",
        "data": { "name": "John" },
        "valid": true
      },
      {
        "description": "invalid data",
        "data": { "name": 123 },
        "valid": false
      }
    ]
  }
]
```

## Next Steps

- [Configuration](configuration.md) - Learn about configuration options
- [Writing Tests](writing-tests.md) - Create compliance test suites
- [API Reference](api-reference.md) - Complete API documentation
