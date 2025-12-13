---
title: Error Handling
description: Comprehensive guide to exception types and error handling strategies for the Cline TOML parser including parse errors, validation errors, and file access errors.
---

Handle parsing and building errors effectively with the comprehensive exception hierarchy provided by the Cline TOML parser.

## Exception Hierarchy

All exceptions implement the `Cline\Toml\Exception\TomlException` interface:

```
TomlException (interface)
├── ParseException
│   ├── FileNotFoundException
│   └── FileNotReadableException
├── SyntaxErrorException
│   ├── ParserSyntaxErrorException
│   ├── ParserUnexpectedTokenException
│   └── UnexpectedTokenException
├── DumpException (building errors)
│   ├── DuplicateKeyException
│   ├── DuplicateTableKeyException
│   ├── DuplicateArrayTableKeyException
│   ├── EmptyKeyException
│   ├── InvalidStringCharactersException
│   ├── MixedArrayTypesException
│   ├── UnsupportedDataTypeException
│   ├── UnquotedKeyRequiredException
│   ├── KeyDefinedAsImplicitTableException
│   └── TableAlreadyDefinedAsArrayException
└── Other exceptions
    ├── InvalidKeyException
    ├── InvalidTableKeyException
    ├── InvalidArrayTableKeyException
    ├── InvalidUtf8Exception
    ├── LexerParseException
    ├── EmptyEosTokenNameException
    └── EmptyNewlineTokenNameException
```

## Parsing Exceptions

### ParseException

Base exception for all parsing errors:

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\ParseException;

try {
    $config = Toml::parse('invalid = [toml syntax');
} catch (ParseException $e) {
    echo "Parse error: " . $e->getMessage();
    echo "Line: " . $e->getParsedLine();
    echo "File: " . $e->getParsedFile(); // null for strings
    echo "Snippet: " . $e->getSnippet();
}
```

Available methods:
- `getMessage()` - Full error message with context
- `getParsedLine()` - Line number where error occurred
- `getParsedFile()` - Filename (null for string parsing)
- `getSnippet()` - Code snippet near the error

### FileNotFoundException

Thrown when parsing a non-existent file:

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\FileNotFoundException;

try {
    $config = Toml::parseFile('missing.toml');
} catch (FileNotFoundException $e) {
    echo "File not found: " . $e->getMessage();
    // Handle missing file (create default, show error, etc.)
}
```

### FileNotReadableException

Thrown when file exists but cannot be read:

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\FileNotReadableException;

try {
    $config = Toml::parseFile('restricted.toml');
} catch (FileNotReadableException $e) {
    echo "Cannot read file: " . $e->getMessage();
    // Check permissions, handle access error
}
```

### InvalidUtf8Exception

Thrown when input contains invalid UTF-8:

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\InvalidUtf8Exception;

try {
    $config = Toml::parse($invalidUtf8String);
} catch (InvalidUtf8Exception $e) {
    echo "Invalid UTF-8 encoding: " . $e->getMessage();
}
```

### SyntaxErrorException

Thrown when TOML syntax is invalid or input ends unexpectedly:

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\SyntaxErrorException;

$toml = <<<'TOML'
[incomplete
TOML;

try {
    $config = Toml::parse($toml);
} catch (SyntaxErrorException $e) {
    echo "Syntax error: " . $e->getMessage();
}
```

## Building Exceptions (DumpException)

### DuplicateKeyException

Thrown when adding a key that already exists:

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\DuplicateKeyException;

try {
    $builder = new TomlBuilder();
    $toml = $builder
        ->addValue('name', 'First')
        ->addValue('name', 'Second') // Duplicate!
        ->getTomlString();
} catch (DuplicateKeyException $e) {
    echo "Duplicate key: " . $e->getMessage();
    // Output: The key "name" has already been defined previously.
}
```

### DuplicateTableKeyException

Thrown when adding a table that already exists:

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\DuplicateTableKeyException;

try {
    $builder = new TomlBuilder();
    $toml = $builder
        ->addTable('database')
        ->addValue('host', 'localhost')
        ->addTable('database') // Duplicate!
        ->addValue('port', 5432)
        ->getTomlString();
} catch (DuplicateTableKeyException $e) {
    echo "Duplicate table: " . $e->getMessage();
}
```

### DuplicateArrayTableKeyException

Thrown when array of table structure conflicts:

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\DuplicateArrayTableKeyException;

try {
    $builder = new TomlBuilder();
    $toml = $builder
        ->addArrayOfTable('items')
        ->addValue('name', 'First')
        ->addTable('items') // Cannot add table after array of tables!
        ->getTomlString();
} catch (DuplicateArrayTableKeyException $e) {
    echo "Invalid structure: " . $e->getMessage();
}
```

### EmptyKeyException

Thrown when key is empty or whitespace-only:

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\EmptyKeyException;

try {
    $builder = new TomlBuilder();
    $toml = $builder
        ->addValue('', 'value') // Empty key!
        ->getTomlString();
} catch (EmptyKeyException $e) {
    echo "Empty key: " . $e->getMessage();
}
```

### InvalidStringCharactersException

Thrown when string contains invalid escape sequences:

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\InvalidStringCharactersException;

try {
    $builder = new TomlBuilder();
    // Invalid escape sequence in string
    $toml = $builder
        ->addValue('path', "invalid\\escape\\sequence")
        ->getTomlString();
} catch (InvalidStringCharactersException $e) {
    echo "Invalid string: " . $e->getMessage();
}
```

### MixedArrayTypesException

Thrown when array contains mixed types:

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\MixedArrayTypesException;

try {
    $builder = new TomlBuilder();
    $toml = $builder
        ->addValue('mixed', [1, 'two', 3.0]) // Mixed types!
        ->getTomlString();
} catch (MixedArrayTypesException $e) {
    echo "Mixed array types: " . $e->getMessage();
}
```

### UnsupportedDataTypeException

Thrown when value contains unsupported data types:

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\UnsupportedDataTypeException;

try {
    $builder = new TomlBuilder();
    $toml = $builder
        ->addValue('invalid', [new stdClass()]) // Objects not supported!
        ->getTomlString();
} catch (UnsupportedDataTypeException $e) {
    echo "Unsupported type: " . $e->getMessage();
}
```

### UnquotedKeyRequiredException

Thrown when table/array keys contain invalid characters:

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\UnquotedKeyRequiredException;

try {
    $builder = new TomlBuilder();
    $toml = $builder
        ->addTable('invalid table name') // Spaces require quotes!
        ->getTomlString();
} catch (UnquotedKeyRequiredException $e) {
    echo "Invalid key: " . $e->getMessage();
}
```

## Error Handling Patterns

### Basic Try-Catch

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\TomlException;

function loadConfig(string $path): array
{
    try {
        return Toml::parseFile($path);
    } catch (TomlException $e) {
        // Log error
        error_log("TOML error: " . $e->getMessage());

        // Return default configuration
        return [
            'app' => ['name' => 'Default App'],
            'debug' => false,
        ];
    }
}
```

### Specific Exception Handling

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\FileNotFoundException;
use Cline\Toml\Exception\ParseException;

function loadConfig(string $path): array
{
    try {
        return Toml::parseFile($path);
    } catch (FileNotFoundException $e) {
        // Create default config file
        $defaultConfig = "name = \"My App\"\nversion = \"1.0.0\"";
        file_put_contents($path, $defaultConfig);
        return Toml::parse($defaultConfig);
    } catch (ParseException $e) {
        // Show detailed error with line number
        throw new \RuntimeException(
            "Invalid TOML syntax in {$path} at line {$e->getParsedLine()}: {$e->getMessage()}"
        );
    }
}
```

### Validation with Custom Messages

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\DuplicateKeyException;
use Cline\Toml\Exception\EmptyKeyException;
use Cline\Toml\Exception\MixedArrayTypesException;

class ConfigBuilder
{
    private TomlBuilder $builder;
    private array $errors = [];

    public function __construct()
    {
        $this->builder = new TomlBuilder();
    }

    public function addSetting(string $key, mixed $value): self
    {
        try {
            $this->builder->addValue($key, $value);
        } catch (DuplicateKeyException $e) {
            $this->errors[] = "Setting '{$key}' is already defined";
        } catch (EmptyKeyException $e) {
            $this->errors[] = "Setting key cannot be empty";
        } catch (MixedArrayTypesException $e) {
            $this->errors[] = "Setting '{$key}' contains mixed array types";
        }

        return $this;
    }

    public function build(): string
    {
        if (!empty($this->errors)) {
            throw new \RuntimeException(
                "Configuration errors:\n- " . implode("\n- ", $this->errors)
            );
        }

        return $this->builder->getTomlString();
    }

    public function getErrors(): array
    {
        return $this->errors;
    }
}

// Usage
$builder = new ConfigBuilder();
$builder
    ->addSetting('name', 'App')
    ->addSetting('name', 'App2') // Duplicate
    ->addSetting('ports', [8080, 'invalid']); // Mixed types

if ($errors = $builder->getErrors()) {
    foreach ($errors as $error) {
        echo "Error: {$error}\n";
    }
}
```

### Graceful Degradation

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\TomlException;

class ConfigLoader
{
    private array $searchPaths;

    public function __construct(array $searchPaths)
    {
        $this->searchPaths = $searchPaths;
    }

    public function load(): array
    {
        foreach ($this->searchPaths as $path) {
            try {
                return Toml::parseFile($path);
            } catch (TomlException $e) {
                // Try next path
                continue;
            }
        }

        // No valid config found, use built-in defaults
        return $this->getDefaults();
    }

    private function getDefaults(): array
    {
        return [
            'app' => ['name' => 'Default Application'],
            'server' => ['host' => 'localhost', 'port' => 8080],
        ];
    }
}

// Usage
$loader = new ConfigLoader([
    '/etc/myapp/config.toml',
    '/usr/local/etc/myapp/config.toml',
    __DIR__ . '/config.toml',
]);

$config = $loader->load();
```

### Detailed Error Reporting

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\ParseException;

function parseWithDetailedErrors(string $toml, string $source = 'string'): array
{
    try {
        return Toml::parse($toml);
    } catch (ParseException $e) {
        $error = [
            'message' => $e->getMessage(),
            'source' => $source,
            'line' => $e->getParsedLine(),
            'snippet' => $e->getSnippet(),
            'file' => $e->getParsedFile(),
        ];

        // Log detailed error
        error_log(json_encode($error, JSON_PRETTY_PRINT));

        // Show user-friendly message
        $userMessage = "Configuration error";
        if ($error['line'] > 0) {
            $userMessage .= " at line {$error['line']}";
        }
        if ($error['snippet']) {
            $userMessage .= ": near \"{$error['snippet']}\"";
        }

        throw new \RuntimeException($userMessage, 0, $e);
    }
}
```

### Validation Helper

```php
use Cline\Toml\TomlBuilder;
use Cline\Toml\Exception\TomlException;

class TomlValidator
{
    public static function validateStructure(array $data): array
    {
        $errors = [];

        try {
            $builder = new TomlBuilder();
            self::buildFromArray($builder, $data);
            $builder->getTomlString();
        } catch (TomlException $e) {
            $errors[] = $e->getMessage();
        }

        return $errors;
    }

    private static function buildFromArray(TomlBuilder $builder, array $data, string $prefix = ''): void
    {
        foreach ($data as $key => $value) {
            if (is_array($value) && !empty($value) && !self::isAssociative($value)) {
                // Indexed array - might be array of tables
                continue;
            }

            if (is_array($value)) {
                $tableName = $prefix ? "{$prefix}.{$key}" : $key;
                $builder->addTable($tableName);
                self::buildFromArray($builder, $value, $tableName);
            } else {
                $builder->addValue($key, $value);
            }
        }
    }

    private static function isAssociative(array $arr): bool
    {
        return array_keys($arr) !== range(0, count($arr) - 1);
    }
}

// Usage
$data = [
    'name' => 'App',
    'name' => 'Duplicate', // Will be caught
    'server' => ['port' => 8080],
];

$errors = TomlValidator::validateStructure($data);
if (!empty($errors)) {
    echo "Validation errors:\n";
    foreach ($errors as $error) {
        echo "- {$error}\n";
    }
}
```

## Best Practices

### Always Handle File Errors

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\FileNotFoundException;
use Cline\Toml\Exception\FileNotReadableException;

function safeParseFile(string $path): ?array
{
    if (!file_exists($path)) {
        return null;
    }

    if (!is_readable($path)) {
        return null;
    }

    try {
        return Toml::parseFile($path);
    } catch (FileNotFoundException | FileNotReadableException $e) {
        return null;
    }
}
```

### Validate User Input

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\ParseException;

function validateUserConfig(string $userInput): bool
{
    try {
        Toml::parse($userInput);
        return true;
    } catch (ParseException $e) {
        return false;
    }
}
```

### Use Type-Specific Catches

```php
use Cline\Toml\Exception\DuplicateKeyException;
use Cline\Toml\Exception\EmptyKeyException;
use Cline\Toml\Exception\MixedArrayTypesException;

try {
    // Build TOML
} catch (DuplicateKeyException $e) {
    // Handle duplicate keys specifically
} catch (EmptyKeyException $e) {
    // Handle empty keys
} catch (MixedArrayTypesException $e) {
    // Handle array type errors
}
```

## Next Steps

- [Getting Started](/toml/getting-started/) - Basic usage and installation
- [Parsing TOML](/toml/parsing/) - Learn about parsing options
- [Building TOML](/toml/building/) - Create TOML programmatically
