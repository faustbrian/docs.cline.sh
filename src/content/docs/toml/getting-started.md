---
title: Getting Started
description: Install and start using the Cline TOML parser for PHP to parse TOML configuration files into PHP arrays or objects.
---

Parse TOML (Tom's Obvious Minimal Language) configuration files into PHP arrays or objects with full support for TOML specification 0.4.0.

**Use case:** Parse TOML configuration files for application settings, package configurations, or any structured data storage needs.

## Installation

Install via Composer:

```bash
composer require cline/toml
```

## Requirements

- PHP 8.5 or higher

## Basic Usage

### Parsing TOML Strings

Parse a TOML string directly into a PHP array:

```php
use Cline\Toml\Toml;

$toml = <<<'TOML'
title = "TOML Example"
version = "1.0.0"

[database]
host = "localhost"
port = 5432
enabled = true
TOML;

$config = Toml::parse($toml);

// Access values
echo $config['title'];              // "TOML Example"
echo $config['database']['host'];   // "localhost"
echo $config['database']['port'];   // 5432
```

### Parsing TOML Files

Load and parse TOML files:

```php
use Cline\Toml\Toml;

// Parse a configuration file
$config = Toml::parseFile('config.toml');

// Use the configuration
$dbHost = $config['database']['host'];
$dbPort = $config['database']['port'];
```

### Returning Objects Instead of Arrays

Get results as stdClass objects:

```php
use Cline\Toml\Toml;

$toml = 'name = "John Doe"';

// Parse as object
$config = Toml::parse($toml, true);

echo $config->name; // "John Doe"
```

## Quick Example

Create a `config.toml` file:

```toml
# Application Configuration
title = "My Application"
version = "2.1.0"

[server]
host = "0.0.0.0"
port = 8080
timeout = 30

[database]
driver = "postgresql"
host = "localhost"
port = 5432
name = "myapp"
username = "dbuser"
password = "secret"

[features]
debug = false
cache = true
logging = true
```

Parse and use it in your PHP application:

```php
use Cline\Toml\Toml;

class Config
{
    private array $config;

    public function __construct(string $configFile)
    {
        $this->config = Toml::parseFile($configFile);
    }

    public function get(string $key, mixed $default = null): mixed
    {
        $keys = explode('.', $key);
        $value = $this->config;

        foreach ($keys as $segment) {
            if (!isset($value[$segment])) {
                return $default;
            }
            $value = $value[$segment];
        }

        return $value;
    }
}

// Usage
$config = new Config('config.toml');

echo $config->get('title');           // "My Application"
echo $config->get('server.port');     // 8080
echo $config->get('database.host');   // "localhost"
echo $config->get('features.debug');  // false
echo $config->get('missing', 'default'); // "default"
```

## Error Handling

The parser throws exceptions when encountering invalid TOML:

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\ParseException;

try {
    $config = Toml::parse('invalid toml syntax');
} catch (ParseException $e) {
    echo "Parse error: " . $e->getMessage();
    echo "Line: " . $e->getParsedLine();
}
```

## What's TOML?

TOML (Tom's Obvious Minimal Language) is a minimal configuration file format that is easy to read due to obvious semantics. It is designed to map unambiguously to a hash table and be easy for humans to read and write.

Example TOML features:

- **Simple key-value pairs:** `name = "value"`
- **Tables (sections):** `[database]`
- **Nested tables:** `[server.production]`
- **Arrays:** `ports = [8080, 8081, 8082]`
- **Inline tables:** `point = { x = 1, y = 2 }`
- **Comments:** `# This is a comment`
- **Multiple data types:** strings, integers, floats, booleans, dates

## Next Steps

- [Parsing](/toml/parsing/) - Learn about parsing options and result formats
- [Building](/toml/building/) - Create TOML programmatically with TomlBuilder
- [Data Types](/toml/data-types/) - Explore all supported TOML data types
- [Error Handling](/toml/error-handling/) - Handle parsing errors effectively
