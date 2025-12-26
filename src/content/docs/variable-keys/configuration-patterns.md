---
title: Configuration Patterns
description: Runtime model registration and configuration strategies for Variable Keys.
---

Learn how to configure and manage variable key types using runtime registration for type safety and explicit model configuration.

## Runtime Registration

Variable Keys uses **strict runtime registration** via the `VariableKeys` facade. Every model using the `HasVariablePrimaryKey` trait **must** be explicitly registered.

### Basic Registration

Register models in your service provider's `boot()` method:

```php
use Cline\VariableKeys\Facades\VariableKeys;
use Cline\VariableKeys\Enums\PrimaryKeyType;
use App\Models\{User, Post, Comment};

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        VariableKeys::map([
            User::class => [
                'primary_key_type' => PrimaryKeyType::ULID,
            ],
            Post::class => [
                'primary_key_type' => PrimaryKeyType::ULID,
            ],
            Comment::class => [
                'primary_key_type' => PrimaryKeyType::ID,
            ],
        ]);
    }
}
```

### Type Safety

Registration requires **enum instances**, not strings:

```php
// ✅ Correct - uses enum
VariableKeys::map([
    User::class => ['primary_key_type' => PrimaryKeyType::ULID],
]);

// ❌ Wrong - strings not allowed
VariableKeys::map([
    User::class => ['primary_key_type' => 'ulid'], // Type error
]);
```

### Strict Validation

Models using the trait **must** be registered or an exception is thrown:

```php
use Cline\VariableKeys\Database\Concerns\HasVariablePrimaryKey;

class User extends Model
{
    use HasVariablePrimaryKey; // Must be registered
}

// If not registered:
$user = new User();
// Throws: ModelNotRegisteredException
```

## Package-Specific Registration

Packages should register their models in their own service providers:

```php
namespace Vendor\Package;

use Cline\VariableKeys\Facades\VariableKeys;
use Cline\VariableKeys\Enums\PrimaryKeyType;
use Vendor\Package\Models\{Ability, Role};

class PackageServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Register package models
        VariableKeys::map([
            Ability::class => [
                'primary_key_type' => PrimaryKeyType::from(
                    config('package.primary_key_type', 'id')
                ),
            ],
            Role::class => [
                'primary_key_type' => PrimaryKeyType::from(
                    config('package.primary_key_type', 'id')
                ),
            ],
        ]);
    }
}
```

## Environment-Based Configuration

Use environment variables with enum conversion:

**.env**
```env
APP_PRIMARY_KEY_TYPE=ulid
```

**Service Provider**
```php
use Cline\VariableKeys\Enums\PrimaryKeyType;

VariableKeys::map([
    User::class => [
        'primary_key_type' => PrimaryKeyType::from(
            env('APP_PRIMARY_KEY_TYPE', 'id')
        ),
    ],
]);
```

### Per-Model Environment Variables

```env
USER_PRIMARY_KEY_TYPE=ulid
ORGANIZATION_PRIMARY_KEY_TYPE=uuid
POST_PRIMARY_KEY_TYPE=id
```

```php
VariableKeys::map([
    User::class => [
        'primary_key_type' => PrimaryKeyType::from(
            env('USER_PRIMARY_KEY_TYPE', 'id')
        ),
    ],
    Organization::class => [
        'primary_key_type' => PrimaryKeyType::from(
            env('ORGANIZATION_PRIMARY_KEY_TYPE', 'id')
        ),
    ],
    Post::class => [
        'primary_key_type' => PrimaryKeyType::from(
            env('POST_PRIMARY_KEY_TYPE', 'id')
        ),
    ],
]);
```

## Configuration Helpers

Create helper methods for consistent configuration:

```php
class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        $this->registerVariableKeys();
    }

    protected function registerVariableKeys(): void
    {
        VariableKeys::map([
            User::class => $this->keyConfig('USER'),
            Organization::class => $this->keyConfig('ORGANIZATION'),
            Post::class => $this->keyConfig('POST'),
        ]);
    }

    protected function keyConfig(string $prefix): array
    {
        return [
            'primary_key_type' => PrimaryKeyType::from(
                env("{$prefix}_PRIMARY_KEY_TYPE", 'id')
            ),
        ];
    }
}
```

## Multi-Tenancy Patterns

### Tenant-Specific Key Types

```php
use Cline\VariableKeys\Facades\VariableKeys;
use Cline\VariableKeys\Enums\PrimaryKeyType;

class TenantServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Global models use ULIDs
        VariableKeys::map([
            Tenant::class => ['primary_key_type' => PrimaryKeyType::ULID],
        ]);

        // Tenant-scoped models use auto-increment
        if (tenancy()->initialized) {
            VariableKeys::map([
                Post::class => ['primary_key_type' => PrimaryKeyType::ID],
                Comment::class => ['primary_key_type' => PrimaryKeyType::ID],
            ]);
        }
    }
}
```

## Polymorphic Configuration

Register models with morph types for polymorphic relationships:

```php
use Cline\VariableKeys\Enums\{PrimaryKeyType, MorphType};

VariableKeys::map([
    User::class => [
        'primary_key_type' => PrimaryKeyType::ULID,
        'morph_type' => MorphType::ULID,
    ],
    Organization::class => [
        'primary_key_type' => PrimaryKeyType::UUID,
        'morph_type' => MorphType::UUID,
    ],
]);
```

## Testing Configuration

### Test-Specific Registration

Override registration in tests:

```php
use Cline\VariableKeys\Facades\VariableKeys;
use Cline\VariableKeys\Enums\PrimaryKeyType;

class UserTest extends TestCase
{
    protected function setUp(): void
    {
        parent::setUp();

        // Use auto-increment in tests for simplicity
        VariableKeys::clear();
        VariableKeys::map([
            User::class => ['primary_key_type' => PrimaryKeyType::ID],
        ]);
    }
}
```

### Feature Flags

```php
VariableKeys::map([
    User::class => [
        'primary_key_type' => Feature::active('use-ulids')
            ? PrimaryKeyType::ULID
            : PrimaryKeyType::ID,
    ],
]);
```

## Migration Consistency

Ensure migrations match model registration:

```php
// Migration
use Cline\VariableKeys\Enums\PrimaryKeyType;

Schema::create('users', function (Blueprint $table) {
    $table->variablePrimaryKey(PrimaryKeyType::ULID);
    $table->string('name');
    $table->timestamps();
});

// Model registration (must match)
VariableKeys::map([
    User::class => ['primary_key_type' => PrimaryKeyType::ULID],
]);
```

## Best Practices

### 1. Register in Service Providers

Always register in service provider `boot()` method, never in models:

```php
// ✅ Good
class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        VariableKeys::map([...]);
    }
}

// ❌ Bad - never in models
class User extends Model
{
    public function __construct()
    {
        VariableKeys::map([...]); // Don't do this
    }
}
```

### 2. Explicit Over Implicit

Explicitly register every model - no fallbacks or wildcards:

```php
// ✅ Good - explicit registration
VariableKeys::map([
    User::class => ['primary_key_type' => PrimaryKeyType::ULID],
    Post::class => ['primary_key_type' => PrimaryKeyType::ULID],
    Comment::class => ['primary_key_type' => PrimaryKeyType::ULID],
]);

// ❌ Bad - no wildcard support
VariableKeys::map([
    '*' => ['primary_key_type' => PrimaryKeyType::ULID], // Not supported
]);
```

### 3. Consistency Across Models

Use the same key type across related models:

```php
$keyType = PrimaryKeyType::ULID;

VariableKeys::map([
    User::class => ['primary_key_type' => $keyType],
    Post::class => ['primary_key_type' => $keyType],
    Comment::class => ['primary_key_type' => $keyType],
]);
```

### 4. Document Decisions

```php
/**
 * Primary Key Strategy: ULID
 *
 * Using ULIDs for:
 * - Distributed database support
 * - Time-ordered queries
 * - URL-safe identifiers
 * - No enumeration attacks
 */
VariableKeys::map([
    User::class => ['primary_key_type' => PrimaryKeyType::ULID],
]);
```
