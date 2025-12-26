---
title: Collections
description: Traits for getting collections of enum names, values, and options.
---

Traits for getting collections of enum names, values, and options.

## Names

Returns a list of case names in the enum.

### Setup

```php
use Cline\Enums\Concerns\Names;

enum TaskStatus: int
{
    use Names;

    case Incomplete = 0;
    case Completed = 1;
    case Canceled = 2;
}

enum Role
{
    use Names;

    case Administrator;
    case Subscriber;
    case Guest;
}
```

### Usage

```php
TaskStatus::names(); // ['Incomplete', 'Completed', 'Canceled']
Role::names(); // ['Administrator', 'Subscriber', 'Guest']
```

## Values

Returns a list of case values for backed enums, or case names for pure enums.

### Setup

```php
use Cline\Enums\Concerns\Values;

enum TaskStatus: int
{
    use Values;

    case Incomplete = 0;
    case Completed = 1;
    case Canceled = 2;
}

enum Role
{
    use Values;

    case Administrator;
    case Subscriber;
    case Guest;
}
```

### Usage

```php
// Backed enum returns values
TaskStatus::values(); // [0, 1, 2]

// Pure enum returns names (same as ::names())
Role::values(); // ['Administrator', 'Subscriber', 'Guest']
```

## Options

Returns an associative array of case names and values for form selects and validation.

### Setup

```php
use Cline\Enums\Concerns\Options;

enum TaskStatus: int
{
    use Options;

    case Incomplete = 0;
    case Completed = 1;
    case Canceled = 2;
}

enum Role
{
    use Options;

    case Administrator;
    case Subscriber;
    case Guest;
}
```

### Basic Usage

```php
// Backed enum returns name => value pairs
TaskStatus::options(); // ['Incomplete' => 0, 'Completed' => 1, 'Canceled' => 2]

// Pure enum returns list of names
Role::options(); // ['Administrator', 'Subscriber', 'Guest']
```

### String Options

Generate string representations of your enum options with `stringOptions()`:

```php
// Custom format with callback and glue
TaskStatus::stringOptions(fn ($name, $value) => "$name => $value", ', ');
// Returns: "Incomplete => 0, Completed => 1, Canceled => 2"
```

For pure enums, both `$name` and `$value` in the callback are the same.

### HTML Options

By default, `stringOptions()` generates HTML `<option>` tags:

```php
// Backed enum generates option tags with values
TaskStatus::stringOptions();
// <option value="0">Incomplete</option>
// <option value="1">Completed</option>
// <option value="2">Canceled</option>

// Pure enum uses name as both value and label
Role::stringOptions();
// <option value="Administrator">Administrator</option>
// <option value="Subscriber">Subscriber</option>
// <option value="Guest">Guest</option>
```

The method automatically converts case names to human-readable format (e.g., `INCOMPLETE` becomes `Incomplete`).

## Use Cases

### Form Selects

```php
<select name="status">
    <?php foreach (TaskStatus::options() as $name => $value): ?>
        <option value="<?= $value ?>"><?= $name ?></option>
    <?php endforeach; ?>
</select>
```

### Validation Rules

```php
$rules = [
    'status' => ['required', Rule::in(TaskStatus::values())],
];
```

### API Documentation

```php
/**
 * @param int $status One of: {TaskStatus::stringOptions(fn($n, $v) => $v, ', ')}
 */
public function updateStatus(int $status): void
{
    // ...
}
```
