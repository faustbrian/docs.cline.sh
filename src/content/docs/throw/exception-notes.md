---
title: Exception Notes
description: Add breadcrumb-style annotations to exceptions for enhanced debugging context, inspired by Python's exception notes.
---

Add breadcrumb-style annotations to exceptions for enhanced debugging context, inspired by Python's exception notes.

## Overview

Exception notes create a chronological trail of context as exceptions propagate through your application, providing valuable debugging insights without requiring exception wrapping.

```php
$exception = new PaymentException('Payment failed');

$exception->addNote('Processing order #12345')
    ->addNote('Customer ID: 789')
    ->addNote('Using Stripe gateway')
    ->addNote('Retry attempt #2');

throw $exception;
```

## Basic Usage

### Adding Single Notes

```php
$exception = new DatabaseException('Query failed');
$exception->addNote('Query: SELECT * FROM users WHERE active = 1');
$exception->addNote('Connection: primary-db');

throw $exception;
```

### Adding Multiple Notes at Once

```php
$exception = new ApiException('External API failed');

$exception->addNotes([
    'Endpoint: /api/v2/users',
    'Method: POST',
    'Timeout: 30s',
    'Retry count: 3',
]);

throw $exception;
```

### Retrieving Notes

```php
try {
    // operation that throws
} catch (Exception $e) {
    $notes = $e->getNotes();

    foreach ($notes as $note) {
        logger()->debug($note);
    }

    // Check if notes exist
    if ($e->hasNotes()) {
        // Handle notes
    }
}
```

### Formatted Output

```php
try {
    // operation
} catch (Exception $e) {
    // Get formatted notes as numbered list
    echo $e->getFormattedNotes();

    /* Output:
    Notes:
      1. Processing order #12345
      2. Using Stripe gateway
      3. Retry attempt #2
    */
}
```

## Real-World Examples

### Tracking Request Flow

```php
class OrderController
{
    public function process(Request $request, int $orderId)
    {
        try {
            $order = Order::findOrFail($orderId);

            $exception = null;

            try {
                $this->validateOrder($order);
            } catch (ValidationException $e) {
                $exception = $e;
                $exception->addNote("Order ID: {$orderId}");
                $exception->addNote("User ID: {$request->user()->id}");
                $exception->addNote('Validation failed in OrderController::process');

                throw $exception;
            }

            try {
                $this->chargePayment($order);
            } catch (PaymentException $e) {
                $e->addNote("Order ID: {$orderId}");
                $e->addNote("Amount: {$order->total}");
                $e->addNote("Payment method: {$order->payment_method}");

                throw $e;
            }

            return response()->json(['success' => true]);
        } catch (Exception $e) {
            logger()->error($e->getMessage(), [
                'notes' => $e->getNotes(),
                'exception' => get_class($e),
            ]);

            return response()->json(['error' => $e->getMessage()], 500);
        }
    }
}
```

### Service Layer Context

```php
class UserService
{
    public function register(array $data): User
    {
        try {
            DB::beginTransaction();

            $user = User::create($data);

            try {
                $this->createProfile($user, $data);
            } catch (Exception $e) {
                $e->addNote("User ID: {$user->id}");
                $e->addNote('Failed during profile creation');

                throw $e;
            }

            try {
                $this->sendWelcomeEmail($user);
            } catch (Exception $e) {
                $e->addNote("User email: {$user->email}");
                $e->addNote('Failed during welcome email');
                $e->addNote('User created successfully');

                // Log but don't fail
                logger()->warning($e->getMessage(), [
                    'notes' => $e->getNotes(),
                ]);
            }

            DB::commit();

            return $user;
        } catch (Exception $e) {
            DB::rollBack();

            $e->addNote('Transaction rolled back');

            throw $e;
        }
    }
}
```

### Background Job Tracking

```php
class ProcessOrderJob implements ShouldQueue
{
    public function handle(): void
    {
        $exception = null;

        try {
            $this->validateInventory();
        } catch (Exception $e) {
            $exception = $e;
            $exception->addNote("Job: ".self::class);
            $exception->addNote("Queue: {$this->queue}");
            $exception->addNote("Attempt: {$this->attempts()}");
            $exception->addNote('Failed at inventory validation');

            throw $exception;
        }

        try {
            $this->processPayment();
        } catch (Exception $e) {
            $e->addNote("Job: ".self::class);
            $e->addNote("Attempt: {$this->attempts()}");
            $e->addNote('Inventory validated successfully');
            $e->addNote('Failed at payment processing');

            throw $e;
        }
    }

    public function failed(Throwable $exception): void
    {
        $exception->addNote('Job permanently failed');

        logger()->critical($exception->getFormattedNotes());
    }
}
```

### API Integration Debugging

```php
class ExternalApiClient
{
    public function fetchUserData(int $userId): array
    {
        $exception = null;

        try {
            $response = Http::timeout(30)
                ->get("/api/users/{$userId}");

            if ($response->failed()) {
                $exception = new ApiException('API request failed');
                $exception->addNote("Endpoint: /api/users/{$userId}");
                $exception->addNote("Status code: {$response->status()}");
                $exception->addNote("Response time: {$response->handlerStats()['total_time']}s");

                throw $exception;
            }

            return $response->json();
        } catch (ConnectionException $e) {
            $e->addNote("Target: {$this->baseUrl}");
            $e->addNote("User ID: {$userId}");
            $e->addNote('Connection timeout');

            throw $e;
        } catch (RequestException $e) {
            $e->addNote("User ID: {$userId}");
            $e->addNote('Request failed');

            throw $e;
        }
    }
}
```

### Multi-Layer Exception Propagation

```php
// Repository layer
class UserRepository
{
    public function findByEmail(string $email): User
    {
        try {
            return User::where('email', $email)->firstOrFail();
        } catch (ModelNotFoundException $e) {
            $e->addNote('Repository: UserRepository');
            $e->addNote("Email: {$email}");

            throw $e;
        }
    }
}

// Service layer
class AuthenticationService
{
    public function authenticate(string $email, string $password): User
    {
        try {
            $user = $this->userRepository->findByEmail($email);
        } catch (ModelNotFoundException $e) {
            $e->addNote('Service: AuthenticationService');
            $e->addNote('Authentication attempt failed');

            throw $e;
        }

        if (!Hash::check($password, $user->password)) {
            $exception = new InvalidCredentialsException();
            $exception->addNote("User ID: {$user->id}");
            $exception->addNote('Password verification failed');

            throw $exception;
        }

        return $user;
    }
}

// Controller layer
class LoginController
{
    public function login(Request $request)
    {
        try {
            $user = $this->authService->authenticate(
                $request->email,
                $request->password
            );
        } catch (Exception $e) {
            $e->addNote('Controller: LoginController');
            $e->addNote("IP: {$request->ip()}");
            $e->addNote("User agent: {$request->userAgent()}");

            logger()->warning('Login failed', [
                'notes' => $e->getNotes(),
                // Notes will include:
                // - Repository: UserRepository
                // - Email: user@example.com
                // - Service: AuthenticationService
                // - Authentication attempt failed
                // - Controller: LoginController
                // - IP: 127.0.0.1
                // - User agent: Mozilla/5.0...
            ]);

            return redirect()->back()->with('error', 'Invalid credentials');
        }

        return redirect()->dashboard();
    }
}
```

## Advanced Patterns

### Conditional Notes

```php
try {
    $result = $this->processData($data);
} catch (Exception $e) {
    $e->addNote("Data size: ".count($data)." items");

    if (app()->environment('local')) {
        $e->addNote("Data dump: ".json_encode($data));
    }

    if ($this->retryCount > 0) {
        $e->addNote("Retry count: {$this->retryCount}");
    }

    throw $e;
}
```

### Combining with Context

```php
try {
    $payment = $this->gateway->charge($amount);
} catch (PaymentException $e) {
    $e->withContext([
        'user_id' => $user->id,
        'amount' => $amount,
        'currency' => 'USD',
    ])
        ->withTags(['payment', 'stripe'])
        ->addNote("Processing payment for order #{$order->id}")
        ->addNote("Gateway: {$this->gateway->name()}")
        ->addNote("Charge ID: {$e->getChargeId()}")
        ->withMetadata([
            'gateway_response' => $e->getGatewayResponse(),
        ]);

    throw $e;
}
```

### Formatted Logging

```php
try {
    $this->processJob();
} catch (Exception $e) {
    logger()->error($e->getMessage()."
".$e->getFormattedNotes(), [
        'exception_class' => get_class($e),
        'file' => $e->getFile(),
        'line' => $e->getLine(),
    ]);

    /* Log output:
    [2024-01-15 10:30:45] production.ERROR: Job processing failed
    Notes:
      1. Job: ProcessOrderJob
      2. Queue: high-priority
      3. Attempt: 3 of 5
      4. Order ID: 12345
      5. Failed at payment processing
    */
}
```

### Exception Notes in Tests

```php
it('tracks execution context through exception chain', function () {
    $exception = new RuntimeException('Error');

    $exception->addNote('Step 1: Validation')
        ->addNote('Step 2: Processing')
        ->addNote('Step 3: Failed at database');

    expect($exception->getNotes())->toHaveCount(3)
        ->and($exception->hasNotes())->toBeTrue()
        ->and($exception->getFormattedNotes())->toContain('Notes:');
});
```

## Best Practices

1. **Add notes early** - Annotate exceptions as soon as you catch them
2. **Be specific** - Include relevant IDs, timestamps, and context
3. **Chronological order** - Notes maintain order, creating a breadcrumb trail
4. **Don't duplicate context** - Use `withContext()` for structured data, notes for narrative
5. **Include layer information** - Identify which layer (controller/service/repository) added the note
6. **Use for debugging** - Notes are perfect for production debugging trails

## Common Patterns

### Before/After State

```php
$exception->addNote("Before: balance = {$user->balance}");
// operation fails
$exception->addNote("After: transaction attempted = {$amount}");
```

### Environmental Context

```php
$exception->addNote("Environment: ".app()->environment());
$exception->addNote("Debug mode: ".(config('app.debug') ? 'enabled' : 'disabled'));
```

### Timing Information

```php
$start = microtime(true);
try {
    // operation
} catch (Exception $e) {
    $duration = round((microtime(true) - $start) * 1000, 2);
    $e->addNote("Execution time: {$duration}ms");

    throw $e;
}
```

## Comparison with Other Approaches

| Feature | Notes | Context | Metadata |
|---------|-------|---------|----------|
| Purpose | Chronological breadcrumbs | Structured data | Debug information |
| Format | Array of strings | Key-value pairs | Key-value pairs |
| When to use | Execution flow | Identifiers/state | Technical details |
| Best for | Debugging trail | Logging/monitoring | Deep debugging |

## See Also

- [Error Context](error-context.md) - Structured context and metadata
- [Error Wrapping](error-wrapping.md) - Wrapping lower-level exceptions
- [Exception Groups](exception-groups.md) - Handling multiple exceptions
