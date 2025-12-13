---
title: Exception Filtering
description: Search and filter through exception chains to find specific errors buried in wrapped exceptions.
---

Search and filter through exception chains to find specific errors buried in wrapped exceptions.

## Overview

Exception filtering provides methods to traverse exception chains (via `getPrevious()`) and find specific exception types or patterns. This is essential when dealing with wrapped exceptions where the root cause is buried several layers deep.

```php
$pdoError = $exception->findFirst(PDOException::class);
if ($pdoError !== null) {
    logger()->error('Database error', [
        'code' => $pdoError->getCode(),
    ]);
}
```

## Available Methods

### findFirst()

Find the first exception of a specific type in the chain.

```php
try {
    $service->process($data);
} catch (Throwable $e) {
    $validation = $e->findFirst(ValidationException::class);
    if ($validation !== null) {
        return response()->json(['errors' => $validation->getErrors()], 422);
    }
}
```

### findAll()

Find all exceptions of a specific type in the chain.

```php
try {
    $service->processMultiple($items);
} catch (Throwable $e) {
    $validationErrors = $e->findAll(ValidationException::class);
    foreach ($validationErrors as $error) {
        logger()->warning($error->getMessage());
    }
}
```

### getChain()

Get the entire exception chain as an array.

```php
foreach ($exception->getChain() as $i => $e) {
    logger()->debug("Exception #{$i}: " . get_class($e) . " - " . $e->getMessage());
}
```

### filterChain()

Filter the exception chain using a custom callback.

```php
$networkErrors = $exception->filterChain(
    fn($e) => str_contains($e->getMessage(), 'connection')
);
```

### hasInChain()

Check if the exception chain contains a specific type.

```php
if ($exception->hasInChain(PDOException::class)) {
    // Handle database error
}
```

### getRootCause()

Get the root cause exception (the last in the chain).

```php
$root = $exception->getRootCause();
logger()->error('Root cause: ' . get_class($root) . ' - ' . $root->getMessage());
```

### getChainDepth()

Get the number of exceptions in the chain.

```php
if ($exception->getChainDepth() > 5) {
    logger()->warning('Deep exception chain detected');
}
```

## Real-World Examples

### Finding Validation Errors in Service Layer

```php
class OrderController
{
    public function create(Request $request)
    {
        try {
            $order = $this->orderService->create($request->all());
            return response()->json($order, 201);
        } catch (Throwable $e) {
            // Service may wrap validation exceptions
            $validation = $e->findFirst(ValidationException::class);

            if ($validation !== null) {
                return response()->json([
                    'message' => 'Validation failed',
                    'errors' => $validation->getErrors(),
                ], 422);
            }

            throw $e;
        }
    }
}
```

### Logging Entire Exception Chain

```php
class ExceptionLogger
{
    public function log(Throwable $exception): void
    {
        if (!method_exists($exception, 'getChain')) {
            logger()->error($exception->getMessage());
            return;
        }

        $chain = $exception->getChain();

        logger()->error('Exception chain detected', [
            'depth' => count($chain),
            'chain' => array_map(function($e) {
                return [
                    'type' => get_class($e),
                    'message' => $e->getMessage(),
                    'code' => $e->getCode(),
                    'file' => $e->getFile(),
                    'line' => $e->getLine(),
                ];
            }, $chain),
        ]);
    }
}
```

### Finding Database Errors in Multi-Layer Architecture

```php
class DatabaseErrorHandler
{
    public function handle(Throwable $exception): void
    {
        if (!method_exists($exception, 'findFirst')) {
            return;
        }

        // Look for PDO exceptions anywhere in the chain
        $pdoError = $exception->findFirst(PDOException::class);

        if ($pdoError !== null) {
            logger()->critical('Database error detected', [
                'error_code' => $pdoError->getCode(),
                'sql_state' => $pdoError->errorInfo[0] ?? null,
                'driver_code' => $pdoError->errorInfo[1] ?? null,
                'message' => $pdoError->errorInfo[2] ?? $pdoError->getMessage(),
            ]);

            // Alert ops team for certain error codes
            if (in_array($pdoError->getCode(), [2002, 2003, 2006])) {
                $this->alertOpsTeam('Database connection lost', $pdoError);
            }
        }
    }
}
```

### Filtering Critical Errors for Alerting

```php
class ErrorAlerter
{
    public function shouldAlert(Throwable $exception): bool
    {
        if (!method_exists($exception, 'filterChain')) {
            return $exception->getCode() >= 500;
        }

        // Find all critical errors in the chain
        $criticalErrors = $exception->filterChain(
            fn($e) => $e->getCode() >= 500
        );

        return count($criticalErrors) > 0;
    }

    public function alert(Throwable $exception): void
    {
        if ($this->shouldAlert($exception)) {
            $criticalErrors = $exception->filterChain(
                fn($e) => $e->getCode() >= 500
            );

            foreach ($criticalErrors as $error) {
                $this->sendAlert([
                    'type' => get_class($error),
                    'message' => $error->getMessage(),
                    'code' => $error->getCode(),
                ]);
            }
        }
    }
}
```

### Finding All Database Errors

```php
class DatabaseMonitor
{
    public function reportDatabaseErrors(Throwable $exception): void
    {
        if (!method_exists($exception, 'findAll')) {
            return;
        }

        $dbErrors = $exception->findAll(PDOException::class);

        if (empty($dbErrors)) {
            return;
        }

        logger()->warning('Multiple database errors detected', [
            'count' => count($dbErrors),
            'errors' => array_map(fn($e) => [
                'message' => $e->getMessage(),
                'code' => $e->getCode(),
            ], $dbErrors),
        ]);

        // Might indicate connection pool issues
        if (count($dbErrors) > 3) {
            $this->checkConnectionPool();
        }
    }
}
```

### Detecting Deep Exception Wrapping

```php
class ExceptionAnalyzer
{
    public function analyze(Throwable $exception): array
    {
        if (!method_exists($exception, 'getChainDepth')) {
            return ['depth' => 1];
        }

        $depth = $exception->getChainDepth();

        // Deep wrapping might indicate architectural issues
        if ($depth > 5) {
            logger()->warning('Deep exception chain detected', [
                'depth' => $depth,
                'chain' => array_map(
                    fn($e) => get_class($e),
                    $exception->getChain()
                ),
            ]);
        }

        return [
            'depth' => $depth,
            'root_cause' => get_class($exception->getRootCause()),
        ];
    }
}
```

### Custom Chain Filtering

```php
class ExceptionFilter
{
    public function findNetworkErrors(Throwable $exception): array
    {
        if (!method_exists($exception, 'filterChain')) {
            return [];
        }

        return $exception->filterChain(function($e) {
            $message = strtolower($e->getMessage());

            return str_contains($message, 'connection')
                || str_contains($message, 'timeout')
                || str_contains($message, 'network')
                || str_contains($message, 'unreachable');
        });
    }

    public function findAppErrors(Throwable $exception): array
    {
        if (!method_exists($exception, 'filterChain')) {
            return [];
        }

        return $exception->filterChain(function($e) {
            return str_starts_with(get_class($e), 'App\Exceptions\')
                && $e->getCode() > 0;
        });
    }
}
```

### Finding Specific Error Codes

```php
class PaymentErrorHandler
{
    public function handlePaymentError(Throwable $exception): void
    {
        if (!method_exists($exception, 'filterChain')) {
            return;
        }

        // Find all payment-related errors
        $paymentErrors = $exception->filterChain(
            fn($e) => $e instanceof PaymentException
        );

        foreach ($paymentErrors as $error) {
            match($error->getCode()) {
                4001 => $this->handleInsufficientFunds($error),
                4002 => $this->handleCardDeclined($error),
                4003 => $this->handleExpiredCard($error),
                default => $this->handleGenericPaymentError($error),
            };
        }
    }
}
```

### Inspecting Exception Chain for Debugging

```php
class DebugController
{
    public function showExceptionChain(Throwable $exception)
    {
        if (!method_exists($exception, 'getChain')) {
            return view('error', ['exception' => $exception]);
        }

        $chain = $exception->getChain();
        $depth = $exception->getChainDepth();
        $root = $exception->getRootCause();

        return view('debug.exception-chain', [
            'exception' => $exception,
            'chain' => $chain,
            'depth' => $depth,
            'root_cause' => $root,
            'has_pdo_errors' => $exception->hasInChain(PDOException::class),
            'has_validation_errors' => $exception->hasInChain(ValidationException::class),
        ]);
    }
}
```

### Finding Specific Layer Errors

```php
class LayeredExceptionHandler
{
    public function identifyFailureLayer(Throwable $exception): string
    {
        if (!method_exists($exception, 'findFirst')) {
            return 'unknown';
        }

        if ($exception->findFirst(RepositoryException::class) !== null) {
            return 'data_layer';
        }

        if ($exception->findFirst(ServiceException::class) !== null) {
            return 'service_layer';
        }

        if ($exception->findFirst(ValidationException::class) !== null) {
            return 'validation_layer';
        }

        if ($exception->findFirst(PDOException::class) !== null) {
            return 'database_layer';
        }

        return 'application_layer';
    }
}
```

### Conditional Error Handling Based on Chain

```php
class SmartErrorHandler
{
    public function handle(Throwable $exception): Response
    {
        if (!method_exists($exception, 'hasInChain')) {
            return $this->defaultErrorResponse($exception);
        }

        // Check for specific error types in chain
        if ($exception->hasInChain(ValidationException::class)) {
            $validation = $exception->findFirst(ValidationException::class);
            return response()->json([
                'message' => 'Validation failed',
                'errors' => $validation->getErrors(),
            ], 422);
        }

        if ($exception->hasInChain(AuthenticationException::class)) {
            return response()->json([
                'message' => 'Authentication required',
            ], 401);
        }

        if ($exception->hasInChain(PDOException::class)) {
            logger()->critical('Database error in chain', [
                'root' => $exception->getRootCause()->getMessage(),
            ]);

            return response()->json([
                'message' => 'Service temporarily unavailable',
            ], 503);
        }

        return $this->defaultErrorResponse($exception);
    }
}
```

### Analyzing Exception Patterns

```php
class ExceptionMetrics
{
    public function recordMetrics(Throwable $exception): void
    {
        if (!method_exists($exception, 'getChain')) {
            $this->recordSimpleException($exception);
            return;
        }

        $chain = $exception->getChain();
        $types = array_map(fn($e) => get_class($e), $chain);

        // Record chain pattern
        $pattern = implode(' -> ', $types);

        Metrics::increment('exception.chains', [
            'pattern' => $pattern,
            'depth' => count($chain),
            'root_type' => get_class($exception->getRootCause()),
        ]);

        // Record specific error types found
        foreach ($chain as $e) {
            Metrics::increment('exception.types', [
                'type' => get_class($e),
                'code' => $e->getCode(),
            ]);
        }
    }
}
```

## Advanced Patterns

### Building Exception Reports

```php
class ExceptionReporter
{
    public function generateReport(Throwable $exception): array
    {
        if (!method_exists($exception, 'getChain')) {
            return $this->simpleReport($exception);
        }

        $chain = $exception->getChain();

        return [
            'summary' => [
                'message' => $exception->getMessage(),
                'type' => get_class($exception),
                'chain_depth' => $exception->getChainDepth(),
                'root_cause_type' => get_class($exception->getRootCause()),
            ],
            'chain' => array_map(fn($e, $i) => [
                'level' => $i,
                'type' => get_class($e),
                'message' => $e->getMessage(),
                'code' => $e->getCode(),
                'file' => $e->getFile(),
                'line' => $e->getLine(),
            ], $chain, array_keys($chain)),
            'analysis' => [
                'has_validation_errors' => $exception->hasInChain(ValidationException::class),
                'has_database_errors' => $exception->hasInChain(PDOException::class),
                'has_network_errors' => !empty($exception->filterChain(
                    fn($e) => str_contains($e->getMessage(), 'connection')
                )),
                'critical_errors_count' => count($exception->filterChain(
                    fn($e) => $e->getCode() >= 500
                )),
            ],
        ];
    }
}
```

### Chain-Aware Retry Logic

```php
class RetryHandler
{
    public function shouldRetry(Throwable $exception): bool
    {
        if (!method_exists($exception, 'hasInChain')) {
            return false;
        }

        // Don't retry validation errors
        if ($exception->hasInChain(ValidationException::class)) {
            return false;
        }

        // Retry connection errors
        if ($exception->hasInChain(ConnectionException::class)) {
            return true;
        }

        // Retry timeout errors
        $timeoutErrors = $exception->filterChain(
            fn($e) => str_contains($e->getMessage(), 'timeout')
        );

        return !empty($timeoutErrors);
    }
}
```

## Best Practices

1. **Check for method existence** - Use `method_exists()` before calling filtering methods on generic `Throwable`
2. **Handle null returns** - `findFirst()` returns `null` when no match is found
3. **Consider performance** - Chain traversal is O(n), cache results if checking multiple types
4. **Log chain depth** - Deep chains might indicate architectural issues
5. **Use specific types** - Filter for specific exception types rather than broad categories
6. **Combine with context** - Use filtering alongside exception context for better debugging

## See Also

- [Error Context](error-context.md) - Structured context and metadata
- [Error Wrapping](error-wrapping.md) - Wrapping lower-level exceptions
- [Exception Notes](exception-notes.md) - Breadcrumb-style debugging
