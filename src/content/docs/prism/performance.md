---
title: Performance
description: Optimize test execution with profiling, benchmarks, and parallel execution.
---

Prism provides comprehensive performance features for fast test execution and performance analysis.

## Parallel Execution

Run tests across multiple worker processes:

```bash
vendor/bin/prism test --parallel 4
```

Or use the short flag:

```bash
vendor/bin/prism test -j 4
```

### How It Works

1. Test files divided into batches
2. Each batch runs in separate PHP process
3. Results collected and merged
4. Total execution time significantly reduced

### Optimal Worker Count

```bash
# Use CPU core count
vendor/bin/prism test -j $(nproc)

# Conservative (50% of cores)
vendor/bin/prism test -j $(($(nproc) / 2))

# Maximum performance
vendor/bin/prism test -j $(nproc --all)
```

### Performance Comparison

```bash
# Sequential execution
vendor/bin/prism test
# Time: 45.2s

# Parallel with 4 workers
vendor/bin/prism test -j 4
# Time: 12.8s (3.5x faster)

# Parallel with 8 workers
vendor/bin/prism test -j 8
# Time: 7.1s (6.4x faster)
```

## Performance Profiling

Identify slowest tests with performance profiling:

```bash
vendor/bin/prism test --profile
```

### Profile Output

```
Test Performance Profile

Slowest Tests:
  1. complex-nested-validation (2.145s)
  2. large-array-validation (1.832s)
  3. recursive-schema-test (1.456s)
  4. deep-object-validation (0.987s)
  5. pattern-matching-test (0.654s)

Total Tests: 1,247
Average Duration: 0.023s
Median Duration: 0.012s
```

### Use Cases

- **Identify bottlenecks**: Find slow tests to optimize
- **Monitor performance**: Track test suite speed over time
- **CI optimization**: Prioritize fast tests in pipelines

## Baseline Benchmarks

Save and compare performance baselines:

### Save Baseline

```bash
vendor/bin/prism test --baseline
```

Save with custom name:

```bash
vendor/bin/prism test --baseline "pre-optimization"
```

### Compare Against Baseline

```bash
vendor/bin/prism test --compare
```

Compare against named baseline:

```bash
vendor/bin/prism test --compare "pre-optimization"
```

### Benchmark Output

```
Benchmark Comparison: default

Performance Changes:
  Total Duration: 45.2s → 38.7s (14.4% faster)
  Average Test: 0.036s → 0.031s (13.9% faster)

Significant Changes:
  ✓ complex-validation: 2.145s → 1.234s (42.5% faster)
  ✓ array-processing: 1.832s → 1.123s (38.7% faster)
  ✗ string-validation: 0.234s → 0.387s (65.4% slower)

Tests Analyzed: 1,247
Improved: 892 (71.5%)
Degraded: 123 (9.9%)
Unchanged: 232 (18.6%)
```

### Workflow Example

```bash
# Save baseline before optimization
vendor/bin/prism test --baseline "before-cache"

# Make optimizations to validator
vim src/Validator.php

# Compare performance
vendor/bin/prism test --compare "before-cache"
```

## Incremental Testing

Run only changed tests for fast iteration:

```bash
vendor/bin/prism test --incremental
```

### How It Works

1. Tracks file modification times
2. Identifies changed test files
3. Only runs tests from changed files
4. Significantly faster during development

### Example Workflow

```bash
# First run - all tests execute
vendor/bin/prism test --incremental
# Time: 45.2s

# Make small change to one test file
vim tests/validation/strings.json

# Only changed file tests run
vendor/bin/prism test --incremental
# Time: 0.8s (56x faster)
```

### Cache Management

Cache stored in `.prism/incremental-cache.json`:

```json
{
  "tests/validation/strings.json": 1703001234,
  "tests/validation/numbers.json": 1703001156,
  "tests/validation/objects.json": 1703000987
}
```

Clear cache to force full run:

```bash
rm .prism/incremental-cache.json
```

## Combining Performance Features

Maximize performance by combining features:

### Development Workflow

Fast iteration with incremental mode and watch:

```bash
vendor/bin/prism test --incremental --watch
```

### CI Optimization

Parallel execution with profiling:

```bash
vendor/bin/prism test -j 8 --profile
```

### Performance Monitoring

Baseline comparison with profiling:

```bash
vendor/bin/prism test --compare "main" --profile
```

## Watch Mode

Automatically re-run tests when files change:

```bash
vendor/bin/prism test --watch
```

### How It Works

1. Runs initial test suite
2. Monitors test files for changes
3. Automatically re-runs tests on change
4. Continues until interrupted (Ctrl+C)

### Watch Output

```
[12:34:56] Initial run completed (45.2s)
[12:35:12] Watching for changes...
[12:36:08] Change detected in tests/validation/strings.json
[12:36:08] Re-running tests...
[12:36:09] Tests completed (0.8s)
[12:36:09] Watching for changes...
```

### Best Practices

Combine watch mode with other features:

```bash
# Watch with incremental testing
vendor/bin/prism test --watch --incremental

# Watch with specific filter
vendor/bin/prism test --watch --filter "string"

# Watch with failures only
vendor/bin/prism test --watch --failures
```

## Performance Best Practices

### 1. Use Parallel Execution in CI

```yaml
# .github/workflows/tests.yml
- name: Run Tests
  run: vendor/bin/prism test -j ${{ steps.cpu.outputs.count }}
```

### 2. Profile Regularly

Monitor test suite performance:

```bash
# Weekly baseline
vendor/bin/prism test --baseline "$(date +%Y-%m-%d)"

# Compare against last week
vendor/bin/prism test --compare "$(date -d '7 days ago' +%Y-%m-%d)"
```

### 3. Optimize Slow Tests

Use profiling to identify and optimize:

```bash
vendor/bin/prism test --profile | grep -A 10 "Slowest Tests"
```

### 4. Incremental During Development

Always use incremental mode when developing:

```bash
alias prism-dev="vendor/bin/prism test --incremental --watch"
```

### 5. Filter Before Profiling

Profile specific test subsets:

```bash
vendor/bin/prism test --filter "validation" --profile
```

## Performance Metrics

### Execution Time

Total time from start to finish:

```bash
vendor/bin/prism test
# Total execution time: 45.234s
```

### Test Duration

Individual test execution times:

```bash
vendor/bin/prism test --profile
# Shows per-test durations
```

### Throughput

Tests executed per second:

```bash
# 1,247 tests in 45.2s = 27.6 tests/second
```

## Next Steps

- Learn about [advanced features](/prism/advanced-features/)
- Explore [output formats](/prism/output-formats/)
- See [custom assertions](/prism/custom-assertions/)
