# Troubleshooting

Common issues and solutions when using ECMA-262 Regex.

## Pattern Compilation Issues

### Incomplete Escape Sequence

**Error:**
```
IncompleteEscapeSequenceException: Incomplete escape sequence at position 5
```

**Cause:** Backslash at end of pattern or invalid escape

**Examples:**
```php
// ✗ Bad
EcmaRegex::test('/test\/', 'test');      // Trailing backslash
EcmaRegex::test('/\/', 'test');          // Lone backslash

// ✓ Good
EcmaRegex::test('/test\\/', 'test\\');   // Escaped backslash
EcmaRegex::test('/test/', 'test');       // No backslash needed
```

**Solution:** Ensure all backslashes are part of valid escape sequences

### Unclosed Character Class

**Error:**
```
UnclosedCharacterClassException: Unclosed character class starting at position 3
```

**Cause:** Missing `]` in character class

**Examples:**
```php
// ✗ Bad
EcmaRegex::test('/[a-z/', 'test');       // Missing ]

// ✓ Good
EcmaRegex::test('/[a-z]/', 'test');      // Properly closed
```

**Solution:** Ensure every `[` has a matching `]`

### Incomplete Group Construct

**Error:**
```
IncompleteGroupConstructException: Incomplete group construct at position 2
```

**Cause:** Missing `)` in group

**Examples:**
```php
// ✗ Bad
EcmaRegex::test('/(test/', 'test');      // Missing )
EcmaRegex::test('/(?:abc/', 'abc');      // Missing )

// ✓ Good
EcmaRegex::test('/(test)/', 'test');     // Properly closed
EcmaRegex::test('/(?:abc)/', 'abc');     // Properly closed
```

**Solution:** Ensure every `(` has a matching `)`

### Invalid Group Construct

**Error:**
```
InvalidGroupConstructException: Invalid group construct: (?x)
```

**Cause:** Unsupported group type or invalid syntax

**Examples:**
```php
// ✗ Bad
EcmaRegex::test('/(?x)/', 'test');       // Invalid group type
EcmaRegex::test('/(?<name)/', 'test');   // Incomplete named group

// ✓ Good
EcmaRegex::test('/(?:test)/', 'test');   // Non-capturing group
EcmaRegex::test('/(?=test)/', 'test');   // Lookahead
EcmaRegex::test('/(?!test)/', 'test');   // Negative lookahead
EcmaRegex::test('/(?<=test)/', 'test');  // Lookbehind
EcmaRegex::test('/(?<!test)/', 'test');  // Negative lookbehind
```

**Supported group types:**
- `(...)` - Capturing group
- `(?:...)` - Non-capturing group
- `(?=...)` - Positive lookahead
- `(?!...)` - Negative lookahead
- `(?<=...)` - Positive lookbehind
- `(?<!...)` - Negative lookbehind

### Invalid Lookbehind Construct

**Error:**
```
InvalidLookbehindConstructException: Invalid lookbehind assertion syntax
```

**Cause:** Malformed lookbehind syntax

**Examples:**
```php
// ✗ Bad
EcmaRegex::test('/(?<test)/', 'test');   // Missing =
EcmaRegex::test('/(?<=)/', 'test');      // Empty assertion

// ✓ Good
EcmaRegex::test('/(?<=@)\w+/', '@test'); // Proper lookbehind
EcmaRegex::test('/(?<!@)\w+/', 'test');  // Proper negative lookbehind
```

### Unexpected Token

**Error:**
```
UnexpectedTokenException: Unexpected token ')' at position 5
```

**Cause:** Token in invalid position

**Examples:**
```php
// ✗ Bad
EcmaRegex::test('/()/', 'test');         // Empty group
EcmaRegex::test('/*/', 'test');          // Quantifier without target

// ✓ Good
EcmaRegex::test('/(test)/', 'test');     // Group with content
EcmaRegex::test('/a*/', 'aaa');          // Quantifier with target
```

## Matching Issues

### Pattern Doesn't Match Expected Input

**Problem:** Pattern compiles but doesn't match when it should

**Debug steps:**

1. **Test with simpler pattern:**
```php
// Start simple
EcmaRegex::test('/test/', 'test');  // Does literal match work?

// Add complexity gradually
EcmaRegex::test('/^test/', 'test');  // Does anchor work?
EcmaRegex::test('/^test$/', 'test'); // Do both anchors work?
```

2. **Check for anchors:**
```php
// ✗ Won't match - anchors too restrictive
EcmaRegex::test('/^test$/', 'test 123');

// ✓ Will match - no anchors
EcmaRegex::test('/test/', 'test 123');
```

3. **Verify case sensitivity:**
```php
// ✗ Case sensitive (default)
EcmaRegex::test('/hello/', 'Hello');  // false

// ✓ Case insensitive
$pattern = EcmaRegex::compile('/hello/', 'i');
$pattern->test('Hello');  // true
```

4. **Check character classes:**
```php
// ✗ Doesn't include uppercase
EcmaRegex::test('/[a-z]+/', 'Hello');  // false

// ✓ Includes both cases
EcmaRegex::test('/[a-zA-Z]+/', 'Hello');  // true
```

5. **Inspect match result:**
```php
$result = EcmaRegex::match('/pattern/', 'input');

echo "Matched: " . ($result->matched ? 'yes' : 'no') . "\n";
echo "Match: {$result->match}\n";
echo "Index: {$result->index}\n";
print_r($result->captures);
```

### Pattern Matches Too Much

**Problem:** Pattern matches more than intended

**Solutions:**

1. **Add anchors:**
```php
// ✗ Matches partial strings
EcmaRegex::test('/\d+/', 'abc123def');  // true

// ✓ Requires entire string
EcmaRegex::test('/^\d+$/', 'abc123def');  // false
EcmaRegex::test('/^\d+$/', '123');        // true
```

2. **Use word boundaries:**
```php
// ✗ Matches within words
EcmaRegex::test('/cat/', 'concatenate');  // true

// ✓ Matches whole words only
EcmaRegex::test('/\bcat\b/', 'concatenate');  // false
EcmaRegex::test('/\bcat\b/', 'cat');          // true
```

3. **Make quantifiers lazy:**
```php
// ✗ Greedy - matches too much
$result = EcmaRegex::match('/".+"/', '"first" "second"');
echo $result->match;  // "first" "second"

// ✓ Lazy - matches minimally
$result = EcmaRegex::match('/".+?"/', '"first" "second"');
echo $result->match;  // "first"
```

### Backreferences Not Working

**Problem:** Backreferences `\1`, `\2` don't match

**Common issues:**

1. **Using non-capturing groups:**
```php
// ✗ Non-capturing - no backreference
EcmaRegex::test('/(?:a+) \1/', 'aa aa');  // false

// ✓ Capturing - backreference works
EcmaRegex::test('/(a+) \1/', 'aa aa');    // true
```

2. **Wrong backreference number:**
```php
// ✗ Only one group, but referencing \2
EcmaRegex::test('/(a+) \2/', 'aa aa');  // false

// ✓ Correct reference
EcmaRegex::test('/(a+) \1/', 'aa aa');  // true
```

3. **Backreference to non-existent group:**
```php
// ✗ No groups defined
EcmaRegex::test('/\1/', 'test');  // false

// ✓ Group defined before backreference
EcmaRegex::test('/(test) \1/', 'test test');  // true
```

## Performance Issues

### Slow Pattern Compilation

**Problem:** First compilation takes too long

**Solutions:**

1. **Enable caching:**
```php
// config/ecma-regex.php
'cache_enabled' => true,
```

2. **Warmup cache:**
```php
// Compile common patterns on deployment
$patterns = [
    '/^[a-z]+$/',
    '/\d{3}-\d{3}-\d{4}/',
    // ... your patterns
];

foreach ($patterns as $pattern) {
    EcmaRegex::compile($pattern);
}
```

3. **Simplify complex patterns:**
```php
// ✗ Overly complex
'/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/'

// ✓ Simpler alternative - validate each rule separately
EcmaRegex::test('/[a-z]/', $password) &&
EcmaRegex::test('/[A-Z]/', $password) &&
EcmaRegex::test('/\d/', $password) &&
EcmaRegex::test('/[@$!%*?&]/', $password) &&
strlen($password) >= 8
```

### Slow Matching

**Problem:** Pattern matching takes too long

**Solutions:**

1. **Reuse compiled patterns:**
```php
// ✗ Recompiles every time
foreach ($inputs as $input) {
    EcmaRegex::test('/pattern/', $input);
}

// ✓ Compile once
$pattern = EcmaRegex::compile('/pattern/');
foreach ($inputs as $input) {
    $pattern->test($input);
}
```

2. **Avoid catastrophic backtracking:**
```php
// ✗ Catastrophic backtracking possible
'/a+a+a+$/'

// ✓ More specific
'/a{3,}$/'
```

3. **Use anchors to fail fast:**
```php
// ✗ Searches entire string
EcmaRegex::test('/test/', $longString);

// ✓ Fails immediately if not at start
EcmaRegex::test('/^test/', $longString);
```

### High Memory Usage

**Problem:** Cache using too much memory

**Solutions:**

1. **Reduce cache TTL:**
```php
// config/ecma-regex.php
'cache_ttl' => 300,  // 5 minutes instead of 1 hour
```

2. **Clear cache periodically:**
```php
// In scheduled task
EcmaRegex::flushCache();
```

3. **Disable persistent cache:**
```php
'cache_enabled' => false,
```

## Integration Issues

### Cache Not Persisting

**Problem:** Patterns recompiled on each request

**Debug:**

1. **Check cache is enabled:**
```php
echo config('ecma-regex.cache_enabled') ? 'enabled' : 'disabled';
```

2. **Verify Laravel cache works:**
```php
Cache::put('test-key', 'test-value', 60);
$value = Cache::get('test-key');
echo $value;  // Should output: test-value
```

3. **Check cache driver:**
```php
echo config('cache.default');  // Should be redis/memcached in production
```

**Solutions:**

1. **Configure cache driver:**
```php
// .env
CACHE_DRIVER=redis
```

2. **Publish config:**
```bash
php artisan vendor:publish --tag="ecma-regex-config"
```

### Facade Not Working

**Problem:** `EcmaRegex` facade not found

**Solutions:**

1. **Clear config cache:**
```bash
php artisan config:clear
```

2. **Verify service provider registered:**
```php
// Should be auto-discovered, but can add manually to config/app.php
'providers' => [
    Cline\EcmaRegex\EcmaRegexServiceProvider::class,
],
```

3. **Check facade alias:**
```php
// config/app.php
'aliases' => [
    'EcmaRegex' => Cline\EcmaRegex\Facades\EcmaRegex::class,
],
```

## ECMA-262 vs PCRE Differences

### Pattern Behavior Differences

**Issue:** Pattern works in JavaScript but not in this package

**Common causes:**

1. **Pattern delimiter confusion:**
```php
// ✗ Forgot delimiters
EcmaRegex::test('^test$', 'test');  // error

// ✓ Include delimiters
EcmaRegex::test('/^test$/', 'test');  // works
```

2. **Flag differences:**
```php
// JavaScript
/pattern/gi

// PHP
EcmaRegex::compile('/pattern/', 'gi');
```

3. **Unicode handling:**
```php
// May need unicode flag
$pattern = EcmaRegex::compile('/café/', 'u');
```

## Getting Help

### Enable Debug Mode

Get detailed error information:

```php
try {
    $result = EcmaRegex::test('/pattern/', 'input');
} catch (EcmaRegexException $e) {
    echo "Error: {$e->getMessage()}\n";
    echo "File: {$e->getFile()}:{$e->getLine()}\n";
    echo "Trace:\n{$e->getTraceAsString()}\n";
}
```

### Test Pattern Incrementally

Build pattern piece by piece:

```php
// Start simple
EcmaRegex::test('/a/', 'a');           // Works?

// Add quantifier
EcmaRegex::test('/a+/', 'aaa');        // Works?

// Add character class
EcmaRegex::test('/[a-z]+/', 'abc');    // Works?

// Add anchors
EcmaRegex::test('/^[a-z]+$/', 'abc');  // Works?
```

### Compare with JavaScript

Test in browser console:

```javascript
// JavaScript
/^[a-z]+$/.test('hello')  // true

// Should match in PHP
EcmaRegex::test('/^[a-z]+$/', 'hello')  // true
```

### Report Issues

If you find a bug:

1. Create minimal reproduction case
2. Test in JavaScript to verify expected behavior
3. Report issue with pattern, input, and expected vs actual result
4. Include error messages and stack traces
