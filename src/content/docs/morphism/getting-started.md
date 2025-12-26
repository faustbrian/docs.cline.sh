---
title: Getting Started
description: Install and start using Morphism for polymorphic relations in Laravel.
---

Morphism provides enhanced polymorphic relationship management for Laravel with strict type enforcement, automatic morphing, and cleaner configuration.

## Installation

```bash
composer require cline/morphism
```

## Basic Setup

### Service Provider

```php
// config/app.php
'providers' => [
    Cline\Morphism\MorphismServiceProvider::class,
],
```

### Configuration

```bash
php artisan vendor:publish --tag=morphism-config
```

```php
// config/morphism.php
return [
    'morphs' => [
        'commentable' => [
            App\Models\Post::class,
            App\Models\Video::class,
            App\Models\Article::class,
        ],
        'taggable' => [
            App\Models\Post::class,
            App\Models\Product::class,
        ],
    ],
];
```

## Basic Usage

```php
use App\Models\Comment;
use App\Models\Post;

// Create polymorphic relation
$post = Post::find(1);
$comment = new Comment(['body' => 'Great post!']);
$post->comments()->save($comment);

// Query polymorphic relation
$comments = Comment::where('commentable_type', Post::class)->get();
```

## Why Morphism?

### Problem: Default Laravel Morphs

```php
// Without Morphism, morph maps use full class names
// commentable_type = "App\Models\Post"

// This breaks if you rename/move classes
// Couples database to PHP namespaces
```

### Solution: Morphism

```php
// With Morphism, morph maps use clean aliases
// commentable_type = "post"

// Rename/move classes freely
// Database stays clean
// Type enforcement prevents invalid relations
```

## Next Steps

- [Basic Usage](/morphism/basic-usage/) - Working with morphs
- [Config-Based Setup](/morphism/config-based-setup/) - Configuration options
- [Strict Enforcement](/morphism/strict-enforcement/) - Type safety
- [Migrations](/morphism/migrations/) - Database setup
- [Testing](/morphism/testing/) - Testing with Morpheus
