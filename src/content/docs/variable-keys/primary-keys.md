---
title: Primary Keys
description: Use the variablePrimaryKey macro to create configurable primary key columns in Laravel migrations.
---

The `variablePrimaryKey()` macro replaces verbose match expressions with a clean, type-safe method for creating primary key columns based on your application's configuration.

## Basic Usage

```php
use Cline\VariableKeys\Enums\PrimaryKeyType;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('users', function (Blueprint $table) {
    $table->variablePrimaryKey(PrimaryKeyType::ID);
    $table->string('name');
    $table->timestamps();
});
```

## Available Primary Key Types

### Auto-Incrementing Integer (ID)

Traditional sequential numeric IDs. Simplest option but reveals record count and ordering.

```php
$table->variablePrimaryKey(PrimaryKeyType::ID);
```

**Equivalent to:**
```php
$table->id();
```

### ULID (Universally Unique Lexicographically Sortable Identifier)

26-character case-insensitive strings that are time-ordered and globally unique. Better performance than UUIDs while maintaining sortability.

```php
$table->variablePrimaryKey(PrimaryKeyType::ULID);
```

**Equivalent to:**
```php
$table->ulid('id')->primary();
```

### UUID (Universally Unique Identifier)

36-character strings (32 hex digits plus 4 hyphens) that are globally unique and cryptographically random. Use when global uniqueness is required without chronological ordering.

```php
$table->variablePrimaryKey(PrimaryKeyType::UUID);
```

**Equivalent to:**
```php
$table->uuid('id')->primary();
```

## Custom Column Name

Specify a custom column name as the second parameter:

```php
$table->variablePrimaryKey(PrimaryKeyType::ULID, 'user_id');
```

## Configuration-Driven Primary Keys

Centralize your primary key type configuration:

```php
// config/database.php
return [
    'primary_key_type' => env('DB_PRIMARY_KEY_TYPE', 'id'),
];
```

Then use it in migrations:

```php
use Cline\VariableKeys\Enums\PrimaryKeyType;

$primaryKeyType = PrimaryKeyType::tryFrom(config('database.primary_key_type'))
    ?? PrimaryKeyType::ID;

Schema::create('users', function (Blueprint $table) use ($primaryKeyType) {
    $table->variablePrimaryKey($primaryKeyType);
    $table->string('name');
    $table->timestamps();
});
```

## Choosing the Right Type

| Type | Use When | Characteristics |
|------|----------|----------------|
| **ID** | Traditional apps, simple requirements | Sequential, predictable, efficient |
| **ULID** | Distributed systems, time-ordered data | Sortable, globally unique, URL-safe |
| **UUID** | Maximum randomness, global uniqueness | Cryptographically random, non-sequential |

## Examples

### Simple ID

```php
Schema::create('posts', function (Blueprint $table) {
    $table->variablePrimaryKey(PrimaryKeyType::ID);
    $table->string('title');
    $table->text('body');
    $table->timestamps();
});
```

### ULID for Distributed System

```php
Schema::create('events', function (Blueprint $table) {
    $table->variablePrimaryKey(PrimaryKeyType::ULID);
    $table->string('type');
    $table->json('payload');
    $table->timestamps();
});
```

### UUID for External API Integration

```php
Schema::create('api_resources', function (Blueprint $table) {
    $table->variablePrimaryKey(PrimaryKeyType::UUID);
    $table->string('external_id');
    $table->json('data');
    $table->timestamps();
});
```
