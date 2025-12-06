---
title: Testing with Morpheus
description: Testing polymorphic relationships with the Morpheus helper.
---

Testing polymorphic relationships with the Morpheus helper.

## Basic Testing

```php
use Cline\Morphism\Testing\Morpheus;
use Tests\TestCase;

class CommentTest extends TestCase
{
    public function test_comment_can_belong_to_post(): void
    {
        $post = Post::factory()->create();
        $comment = Comment::factory()->create([
            'commentable_type' => 'post',
            'commentable_id' => $post->id,
        ]);

        $this->assertInstanceOf(Post::class, $comment->commentable);
        $this->assertTrue($comment->commentable->is($post));
    }
}
```

## Using Morpheus

```php
use Cline\Morphism\Testing\Morpheus;

class MorphismTest extends TestCase
{
    public function test_morph_map_is_configured(): void
    {
        Morpheus::assertMorphMapContains('commentable', [
            'post' => Post::class,
            'video' => Video::class,
        ]);
    }

    public function test_model_has_correct_morph_alias(): void
    {
        Morpheus::assertMorphAlias(Post::class, 'post');
        Morpheus::assertMorphAlias(Video::class, 'video');
    }
}
```

## Testing Strict Mode

```php
use Cline\Morphism\Exceptions\InvalidMorphTypeException;

class StrictModeTest extends TestCase
{
    public function test_invalid_morph_type_throws_exception(): void
    {
        $this->expectException(InvalidMorphTypeException::class);

        $comment = new Comment();
        $comment->commentable_type = 'invalid_type';
        $comment->commentable_id = 1;
        $comment->save();
    }

    public function test_valid_morph_type_saves(): void
    {
        $post = Post::factory()->create();

        $comment = Comment::factory()->make();
        $comment->commentable()->associate($post);
        $comment->save();

        $this->assertDatabaseHas('comments', [
            'id' => $comment->id,
            'commentable_type' => 'post',
            'commentable_id' => $post->id,
        ]);
    }
}
```

## Factory States

```php
// CommentFactory.php
class CommentFactory extends Factory
{
    public function forPost(Post $post = null): static
    {
        return $this->state(fn() => [
            'commentable_type' => 'post',
            'commentable_id' => $post?->id ?? Post::factory(),
        ]);
    }

    public function forVideo(Video $video = null): static
    {
        return $this->state(fn() => [
            'commentable_type' => 'video',
            'commentable_id' => $video?->id ?? Video::factory(),
        ]);
    }
}

// Usage
$comment = Comment::factory()->forPost()->create();
$comment = Comment::factory()->forVideo($video)->create();
```

## Testing Relationships

```php
class RelationshipTest extends TestCase
{
    public function test_post_has_many_comments(): void
    {
        $post = Post::factory()
            ->has(Comment::factory()->count(3), 'comments')
            ->create();

        $this->assertCount(3, $post->comments);
        $this->assertContainsOnlyInstancesOf(Comment::class, $post->comments);
    }

    public function test_comment_belongs_to_morphable(): void
    {
        $post = Post::factory()->create();
        $video = Video::factory()->create();

        $postComment = Comment::factory()->forPost($post)->create();
        $videoComment = Comment::factory()->forVideo($video)->create();

        $this->assertInstanceOf(Post::class, $postComment->commentable);
        $this->assertInstanceOf(Video::class, $videoComment->commentable);
    }
}
```

## Mocking Morphs

```php
use Cline\Morphism\Testing\Morpheus;

class MockedMorphTest extends TestCase
{
    public function test_with_mocked_morphs(): void
    {
        Morpheus::fake([
            'commentable' => [
                'mock' => MockModel::class,
            ],
        ]);

        // Test with mocked morph map
        $this->assertEquals(MockModel::class, Morpheus::getClass('mock', 'commentable'));
    }

    protected function tearDown(): void
    {
        Morpheus::restore();
        parent::tearDown();
    }
}
```
