---
title: Migrations
description: Database migrations for polymorphic relationships.
---

Database migrations for polymorphic relationships.

## Basic Migration

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('comments', function (Blueprint $table) {
            $table->id();
            $table->text('body');
            $table->foreignId('user_id')->constrained();

            // Polymorphic columns
            $table->morphs('commentable');
            // Creates: commentable_type (string), commentable_id (bigint)
            // Creates: index on both columns

            $table->timestamps();
        });
    }
};
```

## Nullable Morphs

```php
Schema::create('images', function (Blueprint $table) {
    $table->id();
    $table->string('path');

    // Nullable polymorphic relation
    $table->nullableMorphs('imageable');

    $table->timestamps();
});
```

## UUID Morphs

```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->text('body');

    // For UUID primary keys
    $table->uuidMorphs('commentable');
    // Creates: commentable_type (string), commentable_id (uuid)

    $table->timestamps();
});

// Nullable UUID morphs
$table->nullableUuidMorphs('commentable');
```

## ULID Morphs

```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->text('body');

    // For ULID primary keys
    $table->ulidMorphs('commentable');

    $table->timestamps();
});
```

## Custom Column Types

```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->text('body');

    // Custom morph columns
    $table->string('commentable_type', 50); // Shorter type column
    $table->unsignedBigInteger('commentable_id');
    $table->index(['commentable_type', 'commentable_id']);

    $table->timestamps();
});
```

## Adding Morphs to Existing Table

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::table('existing_table', function (Blueprint $table) {
            $table->nullableMorphs('taggable');
        });
    }

    public function down(): void
    {
        Schema::table('existing_table', function (Blueprint $table) {
            $table->dropMorphs('taggable');
        });
    }
};
```

## Migrating Morph Types

```php
// Migration to convert class names to aliases
return new class extends Migration
{
    public function up(): void
    {
        // Update existing records to use aliases
        DB::table('comments')
            ->where('commentable_type', 'App\\Models\\Post')
            ->update(['commentable_type' => 'post']);

        DB::table('comments')
            ->where('commentable_type', 'App\\Models\\Video')
            ->update(['commentable_type' => 'video']);
    }
};
```

## Indexing Strategies

```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->morphs('commentable');

    // Additional indexes for common queries
    $table->index(['commentable_type', 'created_at']);
    $table->index(['commentable_id', 'commentable_type']);
});
```
