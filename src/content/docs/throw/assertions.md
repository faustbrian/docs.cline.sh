---
title: Assertions
description: Use the ensure() helper for fluent, readable guard clauses that throw exceptions or abort HTTP requests.
---

Use the `ensure()` helper for fluent, readable guard clauses that throw exceptions or abort HTTP requests.

## Overview

The `ensure()` helper provides a clean alternative to traditional guard clauses:

```php
// Traditional approach
if ($user === null) {
    throw new UserNotFoundException();
}

// With ensure()
ensure($user !== null)->orThrow(UserNotFoundException::class);
```

## Basic Usage

### Throw Exceptions

```php
use function Cline\Throw\ensure;

// With exception class
ensure($user !== null)->orThrow(UserNotFoundException::class);

// With exception class and message
ensure($email !== null)->orThrow(ValidationException::class, 'Email is required');

// With exception instance
ensure($token->isValid())->orThrow(InvalidTokenException::expired());
```

### Abort HTTP Requests

```php
// Abort with status code
ensure($user->isAdmin())->orAbort(403);

// Abort with status code and message
ensure($post !== null)->orAbort(404, 'Post not found');
```

## Common Patterns

### Null Checks

```php
// Ensure value is not null
ensure($user !== null)->orThrow(UserNotFoundException::class);

// Ensure value exists
ensure(isset($data['email']))->orThrow(ValidationException::class, 'Email required');
```

### Type Validation

```php
// Check types
ensure(is_array($data))->orThrow(InvalidTypeException::class, 'Expected array');

ensure(is_string($email))->orThrow(InvalidTypeException::class, 'Email must be string');

ensure($user instanceof User)->orThrow(InvalidTypeException::class);
```

### Range Validation

```php
// Numeric ranges
ensure($age >= 18)->orThrow(ValidationException::class, 'Must be 18 or older');

ensure($quantity > 0 && $quantity <= 100)
    ->orThrow(ValidationException::class, 'Quantity must be between 1 and 100');
```

### String Validation

```php
// String length
ensure(strlen($password) >= 8)
    ->orThrow(ValidationException::class, 'Password must be at least 8 characters');

// String content
ensure(str_contains($email, '@'))
    ->orThrow(ValidationException::class, 'Invalid email format');
```

### Permission Checks

```php
// Authorization
ensure($user->can('edit', $post))
    ->orAbort(403, 'You cannot edit this post');

// Role checks
ensure($user->hasRole('admin'))
    ->orAbort(403, 'Admin access required');
```

## Real-World Examples

### Controller Usage

```php
class PostController extends Controller
{
    public function update(Request $request, int $id)
    {
        $post = Post::find($id);

        // Ensure post exists
        ensure($post !== null)->orAbort(404, 'Post not found');

        // Ensure user can edit
        ensure($request->user()->can('update', $post))
            ->orAbort(403, 'Cannot edit this post');

        $post->update($request->validated());

        return response()->json($post);
    }

    public function destroy(Request $request, int $id)
    {
        $post = Post::find($id);

        ensure($post !== null)->orAbort(404);
        ensure($request->user()->owns($post))->orAbort(403);

        $post->delete();

        return response()->noContent();
    }
}
```

### Service Layer

```php
class PaymentService
{
    public function processPayment(Order $order, PaymentMethod $method): Payment
    {
        // Business rule validation
        ensure($order->canAcceptPayment())
            ->orThrow(OrderException::cannotAcceptPayment());

        ensure($order->total->isPositive())
            ->orThrow(OrderException::invalidAmount());

        ensure($method->isValid())
            ->orThrow(PaymentMethodException::invalid());

        // Process payment...
    }
}
```

### Repository Layer

```php
class UserRepository
{
    public function findByEmail(string $email): User
    {
        ensure(!empty($email))
            ->orThrow(ValidationException::class, 'Email cannot be empty');

        ensure(filter_var($email, FILTER_VALIDATE_EMAIL))
            ->orThrow(ValidationException::class, 'Invalid email format');

        $user = User::where('email', $email)->first();

        ensure($user !== null)
            ->orThrow(UserNotFoundException::class);

        return $user;
    }
}
```

### Domain Models

```php
class Order
{
    public function cancel(): void
    {
        ensure($this->canBeCancelled())
            ->orThrow(OrderException::cannotCancel());

        ensure($this->status !== 'shipped')
            ->orThrow(OrderException::alreadyShipped());

        $this->status = 'cancelled';
        $this->save();
    }

    public function ship(): void
    {
        ensure($this->isPaid())
            ->orThrow(OrderException::notPaid());

        ensure($this->hasAddress())
            ->orThrow(OrderException::missingAddress());

        $this->status = 'shipped';
        $this->shipped_at = now();
        $this->save();
    }
}
```

### Middleware

```php
class RequireApiKey
{
    public function handle(Request $request, Closure $next)
    {
        $apiKey = $request->header('X-API-Key');

        // Ensure API key present
        ensure(!empty($apiKey))
            ->orAbort(401, 'API key required');

        // Ensure API key valid
        ensure($this->isValidApiKey($apiKey))
            ->orAbort(403, 'Invalid API key');

        return $next($request);
    }
}
```

### Form Requests

```php
class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        ensure($this->user()->can('create-users'))
            ->orAbort(403);

        return true;
    }

    protected function prepareForValidation(): void
    {
        // Ensure required data exists
        ensure($this->has('email'))
            ->orThrow(ValidationException::class, 'Email is required');
    }
}
```

## Combining with Exception Features

### With Context

```php
ensure($account->balance >= $amount)
    ->orThrow(
        InsufficientFundsException::forAmount($amount)
            ->withContext([
                'account_id' => $account->id,
                'balance' => $account->balance,
                'required' => $amount,
            ])
    );
```

### With Tags

```php
ensure($subscription->isActive())
    ->orThrow(
        SubscriptionInactiveException::create()
            ->withTags(['subscription', 'access-control'])
    );
```

### With Wrapping

```php
try {
    $result = $this->api->call();
} catch (ApiException $e) {
    ensure($result !== null)
        ->orThrow(
            ExternalServiceException::failed()
                ->wrap($e)
                ->withTags(['external-api', 'critical'])
        );
}
```

## Multiple Assertions

Chain multiple assertions for comprehensive validation:

```php
public function createUser(array $data): User
{
    // Validate all inputs
    ensure(isset($data['email']))
        ->orThrow(ValidationException::class, 'Email required');

    ensure(isset($data['password']))
        ->orThrow(ValidationException::class, 'Password required');

    ensure(filter_var($data['email'], FILTER_VALIDATE_EMAIL))
        ->orThrow(ValidationException::class, 'Invalid email');

    ensure(strlen($data['password']) >= 8)
        ->orThrow(ValidationException::class, 'Password too short');

    return User::create($data);
}
```

## Comparison with Alternatives

### vs throw_if()

```php
// Laravel's throw_if
throw_if($user === null, UserNotFoundException::class);

// ensure() - reads left to right
ensure($user !== null)->orThrow(UserNotFoundException::class);
```

### vs abort_if()

```php
// Laravel's abort_if
abort_if(!$user->isAdmin(), 403);

// ensure() - more explicit
ensure($user->isAdmin())->orAbort(403);
```

### vs Traditional Guards

```php
// Traditional
if ($user === null) {
    throw new UserNotFoundException();
}

// ensure() - more concise
ensure($user !== null)->orThrow(UserNotFoundException::class);
```

## When to Use ensure()

**Use `ensure()` when:**
- Writing guard clauses
- Validating preconditions
- Checking permissions/authorization
- Asserting business rules
- Validating input data

**Don't use `ensure()` when:**
- The condition is part of normal control flow
- You need complex error handling
- Multiple outcomes are valid
- You're checking for expected conditions (use if/else)

## Best Practices

1. **Keep conditions simple** - Complex conditions reduce readability
2. **Use descriptive messages** - Help debugging with clear error messages
3. **Fail early** - Place assertions at the start of methods
4. **Be explicit** - Prefer positive conditions (`!== null` vs `=== null`)
5. **Combine with exceptions** - Use factory methods for rich exception data

## Testing

```php
use function Cline\Throw\ensure;

test('throws when condition fails', function () {
    expect(fn () => ensure(false)->orThrow(RuntimeException::class))
        ->toThrow(RuntimeException::class);
});

test('aborts when condition fails', function () {
    expect(fn () => ensure(false)->orAbort(404))
        ->toThrow(HttpException::class);
});

test('does not throw when condition passes', function () {
    ensure(true)->orThrow(RuntimeException::class);

    expect(true)->toBeTrue();
});
```

## Next Steps

- Learn about [Base Exceptions](base-exceptions.md) for creating domain-specific exceptions
- Explore [Error Context](error-context.md) for adding debugging information
- See [Error Wrapping](error-wrapping.md) for exception chains
