---
title: Filtering
description: Filter tests by name, path, tags, and exclusion patterns.
---

Prism provides powerful filtering capabilities to run specific subsets of your test suite.

## Name Filter

Filter tests by name using regex patterns:

```bash
vendor/bin/prism test --filter "string.*"
```

Matches test descriptions against the provided regex pattern.

### Examples

```bash
# Run only string-related tests
vendor/bin/prism test --filter "^string"

# Run tests containing "validation"
vendor/bin/prism test --filter "validation"

# Run tests matching multiple patterns
vendor/bin/prism test --filter "string|number|boolean"
```

## Path Filter

Filter test files by path using glob patterns:

```bash
vendor/bin/prism test --path-filter "tests/required/**/*.json"
```

Only runs tests from files matching the glob pattern.

### Examples

```bash
# Run tests only from the 'required' directory
vendor/bin/prism test --path-filter "**/required/**"

# Run tests from specific subdirectories
vendor/bin/prism test --path-filter "tests/{draft7,draft2020}/**"

# Run only top-level test files
vendor/bin/prism test --path-filter "tests/*.json"
```

## Exclusion Filter

Exclude tests by name using regex patterns:

```bash
vendor/bin/prism test --exclude "optional|deprecated"
```

Excludes tests whose descriptions match the provided regex.

### Examples

```bash
# Exclude optional tests
vendor/bin/prism test --exclude "optional"

# Exclude multiple patterns
vendor/bin/prism test --exclude "optional|edge-case|experimental"

# Exclude tests with specific keywords
vendor/bin/prism test --exclude "slow|flaky"
```

## Tag Filter

Filter tests by tags defined in test files:

```bash
vendor/bin/prism test --tag "required"
```

Only runs tests that have the specified tag.

### Tag Definition

Define tags in your test files:

```json
{
  "description": "String validation",
  "schema": { "type": "string" },
  "tests": [
    {
      "description": "valid string",
      "data": "hello",
      "valid": true,
      "tags": ["string", "required", "basic"]
    },
    {
      "description": "edge case",
      "data": "",
      "valid": true,
      "tags": ["string", "edge-case"]
    }
  ]
}
```

### Examples

```bash
# Run only required tests
vendor/bin/prism test --tag "required"

# Run edge case tests
vendor/bin/prism test --tag "edge-case"

# Run regression tests
vendor/bin/prism test --tag "regression"
```

## Draft Filter

When using multiple validators, filter by validator name:

```bash
vendor/bin/prism test --draft "draft-7"
```

Runs tests only for the specified draft/validator.

### Examples

```bash
# Test only Draft 7 implementation
vendor/bin/prism test --draft "draft-7"

# Test only Draft 2020-12 implementation
vendor/bin/prism test --draft "draft-2020-12"
```

## Combining Filters

All filters can be combined for precise test selection:

```bash
vendor/bin/prism test \
    --filter "string" \
    --tag "required" \
    --exclude "optional" \
    --path-filter "**/core/**"
```

This runs tests that:
- Match "string" in the name
- Have the "required" tag
- Don't match "optional" in the name
- Are in files under a "core" directory

## Incremental Mode

Run only tests from files that changed since the last run:

```bash
vendor/bin/prism test --incremental
```

Prism tracks file modification times and only runs tests from changed files.

### Use Cases

- **Fast iteration**: Only run tests affected by recent changes
- **CI optimization**: Skip unchanged tests in continuous integration
- **Development workflow**: Quickly verify local changes

### How It Works

1. First run: All tests execute, file timestamps cached
2. Subsequent runs: Only changed files tested
3. Cache location: `.prism/incremental-cache.json`

### Example Workflow

```bash
# Initial run - all tests
vendor/bin/prism test --incremental

# Make changes to test files
vim tests/validation/strings.json

# Only changed tests run
vendor/bin/prism test --incremental
```

## Interactive Filter Configuration

Use interactive mode to configure filters through a menu:

```bash
vendor/bin/prism test --interactive
```

Interactive mode provides:
- Name filter configuration
- Tag selection
- Parallel worker configuration
- Real-time filter preview
- Test execution with configured filters

### Interactive Menu

```
Current Configuration:
  Filter: <none>
  Tag: <none>
  Parallel: 1 worker(s)

Select action:
  > Run tests with current configuration
    Set name filter (regex)
    Set tag filter
    Configure parallel workers
    Clear all filters
    Exit
```

## Performance Tips

### Optimize Filter Performance

1. **Use path filters for large test suites**
   ```bash
   vendor/bin/prism test --path-filter "tests/critical/**"
   ```

2. **Combine with parallel execution**
   ```bash
   vendor/bin/prism test --filter "api" --parallel 4
   ```

3. **Use incremental mode during development**
   ```bash
   vendor/bin/prism test --incremental --watch
   ```

### Filter Precedence

Filters are applied in this order:

1. **Path filter** - Files are filtered first
2. **File-level filters** - `shouldIncludeFile()` in config
3. **Test execution** - Tests run from filtered files
4. **Name filter** - Test names filtered
5. **Tag filter** - Tests filtered by tags
6. **Exclusion filter** - Matching tests excluded

## Next Steps

- Explore [performance features](/prism/performance/)
- Learn about [advanced features](/prism/advanced-features/)
- See [output formats](/prism/output-formats/)
