---
title: Exception Transformation
description: Functionally transform exception properties using map functions, inspired by functional programming patterns.
---

Functionally transform exception properties using map functions, inspired by functional programming patterns.

## Overview

Exception transformation allows you to modify exception messages, context, metadata, tags, and notes using functional transformations as exceptions propagate through your application. This provides a powerful way to normalize, enrich, or sanitize exception data.

```php
$exception->mapMessage(fn($msg) => "Payment Error: {$msg}")
    ->mapContext(fn($ctx) => [...$ctx, 'timestamp' => now()])
    ->mapTags(fn($tags) => array_map('strtolower', $tags));
```

## Available Transformations

### mapMessage()

Transform the exception message.

```php
$exception = new PaymentException('charge failed');

$exception->mapMessage(fn($msg) => ucfirst($msg));
// Message becomes: "Charge failed"

$exception->mapMessage(fn($msg) => "Payment: {$msg}");
// Message becomes: "Payment: charge failed"
```

### mapContext()

Transform the context array.

```php
$exception = new ApiException('Request failed')
    ->withContext(['user_id' => 123]);

$exception->mapContext(fn($ctx) => [...$ctx, 'timestamp' => now()]);
// Context now includes timestamp

$exception->mapContext(fn($ctx) => array_diff_key($ctx, ['password' => 1]));
// Removes password from context
```

### mapMetadata()

Transform the metadata array.

```php
$exception = new DatabaseException('Query failed')
    ->withMetadata(['query' => 'SELECT * FROM users']);

$exception->mapMetadata(fn($meta) => [
    ...$meta,
    'php_version' => PHP_VERSION,
    'memory_usage' => memory_get_usage(true),
]);
```

### mapTags()

Transform the tags array.

```php
$exception = new ValidationException('Invalid input')
    ->withTags(['Payment', 'Critical']);

$exception->mapTags(fn($tags) => array_map('strtolower', $tags));
// Tags become: ['payment', 'critical']

$exception->mapTags(fn($tags) => array_values(array_unique($tags)));
// Deduplicate tags
```

### mapNotes()

Transform the notes array.

```php
$exception = new OrderException('Processing failed')
    ->addNotes(['Note 1', 'Note 2']);

$exception->mapNotes(fn($notes) => array_map(
    fn($n) => "[".now()."] {$n}",
    $notes
));
// Adds timestamps to all notes

$exception->mapNotes(fn($notes) => array_filter(
    $notes,
    fn($n) => strlen($n) > 10
));
// Filters out short notes
```

### transform()

Apply comprehensive transformation to multiple properties.

```php
$exception = new PaymentException('error')
    ->withContext(['user_id' => 123]);

$exception->transform(function($e) {
    $e->mapMessage(fn($msg) => "Critical: {$msg}")
      ->mapContext(fn($ctx) => [...$ctx, 'severity' => 'high'])
      ->addNote('Exception transformed at '.now());
});
```

## Real-World Examples

### Normalizing Error Messages

```php
class ApiClient
{
    public function request(string $endpoint): array
    {
        try {
            return $this->http->get($endpoint);
        } catch (HttpException $e) {
            // Normalize message format
            $e->mapMessage(function($msg) {
                $msg = trim($msg);
                $msg = preg_replace('/\s+/', ' ', $msg);
                return ucfirst($msg);
            });

            throw $e;
        }
    }
}
```

### Adding Layer Context

```php
class OrderRepository
{
    public function find(int $id): Order
    {
        try {
            return Order::findOrFail($id);
        } catch (ModelNotFoundException $e) {
            $e->mapMessage(fn($msg) => "[Repository] {$msg}")
              ->mapContext(fn($ctx) => [...$ctx, 'layer' => 'repository']);

            throw $e;
        }
    }
}

class OrderService
{
    public function processOrder(int $id): void
    {
        try {
            $order = $this->repository->find($id);
        } catch (ModelNotFoundException $e) {
            $e->mapMessage(fn($msg) => "[Service] {$msg}")
              ->mapContext(fn($ctx) => [...$ctx, 'service' => 'OrderService']);

            throw $e;
        }
    }
}

// Final message: "[Service] [Repository] Order not found"
```

### Sanitizing Sensitive Data

```php
class PaymentController
{
    public function charge(Request $request)
    {
        try {
            $this->gateway->charge($request->all());
        } catch (PaymentException $e) {
            $sensitive = ['card_number', 'cvv', 'password'];

            $e->mapContext(fn($ctx) => array_diff_key(
                $ctx,
                array_flip($sensitive)
            ))
              ->mapMetadata(fn($meta) => array_diff_key(
                  $meta,
                  array_flip($sensitive)
              ));

            logger()->error($e->getMessage(), [
                'context' => $e->getContext(),
                'metadata' => $e->getMetadata(),
            ]);

            throw $e;
        }
    }
}
```

### Environment-Based Transformations

```php
try {
    $this->processData($data);
} catch (Exception $e) {
    $e->transform(function($e) use ($data) {
        if (app()->environment('production')) {
            // Remove debug info in production
            $e->mapContext(fn($ctx) => array_diff_key($ctx, ['debug' => 1]))
              ->mapMetadata(fn($meta) => array_diff_key($meta, ['trace' => 1]));
        } else {
            // Add extra debug info in development
            $e->mapContext(fn($ctx) => [
                ...$ctx,
                'raw_data' => $data,
                'debug' => true,
            ])
              ->addNote('Development mode enabled');
        }
    });

    throw $e;
}
```

### Chaining Multiple Transformations

```php
$exception = new ValidationException('invalid email')
    ->withContext(['email' => 'bad-email'])
    ->withTags(['Validation', 'Critical'])
    ->addNote('Validation started');

$exception->mapMessage(fn($msg) => ucfirst($msg))
    ->mapMessage(fn($msg) => "User Input Error: {$msg}")
    ->mapContext(fn($ctx) => [...$ctx, 'timestamp' => now()])
    ->mapContext(fn($ctx) => [...$ctx, 'ip' => request()->ip()])
    ->mapTags(fn($tags) => array_map('strtolower', $tags))
    ->mapTags(fn($tags) => [...$tags, 'user-error'])
    ->mapNotes(fn($notes) => [...$notes, 'Transformation complete']);

// Final state:
// Message: "User Input Error: Invalid email"
// Context: ['email' => 'bad-email', 'timestamp' => ..., 'ip' => '127.0.0.1']
// Tags: ['validation', 'critical', 'user-error']
// Notes: ['Validation started', 'Transformation complete']
```

### Filtering Large Metadata

```php
class LogRepository
{
    public function store(array $data): void
    {
        try {
            Log::create($data);
        } catch (DatabaseException $e) {
            $e->mapMetadata(function($meta) {
                // Filter out large values
                return array_filter(
                    $meta,
                    fn($v) => strlen(json_encode($v)) < 1000
                );
            })
              ->addNote('Large metadata values removed for logging');

            logger()->error($e->getMessage(), [
                'metadata' => $e->getMetadata(),
            ]);

            throw $e;
        }
    }
}
```

### Adding Contextual Prefixes

```php
class MultiTenantRepository
{
    public function query(int $tenantId): Collection
    {
        try {
            return DB::connection("tenant_{$tenantId}")
                ->table('records')
                ->get();
        } catch (QueryException $e) {
            $e->mapMessage(fn($msg) => "[Tenant {$tenantId}] {$msg}")
              ->mapContext(fn($ctx) => [...$ctx, 'tenant_id' => $tenantId]);

            throw $e;
        }
    }
}
```

### Normalizing Tags

```php
class ExceptionHandler
{
    public function report(Throwable $exception): void
    {
        if (method_exists($exception, 'mapTags')) {
            $exception->mapTags(function($tags) {
                // Normalize to lowercase
                $tags = array_map('strtolower', $tags);

                // Deduplicate
                $tags = array_unique($tags);

                // Add environment tag
                $tags[] = app()->environment();

                return array_values($tags);
            });
        }

        parent::report($exception);
    }
}
```

### Complex Multi-Step Transformation

```php
class OrderProcessor
{
    public function process(Order $order): void
    {
        try {
            $this->validateOrder($order);
            $this->chargePayment($order);
            $this->fulfillOrder($order);
        } catch (Exception $e) {
            $e->transform(function($e) use ($order) {
                // Step 1: Normalize message
                $e->mapMessage(function($msg) {
                    $msg = trim($msg);
                    $msg = preg_replace('/\s+/', ' ', $msg);
                    return ucfirst(rtrim($msg, '.'));
                });

                // Step 2: Enrich context
                $e->mapContext(fn($ctx) => [
                    ...$ctx,
                    'order_id' => $order->id,
                    'customer_id' => $order->customer_id,
                    'total' => $order->total,
                    'status' => $order->status,
                    'timestamp' => now()->toIso8601String(),
                ]);

                // Step 3: Sanitize sensitive data
                $sensitive = ['password', 'card_number', 'cvv', 'ssn'];
                $e->mapContext(fn($ctx) => array_diff_key(
                    $ctx,
                    array_flip($sensitive)
                ));

                // Step 4: Normalize tags
                $e->mapTags(fn($tags) => array_values(
                    array_unique(
                        array_map('strtolower', $tags)
                    )
                ));

                // Step 5: Add processing notes
                $e->addNote("Processed in OrderProcessor at ".now());
                $e->addNote("Order status: {$order->status}");
            });

            throw $e;
        }
    }
}
```

### Conditional Transformations Based on Exception Type

```php
class ExceptionTransformer
{
    public function transform(Throwable $exception): void
    {
        if (!method_exists($exception, 'transform')) {
            return;
        }

        $exception->transform(function($e) {
            // Add timestamp to all exceptions
            $e->mapContext(fn($ctx) => [...$ctx, 'timestamp' => now()]);

            // Type-specific transformations
            if ($e instanceof ValidationException) {
                $e->mapMessage(fn($msg) => "Validation Error: {$msg}")
                  ->mapTags(fn($tags) => [...$tags, 'user-error']);
            }

            if ($e instanceof DatabaseException) {
                $e->mapMessage(fn($msg) => "Database Error: {$msg}")
                  ->mapTags(fn($tags) => [...$tags, 'infrastructure']);
            }

            if ($e instanceof PaymentException) {
                $e->mapMessage(fn($msg) => "Payment Error: {$msg}")
                  ->mapTags(fn($tags) => [...$tags, 'critical', 'payment']);
            }
        });
    }
}
```

## Advanced Patterns

### Middleware-Style Transformations

```php
class ExceptionPipeline
{
    protected array $transformers = [];

    public function pipe(callable $transformer): self
    {
        $this->transformers[] = $transformer;
        return $this;
    }

    public function process(Throwable $exception): void
    {
        if (!method_exists($exception, 'transform')) {
            return;
        }

        foreach ($this->transformers as $transformer) {
            $exception->transform($transformer);
        }
    }
}

// Usage
$pipeline = new ExceptionPipeline();

$pipeline->pipe(fn($e) => $e->mapMessage(fn($m) => ucfirst($m)))
    ->pipe(fn($e) => $e->mapContext(fn($c) => [...$c, 'timestamp' => now()]))
    ->pipe(fn($e) => $e->mapTags(fn($t) => array_map('strtolower', $t)))
    ->process($exception);
```

### Transformation Based on Context

```php
try {
    $this->performOperation($data);
} catch (Exception $e) {
    $e->transform(function($e) use ($data) {
        // Transform based on data size
        if (count($data) > 1000) {
            $e->addNote('Large dataset: '.count($data).' items')
              ->mapContext(fn($ctx) => [...$ctx, 'dataset_size' => 'large']);
        }

        // Transform based on user role
        if (auth()->user()->isAdmin()) {
            $e->mapContext(fn($ctx) => [...$ctx, 'admin_view' => true]);
        } else {
            // Remove sensitive data for non-admins
            $e->mapMetadata(fn($meta) => array_filter(
                $meta,
                fn($k) => !in_array($k, ['query', 'trace']),
                ARRAY_FILTER_USE_KEY
            ));
        }
    });

    throw $e;
}
```

## Best Practices

1. **Transform early** - Apply transformations as soon as you catch exceptions
2. **Chain related transformations** - Group related modifications together
3. **Be consistent** - Use the same transformation patterns across your application
4. **Don't lose information** - Add, don't replace critical context
5. **Sanitize sensitive data** - Always remove passwords, keys, tokens
6. **Use `transform()` for complex changes** - Multiple properties at once
7. **Keep transformations pure** - Don't cause side effects in transformation functions

## Comparison with Other Approaches

| Approach | Use Case | Example |
|----------|----------|---------|
| `mapMessage()` | Single message change | Prefix, normalize format |
| `mapContext()` | Add/remove context data | Timestamps, sanitization |
| `mapMetadata()` | Debug information | Environment info, metrics |
| `mapTags()` | Categorization | Normalize, deduplicate |
| `mapNotes()` | Breadcrumb trail | Add timestamps, filter |
| `transform()` | Multiple properties | Complex multi-step changes |

## See Also

- [Error Context](error-context.md) - Structured context and metadata
- [Exception Notes](exception-notes.md) - Breadcrumb-style debugging
- [Error Wrapping](error-wrapping.md) - Wrapping lower-level exceptions
