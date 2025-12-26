---
title: Config-Based Setup
description: Configure polymorphic relationships centrally.
---

Configure polymorphic relationships centrally.

## Configuration File

```php
// config/morphism.php
return [
    /*
    |--------------------------------------------------------------------------
    | Morph Maps
    |--------------------------------------------------------------------------
    |
    | Define all polymorphic relationship types and their allowed models.
    |
    */
    'morphs' => [
        'commentable' => [
            'post' => App\Models\Post::class,
            'video' => App\Models\Video::class,
            'article' => App\Models\Article::class,
        ],

        'taggable' => [
            'post' => App\Models\Post::class,
            'product' => App\Models\Product::class,
        ],

        'imageable' => [
            'user' => App\Models\User::class,
            'product' => App\Models\Product::class,
            'category' => App\Models\Category::class,
        ],
    ],

    /*
    |--------------------------------------------------------------------------
    | Strict Mode
    |--------------------------------------------------------------------------
    |
    | When enabled, only models listed in morphs can be used in relations.
    |
    */
    'strict' => true,

    /*
    |--------------------------------------------------------------------------
    | Auto-Discovery
    |--------------------------------------------------------------------------
    |
    | Automatically discover morph maps from model attributes.
    |
    */
    'auto_discover' => false,
];
```

## Publishing Configuration

```bash
php artisan vendor:publish --tag=morphism-config
```

## Environment-Based Config

```php
// config/morphism.php
return [
    'strict' => env('MORPHISM_STRICT', true),

    'morphs' => [
        'commentable' => array_filter([
            'post' => App\Models\Post::class,
            'video' => App\Models\Video::class,
            // Only in non-production
            'test' => app()->environment('local') ? App\Models\TestModel::class : null,
        ]),
    ],
];
```

## Dynamic Configuration

```php
use Cline\Morphism\Facades\Morphism;

// Add morphs at runtime
Morphism::register('commentable', [
    'article' => App\Models\Article::class,
]);

// Get all morphs for a type
$morphs = Morphism::getMorphs('commentable');

// Check if type is registered
Morphism::has('commentable'); // true

// Get alias for class
Morphism::getAlias(Post::class, 'commentable'); // 'post'

// Get class for alias
Morphism::getClass('post', 'commentable'); // App\Models\Post::class
```

## Model-Based Configuration

```php
use Cline\Morphism\Attributes\Morphable;

#[Morphable('commentable', alias: 'post')]
#[Morphable('taggable', alias: 'post')]
class Post extends Model
{
    // This model can be used in commentable and taggable relations
}

// Enable auto-discovery
// config/morphism.php
return [
    'auto_discover' => true,
    'discover_paths' => [
        app_path('Models'),
    ],
];
```

## Validation

```php
// Validate configuration on boot (development)
Morphism::validate();

// Check for issues
$issues = Morphism::diagnose();
foreach ($issues as $issue) {
    logger()->warning($issue);
}
```
