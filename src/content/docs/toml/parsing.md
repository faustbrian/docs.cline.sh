---
title: Parsing TOML
description: Parse TOML strings and files into PHP arrays or objects with comprehensive error handling and validation.
---

Parse TOML content from strings or files into native PHP data structures with full TOML 0.4.0 specification support.

## Parsing Methods

### Parse from String

Parse TOML content directly from a string:

```php
use Cline\Toml\Toml;

$toml = <<<'TOML'
name = "Configuration"
version = "1.0.0"
TOML;

$result = Toml::parse($toml);

// Returns: ['name' => 'Configuration', 'version' => '1.0.0']
```

### Parse from File

Load and parse a TOML file:

```php
use Cline\Toml\Toml;

$config = Toml::parseFile('/path/to/config.toml');
```

The parser validates that:
- File exists (throws `FileNotFoundException`)
- File is readable (throws `FileNotReadableException`)
- Content is valid TOML (throws `ParseException`)

## Result Formats

### Array Format (Default)

By default, parsed TOML returns a PHP array:

```php
$toml = <<<'TOML'
title = "My App"

[database]
host = "localhost"
port = 5432
TOML;

$config = Toml::parse($toml);

// Array access
echo $config['title'];              // "My App"
echo $config['database']['host'];   // "localhost"
```

### Object Format

Return results as `stdClass` objects:

```php
$config = Toml::parse($toml, true);

// Object access
echo $config->title;              // "My App"
echo $config->database->host;     // "localhost"
```

### Empty Input

Empty TOML strings return `null`:

```php
$result = Toml::parse('');
// Returns: null
```

## Parsing Tables

### Simple Tables

```php
$toml = <<<'TOML'
[server]
host = "localhost"
port = 8080

[database]
host = "db.internal"
port = 5432
TOML;

$config = Toml::parse($toml);

echo $config['server']['host'];   // "localhost"
echo $config['database']['port']; // 5432
```

### Nested Tables

```php
$toml = <<<'TOML'
[application]
name = "MyApp"

[application.server]
host = "0.0.0.0"
port = 8080

[application.server.ssl]
enabled = true
cert = "/path/to/cert.pem"
TOML;

$config = Toml::parse($toml);

echo $config['application']['name'];                  // "MyApp"
echo $config['application']['server']['host'];        // "0.0.0.0"
echo $config['application']['server']['ssl']['cert']; // "/path/to/cert.pem"
```

### Dotted Keys

TOML supports dotted keys as an alternative syntax:

```php
$toml = <<<'TOML'
name = "App"
server.host = "localhost"
server.port = 8080
TOML;

$config = Toml::parse($toml);

// Same structure as table syntax
echo $config['name'];           // "App"
echo $config['server']['host']; // "localhost"
```

## Parsing Arrays

### Simple Arrays

```php
$toml = 'ports = [8080, 8081, 8082]';

$config = Toml::parse($toml);

foreach ($config['ports'] as $port) {
    echo $port . "\n";
}
// Output: 8080, 8081, 8082
```

### Nested Arrays

```php
$toml = 'matrix = [[1, 2], [3, 4], [5, 6]]';

$config = Toml::parse($toml);

echo $config['matrix'][0][0]; // 1
echo $config['matrix'][1][1]; // 4
```

### Array of Tables

```php
$toml = <<<'TOML'
[[products]]
name = "Hammer"
sku = 738594937

[[products]]
name = "Nail"
sku = 284758393
TOML;

$config = Toml::parse($toml);

foreach ($config['products'] as $product) {
    echo $product['name'] . ": " . $product['sku'] . "\n";
}
// Output:
// Hammer: 738594937
// Nail: 284758393
```

### Nested Array of Tables

```php
$toml = <<<'TOML'
[[fruit]]
name = "apple"

[[fruit.variety]]
name = "red delicious"

[[fruit.variety]]
name = "granny smith"

[[fruit]]
name = "banana"

[[fruit.variety]]
name = "plantain"
TOML;

$config = Toml::parse($toml);

// Access nested structure
echo $config['fruit'][0]['name'];                   // "apple"
echo $config['fruit'][0]['variety'][0]['name'];     // "red delicious"
echo $config['fruit'][0]['variety'][1]['name'];     // "granny smith"
echo $config['fruit'][1]['name'];                   // "banana"
echo $config['fruit'][1]['variety'][0]['name'];     // "plantain"
```

## Parsing Inline Tables

Inline tables are enclosed in curly braces:

```php
$toml = <<<'TOML'
name = { first = "Tom", last = "Preston-Werner" }
point = { x = 1, y = 2 }
TOML;

$config = Toml::parse($toml);

echo $config['name']['first'];  // "Tom"
echo $config['point']['x'];     // 1
```

## Real-World Example

Parse a complete application configuration:

```php
use Cline\Toml\Toml;
use Cline\Toml\Exception\ParseException;
use Cline\Toml\Exception\FileNotFoundException;

class ConfigManager
{
    private array $config;

    public function __construct(string $configPath)
    {
        try {
            $this->config = Toml::parseFile($configPath);
        } catch (FileNotFoundException $e) {
            throw new \RuntimeException("Config file not found: {$configPath}");
        } catch (ParseException $e) {
            throw new \RuntimeException(
                "Invalid TOML at line {$e->getParsedLine()}: {$e->getMessage()}"
            );
        }
    }

    public function getDatabaseConfig(string $connection = 'default'): array
    {
        if (!isset($this->config['database'][$connection])) {
            throw new \InvalidArgumentException("Database connection '{$connection}' not found");
        }

        return $this->config['database'][$connection];
    }

    public function getServers(): array
    {
        return $this->config['servers'] ?? [];
    }

    public function isFeatureEnabled(string $feature): bool
    {
        return $this->config['features'][$feature] ?? false;
    }
}

// config.toml
/*
[database.default]
driver = "mysql"
host = "localhost"
port = 3306

[database.cache]
driver = "redis"
host = "localhost"
port = 6379

[[servers]]
name = "web-1"
ip = "10.0.1.1"

[[servers]]
name = "web-2"
ip = "10.0.1.2"

[features]
api = true
webhooks = false
*/

// Usage
$config = new ConfigManager('config.toml');

$dbConfig = $config->getDatabaseConfig('default');
echo $dbConfig['host']; // "localhost"

foreach ($config->getServers() as $server) {
    echo "{$server['name']}: {$server['ip']}\n";
}

if ($config->isFeatureEnabled('api')) {
    echo "API is enabled\n";
}
```

## Performance Tips

### Caching Parsed Results

For frequently accessed configuration files, cache the parsed result:

```php
use Cline\Toml\Toml;

class CachedConfig
{
    private static ?array $cache = null;

    public static function load(string $path): array
    {
        if (self::$cache === null) {
            self::$cache = Toml::parseFile($path);
        }

        return self::$cache;
    }

    public static function clearCache(): void
    {
        self::$cache = null;
    }
}
```

### Parse Once, Use Many Times

```php
// Parse once
$config = Toml::parseFile('config.toml');

// Store in container, service, or static property
$app->instance('config', $config);

// Use throughout application
$dbHost = $app->get('config')['database']['host'];
```

## File Path Handling

The parser accepts both absolute and relative paths:

```php
// Absolute path
$config = Toml::parseFile('/var/www/config.toml');

// Relative path
$config = Toml::parseFile('config/app.toml');

// Using constants
$config = Toml::parseFile(__DIR__ . '/config.toml');

// With environment variables
$configPath = getenv('CONFIG_PATH') ?: 'config.toml';
$config = Toml::parseFile($configPath);
```

## Comments in TOML

Comments are ignored during parsing:

```php
$toml = <<<'TOML'
# This is a comment
name = "App" # This is also a comment

# Comments can be on their own line
[database]
host = "localhost" # End-of-line comment
TOML;

$config = Toml::parse($toml);
// Comments are not included in the result
```

## Next Steps

- [Building TOML](/toml/building/) - Create TOML programmatically
- [Data Types](/toml/data-types/) - Learn about all supported data types
- [Error Handling](/toml/error-handling/) - Handle parsing errors effectively
