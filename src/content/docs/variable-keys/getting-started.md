---
title: Getting Started
description: Install and configure Variable Keys for clean, type-safe Laravel migrations with configurable primary keys and polymorphic relationships.
---

Eliminate repetitive match expressions in your Laravel migrations with type-safe Blueprint macros for variable primary keys, foreign keys, and polymorphic relationships.

## Requirements

Variable Keys v1.0 requires PHP 8.5+ and Laravel 12+.

## Installation

Install Variable Keys with composer:

```bash
composer require cline/variable-keys
```

The package will automatically register its service provider and Blueprint macros.

## Quick Example

Instead of writing verbose match expressions in migrations:

```php
use Cline\VariableKeys\Enums\PrimaryKeyType;

Schema::create('users', function (Blueprint $table) {
    // Before: verbose match expression
    match (PrimaryKeyType::ULID) {
        PrimaryKeyType::ULID => $table->ulid('id')->primary(),
        PrimaryKeyType::UUID => $table->uuid('id')->primary(),
        PrimaryKeyType::ID => $table->id(),
    };

    $table->string('name');
    $table->timestamps();
});
```

Use clean, type-safe macros:

```php
use Cline\VariableKeys\Enums\PrimaryKeyType;

Schema::create('users', function (Blueprint $table) {
    // After: clean macro
    $table->variablePrimaryKey(PrimaryKeyType::ULID);

    $table->string('name');
    $table->timestamps();
});
```

## Model Integration

Register models in your service provider to enable automatic primary key generation:

```php
use Cline\VariableKeys\Facades\VariableKeys;
use Cline\VariableKeys\Enums\PrimaryKeyType;
use App\Models\User;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        VariableKeys::map([
            User::class => [
                'primary_key_type' => PrimaryKeyType::ULID,
            ],
        ]);
    }
}
```

Add the trait to your models:

```php
use Cline\VariableKeys\Database\Concerns\HasVariablePrimaryKey;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    use HasVariablePrimaryKey;
}

## What's Included

Variable Keys provides three Blueprint macros and two enums:

### Enums

- **`PrimaryKeyType`** - ID, ULID, UUID
- **`MorphType`** - String, Numeric, UUID, ULID

### Blueprint Macros

- **`variablePrimaryKey()`** - Primary key columns
- **`variableForeignKey()`** - Foreign key columns
- **`variableMorphs()`** - Polymorphic relationship columns

## Next Steps

- [Primary Keys](primary-keys.md) - Configure variable primary keys
- [Foreign Keys](foreign-keys.md) - Manage foreign key relationships
- [Polymorphic Relations](polymorphic-relations.md) - Handle polymorphic relationships
- [Configuration Patterns](configuration-patterns.md) - Centralize key type configuration
