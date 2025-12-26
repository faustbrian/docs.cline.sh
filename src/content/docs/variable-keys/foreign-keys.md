---
title: Foreign Keys
description: Use the variableForeignKey macro to create type-matched foreign key columns in Laravel migrations.
---

The `variableForeignKey()` macro creates foreign key columns that match your primary key type, eliminating the need for verbose match expressions.

## Basic Usage

```php
use Cline\VariableKeys\Enums\PrimaryKeyType;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('posts', function (Blueprint $table) {
    $table->variablePrimaryKey(PrimaryKeyType::ULID);
    $table->string('title');

    // Foreign key automatically matches ULID type
    $table->variableForeignKey('user_id', PrimaryKeyType::ULID)
          ->constrained('users')
          ->cascadeOnDelete();

    $table->timestamps();
});
```

## Method Signature

```php
$table->variableForeignKey(string $column, PrimaryKeyType $type)
```

**Parameters:**
- `$column` - The foreign key column name (e.g., `'user_id'`, `'category_id'`)
- `$type` - The primary key type enum value

**Returns:** A `ForeignKeyDefinition` instance for chaining constraints.

## Type Matching

The macro automatically selects the correct foreign key method based on the type:

| PrimaryKeyType | Foreign Key Method |
|----------------|-------------------|
| `ID` | `foreignId()` |
| `ULID` | `foreignUlid()` |
| `UUID` | `foreignUuid()` |

## Chaining Constraints

The macro returns a foreign key definition, allowing you to chain Laravel's standard constraint methods:

```php
$table->variableForeignKey('user_id', PrimaryKeyType::UUID)
      ->constrained('users')
      ->cascadeOnUpdate()
      ->cascadeOnDelete();
```

### Common Constraints

```php
// Reference specific table and column
$table->variableForeignKey('author_id', PrimaryKeyType::ULID)
      ->constrained('users', 'id');

// Cascade on delete
$table->variableForeignKey('category_id', PrimaryKeyType::ID)
      ->constrained()
      ->cascadeOnDelete();

// Set null on delete
$table->variableForeignKey('parent_id', PrimaryKeyType::UUID)
      ->nullable()
      ->constrained('posts')
      ->nullOnDelete();

// Restrict deletion
$table->variableForeignKey('organization_id', PrimaryKeyType::ULID)
      ->constrained()
      ->restrictOnDelete();

// Add index
$table->variableForeignKey('team_id', PrimaryKeyType::ID)
      ->constrained()
      ->index();
```

## Configuration-Driven Foreign Keys

Centralize your primary key type to ensure consistency:

```php
$primaryKeyType = PrimaryKeyType::tryFrom(config('database.primary_key_type'))
    ?? PrimaryKeyType::ID;

Schema::create('comments', function (Blueprint $table) use ($primaryKeyType) {
    $table->variablePrimaryKey($primaryKeyType);
    $table->text('body');

    $table->variableForeignKey('post_id', $primaryKeyType)
          ->constrained()
          ->cascadeOnDelete();

    $table->variableForeignKey('user_id', $primaryKeyType)
          ->constrained('users');

    $table->timestamps();
});
```

## Before and After

### Before: Verbose Match Expression

```php
match ($primaryKeyType) {
    PrimaryKeyType::ULID => $table->foreignUlid('user_id')
                                  ->constrained('users')
                                  ->cascadeOnDelete(),
    PrimaryKeyType::UUID => $table->foreignUuid('user_id')
                                  ->constrained('users')
                                  ->cascadeOnDelete(),
    PrimaryKeyType::ID => $table->foreignId('user_id')
                                ->constrained('users')
                                ->cascadeOnDelete(),
};
```

### After: Clean Macro

```php
$table->variableForeignKey('user_id', $primaryKeyType)
      ->constrained('users')
      ->cascadeOnDelete();
```

## Examples

### Blog System with ULIDs

```php
$primaryKeyType = PrimaryKeyType::ULID;

Schema::create('posts', function (Blueprint $table) use ($primaryKeyType) {
    $table->variablePrimaryKey($primaryKeyType);
    $table->string('title');
    $table->text('body');

    $table->variableForeignKey('author_id', $primaryKeyType)
          ->constrained('users', 'id')
          ->cascadeOnDelete();

    $table->variableForeignKey('category_id', $primaryKeyType)
          ->constrained('categories')
          ->restrictOnDelete();

    $table->timestamps();
});
```

### Multi-Tenancy with UUIDs

```php
$primaryKeyType = PrimaryKeyType::UUID;

Schema::create('projects', function (Blueprint $table) use ($primaryKeyType) {
    $table->variablePrimaryKey($primaryKeyType);
    $table->string('name');

    $table->variableForeignKey('tenant_id', $primaryKeyType)
          ->constrained('tenants')
          ->restrictOnDelete();

    $table->variableForeignKey('created_by', $primaryKeyType)
          ->constrained('users', 'id');

    $table->timestamps();
});
```

### Self-Referential Relationships

```php
Schema::create('categories', function (Blueprint $table) {
    $table->variablePrimaryKey(PrimaryKeyType::ID);
    $table->string('name');

    $table->variableForeignKey('parent_id', PrimaryKeyType::ID)
          ->nullable()
          ->constrained('categories')
          ->cascadeOnDelete();

    $table->timestamps();
});
```
