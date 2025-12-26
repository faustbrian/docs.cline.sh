---
title: Integration Patterns
description: This guide demonstrates how to integrate the Throw package with common Laravel patterns and third-party services.
---

This guide demonstrates how to integrate the Throw package with common Laravel patterns and third-party services.

## Table of Contents
- [Assertion API (ensure)](#assertion-api-ensure)
- [Wrapping Laravel Exceptions](#wrapping-laravel-exceptions)
- [Form Request Validation](#form-request-validation)
- [Eloquent Model Guards](#eloquent-model-guards)
- [Service Classes](#service-classes)
- [API Resources](#api-resources)
- [Jobs and Queues](#jobs-and-queues)
- [Error Monitoring](#error-monitoring)
- [Third-Party API Clients](#third-party-api-clients)

## Assertion API (ensure)

The `ensure()` helper provides a fluent API for conditional throwing and aborting. It supports both boolean values and callbacks for lazy evaluation.

### Basic Usage

```php
use function Cline\Thrownsure;

// Boolean conditions
ensure($user !== null)->orThrow(UserNotFoundException::class);
ensure($user->isAdmin())->orForbidden();
ensure($token->isValid())->orUnauthorized();

// Callback for lazy evaluation (only evaluated when needed)
ensure(fn() => expensive_check())->orThrow(Exception::class);
ensure(fn() => $user->isAdmin())->orForbidden();
```

### Throw Methods

```php
use function Cline\Thrownsure;

// orThrow - throws when condition is FALSE
ensure($user !== null)->orThrow(UserNotFoundException::class);
ensure(false)->orThrow(RuntimeException::class, 'Failed'); // throws

// throwIf - throws when condition is TRUE
ensure(true)->throwIf(RuntimeException::class); // throws
ensure($user->isBanned())->throwIf(UserBannedException::class);

// throwUnless - throws when condition is FALSE (alias for orThrow)
ensure(false)->throwUnless(RuntimeException::class); // throws
ensure($token->isValid())->throwUnless(InvalidTokenException::class);
```

### Abort Methods (HTTP Status Codes)

```php
use Cline\Throw\Support\HttpStatusCode;
use function Cline\Thrownsure;

// orAbort - aborts when condition is FALSE
ensure($user->can('admin'))->orAbort(HttpStatusCode::Forbidden);
ensure(false)->orAbort(HttpStatusCode::NotFound); // aborts with 404

// abortIf - aborts when condition is TRUE
ensure(true)->abortIf(HttpStatusCode::BadRequest); // aborts with 400
ensure($user->isBanned())->abortIf(HttpStatusCode::Forbidden, 'Account banned');

// abortUnless - aborts when condition is FALSE (alias for orAbort)
ensure(false)->abortUnless(HttpStatusCode::NotFound); // aborts with 404
ensure($user !== null)->abortUnless(HttpStatusCode::NotFound);
```

### HTTP Helper Methods

Convenient shorthand for common HTTP status codes:

```php
use function Cline\Thrownsure;

// 4xx Client Errors
ensure($input->isValid())->orBadRequest('Invalid input');           // 400
ensure($token !== null)->orUnauthorized('Authentication required'); // 401
ensure($user->can('admin'))->orForbidden('Access denied');         // 403
ensure($post !== null)->orNotFound('Post not found');              // 404
ensure(!$user->exists())->orConflict('User already exists');       // 409
ensure($validation->passes())->orUnprocessable('Validation failed'); // 422
ensure($rateLimiter->allow())->orTooManyRequests();                // 429

// 5xx Server Errors
ensure($service->isHealthy())->orServerError();                    // 500
ensure(!$maintenance->isEnabled())->orServiceUnavailable();        // 503

// Additional helpers
ensure($request->isMethod('POST'))->orMethodNotAllowed();          // 405
ensure($request->accepts('json'))->orNotAcceptable();              // 406
ensure($request->timedOut())->orRequestTimeout();                  // 408
ensure(!$resource->isDeleted())->orGone('Resource deleted');       // 410
ensure($file->size() <= $max)->orPayloadTooLarge();               // 413
ensure($request->isJson())->orUnsupportedMediaType();             // 415
ensure(!$request->wantsCoffee())->orImATeapot();                  // 418
ensure(!$resource->isLocked())->orLocked('Resource locked');      // 423
ensure($hasHeader)->orPreconditionRequired();                      // 428
ensure($feature->isImplemented())->orNotImplemented();            // 501
ensure($upstream->isResponding())->orBadGateway();                // 502
ensure($upstream->respondedInTime())->orGatewayTimeout();         // 504
```

### Lazy Evaluation with Callbacks

Use callbacks to defer expensive checks until needed:

```php
use function Cline\Thrownsure;

// Expensive database query only runs if previous checks pass
ensure(fn() => User::where('email', $email)->exists())
    ->orThrow(UserNotFoundException::class);

// Complex permission check with multiple queries
ensure(fn() => $user->can('edit', $post) && !$post->isLocked())
    ->orForbidden('Cannot edit this post');

// Rate limiting check (closure only called if needed)
ensure(fn() => !RateLimiter::tooManyAttempts($key, 60))
    ->orTooManyRequests('Rate limit exceeded');
```

### Comparison: Assertion vs Exception Trait Methods

**Important:** The `ensure()` Assertion API has different semantics than Exception trait methods:

```php
use function Cline\Thrownsure;

// Assertion API (ensure helper)
ensure($condition)->throwIf($exception);     // throws when $condition is TRUE
ensure($condition)->throwUnless($exception); // throws when $condition is FALSE

// Exception Trait API (on exception instances)
Exception::create()->throwIf($condition);    // throws when $condition parameter is TRUE
Exception::create()->throwUnless($condition); // throws when $condition parameter is FALSE
```

## Wrapping Laravel Exceptions

Convert framework exceptions into domain-specific exceptions while preserving the original error.

```php
use Cline\Throw\Exceptions\DatabaseException;
use Illuminate\Database\QueryException;

class UserRepository
{
    public function find(int $id): User
    {
        try {
            return User::findOrFail($id);
        } catch (ModelNotFoundException $e) {
            throw UserNotFoundException::withId($id)->wrap($e);
        } catch (QueryException $e) {
            throw DatabaseException::queryFailed()
                ->wrap($e)
                ->withContext(['query' => $e->getSql()]);
        }
    }
}
```

## Form Request Validation

Use assertions in form requests for authorization and validation.

```php
use Cline\Throw\Support\HttpStatusCode;
use Illuminate\Foundation\Http\FormRequest;

use function Cline\Thrownsure;

class UpdatePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        $post = $this->route('post');

        // Explicit authorization with typed status codes
        ensure($post->user_id === $this->user()->id)
            ->orForbidden('You cannot edit this post');

        return true;
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'content' => 'required|string',
        ];
    }
}
```

## Eloquent Model Guards

Add guard methods to your models for business rule enforcement.

```php
use Cline\Throw\Exceptions\BusinessRuleException;
use Illuminate\Database\Eloquent\Model;

use function Cline\Thrownsure;

class Subscription extends Model
{
    public function cancel(): void
    {
        ensure(!$this->isCancelled())
            ->orThrow(BusinessRuleException::subscriptionAlreadyCancelled());

        ensure($this->canBeCancelled())
            ->orThrow(BusinessRuleException::subscriptionNotCancellable());

        $this->update(['cancelled_at' => now()]);
    }

    public function charge(int $amount): void
    {
        ensure($this->isActive())
            ->orThrow(BusinessRuleException::subscriptionNotActive());

        ensure($amount > 0)
            ->orThrow(BusinessRuleException::invalidAmount());

        // Process charge...
    }
}
```

## Service Classes

Structure service classes with explicit error handling.

```php
use Cline\Throw\Exceptions\PaymentException;
use Cline\Throw\Support\Errors;
use Stripe\Exception\CardException;

use function Cline\Thrownsure;

class PaymentService
{
    public function charge(User $user, int $amount): Payment
    {
        ensure($amount > 0)
            ->orThrow(PaymentException::invalidAmount($amount));

        ensure($user->hasPaymentMethod())
            ->orThrow(PaymentException::noPaymentMethod());

        try {
            $charge = $this->stripe->charges->create([
                'amount' => $amount,
                'currency' => 'usd',
                'customer' => $user->stripe_id,
            ]);

            return Payment::create([
                'user_id' => $user->id,
                'amount' => $amount,
                'stripe_charge_id' => $charge->id,
            ]);
        } catch (CardException $e) {
            throw PaymentException::cardDeclined()
                ->wrap($e)
                ->withContext([
                    'user_id' => $user->id,
                    'amount' => $amount,
                    'decline_code' => $e->getDeclineCode(),
                ]);
        }
    }
}
```

## API Resources

Guard against missing data in API resources.

```php
use Cline\Throw\Exceptions\ResourceException;
use Illuminate\Http\Resources\Json\JsonResource;

use function Cline\Thrownsure;

class UserResource extends JsonResource
{
    public function toArray($request): array
    {
        ensure($this->resource !== null)
            ->orThrow(ResourceException::missingData());

        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'profile' => new ProfileResource($this->whenLoaded('profile')),
        ];
    }
}
```

## Jobs and Queues

Handle job failures with detailed error context.

```php
use Cline\Throw\Exceptions\ExternalServiceException;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

use function Cline\Thrownsure;

class ProcessWebhook implements ShouldQueue
{
    use InteractsWithQueue, Queueable;

    public function __construct(
        private array $payload,
    ) {}

    public function handle(): void
    {
        ensure(isset($this->payload['event']))
            ->orThrow(ExternalServiceException::invalidWebhookPayload()
                ->withContext(['payload' => $this->payload]));

        match ($this->payload['event']) {
            'payment.succeeded' => $this->handlePaymentSucceeded(),
            'payment.failed' => $this->handlePaymentFailed(),
            default => throw ExternalServiceException::unknownWebhookEvent()
                ->withContext(['event' => $this->payload['event']]),
        };
    }

    public function failed(Throwable $exception): void
    {
        logger()->error('Webhook processing failed', [
            'exception' => $exception->getMessage(),
            'payload' => $this->payload,
            'context' => method_exists($exception, 'getContext')
                ? $exception->getContext()
                : [],
        ]);
    }
}
```

## Error Monitoring

Integrate with Sentry, Flare, or other monitoring services.

```php
use Cline\Throw\Exceptions\DomainException;
use Cline\Throw\Support\Errors;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Sentry\State\Scope;

class Handler extends ExceptionHandler
{
    public function register(): void
    {
        $this->reportable(function (Throwable $e) {
            if (app()->bound('sentry')) {
                app('sentry')->captureException($e, function (Scope $scope) use ($e) {
                    // Add custom context from HasErrorContext trait
                    if (method_exists($e, 'getContext')) {
                        $scope->setContext('error_context', $e->getContext());
                    }

                    // Add tags from exception
                    if (method_exists($e, 'getTags')) {
                        foreach ($e->getTags() as $tag) {
                            $scope->setTag('custom', $tag);
                        }
                    }

                    // Add metadata
                    if (method_exists($e, 'getMetadata')) {
                        $scope->setContext('metadata', $e->getMetadata());
                    }

                    // Check for wrapped exceptions
                    if (method_exists($e, 'getWrapped') && $e->getWrapped()) {
                        $scope->setContext('wrapped_exception', [
                            'class' => get_class($e->getWrapped()),
                            'message' => $e->getWrapped()->getMessage(),
                        ]);
                    }
                });
            }
        });

        // Handle domain exceptions differently
        $this->renderable(function (DomainException $e, $request) {
            if ($request->expectsJson()) {
                return response()->json([
                    'error' => $e->getMessage(),
                    'context' => $e->getContext(),
                ], 422);
            }
        });
    }
}
```

## Third-Party API Clients

Wrap external service errors with domain exceptions.

```php
use Cline\Throw\Exceptions\ExternalServiceException;
use Cline\Throw\Support\Errors;
use GuzzleHttp\Client;
use GuzzleHttp\Exception\RequestException;

use function Cline\Thrownsure;

class GitHubClient
{
    public function __construct(
        private Client $client,
        private string $token,
    ) {}

    public function getUser(string $username): array
    {
        ensure($username !== '')
            ->orThrow(ExternalServiceException::invalidParameter('username'));

        try {
            $response = $this->client->get("https://api.github.com/users/{$username}", [
                'headers' => [
                    'Authorization' => "Bearer {$this->token}",
                    'Accept' => 'application/vnd.github.v3+json',
                ],
            ]);

            return json_decode($response->getBody()->getContents(), true);
        } catch (RequestException $e) {
            // Check if it's a 404
            if (Errors::is($e, RequestException::class) && $e->getResponse()?->getStatusCode() === 404) {
                throw ExternalServiceException::userNotFound($username)
                    ->wrap($e)
                    ->withTags(['github', 'not-found']);
            }

            // Check if it's rate limiting
            if ($e->getResponse()?->getStatusCode() === 429) {
                throw ExternalServiceException::rateLimitExceeded()
                    ->wrap($e)
                    ->withTags(['github', 'rate-limit'])
                    ->withMetadata([
                        'retry_after' => $e->getResponse()->getHeader('Retry-After'),
                    ]);
            }

            // Generic API error
            throw ExternalServiceException::apiRequestFailed()
                ->wrap($e)
                ->withContext([
                    'service' => 'github',
                    'endpoint' => "users/{$username}",
                    'status_code' => $e->getResponse()?->getStatusCode(),
                ]);
        }
    }
}
```

## Controller Integration

Clean controller actions with explicit error handling.

```php
use Cline\Throw\Support\HttpStatusCode;
use Illuminate\Http\JsonResponse;

use function Cline\Thrownsure;

class PostController extends Controller
{
    public function show(Post $post): JsonResponse
    {
        ensure($post->isPublished() || $post->user_id === auth()->id())
            ->orForbidden();

        return response()->json(new PostResource($post));
    }

    public function update(UpdatePostRequest $request, Post $post): JsonResponse
    {
        ensure($post->user_id === auth()->id())
            ->orForbidden('You cannot edit this post');

        ensure(!$post->isLocked())
            ->orAbort(HttpStatusCode::Conflict, 'Post is locked');

        $post->update($request->validated());

        return response()->json(new PostResource($post));
    }

    public function destroy(Post $post): JsonResponse
    {
        ensure($post->user_id === auth()->id())
            ->orForbidden();

        ensure($post->canBeDeleted())
            ->orAbort(HttpStatusCode::Conflict, 'Post cannot be deleted');

        $post->delete();

        return response()->json(['message' => 'Post deleted']);
    }
}
```

## Middleware Integration

Use assertions in middleware for request validation.

```php
use Cline\Throw\Support\HttpStatusCode;
use Closure;
use Illuminate\Http\Request;

use function Cline\Thrownsure;

class EnsureUserIsVerified
{
    public function handle(Request $request, Closure $next)
    {
        ensure($request->user()?->hasVerifiedEmail())
            ->orAbort(HttpStatusCode::Forbidden, 'Email must be verified');

        return $next($request);
    }
}

class EnsureApiRateLimit
{
    public function handle(Request $request, Closure $next)
    {
        $limiter = app(RateLimiter::class);

        ensure($limiter->attempt($request->user()->id, 60, fn() => true))
            ->orTooManyRequests('Rate limit exceeded');

        return $next($request);
    }
}
```
