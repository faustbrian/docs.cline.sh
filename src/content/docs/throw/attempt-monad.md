---
title: Attempt Monad (Scala-Style Try)
description: The attempt() helper provides a Scala-inspired Try monad for handling exceptions fluently. It wraps code execution and lets you handle success/failure with various strategies.
---

The `attempt()` helper provides a Scala-inspired Try monad for handling exceptions fluently. It wraps code execution and lets you handle success/failure with various strategies.

## Core Concepts

### Success vs Failure

The `Attempt` class represents a computation that may either succeed (Success) or fail (Failure). Unlike traditional try-catch, you handle both cases fluently:

```php
use function Cline\Throw\attempt;

// Execute and get result
$user = attempt(fn() => User::findOrFail($id))->get();

// Execute with fallback
$user = attempt(fn() => User::find($id))->getOrElse(null);

// Convert to Option-like (null on failure)
$user = attempt(fn() => loadUser())->toOption();

// Recover from failure
$data = attempt(fn() => fetchFromApi())->recover(fn($e) => getCached());
```

## Scala-Style Methods

### `get()`

Unwrap the result or throw the original exception:

```php
// Throws original exception if it fails
$user = attempt(fn() => User::findOrFail($id))->get();

// Equivalent to traditional:
try {
    $user = User::findOrFail($id);
} catch (Throwable $e) {
    throw $e;
}
```

### `getOrElse()`

Get result or return a default value:

```php
// Return null if not found
$user = attempt(fn() => User::find($id))->getOrElse(null);

// Return empty array
$items = attempt(fn() => fetchItems())->getOrElse([]);

// Return default object
$config = attempt(fn() => loadConfig())->getOrElse(new Config());
```

### `toOption()`

Convert to Option monad (`Some<T>` or `None`):

```php
use Cline\Monad\Option\Option;

// Returns Some<User> or None
$user = attempt(fn() => loadUser())->toOption();

// Chain with Option methods
$name = attempt(fn() => $user->profile->name)
    ->toOption()
    ->map(fn($n) => strtoupper($n))
    ->unwrapOr('GUEST');

// Use with Option's unwrapOr
$result = attempt(fn() => dangerousOp())
    ->toOption()
    ->unwrapOr('default');

// Transform with map/filter
$email = attempt(fn() => $user->email)
    ->toOption()
    ->filter(fn($e) => str_contains($e, '@'))
    ->map(fn($e) => strtolower($e))
    ->unwrapOr('no-email@example.com');
```

### `recover()`

Execute a callback to recover from failure:

```php
// Recover with cached data
$data = attempt(fn() => fetchFromApi())
    ->recover(fn($e) => Cache::get('data', []));

// Log and return fallback
$result = attempt(fn() => processPayment($order))
    ->recover(function (Throwable $e) {
        Log::error('Payment failed', ['error' => $e->getMessage()]);
        return ['status' => 'pending'];
    });

// Transform exception into value
$status = attempt(fn() => checkService())
    ->recover(fn($e) => 'offline');
```

## Custom Exception Handling

### `orThrow()`

Throw a custom exception instead of the original:

```php
// Throw custom exception class
attempt(fn() => parseJson($data))
    ->orThrow(InvalidJsonException::class, 'Invalid JSON provided');

// Throw exception instance
attempt(fn() => loadResource($id))
    ->orThrow(ResourceNotFoundException::notFound($id));

// Original exception is wrapped as previous
try {
    attempt(fn() => dangerousOp())
        ->orThrow(ServiceException::class, 'Service failed');
} catch (ServiceException $e) {
    $original = $e->getPrevious(); // Original exception
}
```

## HTTP Abort Helpers

### `abort()`

Abort HTTP request with status code:

```php
use Cline\Throw\Support\HttpStatusCode;

attempt(fn() => authorize($user))
    ->abort(HttpStatusCode::Forbidden, 'Access denied');
```

### Convenience Helpers

```php
// 400 Bad Request
attempt(fn() => validateInput($data))
    ->orBadRequest('Invalid input');

// 401 Unauthorized
attempt(fn() => authenticate($token))
    ->orUnauthorized('Authentication required');

// 403 Forbidden
attempt(fn() => checkPermission($user, 'admin'))
    ->orForbidden();

// 404 Not Found
attempt(fn() => Post::findOrFail($id))
    ->orNotFound('Post not found');

// 409 Conflict
attempt(fn() => createUser($email))
    ->orConflict('Email already exists');

// 422 Unprocessable Entity
attempt(fn() => validator($data)->validate())
    ->orUnprocessable('Validation failed');

// 429 Too Many Requests
attempt(fn() => rateLimiter()->attempt($key))
    ->orTooManyRequests();

// 500 Internal Server Error
attempt(fn() => criticalOperation())
    ->orServerError();
```

## Executing Different Callables

### Closures

```php
$result = attempt(fn() => expensiveComputation())->get();
```

### Invokable Classes

```php
class ProcessPayment
{
    public function __invoke(Order $order): PaymentResult
    {
        // Process payment
    }
}

$result = attempt(ProcessPayment::class)->getOrElse(null);
$result = attempt(new ProcessPayment())->get();
```

### Classes with `handle()` Method

```php
class ProcessOrder
{
    public function handle(): OrderResult
    {
        // Process order
    }
}

$result = attempt(new ProcessOrder())->get();
```

### Callable Arrays

```php
// Static methods
$result = attempt([User::class, 'findByEmail'])
    ->getOrElse(null);

// Instance methods
$service = new ApiService();
$data = attempt([$service, 'fetchData'])
    ->recover(fn($e) => []);
```

## Real-World Examples

### API Calls with Fallbacks

```php
// Try API, fall back to cache, then default
$data = attempt(fn() => Http::get($url)->json())
    ->recover(fn($e) => Cache::get("api:{$url}"))
    ?? [];
```

### Safe Property Access

```php
// Deeply nested property access with Option
$city = attempt(fn() => $user->address->location->city)
    ->toOption()
    ->unwrapOr('Unknown');

// Chain transformations
$displayName = attempt(fn() => $user->profile->fullName)
    ->toOption()
    ->map(fn($name) => trim($name))
    ->filter(fn($name) => $name !== '')
    ->unwrapOr('Anonymous');
```

### Database Operations

```php
// Find or create pattern
$user = attempt(fn() => User::where('email', $email)->firstOrFail())
    ->recover(fn($e) => User::create(['email' => $email]));
```

### File Operations

```php
// Read file with fallback
$content = attempt(fn() => file_get_contents($path))
    ->getOrElse('');

// Parse JSON safely
$data = attempt(fn() => json_decode($json, true, flags: JSON_THROW_ON_ERROR))
    ->recover(fn($e) => []);
```

### Service Layer

```php
class PaymentService
{
    public function charge(Order $order): PaymentResult
    {
        return attempt(fn() => $this->gateway->charge($order))
            ->recover(function (Throwable $e) use ($order) {
                Log::error('Payment failed', [
                    'order_id' => $order->id,
                    'error' => $e->getMessage(),
                ]);

                return PaymentResult::failed($e->getMessage());
            });
    }
}
```

### Controller Actions

```php
class PostController
{
    public function show(int $id)
    {
        $post = attempt(fn() => Post::findOrFail($id))
            ->orNotFound('Post not found');

        return view('posts.show', compact('post'));
    }

    public function update(Request $request, int $id)
    {
        attempt(fn() => $this->authorize('update', Post::findOrFail($id)))
            ->orForbidden();

        $post = attempt(fn() => $this->postService->update($id, $request->validated()))
            ->orServerError('Failed to update post');

        return redirect()->route('posts.show', $post);
    }
}
```

### Multi-Step Operations

```php
// Chain multiple risky operations
$result = attempt(fn() => $this->validateData($input))
    ->recover(fn($e) => throw ValidationException::invalid($e->getMessage()));

$processed = attempt(fn() => $this->processData($result))
    ->recover(fn($e) => $this->fallbackProcessor($result));

$saved = attempt(fn() => $this->saveData($processed))
    ->orThrow(PersistenceException::class, 'Failed to save');
```

## Pattern Comparison

### Traditional Try-Catch

```php
try {
    $user = User::findOrFail($id);
} catch (ModelNotFoundException $e) {
    $user = null;
}
```

### With Attempt + Option

```php
$user = attempt(fn() => User::findOrFail($id))
    ->toOption()
    ->unwrapOr(null);

// Or use the Option directly
$user = attempt(fn() => User::findOrFail($id))->toOption();
// Returns Some<User> or None
```

### Traditional Try-Catch with Custom Exception

```php
try {
    $data = fetchFromApi();
} catch (Throwable $e) {
    Log::error('API failed', ['error' => $e->getMessage()]);
    throw new ServiceException('Service unavailable', previous: $e);
}
```

### With Attempt

```php
$data = attempt(fn() => fetchFromApi())
    ->recover(function ($e) {
        Log::error('API failed', ['error' => $e->getMessage()]);
        throw new ServiceException('Service unavailable', previous: $e);
    });
```

## Best Practices

### 1. Use `get()` for Pure Unwrapping

When you want to propagate the original exception:

```php
$user = attempt(fn() => User::findOrFail($id))->get();
```

### 2. Use `getOrElse()` for Safe Defaults

When you have a reasonable default value:

```php
$config = attempt(fn() => loadConfig())->getOrElse([]);
```

### 3. Use `toOption()` for Option Monad Chaining

When you want to leverage Option's map/filter/flatMap:

```php
$name = attempt(fn() => $user->name)
    ->toOption()
    ->map(fn($n) => strtoupper($n))
    ->unwrapOr('GUEST');

// Complex transformations
$validEmail = attempt(fn() => $user->email)
    ->toOption()
    ->filter(fn($e) => filter_var($e, FILTER_VALIDATE_EMAIL))
    ->map(fn($e) => strtolower($e))
    ->unwrapOr(null);
```

### 4. Use `recover()` for Side Effects

When you need to log, notify, or transform errors:

```php
attempt(fn() => sendEmail($user))
    ->recover(function ($e) use ($user) {
        Log::warning('Email failed', ['user' => $user->id]);
        Queue::push(new SendEmailJob($user));
    });
```

### 5. Use `orThrow()` for Custom Exceptions

When you want to transform exceptions into your domain:

```php
attempt(fn() => $this->externalApi->call())
    ->orThrow(ServiceUnavailableException::class);
```

### 6. Use HTTP Helpers in Controllers

Keep controller code clean and expressive:

```php
public function destroy(int $id)
{
    $post = attempt(fn() => Post::findOrFail($id))
        ->orNotFound();

    attempt(fn() => $this->authorize('delete', $post))
        ->orForbidden();

    attempt(fn() => $post->delete())
        ->orServerError();

    return redirect()->route('posts.index');
}
```

## Type Safety

The `Attempt` class is fully typed with generics:

```php
/** @var Attempt<User> */
$userAttempt = attempt(fn() => User::find($id));

/** @var User|null */
$user = $userAttempt->toOption();

/** @var User */
$user = $userAttempt->get(); // Throws if failed

/** @var User|Guest */
$user = $userAttempt->getOrElse(new Guest());
```
