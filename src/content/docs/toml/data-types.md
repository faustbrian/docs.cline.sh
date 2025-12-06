---
title: Data Types
description: Complete guide to all TOML data types supported by the Cline TOML parser including strings, numbers, booleans, dates, arrays, tables, and inline tables.
---

The Cline TOML parser supports all data types defined in the TOML 0.4.0 specification. Learn how each type is parsed and represented in PHP.

## Strings

### Basic Strings

Enclosed in double quotes with escape sequences:

```toml
title = "TOML Example"
description = "A string with \"quotes\""
```

Parsed as:

```php
[
    'title' => 'TOML Example',
    'description' => 'A string with "quotes"',
]
```

### Escape Sequences

Supported escape sequences:

```toml
backspace = "Value \b here"
tab = "Value \t here"
newline = "Value \n here"
formfeed = "Value \f here"
carriage = "Value \r here"
quote = "Value \" here"
backslash = "Value \\ here"
```

### Unicode Escapes

```toml
unicode = "Greek delta: \u03B4"
unicode32 = "Emoji: \U0001F600"
```

Parsed as:

```php
[
    'unicode' => 'Greek delta: δ',
    'unicode32' => 'Emoji: 😀',
]
```

### Literal Strings

Enclosed in single quotes, no escaping:

```toml
path = 'C:\Users\nodejs\templates'
regex = '<\i\c*\s*>'
```

Parsed as:

```php
[
    'path' => 'C:\Users\nodejs\templates',
    'regex' => '<\i\c*\s*>',
]
```

When building with TomlBuilder, prefix with `@`:

```php
$builder->addValue('regex', '@<\i\c*\s*>');
// Output: regex = '<\i\c*\s*>'
```

### Multi-line Basic Strings

```toml
description = """
Multi-line strings
are supported.
Line breaks are preserved."""
```

### Multi-line Literal Strings

```toml
regex = '''
I [dw]on't need \d{2} apples
'''
```

## Integers

### Standard Integers

```toml
positive = 99
negative = -17
zero = 0
```

Parsed as PHP `int`:

```php
[
    'positive' => 99,
    'negative' => -17,
    'zero' => 0,
]
```

### Large Integers

```toml
# Underscores for readability
large = 1_000_000
very_large = 9_223_372_036_854_775_807
```

Parsed as:

```php
[
    'large' => 1000000,
    'very_large' => 9223372036854775807,
]
```

### Other Bases

```toml
hex = 0xDEADBEEF
octal = 0o755
binary = 0b11010110
```

Parsed as decimal integers:

```php
[
    'hex' => 3735928559,
    'octal' => 493,
    'binary' => 214,
]
```

## Floats

### Standard Floats

```toml
pi = 3.14159
negative = -0.01
```

Parsed as PHP `float`:

```php
[
    'pi' => 3.14159,
    'negative' => -0.01,
]
```

### Scientific Notation

```toml
exponent = 5e+22
negative_exp = -2e-2
```

Parsed as:

```php
[
    'exponent' => 5.0e+22,
    'negative_exp' => -0.02,
]
```

### Special Float Values

```toml
# Infinity
infinity = inf
positive_infinity = +inf
negative_infinity = -inf

# Not a number
not_a_number = nan
```

Parsed as:

```php
[
    'infinity' => INF,
    'positive_infinity' => INF,
    'negative_infinity' => -INF,
    'not_a_number' => NAN,
]
```

### Building Floats

When building, ensure decimal point is included:

```php
$builder->addValue('price', 19.99);   // price = 19.99
$builder->addValue('whole', 42.0);    // whole = 42.0
```

## Booleans

```toml
enabled = true
disabled = false
```

Parsed as PHP `bool`:

```php
[
    'enabled' => true,
    'disabled' => false,
]
```

When building:

```php
$builder->addValue('enabled', true);   // enabled = true
$builder->addValue('disabled', false); // disabled = false
```

## Dates and Times

### Offset Date-Time

```toml
odt1 = 1979-05-27T07:32:00Z
odt2 = 1979-05-27T00:32:00-07:00
odt3 = 1979-05-27T00:32:00.999999-07:00
```

Parsed as PHP `DateTime` objects with timezone:

```php
$date = $config['odt1'];
echo $date->format('Y-m-d H:i:s'); // "1979-05-27 07:32:00"
```

### Local Date-Time

```toml
ldt1 = 1979-05-27T07:32:00
ldt2 = 1979-05-27T00:32:00.999999
```

### Local Date

```toml
ld1 = 1979-05-27
```

### Local Time

```toml
lt1 = 07:32:00
lt2 = 00:32:00.999999
```

### Building Dates

```php
$builder->addValue('created', new DateTime('2024-01-15 10:30:00'));
// Output: created = 2024-01-15T10:30:00Z
```

## Arrays

### Homogeneous Arrays

All elements must be the same type:

```toml
integers = [1, 2, 3]
strings = ["red", "yellow", "green"]
floats = [1.1, 2.2, 3.3]
booleans = [true, false, true]
dates = [1979-05-27T07:32:00Z, 1980-01-01T00:00:00Z]
```

Parsed as PHP arrays:

```php
[
    'integers' => [1, 2, 3],
    'strings' => ['red', 'yellow', 'green'],
    'floats' => [1.1, 2.2, 3.3],
    'booleans' => [true, false, true],
    'dates' => [
        new DateTime('1979-05-27 07:32:00'),
        new DateTime('1980-01-01 00:00:00'),
    ],
]
```

### Nested Arrays

```toml
matrix = [[1, 2], [3, 4], [5, 6]]
mixed_types = [[1, 2], ["a", "b"], [1.1, 2.2]]
```

Parsed as:

```php
[
    'matrix' => [[1, 2], [3, 4], [5, 6]],
    'mixed_types' => [[1, 2], ['a', 'b'], [1.1, 2.2]],
]
```

### Arrays with Line Breaks

```toml
hosts = [
    "alpha",
    "beta",
    "gamma"
]
```

### Empty Arrays

```toml
empty = []
```

Parsed as:

```php
['empty' => []]
```

### Building Arrays

```php
$builder->addValue('ports', [8080, 8081, 8082]);
$builder->addValue('hosts', ['localhost', 'example.com']);
$builder->addValue('matrix', [[1, 2], [3, 4]]);
```

## Tables

### Basic Tables

```toml
[database]
host = "localhost"
port = 5432
enabled = true
```

Parsed as:

```php
[
    'database' => [
        'host' => 'localhost',
        'port' => 5432,
        'enabled' => true,
    ],
]
```

### Nested Tables

```toml
[application]
name = "MyApp"

[application.server]
host = "0.0.0.0"
port = 8080

[application.server.ssl]
enabled = true
```

Parsed as:

```php
[
    'application' => [
        'name' => 'MyApp',
        'server' => [
            'host' => '0.0.0.0',
            'port' => 8080,
            'ssl' => [
                'enabled' => true,
            ],
        ],
    ],
]
```

### Dotted Keys

Alternative to nested tables:

```toml
[application]
name = "MyApp"
server.host = "0.0.0.0"
server.port = 8080
```

Produces the same structure as nested tables.

### Building Tables

```php
$builder
    ->addTable('database')
    ->addValue('host', 'localhost')
    ->addValue('port', 5432)
    ->addTable('database.replica')
    ->addValue('host', 'replica.internal')
    ->addValue('port', 5432);
```

## Inline Tables

Compact single-line table format:

```toml
name = { first = "Tom", last = "Preston-Werner" }
point = { x = 1, y = 2 }
colors = { red = 255, green = 128, blue = 0 }
```

Parsed as:

```php
[
    'name' => [
        'first' => 'Tom',
        'last' => 'Preston-Werner',
    ],
    'point' => ['x' => 1, 'y' => 2],
    'colors' => ['red' => 255, 'green' => 128, 'blue' => 0],
]
```

## Array of Tables

### Basic Array of Tables

```toml
[[products]]
name = "Hammer"
sku = 738594937

[[products]]
name = "Nail"
sku = 284758393
```

Parsed as:

```php
[
    'products' => [
        ['name' => 'Hammer', 'sku' => 738594937],
        ['name' => 'Nail', 'sku' => 284758393],
    ],
]
```

### Nested Array of Tables

```toml
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
```

Parsed as:

```php
[
    'fruit' => [
        [
            'name' => 'apple',
            'variety' => [
                ['name' => 'red delicious'],
                ['name' => 'granny smith'],
            ],
        ],
        [
            'name' => 'banana',
            'variety' => [
                ['name' => 'plantain'],
            ],
        ],
    ],
]
```

### Building Array of Tables

```php
$builder
    ->addArrayOfTable('servers')
    ->addValue('name', 'web-1')
    ->addValue('ip', '10.0.1.1')
    ->addArrayOfTable('servers')
    ->addValue('name', 'web-2')
    ->addValue('ip', '10.0.1.2');
```

## Type Conversion Examples

### Parsing Example

```php
use Cline\Toml\Toml;

$toml = <<<'TOML'
# String types
title = "My Config"
path = 'C:\Windows\System32'

# Numeric types
count = 42
price = 19.99

# Boolean
enabled = true

# Date
created = 2024-01-15T10:30:00Z

# Array
tags = ["production", "database", "critical"]

# Table
[server]
host = "localhost"
port = 8080

# Array of tables
[[backups]]
name = "daily"
time = "02:00"

[[backups]]
name = "weekly"
time = "03:00"
TOML;

$config = Toml::parse($toml);

// Access values with proper PHP types
$title = $config['title'];                    // string
$count = $config['count'];                    // int
$price = $config['price'];                    // float
$enabled = $config['enabled'];                // bool
$created = $config['created'];                // DateTime
$tags = $config['tags'];                      // array
$server = $config['server'];                  // array
$backups = $config['backups'];                // array of arrays
```

### Building Example

```php
use Cline\Toml\TomlBuilder;

$builder = new TomlBuilder();

$toml = $builder
    ->addValue('title', 'My Config')
    ->addValue('path', '@C:\Windows\System32')
    ->addValue('count', 42)
    ->addValue('price', 19.99)
    ->addValue('enabled', true)
    ->addValue('created', new DateTime())
    ->addValue('tags', ['production', 'database', 'critical'])
    ->addTable('server')
    ->addValue('host', 'localhost')
    ->addValue('port', 8080)
    ->addArrayOfTable('backups')
    ->addValue('name', 'daily')
    ->addValue('time', '02:00')
    ->addArrayOfTable('backups')
    ->addValue('name', 'weekly')
    ->addValue('time', '03:00')
    ->getTomlString();
```

## Type Restrictions

### Array Homogeneity

Arrays must contain elements of the same type:

```toml
# Valid - all integers
valid = [1, 2, 3]

# Invalid - mixed types
invalid = [1, "two", 3.0]  # ParseException
```

### Key Restrictions

Keys must be:
- Non-empty
- Valid unquoted keys (letters, numbers, underscores, dashes) or quoted

```toml
# Valid keys
name = "value"
first-name = "value"
"special key" = "value"

# Invalid - empty key
"" = "value"  # EmptyKeyException
```

## Next Steps

- [Parsing TOML](/toml/parsing/) - Learn how to parse TOML files
- [Building TOML](/toml/building/) - Create TOML programmatically
- [Error Handling](/toml/error-handling/) - Handle type-related errors
