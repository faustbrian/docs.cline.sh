---
title: Strict Enforcement
description: Type-safe polymorphic relationships with strict mode.
---

Type-safe polymorphic relationships with strict mode.

## Enabling Strict Mode

```php
// config/morphism.php
return [
    'strict' => true,

    'morphs' => [
        'commentable' => [
            'post' => App\Models\Post::class,
            'video' => App\Models\Video::class,
        ],
    ],
];
```

## What Strict Mode Does

### Prevents Invalid Types

```php
// With strict mode ON
$comment = new Comment();
$comment->commentable()->associate($product); // Throws exception!
// "Product is not a valid commentable type"

// Only configured types allowed
$comment->commentable()->associate($post); // Works
$comment->commentable()->associate($video); // Works
```

### Validates on Save

```php
// Even manual assignment is validated
$comment = new Comment();
$comment->commentable_type = 'product'; // Invalid
$comment->commentable_id = 1;
$comment->save(); // Throws MorphismException!
```

### Validates on Query

```php
// Invalid morph type in query
Comment::where('commentable_type', 'invalid')->get();
// Throws exception in strict mode
```

## Strict Mode Levels

```php
// config/morphism.php
return [
    // Full strict - validates everything
    'strict' => true,

    // Or granular control
    'strict' => [
        'save' => true,      // Validate on model save
        'query' => true,     // Validate in queries
        'associate' => true, // Validate on associate()
        'create' => true,    // Validate on create/make
    ],
];
```

## Exception Handling

```php
use Cline\Morphism\Exceptions\InvalidMorphTypeException;

try {
    $comment->commentable()->associate($invalidModel);
} catch (InvalidMorphTypeException $e) {
    // Handle invalid type
    $validTypes = $e->getValidTypes();
    $attemptedType = $e->getAttemptedType();
    $morphName = $e->getMorphName();
}
```

## Disabling Per-Operation

```php
use Cline\Morphism\Facades\Morphism;

// Temporarily disable strict mode
Morphism::withoutStrict(function () {
    $comment->commentable()->associate($anyModel);
});

// Or for specific morph
$comment->commentable()->withoutStrict()->associate($anyModel);
```

## Development vs Production

```php
// config/morphism.php
return [
    // Strict in development, permissive in production
    'strict' => env('APP_ENV') !== 'production',

    // Or always strict but log instead of throw in production
    'strict' => true,
    'strict_action' => env('APP_ENV') === 'production' ? 'log' : 'throw',
];
```

## Custom Validation

```php
use Cline\Morphism\Facades\Morphism;

Morphism::validateUsing('commentable', function ($model, $type) {
    // Custom validation logic
    if ($model instanceof Post && !$model->allow_comments) {
        return false;
    }
    return true;
});
```
