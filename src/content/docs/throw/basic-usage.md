---
title: Basic Usage
description: The ConditionallyThrowable trait provides four core methods for conditional exception throwing. All methods support both boolean values and callbacks for lazy evaluation.
---

The `ConditionallyThrowable` trait provides four core methods for conditional exception throwing. All methods support both boolean values and callbacks for lazy evaluation.

## throwIf()

Throws the exception when the condition evaluates to `true`:

```php
use App\Exceptions\ValidationException;

// Basic guard clause
ValidationException::invalidEmail()->throwIf(!filter_var($email, FILTER_VALIDATE_EMAIL));

// Null checking
MissingResourceException::notFound()->throwIf($user === null);

// Type validation
InvalidTypeException::expectedArray()->throwIf(!is_array($data));
```

### Real-World Example

```php
class UserService
{
    public function findOrFail(int $id): User
    {
        $user = User::find($id);

        UserNotFoundException::withId($id)->throwIf($user === null);

        return $user;
    }
}
```

## throwUnless()

Throws the exception when the condition evaluates to `false`:

```php
// Interface validation
InvalidTokenableException::mustImplementInterface()
    ->throwUnless($tokenable instanceof HasApiTokens);

// Permission checking
UnauthorizedException::missingPermission()
    ->throwUnless($user->can('admin'));

// State validation
InvalidStateException::notPublished()
    ->throwUnless($post->isPublished());
```

### Real-World Example

```php
class PaymentProcessor
{
    public function process(Order $order): void
    {
        PaymentException::orderNotPayable()
            ->throwUnless($order->canAcceptPayment());

        // Process payment...
    }
}
```

## Comparison with Laravel Helpers

### Before (using throw_if helper)

```php
throw_if($user === null, UserNotFoundException::withId($id));
throw_unless($user->can('admin'), UnauthorizedException::class);
```

### After (using Throw trait)

```php
UserNotFoundException::withId($id)->throwIf($user === null);
UnauthorizedException::missingPermission()->throwUnless($user->can('admin'));
```

### Readability Benefits

The fluent pattern offers several advantages:

1. **Left-to-right reading**: Exception details come first, then the condition
2. **Natural chaining**: Works seamlessly with static factory methods
3. **Better autocomplete**: IDEs suggest all available factory methods
4. **More explicit**: The exception type and message are immediately visible

## Chaining with Static Factories

Throw works beautifully with named constructors:

```php
final class AuthenticationException extends RuntimeException
{
    use ConditionallyThrowable;

    public static function invalidCredentials(): self
    {
        return new self('Invalid username or password');
    }

    public static function accountLocked(): self
    {
        return new self('Account has been locked due to too many failed attempts');
    }

    public static function sessionExpired(): self
    {
        return new self('Your session has expired. Please log in again');
    }
}

// Usage
AuthenticationException::invalidCredentials()->throwIf(!Hash::check($password, $user->password));
AuthenticationException::accountLocked()->throwIf($user->isLocked());
AuthenticationException::sessionExpired()->throwIf($session->isExpired());
```

## Complex Conditions

You can use any boolean expression:

```php
// Multiple conditions with &&
InvalidConfigurationException::missingRequiredFields()
    ->throwIf(empty($config['api_key']) && empty($config['secret']));

// Negation
InvalidStateException::unexpectedStatus()
    ->throwIf(!in_array($order->status, ['pending', 'processing']));

// Method calls
RateLimitException::tooManyAttempts()
    ->throwIf($this->rateLimiter->tooManyAttempts($key));
```

## Lazy Evaluation with Callbacks

All conditional methods support callbacks for lazy evaluation, which only execute when needed:

```php
// Expensive database check - only runs if needed
UserNotFoundException::notFound()
    ->throwIf(fn() => User::where('email', $email)->doesntExist());

// Complex permission check
UnauthorizedException::forbidden()
    ->throwUnless(fn() => $user->can('edit', $post) && !$post->isLocked());

// Rate limiting check
RateLimitException::exceeded()
    ->throwIf(fn() => !$this->rateLimiter->allow($key, 60));

// Multiple queries deferred
DataException::invalid()
    ->throwUnless(fn() => $this->validator->passes() && $this->exists($id));
```

### When to Use Callbacks

Use callbacks when:
- The condition involves expensive operations (database queries, API calls)
- The check should only run if previous conditions pass
- You want to defer evaluation for performance
- The condition has side effects you want to control

## Supporting Both Patterns

You can support both the traditional Laravel helper and the fluent API:

```php
// Traditional
throw_if($tokenable === null, MissingTokenableException::forParentToken());

// Fluent
MissingTokenableException::forParentToken()->throwIf($tokenable === null);
```

Both work identically - choose based on your team's preference or existing codebase conventions.
