# Performance

Optimization strategies and caching for ECMA-262 Regex.

## Caching Architecture

The package uses a two-tier caching system for optimal performance:

### 1. Memory Cache (L1)

In-memory array cache for instant access within a request:

```php
use Cline\EcmaRegex\Facades\EcmaRegex;

// First call: compiles pattern
$pattern = EcmaRegex::compile('/^[a-z]+$/');

// Subsequent calls in same request: instant retrieval
$pattern = EcmaRegex::compile('/^[a-z]+$/');  // From memory
```

### 2. Persistent Cache (L2)

Laravel Cache for cross-request pattern reuse:

```php
// First request: compiles and caches
$pattern = EcmaRegex::compile('/complex-pattern/');

// Later requests: retrieves from persistent cache
$pattern = EcmaRegex::compile('/complex-pattern/');  // From Laravel Cache
```

## Cache Configuration

Configure caching in `config/ecma-regex.php`:

```php
return [
    // Enable/disable caching
    'cache_enabled' => env('ECMA_REGEX_CACHE_ENABLED', true),

    // Cache lifetime in seconds (default: 1 hour)
    'cache_ttl' => env('ECMA_REGEX_CACHE_TTL', 3600),
];
```

Or via environment variables:

```bash
ECMA_REGEX_CACHE_ENABLED=true
ECMA_REGEX_CACHE_TTL=7200  # 2 hours
```

## Performance Best Practices

### 1. Compile Once, Use Many Times

**Bad:**
```php
foreach ($emails as $email) {
    // Recompiles pattern every iteration
    EcmaRegex::test('/^[a-z]+@[a-z]+\.[a-z]+$/', $email);
}
```

**Good:**
```php
// Compile once
$pattern = EcmaRegex::compile('/^[a-z]+@[a-z]+\.[a-z]+$/');

foreach ($emails as $email) {
    // Reuses compiled pattern
    $pattern->test($email);
}
```

### 2. Use Facade for Single Tests

For one-off tests, use the facade directly:

```php
// Efficient for single use
if (EcmaRegex::test('/^\d+$/', $input)) {
    // Handle numeric input
}
```

### 3. Leverage Automatic Caching

The manager automatically caches compiled patterns:

```php
// Compiles and caches
EcmaRegex::compile('/pattern/');

// Retrieved from cache (fast)
EcmaRegex::compile('/pattern/');
```

### 4. Choose Appropriate Cache TTL

**Short-lived patterns** (frequently changing):
```php
// 5 minutes
'cache_ttl' => 300,
```

**Long-lived patterns** (static validation):
```php
// 24 hours
'cache_ttl' => 86400,
```

**Permanent patterns** (no expiration):
```php
// No expiration
'cache_ttl' => null,
```

## Benchmarks

Typical performance characteristics:

### Pattern Compilation

```
First compilation:  ~2-5ms  (parse + AST generation)
From memory cache:  ~0.01ms (instant retrieval)
From persistent:    ~0.1ms  (cache lookup)
```

### Pattern Matching

```
Simple patterns:    ~0.05-0.1ms per match
Complex patterns:   ~0.2-0.5ms per match
Backtracking:       ~0.5-2ms per match
```

### Real-World Example

Validating 10,000 emails:

**Without caching:**
```
10,000 compilations × 3ms = 30,000ms (30 seconds)
10,000 matches × 0.1ms = 1,000ms (1 second)
Total: ~31 seconds
```

**With caching:**
```
1 compilation × 3ms = 3ms
10,000 matches × 0.1ms = 1,000ms (1 second)
Total: ~1 second
```

**30x performance improvement!**

## Memory Usage

Typical memory footprint:

- **Compiled pattern**: ~5-20KB depending on complexity
- **Memory cache**: Grows with unique patterns (cleared per request)
- **Persistent cache**: Configurable via Laravel Cache settings

### Monitoring Cache Size

```php
// Count cached patterns (implement custom tracking)
class CacheMonitor
{
    public function getCachedPatternCount(): int
    {
        $cache = app('cache');
        $patterns = 0;

        // Scan cache for ecma-regex keys
        // Implementation depends on cache driver

        return $patterns;
    }
}
```

## Cache Invalidation

### Manual Flush

Clear all cached patterns:

```php
EcmaRegex::flushCache();
```

### Automatic Expiration

Patterns expire based on `cache_ttl`:

```php
// Cached for 1 hour
'cache_ttl' => 3600,
```

### Selective Invalidation

Clear specific patterns by changing the pattern string:

```php
// Old pattern (cached)
$pattern1 = EcmaRegex::compile('/old-pattern/');

// New pattern (separate cache entry)
$pattern2 = EcmaRegex::compile('/new-pattern/');
```

## Production Optimizations

### 1. Enable Persistent Cache

Use Redis or Memcached for best performance:

**Redis** (recommended):
```php
// config/cache.php
'default' => 'redis',

'stores' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'cache',
    ],
],
```

**Memcached**:
```php
'default' => 'memcached',

'stores' => [
    'memcached' => [
        'driver' => 'memcached',
        'servers' => [
            ['host' => '127.0.0.1', 'port' => 11211, 'weight' => 100],
        ],
    ],
],
```

### 2. Optimize Cache TTL

Match TTL to your deployment frequency:

```php
// Deploy daily → 24 hour cache
'cache_ttl' => 86400,

// Deploy hourly → 1 hour cache
'cache_ttl' => 3600,
```

### 3. Warmup Cache

Pre-compile common patterns on deployment:

```php
// database/seeders/RegexCacheSeeder.php
use Cline\EcmaRegex\Facades\EcmaRegex;

class RegexCacheSeeder extends Seeder
{
    public function run(): void
    {
        $patterns = [
            '/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/',
            '/^\d{3}-\d{3}-\d{4}$/',
            '/^https?:\/\/.+/',
            // ... add your common patterns
        ];

        foreach ($patterns as $pattern) {
            EcmaRegex::compile($pattern);
        }
    }
}
```

Run after deployment:
```bash
php artisan db:seed --class=RegexCacheSeeder
```

### 4. Monitor Performance

Log slow pattern compilations:

```php
use Illuminate\Support\Facades\Log;

$start = microtime(true);
$pattern = EcmaRegex::compile('/complex-pattern/');
$duration = (microtime(true) - $start) * 1000;

if ($duration > 10) {  // 10ms threshold
    Log::warning("Slow regex compilation: {$duration}ms", [
        'pattern' => '/complex-pattern/',
    ]);
}
```

## Optimization Checklist

- [ ] Enable caching (`cache_enabled = true`)
- [ ] Set appropriate TTL for your deployment frequency
- [ ] Use Redis or Memcached in production
- [ ] Compile patterns once, reuse many times
- [ ] Warmup cache with common patterns on deployment
- [ ] Monitor cache hit rates and compilation times
- [ ] Flush cache after major updates if needed

## Profiling

### Pattern Compilation Time

```php
$start = microtime(true);
$pattern = EcmaRegex::compile('/your-pattern/');
$compileTime = (microtime(true) - $start) * 1000;

echo "Compilation: {$compileTime}ms\n";
```

### Match Execution Time

```php
$pattern = EcmaRegex::compile('/your-pattern/');

$start = microtime(true);
$result = $pattern->test('your input');
$matchTime = (microtime(true) - $start) * 1000;

echo "Match: {$matchTime}ms\n";
```

### Bulk Testing

```php
$pattern = EcmaRegex::compile('/your-pattern/');
$inputs = [...];  // 1000 test strings

$start = microtime(true);
foreach ($inputs as $input) {
    $pattern->test($input);
}
$totalTime = (microtime(true) - $start) * 1000;
$avgTime = $totalTime / count($inputs);

echo "Total: {$totalTime}ms\n";
echo "Average: {$avgTime}ms per match\n";
```

## Troubleshooting

### High Memory Usage

If memory usage is high:

1. **Reduce cache TTL** - shorter lifetime = less cached data
2. **Disable persistent cache** - use memory cache only
3. **Clear cache periodically** - call `flushCache()` during maintenance

```php
// In a scheduled task
Schedule::call(function () {
    EcmaRegex::flushCache();
})->daily();
```

### Slow Performance

If matching is slow:

1. **Enable caching** - ensure `cache_enabled = true`
2. **Compile patterns once** - avoid recompilation in loops
3. **Simplify patterns** - complex backtracking patterns are slower
4. **Use simpler alternatives** - consider if full regex is needed

### Cache Not Working

Verify cache is enabled and configured:

```php
// Check cache status
$config = config('ecma-regex.cache_enabled');
echo "Cache enabled: " . ($config ? 'yes' : 'no') . "\n";

// Check Laravel cache is working
Cache::put('test-key', 'test-value', 60);
echo "Laravel cache: " . Cache::get('test-key') . "\n";
```
