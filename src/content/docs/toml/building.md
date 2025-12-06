---
title: Building TOML
description: Create TOML content programmatically using the fluent TomlBuilder API with support for all TOML data types and structures.
---

Build TOML documents programmatically using the fluent `TomlBuilder` API. Perfect for generating configuration files, exports, or dynamic TOML content.

## Basic Usage

### Creating Simple Key-Value Pairs

```php
use Cline\Toml\TomlBuilder;

$builder = new TomlBuilder();

$toml = $builder
    ->addValue('name', 'My Application')
    ->addValue('version', '1.0.0')
    ->addValue('debug', false)
    ->getTomlString();

echo $toml;
/*
name = "My Application"
version = "1.0.0"
debug = false
*/
```

### Adding Comments

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addComment('Application Configuration')
    ->addValue('name', 'MyApp')
    ->addComment('Version information')
    ->addValue('version', '2.0.0')
    ->getTomlString();

echo $toml;
/*
#Application Configuration
name = "MyApp"
#Version information
version = "2.0.0"
*/
```

### Inline Comments

Add comments to individual values:

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addValue('port', 8080, 'HTTP port')
    ->addValue('timeout', 30, 'Connection timeout in seconds')
    ->getTomlString();

echo $toml;
/*
port = 8080 #HTTP port
timeout = 30 #Connection timeout in seconds
*/
```

## Working with Tables

### Simple Tables

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addTable('server')
    ->addValue('host', '0.0.0.0')
    ->addValue('port', 8080)
    ->addTable('database')
    ->addValue('host', 'localhost')
    ->addValue('port', 5432)
    ->getTomlString();

echo $toml;
/*
[server]
host = "0.0.0.0"
port = 8080

[database]
host = "localhost"
port = 5432
*/
```

### Nested Tables

Use dot notation for nested tables:

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addTable('application')
    ->addValue('name', 'MyApp')
    ->addTable('application.server')
    ->addValue('host', 'localhost')
    ->addValue('port', 8080)
    ->addTable('application.server.ssl')
    ->addValue('enabled', true)
    ->addValue('cert', '/path/to/cert.pem')
    ->getTomlString();

echo $toml;
/*
[application]
name = "MyApp"

[application.server]
host = "localhost"
port = 8080

[application.server.ssl]
enabled = true
cert = "/path/to/cert.pem"
*/
```

## Array of Tables

Create array of tables with `addArrayOfTable()`:

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addArrayOfTable('products')
    ->addValue('name', 'Hammer')
    ->addValue('sku', 738594937)
    ->addArrayOfTable('products')
    ->addValue('name', 'Nail')
    ->addValue('sku', 284758393)
    ->getTomlString();

echo $toml;
/*
[[products]]
name = "Hammer"
sku = 738594937

[[products]]
name = "Nail"
sku = 284758393
*/
```

### Nested Array of Tables

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addArrayOfTable('fruit')
    ->addValue('name', 'apple')
    ->addArrayOfTable('fruit.variety')
    ->addValue('name', 'red delicious')
    ->addArrayOfTable('fruit.variety')
    ->addValue('name', 'granny smith')
    ->addArrayOfTable('fruit')
    ->addValue('name', 'banana')
    ->addArrayOfTable('fruit.variety')
    ->addValue('name', 'plantain')
    ->getTomlString();

echo $toml;
/*
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
*/
```

## Data Types

### Strings

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addValue('title', 'TOML Example')
    ->addValue('description', 'A string with "quotes"')
    ->addValue('path', 'C:\\Users\\nodejs\\templates')
    ->getTomlString();
```

### Literal Strings

Use `@` prefix for literal strings (no escaping):

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addValue('regex', '@<\i\c*\s*>')
    ->getTomlString();

// Output: regex = '<\i\c*\s*>'
// Backslashes are preserved without escaping
```

To include a literal `@` at the start, use `@@`:

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addValue('email', '@@user@example.com')
    ->getTomlString();

// Output: email = '@user@example.com'
```

### Numbers

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addValue('integer', 42)
    ->addValue('negative', -17)
    ->addValue('float', 3.14)
    ->addValue('scientific', 5e22)
    ->getTomlString();

/*
integer = 42
negative = -17
float = 3.14
scientific = 5.0e+22
*/
```

### Booleans

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addValue('enabled', true)
    ->addValue('disabled', false)
    ->getTomlString();

/*
enabled = true
disabled = false
*/
```

### Dates and Times

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addValue('created', new DateTime('2024-01-15 10:30:00'))
    ->getTomlString();

/*
created = 2024-01-15T10:30:00Z
*/
```

### Arrays

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addValue('ports', [8080, 8081, 8082])
    ->addValue('hosts', ['localhost', 'example.com'])
    ->addValue('matrix', [[1, 2], [3, 4]])
    ->getTomlString();

/*
ports = [8080, 8081, 8082]
hosts = ["localhost", "example.com"]
matrix = [[1, 2], [3, 4]]
*/
```

Arrays with DateTime objects:

```php
$builder = new TomlBuilder();

$toml = $builder
    ->addValue('timestamps', [
        new DateTime('2024-01-01'),
        new DateTime('2024-06-01'),
        new DateTime('2024-12-01')
    ])
    ->getTomlString();
```

## Indentation

Control indentation (default is 4 spaces):

```php
// No indentation
$builder = new TomlBuilder(0);

// 2 spaces
$builder = new TomlBuilder(2);

// 4 spaces (default)
$builder = new TomlBuilder(4);
```

## Real-World Examples

### Generate Application Config

```php
use Cline\Toml\TomlBuilder;

function generateAppConfig(array $config): string
{
    $builder = new TomlBuilder();

    $builder->addComment('Application Configuration');
    $builder->addValue('name', $config['name']);
    $builder->addValue('version', $config['version']);
    $builder->addValue('debug', $config['debug']);

    $builder->addTable('server');
    $builder->addValue('host', $config['server']['host']);
    $builder->addValue('port', $config['server']['port']);
    $builder->addValue('timeout', $config['server']['timeout'], 'seconds');

    $builder->addTable('database');
    $builder->addValue('driver', $config['database']['driver']);
    $builder->addValue('host', $config['database']['host']);
    $builder->addValue('port', $config['database']['port']);
    $builder->addValue('name', $config['database']['name']);

    return $builder->getTomlString();
}

$config = [
    'name' => 'My Application',
    'version' => '1.0.0',
    'debug' => false,
    'server' => [
        'host' => '0.0.0.0',
        'port' => 8080,
        'timeout' => 30,
    ],
    'database' => [
        'driver' => 'mysql',
        'host' => 'localhost',
        'port' => 3306,
        'name' => 'myapp',
    ],
];

$toml = generateAppConfig($config);
file_put_contents('config.toml', $toml);
```

### Export Database Connections

```php
use Cline\Toml\TomlBuilder;

class DatabaseExporter
{
    public function export(array $connections): string
    {
        $builder = new TomlBuilder();

        $builder->addComment('Database Connections');

        foreach ($connections as $name => $connection) {
            $builder->addTable("database.{$name}");
            $builder->addValue('driver', $connection['driver']);
            $builder->addValue('host', $connection['host']);
            $builder->addValue('port', $connection['port']);
            $builder->addValue('username', $connection['username']);
            $builder->addValue('database', $connection['database']);
        }

        return $builder->getTomlString();
    }
}

$connections = [
    'production' => [
        'driver' => 'pgsql',
        'host' => 'db.prod.internal',
        'port' => 5432,
        'username' => 'app_user',
        'database' => 'myapp',
    ],
    'staging' => [
        'driver' => 'pgsql',
        'host' => 'db.staging.internal',
        'port' => 5432,
        'username' => 'app_user',
        'database' => 'myapp_staging',
    ],
];

$exporter = new DatabaseExporter();
$toml = $exporter->export($connections);

file_put_contents('database.toml', $toml);
```

### Generate Server List

```php
use Cline\Toml\TomlBuilder;

class ServerConfigGenerator
{
    public function generate(array $servers): string
    {
        $builder = new TomlBuilder();

        $builder->addComment('Server Configuration');
        $builder->addValue('updated', new DateTime());

        foreach ($servers as $server) {
            $builder->addArrayOfTable('servers');
            $builder->addValue('name', $server['name']);
            $builder->addValue('ip', $server['ip']);
            $builder->addValue('port', $server['port']);
            $builder->addValue('active', $server['active']);

            if (!empty($server['tags'])) {
                $builder->addValue('tags', $server['tags']);
            }
        }

        return $builder->getTomlString();
    }
}

$servers = [
    [
        'name' => 'web-1',
        'ip' => '10.0.1.1',
        'port' => 80,
        'active' => true,
        'tags' => ['web', 'production'],
    ],
    [
        'name' => 'web-2',
        'ip' => '10.0.1.2',
        'port' => 80,
        'active' => true,
        'tags' => ['web', 'production'],
    ],
    [
        'name' => 'db-1',
        'ip' => '10.0.2.1',
        'port' => 5432,
        'active' => true,
        'tags' => ['database', 'production'],
    ],
];

$generator = new ServerConfigGenerator();
$toml = $generator->generate($servers);

file_put_contents('servers.toml', $toml);
```

## Validation

The builder automatically validates:

- Key names (cannot be empty)
- Table names (must be valid unquoted keys)
- Array types (all elements must be same type)
- String characters (special characters are escaped)
- Duplicate keys and tables

## Method Chaining

All builder methods return the builder instance for fluent chaining:

```php
$toml = (new TomlBuilder())
    ->addComment('Configuration')
    ->addValue('version', '1.0')
    ->addTable('server')
    ->addValue('port', 8080)
    ->getTomlString();
```

## Next Steps

- [Data Types](/toml/data-types/) - Learn about all supported data types
- [Error Handling](/toml/error-handling/) - Handle builder exceptions
- [Parsing TOML](/toml/parsing/) - Parse TOML back into PHP
