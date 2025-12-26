---
title: File System Assertions
description: Validate files, directories, and file permissions.
---

File system assertions validate files, directories, and file permissions.

## Available Assertions

### file()

Assert that a file exists.

```php
use Cline\Assert\Assertions\Assertion;

Assertion::file('/path/to/file.txt');
Assertion::file($configPath, 'Config file not found');
```

### directory()

Assert that a directory exists.

```php
Assertion::directory('/path/to/dir');
Assertion::directory($uploadsPath, 'Uploads directory not found');
```

### readable()

Assert that a file or directory is readable.

```php
Assertion::readable('/path/to/file.txt');
Assertion::readable($logFile, 'Cannot read log file');
```

### writeable()

Assert that a file or directory is writeable.

```php
Assertion::writeable('/path/to/file.txt');
Assertion::writeable($cacheDir, 'Cache directory is not writeable');
```

## Chaining File System Assertions

```php
use Cline\Assert\Assert;

Assert::that($configFile)
    ->string()
    ->notEmpty()
    ->file('Config file does not exist')
    ->readable('Config file is not readable');

Assert::that($uploadDir)
    ->directory('Upload directory missing')
    ->writeable('Upload directory is not writeable');
```

## Common Patterns

### Configuration File Validation

```php
$configPath = __DIR__ . '/config/app.php';

Assert::that($configPath)
    ->file('Configuration file not found')
    ->readable('Cannot read configuration file');

$config = require $configPath;
```

### Upload Directory Validation

```php
$uploadPath = storage_path('uploads');

Assert::that($uploadPath)
    ->directory('Upload directory does not exist')
    ->writeable('Cannot write to upload directory');
```

### Multiple Directory Validation

```php
$directories = [
    storage_path('cache'),
    storage_path('sessions'),
    storage_path('views'),
];

foreach ($directories as $dir) {
    Assert::that($dir)
        ->directory("Directory does not exist: {$dir}")
        ->writeable("Directory not writeable: {$dir}");
}
```

## Permission Checks

### Read and Write

```php
Assert::that($dataFile)
    ->file()
    ->readable('Cannot read data file')
    ->writeable('Cannot write to data file');
```

## Best Practices

### Check Existence First

```php
Assert::that($file)
    ->file('File does not exist')
    ->readable('File is not readable');
```

### Validate Before Operations

```php
Assert::that($sourceFile)->file()->readable();
Assert::that($destDir)->directory()->writeable();

copy($sourceFile, $destDir . '/' . basename($sourceFile));
```

### Use Absolute Paths

```php
// Use absolute path
Assertion::file(__DIR__ . '/config/app.php');

// Or use path helper
Assertion::file(config_path('app.php'));
```

## Application Examples

### Asset Loading

```php
public function loadAsset(string $name): string
{
    $assetPath = public_path("assets/{$name}");

    Assert::that($assetPath)
        ->file("Asset not found: {$name}")
        ->readable("Cannot read asset: {$name}");

    return file_get_contents($assetPath);
}
```

### Cache Management

```php
public function setupCache(): void
{
    $cacheDir = storage_path('framework/cache');

    if (!is_dir($cacheDir)) {
        mkdir($cacheDir, 0755, true);
    }

    Assert::that($cacheDir)
        ->directory('Failed to create cache directory')
        ->writeable('Cache directory must be writeable');
}
```

## Next Steps

- [String Assertions](/assert/string-assertions/) - Path validation
- [Custom Assertions](/assert/custom-assertions/) - Custom file validators
- [Lazy Assertions](/assert/lazy-assertions/) - Validate multiple paths
