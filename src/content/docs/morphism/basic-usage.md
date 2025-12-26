---
title: Basic Usage
description: Working with polymorphic relationships using Morphism.
---

Working with polymorphic relationships using Morphism.

## Defining Morphs

### In Configuration

```php
// config/morphism.php
return [
    'morphs' => [
        'commentable' => [
            App\Models\Post::class,
            App\Models\Video::class,
        ],
    ],
];
```

### In Models

```php
use Cline\Morphism\Traits\HasMorphs;

class Comment extends Model
{
    use HasMorphs;

    public function commentable()
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    use HasMorphs;

    public function comments()
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

## Creating Relations

```php
// From parent model
$post = Post::find(1);
$post->comments()->create([
    'body' => 'Great content!',
    'user_id' => auth()->id(),
]);

// From child model
$comment = Comment::create([
    'body' => 'Nice video!',
    'user_id' => auth()->id(),
    'commentable_type' => 'video',  // Uses alias, not class name
    'commentable_id' => $video->id,
]);

// Associate
$comment = new Comment(['body' => 'Hello']);
$comment->commentable()->associate($post);
$comment->save();
```

## Querying Relations

```php
// Get all comments for a post
$comments = $post->comments;

// Get the parent model
$parent = $comment->commentable;

// Query by type
$postComments = Comment::where('commentable_type', 'post')->get();

// With morphed type
$postComments = Comment::whereMorphedTo('commentable', Post::class)->get();
```

## Multiple Morph Types

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
            App\Models\Category::class,
        ],
        'likeable' => [
            App\Models\Post::class,
            App\Models\Comment::class,
            App\Models\Photo::class,
        ],
    ],
];
```

## Morph Map Aliases

```php
// Custom aliases
return [
    'morphs' => [
        'commentable' => [
            'post' => App\Models\Post::class,
            'video' => App\Models\Video::class,
            'article' => App\Models\Article::class,
        ],
    ],
];

// Database stores 'post', 'video', 'article' instead of full class names
```

## Eager Loading

```php
// Eager load morph relation
$comments = Comment::with('commentable')->get();

// Constrained eager load
$comments = Comment::with(['commentable' => function ($query) {
    $query->where('published', true);
}])->get();

// Morph with specific types
$comments = Comment::with([
    'commentable' => function (MorphTo $morphTo) {
        $morphTo->morphWith([
            Post::class => ['author'],
            Video::class => ['channel'],
        ]);
    },
])->get();
```
