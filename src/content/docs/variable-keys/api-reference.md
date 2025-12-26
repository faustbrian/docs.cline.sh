---
title: API Reference
description: Complete reference for all Variable Keys enums, macros, and methods.
---

Complete API documentation for Variable Keys enums and Blueprint macros.

## Enums

### PrimaryKeyType

Represents the available primary key generation strategies for Laravel migrations.

**Namespace:** `Cline\VariableKeys\Enums\PrimaryKeyType`

#### Cases

##### ID

```php
PrimaryKeyType::ID
```

Traditional auto-incrementing integer primary keys. Simplest option but reveals record count and ordering.

**String value:** `'id'`

##### ULID

```php
PrimaryKeyType::ULID
```

Universally Unique Lexicographically Sortable Identifiers. 26-character case-insensitive strings that are time-ordered and globally unique.

**String value:** `'ulid'`

##### UUID

```php
PrimaryKeyType::UUID
```

Universally Unique Identifiers (version 4). 36-character strings (32 hex digits plus 4 hyphens) that are globally unique and cryptographically random.

**String value:** `'uuid'`

#### Usage

```php
use Cline\VariableKeys\Enums\PrimaryKeyType;

// Direct usage
$type = PrimaryKeyType::ULID;

// From string value
$type = PrimaryKeyType::tryFrom('ulid') ?? PrimaryKeyType::ID;

// From configuration
$type = PrimaryKeyType::tryFrom(config('database.primary_key_type'))
    ?? PrimaryKeyType::ID;
```

---

### MorphType

Represents the available polymorphic relationship types for Laravel migrations.

**Namespace:** `Cline\VariableKeys\Enums\MorphType`

#### Cases

##### String

```php
MorphType::String
```

Standard polymorphic relationship with auto-detected IDs. Uses Laravel's default morphs() method.

**String value:** `'string'`

##### Numeric

```php
MorphType::Numeric
```

Polymorphic relationship with numeric (integer) IDs. Suitable for auto-incrementing integer primary keys.

**String value:** `'numeric'`

##### UUID

```php
MorphType::UUID
```

Polymorphic relationship with UUID identifiers. Uses 36-character UUIDs.

**String value:** `'uuid'`

##### ULID

```php
MorphType::ULID
```

Polymorphic relationship with ULID identifiers. Uses 26-character ULIDs.

**String value:** `'ulid'`

#### Usage

```php
use Cline\VariableKeys\Enums\MorphType;

// Direct usage
$type = MorphType::ULID;

// From string value
$type = MorphType::tryFrom('ulid') ?? MorphType::String;

// From configuration
$type = MorphType::tryFrom(config('database.morph_type'))
    ?? MorphType::String;
```

---

## Blueprint Macros

All macros are automatically registered when the package is installed. They extend Laravel's `Illuminate\Database\Schema\Blueprint` class.

### variablePrimaryKey()

Creates a primary key column based on the specified type.

#### Signature

```php
public function variablePrimaryKey(
    PrimaryKeyType $type,
    string $column = 'id'
): \Illuminate\Database\Schema\ColumnDefinition
```

#### Parameters

**`$type`** (PrimaryKeyType, required)
- The primary key type to use
- Must be one of: `PrimaryKeyType::ID`, `PrimaryKeyType::ULID`, or `PrimaryKeyType::UUID`

**`$column`** (string, optional)
- The column name for the primary key
- Default: `'id'`

#### Returns

`ColumnDefinition` - A column definition instance for the created primary key

#### Examples

```php
// Auto-incrementing integer ID
$table->variablePrimaryKey(PrimaryKeyType::ID);

// ULID primary key
$table->variablePrimaryKey(PrimaryKeyType::ULID);

// UUID primary key
$table->variablePrimaryKey(PrimaryKeyType::UUID);

// Custom column name
$table->variablePrimaryKey(PrimaryKeyType::ULID, 'user_id');
```

#### Equivalent Laravel Methods

```php
// PrimaryKeyType::ID
$table->id();
$table->id('user_id');

// PrimaryKeyType::ULID
$table->ulid('id')->primary();
$table->ulid('user_id')->primary();

// PrimaryKeyType::UUID
$table->uuid('id')->primary();
$table->uuid('user_id')->primary();
```

---

### variableForeignKey()

Creates a foreign key column that matches the specified primary key type.

#### Signature

```php
public function variableForeignKey(
    string $column,
    PrimaryKeyType $type
): \Illuminate\Database\Schema\ForeignKeyDefinition
```

#### Parameters

**`$column`** (string, required)
- The foreign key column name
- Example: `'user_id'`, `'category_id'`, `'parent_id'`

**`$type`** (PrimaryKeyType, required)
- The primary key type to match
- Must be one of: `PrimaryKeyType::ID`, `PrimaryKeyType::ULID`, or `PrimaryKeyType::UUID`

#### Returns

`ForeignKeyDefinition` - A foreign key definition instance for chaining constraints

#### Examples

```php
// Integer foreign key
$table->variableForeignKey('user_id', PrimaryKeyType::ID)
      ->constrained()
      ->cascadeOnDelete();

// ULID foreign key
$table->variableForeignKey('author_id', PrimaryKeyType::ULID)
      ->constrained('users', 'id');

// UUID foreign key with nullable
$table->variableForeignKey('parent_id', PrimaryKeyType::UUID)
      ->nullable()
      ->constrained('posts')
      ->nullOnDelete();
```

#### Chainable Methods

The returned `ForeignKeyDefinition` supports all standard Laravel foreign key methods:

```php
->constrained(?string $table = null, ?string $column = 'id')
->cascadeOnUpdate()
->restrictOnUpdate()
->cascadeOnDelete()
->restrictOnDelete()
->nullOnDelete()
->noActionOnDelete()
->index()
->nullable()
```

#### Equivalent Laravel Methods

```php
// PrimaryKeyType::ID
$table->foreignId('user_id');

// PrimaryKeyType::ULID
$table->foreignUlid('user_id');

// PrimaryKeyType::UUID
$table->foreignUuid('user_id');
```

---

### variableMorphs()

Creates polymorphic relationship columns (type and ID) based on the specified morph type.

#### Signature

```php
public function variableMorphs(
    string $name,
    MorphType $type,
    bool $nullable = false
): void
```

#### Parameters

**`$name`** (string, required)
- The morph relationship name
- Creates `{name}_type` and `{name}_id` columns
- Example: `'commentable'` creates `commentable_type` and `commentable_id`

**`$type`** (MorphType, required)
- The morph type to use
- Must be one of: `MorphType::String`, `MorphType::Numeric`, `MorphType::UUID`, or `MorphType::ULID`

**`$nullable`** (bool, optional)
- Whether the relationship columns should be nullable
- Default: `false`
- When `true`, uses Laravel's `nullableXxxMorphs()` methods

#### Returns

`void` - Creates the morph columns but returns nothing

#### Examples

```php
// String morphs (auto-detected)
$table->variableMorphs('commentable', MorphType::String);
// Creates: commentable_type (string), commentable_id (unsignedBigInteger)

// Numeric morphs
$table->variableMorphs('taggable', MorphType::Numeric);
// Creates: taggable_type (string), taggable_id (unsignedBigInteger)

// UUID morphs
$table->variableMorphs('imageable', MorphType::UUID);
// Creates: imageable_type (string), imageable_id (char(36))

// ULID morphs
$table->variableMorphs('attachable', MorphType::ULID);
// Creates: attachable_type (string), attachable_id (char(26))

// Nullable relationship
$table->variableMorphs('parent', MorphType::ULID, nullable: true);
// Creates nullable versions of both columns
```

#### Equivalent Laravel Methods

```php
// MorphType::String
$table->morphs('commentable');
$table->nullableMorphs('commentable'); // when nullable: true

// MorphType::Numeric
$table->numericMorphs('commentable');
$table->nullableNumericMorphs('commentable'); // when nullable: true

// MorphType::UUID
$table->uuidMorphs('commentable');
$table->nullableUuidMorphs('commentable'); // when nullable: true

// MorphType::ULID
$table->ulidMorphs('commentable');
$table->nullableUlidMorphs('commentable'); // when nullable: true
```

---

## Type Conversion

### From String to Enum

Both enums support `tryFrom()` for safe conversion from string values:

```php
use Cline\VariableKeys\Enums\PrimaryKeyType;
use Cline\VariableKeys\Enums\MorphType;

// Returns enum case or null
$primaryKey = PrimaryKeyType::tryFrom('ulid');
// Result: PrimaryKeyType::ULID

$morph = MorphType::tryFrom('invalid');
// Result: null

// With fallback
$primaryKey = PrimaryKeyType::tryFrom($value) ?? PrimaryKeyType::ID;
$morph = MorphType::tryFrom($value) ?? MorphType::String;
```

### Enum to String

Access the backing string value:

```php
$type = PrimaryKeyType::ULID;
echo $type->value; // "ulid"

$morph = MorphType::UUID;
echo $morph->value; // "uuid"
```

---

## Service Provider

The package automatically registers its service provider when installed via composer.

**Class:** `Cline\VariableKeys\VariableKeysServiceProvider`

**Namespace:** `Cline\VariableKeys`

### Registered Macros

The service provider registers the following Blueprint macros during boot:

1. `variablePrimaryKey()`
2. `variableForeignKey()`
3. `variableMorphs()`

No manual registration is required.
