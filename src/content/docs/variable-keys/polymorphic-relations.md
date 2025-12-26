---
title: Polymorphic Relations
description: Use the variableMorphs macro to create configurable polymorphic relationship columns in Laravel migrations.
---

The `variableMorphs()` macro eliminates verbose match expressions when defining polymorphic relationship columns, making your migrations cleaner and type-safe.

## Basic Usage

```php
use Cline\VariableKeys\Enums\MorphType;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->text('body');

    // Polymorphic relationship
    $table->variableMorphs('commentable', MorphType::ULID);

    $table->timestamps();
});
```

## Method Signature

```php
$table->variableMorphs(string $name, MorphType $type, bool $nullable = false)
```

**Parameters:**
- `$name` - The morph relationship name (e.g., `'commentable'`, `'taggable'`)
- `$type` - The morph type enum value
- `$nullable` - Whether the relationship is optional (default: `false`)

## Available Morph Types

### String (Auto-Detected)

Laravel's default morphs that automatically detect the appropriate column type.

```php
$table->variableMorphs('commentable', MorphType::String);
```

**Equivalent to:**
```php
$table->morphs('commentable');
```

### Numeric (Integer IDs)

Explicitly use integer foreign keys for the morph relationship. Best when models use auto-incrementing integer primary keys.

```php
$table->variableMorphs('taggable', MorphType::Numeric);
```

**Equivalent to:**
```php
$table->numericMorphs('taggable');
```

### UUID

Use 36-character UUIDs for the morph relationship.

```php
$table->variableMorphs('imageable', MorphType::UUID);
```

**Equivalent to:**
```php
$table->uuidMorphs('imageable');
```

### ULID

Use 26-character ULIDs for the morph relationship.

```php
$table->variableMorphs('attachable', MorphType::ULID);
```

**Equivalent to:**
```php
$table->ulidMorphs('attachable');
```

## Nullable Relationships

Set the third parameter to `true` for optional polymorphic relationships:

```php
$table->variableMorphs('parent', MorphType::UUID, nullable: true);
```

This creates nullable columns for both the type and ID:

```php
// Equivalent to:
$table->nullableUuidMorphs('parent');
```

## Configuration-Driven Morphs

Centralize your morph type configuration:

```php
// config/database.php
return [
    'morph_type' => env('DB_MORPH_TYPE', 'string'),
];
```

Use in migrations:

```php
use Cline\VariableKeys\Enums\MorphType;

$morphType = MorphType::tryFrom(config('database.morph_type'))
    ?? MorphType::String;

Schema::create('images', function (Blueprint $table) use ($morphType) {
    $table->id();
    $table->string('url');

    $table->variableMorphs('imageable', $morphType);

    $table->timestamps();
});
```

## Before and After

### Before: Verbose Match Expression

```php
match ($morphType) {
    MorphType::ULID => $table->ulidMorphs('commentable'),
    MorphType::UUID => $table->uuidMorphs('commentable'),
    MorphType::Numeric => $table->numericMorphs('commentable'),
    MorphType::String => $table->morphs('commentable'),
};
```

### After: Clean Macro

```php
$table->variableMorphs('commentable', $morphType);
```

## Examples

### Comments System with ULIDs

```php
use Cline\VariableKeys\Enums\MorphType;

Schema::create('comments', function (Blueprint $table) {
    $table->ulid('id')->primary();
    $table->text('body');
    $table->foreignUlid('user_id')->constrained()->cascadeOnDelete();

    // Comments can belong to posts, videos, etc.
    $table->variableMorphs('commentable', MorphType::ULID);

    $table->timestamps();
});
```

### Tagging System with Numeric IDs

```php
Schema::create('taggables', function (Blueprint $table) {
    $table->id();

    $table->foreignId('tag_id')->constrained()->cascadeOnDelete();

    // Tags can be attached to posts, products, etc.
    $table->variableMorphs('taggable', MorphType::Numeric);

    $table->unique(['tag_id', 'taggable_type', 'taggable_id']);
    $table->timestamps();
});
```

### Media Library with UUIDs

```php
Schema::create('media', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->string('file_name');
    $table->string('mime_type');
    $table->unsignedBigInteger('size');

    // Media can be attached to users, posts, products, etc.
    $table->variableMorphs('mediable', MorphType::UUID);

    $table->timestamps();
});
```

### Activity Log with Nullable Subject

```php
Schema::create('activity_log', function (Blueprint $table) {
    $table->id();
    $table->string('description');

    $table->variableMorphs('causer', MorphType::ULID);

    // Subject is optional (some activities don't have a subject)
    $table->variableMorphs('subject', MorphType::ULID, nullable: true);

    $table->timestamps();
});
```

### Notification System

```php
Schema::create('notifications', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->string('type');
    $table->text('data');

    // The entity that can receive notifications
    $table->variableMorphs('notifiable', MorphType::UUID);

    $table->timestamp('read_at')->nullable();
    $table->timestamps();
});
```

## Choosing the Right Morph Type

| Type | Use When | Characteristics |
|------|----------|----------------|
| **String** | Default, flexible setup | Auto-detects column type |
| **Numeric** | Integer primary keys | Efficient, traditional IDs |
| **UUID** | UUID primary keys | Globally unique, random |
| **ULID** | ULID primary keys | Sortable, time-ordered |

## Relationship Configuration

After creating the morph columns in your migration, define the relationship in your models:

```php
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Comment extends Model
{
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```
