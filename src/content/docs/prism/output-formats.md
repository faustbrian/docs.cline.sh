---
title: Output Formats
description: Configure test result output formats including text, JSON, and JUnit XML.
---

Prism supports multiple output formats for test results, suitable for different use cases.

## Text Format (Default)

Beautiful, color-coded terminal output with Termwind:

```bash
vendor/bin/prism test
```

### Enhanced Terminal Output

```
┌─────────────────────────────────────────────┐
│ Test Suite: my-validator                   │
├─────────────────────────────────────────────┤
│ Total Tests: 1,247                          │
│ Passed: 1,198 ✓                             │
│ Failed: 49 ✗                                │
│ Pass Rate: 96.1%                            │
│ Duration: 45.234s                           │
└─────────────────────────────────────────────┘

✓ String validation tests (234/234)
✓ Number validation tests (187/187)
✗ Object validation tests (156/162)
✓ Array validation tests (143/143)
```

### Show Detailed Failures

Display failures with syntax-highlighted JSON diff:

```bash
vendor/bin/prism test --failures
```

```
1. additional properties should not be valid
   Test ID: draft7:properties:3:2
   File: properties.json
   Group: properties validation
   Expected Validation: INVALID
   Actual Validation: VALID

   Error: Validation should have failed

   Duration: 23.50ms

   Test Data:
      {
        "foo": 1,
        "bar": "extra"
      }
────────────────────────────────────────────────────────────────────────────────
```

**Enhanced Failure Display Features:**
- **Syntax highlighting**: JSON values color-coded by type (strings=yellow, numbers=cyan, booleans/nulls=magenta)
- **Test metadata**: ID, file, group, expected vs actual validation state
- **Error messages**: Clear description of validation failures
- **Performance metrics**: Duration in milliseconds
- **Structured layout**: Easy to scan and debug

## CI-Friendly Output

Plain text output without Termwind formatting:

```bash
vendor/bin/prism test --ci
```

### CI Output Format

```
Test Suite: my-validator
Total: 1247
Passed: 1198
Failed: 49
Pass Rate: 96.07%
Duration: 45.234s

PASS draft7:strings:0:0
PASS draft7:strings:0:1
FAIL draft7:properties:3:2
PASS draft7:numbers:1:0
```

### Use Cases

- **Continuous Integration**: GitHub Actions, GitLab CI, Jenkins
- **Logging**: Structured logs for parsing
- **Automation**: Scripts processing test results

## JSON Format

Machine-readable JSON output:

```bash
vendor/bin/prism test --format json
```

### JSON Structure

```json
{
  "suites": [
    {
      "name": "my-validator",
      "total_tests": 1247,
      "passed_tests": 1198,
      "failed_tests": 49,
      "pass_rate": 96.07,
      "duration": 45.234,
      "results": [
        {
          "id": "draft7:strings:0:0",
          "file": "strings.json",
          "group": "string validation",
          "description": "valid string",
          "data": "hello",
          "expected_valid": true,
          "actual_valid": true,
          "passed": true,
          "error": null,
          "duration": 0.012,
          "tags": ["string", "basic"]
        },
        {
          "id": "draft7:properties:3:2",
          "file": "properties.json",
          "group": "properties validation",
          "description": "additional properties invalid",
          "data": {"foo": 1, "bar": "extra"},
          "expected_valid": false,
          "actual_valid": true,
          "passed": false,
          "error": null,
          "duration": 0.023,
          "tags": ["object", "properties"]
        }
      ]
    }
  ]
}
```

### Use Cases

- **API Integration**: Send results to monitoring systems
- **Data Analysis**: Process results programmatically
- **Storage**: Archive test results in databases

### Processing JSON Output

```bash
# Save to file
vendor/bin/prism test --format json > results.json

# Parse with jq
vendor/bin/prism test --format json | jq '.suites[0].pass_rate'

# Count failures
vendor/bin/prism test --format json | jq '[.suites[].results[] | select(.passed == false)] | length'
```

## JUnit XML Format

Standards-compliant JUnit XML output for CI/CD integration:

```bash
vendor/bin/prism test --format xml
```

### XML Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testsuites>
  <testsuite name="my-validator" tests="1247" failures="49" time="45.234">
    <testcase
      name="draft7:strings:0:0"
      classname="string validation"
      time="0.012">
    </testcase>
    <testcase
      name="draft7:properties:3:2"
      classname="properties validation"
      time="0.023">
      <failure type="ValidationFailure">
        Expected: invalid
        Actual: valid
        Data: {"foo":1,"bar":"extra"}
      </failure>
    </testcase>
  </testsuite>
</testsuites>
```

### Use Cases

- **JUnit Integration**: Compatible with JUnit XML format
- **CI Systems**: Jenkins, Bamboo, TeamCity
- **IDE Integration**: PhpStorm, VS Code test runners

## Combining Output with Features

### JSON with Profiling

```bash
vendor/bin/prism test --format json --profile > results.json
```

### XML for CI with Failures

```bash
vendor/bin/prism test --format xml --ci --failures > junit.xml
```

## Redirecting Output

### Save to File

```bash
vendor/bin/prism test > test-results.txt
vendor/bin/prism test --format json > results.json
vendor/bin/prism test --format xml > junit.xml
```

### Pipe to Other Tools

```bash
# Filter with grep
vendor/bin/prism test --ci | grep FAIL

# Parse with jq
vendor/bin/prism test --format json | jq '.suites[].pass_rate'

# Parse XML with xmllint
vendor/bin/prism test --format xml | xmllint --format -
```

## Output Customization

### Show Only Failures

Combine format with failure filter:

```bash
vendor/bin/prism test --failures --ci
```

### Detailed JSON Output

Include failure details in JSON:

```bash
vendor/bin/prism test --format json --failures
```

### Profile with Text Output

Combine profiling with default format:

```bash
vendor/bin/prism test --profile
```

## Integration Examples

### GitHub Actions

```yaml
- name: Run Tests
  run: vendor/bin/prism test --format json --ci > results.json

- name: Upload Results
  uses: actions/upload-artifact@v3
  with:
    name: test-results
    path: results.json
```

### GitLab CI

```yaml
test:
  script:
    - vendor/bin/prism test --format junit > junit.xml
  artifacts:
    reports:
      junit: junit.xml
```

### Jenkins

```groovy
stage('Test') {
    steps {
        sh 'vendor/bin/prism test --format xml > junit.xml'
    }
    post {
        always {
            junit 'junit.xml'
        }
    }
}
```

### Custom Monitoring

```bash
#!/bin/bash
# Send results to monitoring service

vendor/bin/prism test --format json | \
  jq '{
    pass_rate: .suites[0].pass_rate,
    duration: .suites[0].duration,
    failures: .suites[0].failed_tests
  }' | \
  curl -X POST https://monitoring.example.com/api/metrics \
    -H "Content-Type: application/json" \
    -d @-
```

## Next Steps

- Learn about [custom assertions](/prism/custom-assertions/)
- Explore [filtering options](/prism/filtering/)
- See [performance features](/prism/performance/)
