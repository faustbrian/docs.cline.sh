---
title: Advanced Features
description: Explore advanced Prism features including snapshots, fuzzing, validator comparison, and coverage analysis.
---

Prism provides advanced features for comprehensive test validation and analysis.

## Snapshot Testing

Capture and compare test results against saved snapshots.

### Update Snapshots

Save current test results as the expected baseline:

```bash
vendor/bin/prism test --update-snapshots
```

### Check Against Snapshots

Compare current results against saved snapshots:

```bash
vendor/bin/prism test --check-snapshots
```

### Snapshot Output

```
Snapshot Comparison: my-validator

Changes Detected:
  ✓ New passing tests: 12
  ✗ New failing tests: 3
  ~ Changed tests: 7

Detailed Changes:
  + test-123: null → PASS
  - test-456: PASS → FAIL
  ~ test-789: PASS → PASS (duration: 0.12s → 0.34s)

Recommendation: Review changes before updating snapshots
```

### Use Cases

- **Regression detection**: Identify unexpected behavior changes
- **Refactoring safety**: Ensure validator behavior unchanged
- **CI validation**: Verify test results match expectations

### Workflow Example

```bash
# Establish baseline
vendor/bin/prism test --update-snapshots

# Make changes to validator
vim src/Validator.php

# Verify no regressions
vendor/bin/prism test --check-snapshots
```

## Fuzzing and Test Generation

Generate random test cases to discover edge cases:

```bash
vendor/bin/prism test --fuzz 1000
```

Generates and runs 1000 random test cases per validator.

### What Gets Generated

1. **Edge Cases**
   - null, true, false
   - Integer boundaries (PHP_INT_MAX, PHP_INT_MIN, 0, -1, 1)
   - Float edge cases (0.0, -0.0)
   - String edge cases (empty, whitespace, single char)
   - Large strings (1000, 10000 characters)
   - Array edge cases (empty, nested, mixed types)
   - Object structures

2. **Random Data**
   - Random integers (-1000 to 1000)
   - Random floats
   - Random strings (0-100 characters)
   - Random arrays (0-10 elements)
   - Random objects (0-5 properties)

### Fuzzing Output

```
Fuzzing Test Results

Total Fuzzed Tests: 1,000
Passed: 967
Failed: 33

Found 33 failure(s) during fuzzing:

1. fuzz-234
   Description: Fuzzed test with array data
   Error: Maximum nesting depth exceeded
   Duration: 0.0234s

2. fuzz-567
   Description: Fuzzed test with string data
   Error: Invalid UTF-8 sequence
   Duration: 0.0123s
```

### Use Cases

- **Edge case discovery**: Find unexpected failures
- **Robustness testing**: Verify validator handles random input
- **Security testing**: Identify potential vulnerabilities

### Example Workflow

```bash
# Run fuzzing to discover issues
vendor/bin/prism test --fuzz 5000

# Fix discovered issues
vim src/Validator.php

# Re-run fuzzing to verify fixes
vendor/bin/prism test --fuzz 5000
```

## Validator Comparison

Compare results across multiple validator implementations:

```bash
vendor/bin/prism test --compare-validators
```

### Configuration

Define multiple validators in `prism.php`:

```php
<?php

return [
    new OpisJsonSchemaValidator(),
    new JustValidateValidator(),
    new SwaggestValidator(),
];
```

### Comparison Output

```
Validator Comparison Report

Validators: opis-json-schema, justvalidate, swaggest
Total Tests: 1,247
Discrepancies: 23

Found 23 test(s) with differing results:

1. draft7/strings/pattern-01
   Description: Pattern validation with anchors
   Agreement: 66.7%
   Results:
     opis-json-schema: PASS (expected: valid, got: valid)
     justvalidate: FAIL (expected: valid, got: invalid)
     swaggest: PASS (expected: valid, got: valid)

2. draft7/numbers/multipleOf-02
   Description: Multiple of with floating point
   Agreement: 33.3%
   Results:
     opis-json-schema: FAIL (expected: valid, got: invalid)
     justvalidate: FAIL (expected: valid, got: invalid)
     swaggest: PASS (expected: valid, got: valid)
```

### Use Cases

- **Spec compliance**: Verify implementations agree
- **Edge case identification**: Find interpretation differences
- **Migration validation**: Compare old vs new validators

## Coverage Analysis

Analyze test coverage across groups, files, and tags:

```bash
vendor/bin/prism test --coverage
```

### Coverage Output

```
Test Coverage Analysis

Total Tests: 1,247
Passed: 1,198
Failed: 49
Pass Rate: 96.1%
Coverage Score: 87.3/100

Groups Coverage: 45 unique groups
  properties: 234 tests
  items: 187 tests
  required: 156 tests
  type: 143 tests
  anyOf: 98 tests
  ... and 40 more groups

Files Coverage: 78 unique files
  properties.json: 234 tests
  items.json: 187 tests
  required.json: 156 tests
  type.json: 143 tests
  anyOf.json: 98 tests
  ... and 73 more files

Tags Coverage: 12 unique tags
  required: 687 tests
  optional: 312 tests
  edge-case: 156 tests
  regression: 89 tests
  performance: 3 tests
  ... and 7 more tags
```

### Coverage Score Calculation

Weighted formula:
- 60% pass rate
- 20% group diversity (unique test groups)
- 20% file diversity (unique test files)

### Use Cases

- **Completeness check**: Verify comprehensive coverage
- **Gap identification**: Find untested areas
- **Quality metrics**: Track test suite health

## Interactive Mode

Menu-driven test configuration and execution:

```bash
vendor/bin/prism test --interactive
```

### Interactive Menu

```
Interactive Mode - Configure and run tests

Current Configuration:
  Filter: <none>
  Tag: <none>
  Parallel: 1 worker(s)
  Incremental: disabled
  Watch: disabled

Select action:
  > Run tests with current configuration
    Set name filter (regex)
    Set tag filter
    Configure parallel workers
    Toggle incremental mode
    Toggle watch mode
    Clear all filters
    Exit
```

### Features

- **Real-time configuration**: Adjust settings before running
- **Filter preview**: See current filter configuration
- **Immediate execution**: Run with configured options
- **Persistent session**: Keep testing until exit

### Use Cases

- **Exploratory testing**: Try different filter combinations
- **Learning**: Understand filter behavior interactively
- **Quick iteration**: Adjust and re-run easily

## Multiple Output Formats

Generate test results in various formats:

### JSON Output

```bash
vendor/bin/prism test --format json
```

### JUnit XML Output

```bash
vendor/bin/prism test --format xml
```

See [Output Formats](/prism/output-formats/) for detailed format specifications.

## Combining Advanced Features

### Comprehensive CI Pipeline

```bash
vendor/bin/prism test \
    -j 8 \
    --profile \
    --coverage \
    --check-snapshots \
    --format json > results.json
```

### Development Workflow

```bash
vendor/bin/prism test \
    --watch \
    --incremental \
    --interactive \
    --failures
```

### Quality Assurance

```bash
vendor/bin/prism test \
    --compare-validators \
    --fuzz 1000 \
    --coverage \
    --profile
```

## Next Steps

- See [output formats](/prism/output-formats/) for result formats
- Learn about [custom assertions](/prism/custom-assertions/)
- Explore [filtering options](/prism/filtering/)
