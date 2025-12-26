# Configuration

Learn how to configure compliance tests for your PHP projects.

## Configuration File

The compliance tool looks for a `compliance.php` file in your project root. This file should return an array of `ComplianceTestInterface` implementations.

### Basic Configuration

```php
<?php

declare(strict_types=1);

use Cline\Compliance\Contracts\ComplianceTestInterface;
use Cline\Compliance\Contracts\ValidationResult;

return [
    new class implements ComplianceTestInterface {
        public function getName(): string
        {
            return 'My Test Suite';
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

### Multiple Test Suites

You can define multiple test suites in a single configuration file:

```php
<?php

declare(strict_types=1);

use Cline\JsonSchema\Adapters\ComplianceAdapter;
use Cline\JsonSchema\Support\SchemaLoader;
use Cline\JsonSchema\Validators\Draft04Validator;
use Cline\JsonSchema\Validators\Draft07Validator;

$schemaLoader = new SchemaLoader(__DIR__.'/compliance/remotes');

return [
    new ComplianceAdapter(
        name: 'Draft 04',
        validatorClass: Draft04Validator::class,
        testDirectory: __DIR__.'/compliance/tests/draft4',
        schemaLoader: $schemaLoader,
    ),
    new ComplianceAdapter(
        name: 'Draft 07',
        validatorClass: Draft07Validator::class,
        testDirectory: __DIR__.'/compliance/tests/draft7',
        schemaLoader: $schemaLoader,
    ),
];
```

## ComplianceTestInterface Methods

### getName(): string

Returns the human-readable name of the test suite. This name appears in test output.

```php
public function getName(): string
{
    return 'JSON Schema Draft 04';
}
```

### getValidatorClass(): string

Returns the fully-qualified class name of the validator to use.

```php
public function getValidatorClass(): string
{
    return Draft04Validator::class;
}
```

**Returns:** `class-string`

### getTestDirectory(): string

Returns the absolute path to the directory containing test files.

```php
public function getTestDirectory(): string
{
    return __DIR__.'/tests/compliance/draft4';
}
```

### validate(mixed $data, mixed $schema): ValidationResult

Validates data against a schema using your validator implementation.

```php
public function validate(mixed $data, mixed $schema): ValidationResult
{
    $validator = new ($this->getValidatorClass())();

    return $validator->validate($data, $schema);
}
```

**Parameters:**
- `$data` (mixed) - The data to validate
- `$schema` (mixed) - The schema to validate against

**Returns:** `ValidationResult`

### getTestFilePatterns(): array

Returns an array of glob patterns for finding test files.

```php
public function getTestFilePatterns(): array
{
    return ['*.json', '**/*.json'];
}
```

**Returns:** `array<int, string>`

Common patterns:
- `['*.json']` - All JSON files in test directory
- `['**/*.json']` - All JSON files recursively
- `['draft4/**/*.json']` - JSON files in specific subdirectory

### decodeJson(string $json): mixed

Decodes JSON strings while preserving type distinctions (e.g., `{}` vs `[]`).

```php
public function decodeJson(string $json): mixed
{
    return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
}
```

**Parameters:**
- `$json` (string) - The JSON string to decode

**Returns:** `mixed`

**Flags:**
- Use `JSON_THROW_ON_ERROR` for proper error handling
- Consider `JSON_OBJECT_AS_ARRAY` vs object return type based on your needs

## ValidationResult

Your `validate()` method must return an instance of `ValidationResult`:

```php
use Cline\Compliance\Contracts\ValidationResult;

final readonly class MyValidationResult implements ValidationResult
{
    public function __construct(
        public bool $valid,
        public ?string $error = null,
    ) {}

    public function isValid(): bool
    {
        return $this->valid;
    }

    public function getError(): ?string
    {
        return $this->error;
    }
}
```

### Interface Methods

#### isValid(): bool

Returns whether validation passed.

```php
public function isValid(): bool
{
    return $this->valid;
}
```

#### getError(): ?string

Returns the validation error message, or null if validation passed.

```php
public function getError(): ?string
{
    return $this->error;
}
```

## Directory Structure

Recommended project structure:

```
your-project/
├── compliance.php              # Configuration file
├── compliance/
│   ├── tests/
│   │   ├── draft4/
│   │   │   ├── basic.json
│   │   │   └── advanced.json
│   │   └── draft7/
│   │       ├── basic.json
│   │       └── advanced.json
│   └── remotes/               # Remote schema references (optional)
└── vendor/
    └── bin/
        └── compliance         # CLI binary
```

## Advanced Configuration

### Custom Adapter Class

Create a reusable adapter class:

```php
<?php

declare(strict_types=1);

namespace App\Compliance;

use Cline\Compliance\Contracts\ComplianceTestInterface;
use Cline\Compliance\Contracts\ValidationResult;

final readonly class CustomAdapter implements ComplianceTestInterface
{
    public function __construct(
        private string $name,
        private string $validatorClass,
        private string $testDirectory,
        private array $testFilePatterns = ['*.json'],
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
        $validatorClass = $this->validatorClass;
        $validator = new $validatorClass();

        return $validator->validate($data, $schema);
    }

    public function getTestFilePatterns(): array
    {
        return $this->testFilePatterns;
    }

    public function decodeJson(string $json): mixed
    {
        return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
    }
}
```

Usage in `compliance.php`:

```php
<?php

use App\Compliance\CustomAdapter;
use App\Validators\MyValidator;

return [
    new CustomAdapter(
        name: 'My Custom Tests',
        validatorClass: MyValidator::class,
        testDirectory: __DIR__.'/tests/compliance',
        testFilePatterns: ['*.json', 'custom/**/*.json'],
    ),
];
```

## Configuration Discovery

The compliance tool searches for `compliance.php` in the following order:

1. Current working directory
2. Parent directories (up to project root)

If no configuration file is found, the tool displays an error message.

## Next Steps

- [Writing Tests](writing-tests.md) - Create compliance test suites
- [API Reference](api-reference.md) - Complete API documentation
