---
title: Deferred Cleanup
description: Zig-inspired deferred cleanup that executes only when errors occur, ensuring resources are properly cleaned up on error paths.
---

Zig-inspired deferred cleanup that executes only when errors occur, ensuring resources are properly cleaned up on error paths.

## Overview

Deferred cleanup allows you to register callbacks that execute only when an exception is thrown, inspired by Zig's `errdefer`. This eliminates the need for explicit try-catch blocks for cleanup while ensuring resources are released on error paths.

```php
$cleanup = errdefer();
$cleanup->onError(fn() => DB::rollBack());

DB::beginTransaction();
$user = User::create($data); // If this throws, rollback runs automatically
DB::commit();
```

## Basic Usage

### Registering Cleanup Callbacks

```php
use function Cline\Throw\errdefer;

$cleanup = errdefer();

// Register cleanup that runs on error
$cleanup->onError(function() {
    fclose($fileHandle);
});
```

### Using with run()

```php
$cleanup = errdefer();
$cleanup->onError(fn() => unlink($tempFile));

$result = $cleanup->run(function() {
    // If this throws, temp file is deleted
    return processFile($tempFile);
});
```

### Manual Cleanup Trigger

```php
$cleanup = errdefer();
$cleanup->onError(fn() => rollbackTransaction());

if ($error) {
    $cleanup->cleanup(); // Manually trigger cleanup
}
```

## Real-World Examples

### Database Transactions

```php
class OrderService
{
    public function createOrder(array $data): Order
    {
        $cleanup = errdefer();
        $cleanup->onError(fn() => DB::rollBack());

        DB::beginTransaction();

        $order = Order::create($data);

        foreach ($data['items'] as $item) {
            $order->items()->create($item);
        }

        DB::commit();

        return $order;
    }
}
```

### File Upload with Cleanup

```php
class FileUploader
{
    public function upload(UploadedFile $file): string
    {
        $cleanup = errdefer();

        // Upload to temporary location
        $tempPath = $file->storeAs('temp', $file->hashName());
        $cleanup->onError(fn() => Storage::delete($tempPath));

        // Validate file
        $this->validate($tempPath);

        // Process file
        $processedPath = $this->process($tempPath);
        $cleanup->onError(fn() => Storage::delete($processedPath));

        // Move to final location
        $finalPath = $this->moveToFinal($processedPath);

        // Success - cleanup won't run
        return $finalPath;
    }
}
```

### Resource Management

```php
class ResourceManager
{
    public function processWithResources(): mixed
    {
        $cleanup = errdefer();

        $file = fopen('data.txt', 'w');
        $cleanup->onError(fn() => fclose($file));

        $lock = $this->acquireLock('processing');
        $cleanup->onError(fn() => $this->releaseLock($lock));

        $connection = $this->openConnection();
        $cleanup->onError(fn() => $connection->close());

        // Do work - if exception occurs, all resources are cleaned up
        return $this->processData($file, $connection);
    }
}
```

### API Client with Cleanup

```php
class ApiClient
{
    public function fetchWithRetry(string $endpoint): array
    {
        $cleanup = errdefer();

        $tempFile = tmpfile();
        $cleanup->onError(function() use ($tempFile) {
            if (is_resource($tempFile)) {
                fclose($tempFile);
            }
        });

        $response = Http::timeout(30)
            ->get($endpoint)
            ->throw();

        fwrite($tempFile, $response->body());

        return $this->processResponse($tempFile);
    }
}
```

### Multi-Step Operation

```php
class DeploymentService
{
    public function deploy(string $version): void
    {
        $cleanup = errdefer();

        // Step 1: Backup current version
        $backupPath = $this->backup();
        $cleanup->onError(fn() => $this->restore($backupPath));

        // Step 2: Download new version
        $downloadPath = $this->download($version);
        $cleanup->onError(fn() => unlink($downloadPath));

        // Step 3: Extract
        $extractPath = $this->extract($downloadPath);
        $cleanup->onError(fn() => $this->removeDirectory($extractPath));

        // Step 4: Deploy
        $this->installNewVersion($extractPath);

        // Success - no cleanup needed
    }
}
```

### Cache with Cleanup

```php
class CacheWriter
{
    public function writeToCache(string $key, mixed $data): void
    {
        $cleanup = errdefer();

        // Acquire write lock
        $lock = Cache::lock("write:{$key}", 10);
        $lock->get();
        $cleanup->onError(fn() => $lock->release());

        // Write to temp key first
        $tempKey = "{$key}:temp";
        Cache::put($tempKey, $data, 60);
        $cleanup->onError(fn() => Cache::forget($tempKey));

        // Validate data
        $this->validate(Cache::get($tempKey));

        // Swap to final key
        Cache::put($key, $data, 3600);
        Cache::forget($tempKey);

        $lock->release();
    }
}
```

### Background Job with Cleanup

```php
class ProcessOrderJob implements ShouldQueue
{
    public function handle(): void
    {
        $cleanup = errdefer();

        // Lock the order
        $lock = Cache::lock("order:{$this->orderId}", 300);
        $lock->get();
        $cleanup->onError(fn() => $lock->release());

        $order = Order::find($this->orderId);
        $order->update(['status' => 'processing']);
        $cleanup->onError(fn() => $order->update(['status' => 'pending']));

        // Process payment
        $this->processPayment($order);

        // Send confirmation
        $this->sendConfirmation($order);

        $order->update(['status' => 'completed']);
        $lock->release();
    }
}
```

### Nested Operations

```php
class UserRegistration
{
    public function register(array $data): User
    {
        $outerCleanup = errdefer();

        return $outerCleanup->run(function() use ($data) {
            DB::beginTransaction();
            $outerCleanup->onError(fn() => DB::rollBack());

            $user = User::create($data);

            // Nested operation with its own cleanup
            $innerCleanup = errdefer();
            $innerCleanup->run(function() use ($user) {
                $profile = $user->profile()->create([
                    'display_name' => $user->name,
                ]);

                $this->uploadAvatar($user, $profile);
            });

            DB::commit();

            return $user;
        });
    }
}
```

### Cleanup Execution Order

```php
class OrderProcessor
{
    public function process(Order $order): void
    {
        $cleanup = errdefer();

        // Cleanup executes in reverse order (LIFO)
        $cleanup->onError(fn() => Log::info('Cleanup step 1'));
        $cleanup->onError(fn() => Log::info('Cleanup step 2'));
        $cleanup->onError(fn() => Log::info('Cleanup step 3'));

        throw new RuntimeException('Error');

        // Output on error:
        // Cleanup step 3
        // Cleanup step 2
        // Cleanup step 1
    }
}
```

### Using with Attempt

```php
use function Cline\Throw\attempt;
use function Cline\Throw\errdefer;

class FileProcessor
{
    public function processFile(string $path): Result
    {
        $cleanup = errdefer();

        return attempt(function() use ($path, $cleanup) {
            $handle = fopen($path, 'r');
            $cleanup->onError(fn() => fclose($handle));

            $data = $this->readData($handle);
            $processed = $this->processData($data);

            fclose($handle);

            return $processed;
        })->toResult();
    }
}
```

## Advanced Patterns

### Conditional Cleanup

```php
$cleanup = errdefer();
$resourceAcquired = false;

$resource = $this->acquireResource();
$resourceAcquired = true;
$cleanup->onError(function() use (&$resourceAcquired, $resource) {
    if ($resourceAcquired) {
        $resource->release();
    }
});
```

### Cleanup with State

```php
class StatefulProcessor
{
    private array $acquiredResources = [];

    public function process(): void
    {
        $cleanup = errdefer();
        $cleanup->onError(fn() => $this->cleanupAll());

        foreach ($this->steps as $step) {
            $resource = $this->acquireResource($step);
            $this->acquiredResources[] = $resource;

            $this->executeStep($step, $resource);
        }

        // Success - clear acquired resources
        $this->acquiredResources = [];
    }

    private function cleanupAll(): void
    {
        foreach ($this->acquiredResources as $resource) {
            $resource->release();
        }

        $this->acquiredResources = [];
    }
}
```

### Cleanup Logging

```php
$cleanup = errdefer();

$cleanup->onError(function() {
    Log::warning('Operation failed, running cleanup', [
        'timestamp' => now(),
        'user_id' => auth()->id(),
    ]);

    $this->performCleanup();

    Log::info('Cleanup completed');
});
```

## Comparison with Try-Catch

### Traditional Approach

```php
try {
    DB::beginTransaction();

    $user = User::create($data);
    $profile = $user->profile()->create($profileData);

    DB::commit();
} catch (Throwable $e) {
    DB::rollBack();
    throw $e;
}
```

### Deferred Cleanup Approach

```php
$cleanup = errdefer();
$cleanup->onError(fn() => DB::rollBack());

DB::beginTransaction();
$user = User::create($data);
$profile = $user->profile()->create($profileData);
DB::commit();
```

## Best Practices

1. **Register cleanup immediately** - Right after acquiring a resource
2. **LIFO order** - Cleanup executes in reverse order of registration
3. **Idempotent cleanup** - Cleanup should be safe to run multiple times
4. **Single responsibility** - One cleanup callback per resource
5. **Use with run()** - For automatic cleanup on exceptions
6. **Manual cleanup** - Call `cleanup()` explicitly when needed
7. **Don't mix patterns** - Use either `run()` or destructor, not both

## See Also

- [Error Wrapping](error-wrapping.md) - Wrapping lower-level exceptions
- [Exception Notes](exception-notes.md) - Breadcrumb-style debugging
- [Result Integration](result-integration.md) - Converting attempts to Result types
