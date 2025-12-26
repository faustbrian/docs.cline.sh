# Writing Tests

Learn how to create comprehensive compliance test suites.

## Test File Format

Compliance tests are defined in JSON files with a specific structure:

```json
[
  {
    "description": "Test suite description",
    "schema": {
      "type": "object",
      "properties": {
        "name": { "type": "string" }
      }
    },
    "tests": [
      {
        "description": "valid data",
        "data": { "name": "John" },
        "valid": true
      },
      {
        "description": "invalid data",
        "data": { "name": 123 },
        "valid": false
      }
    ]
  }
]
```

## Test Structure

Each test file contains an array of test suites. Each suite has:

- `description` (string) - Human-readable description of what's being tested
- `schema` (object) - The schema to validate against
- `tests` (array) - Individual test cases

Each test case has:

- `description` (string) - What this specific test validates
- `data` (mixed) - The data to validate
- `valid` (boolean) - Expected validation result (true = should pass, false = should fail)

## Basic Examples

### Testing Primitive Types

```json
[
  {
    "description": "string type validation",
    "schema": { "type": "string" },
    "tests": [
      {
        "description": "a valid string",
        "data": "hello",
        "valid": true
      },
      {
        "description": "an integer is invalid",
        "data": 42,
        "valid": false
      },
      {
        "description": "a boolean is invalid",
        "data": true,
        "valid": false
      }
    ]
  }
]
```

### Testing Objects

```json
[
  {
    "description": "object properties validation",
    "schema": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "age": { "type": "integer" }
      },
      "required": ["name"]
    },
    "tests": [
      {
        "description": "valid object with all properties",
        "data": { "name": "John", "age": 30 },
        "valid": true
      },
      {
        "description": "valid object with only required property",
        "data": { "name": "Jane" },
        "valid": true
      },
      {
        "description": "invalid - missing required property",
        "data": { "age": 25 },
        "valid": false
      },
      {
        "description": "invalid - wrong type for property",
        "data": { "name": 123 },
        "valid": false
      }
    ]
  }
]
```

### Testing Arrays

```json
[
  {
    "description": "array validation",
    "schema": {
      "type": "array",
      "items": { "type": "string" },
      "minItems": 1,
      "maxItems": 3
    },
    "tests": [
      {
        "description": "valid array with strings",
        "data": ["a", "b", "c"],
        "valid": true
      },
      {
        "description": "invalid - contains non-string",
        "data": ["a", 1, "c"],
        "valid": false
      },
      {
        "description": "invalid - too few items",
        "data": [],
        "valid": false
      },
      {
        "description": "invalid - too many items",
        "data": ["a", "b", "c", "d"],
        "valid": false
      }
    ]
  }
]
```

## Advanced Patterns

### Multiple Schemas in One File

```json
[
  {
    "description": "integer validation",
    "schema": { "type": "integer" },
    "tests": [
      { "description": "valid integer", "data": 42, "valid": true },
      { "description": "invalid string", "data": "42", "valid": false }
    ]
  },
  {
    "description": "number with constraints",
    "schema": {
      "type": "number",
      "minimum": 0,
      "maximum": 100
    },
    "tests": [
      { "description": "valid number in range", "data": 50, "valid": true },
      { "description": "invalid - below minimum", "data": -1, "valid": false },
      { "description": "invalid - above maximum", "data": 101, "valid": false }
    ]
  }
]
```

### Testing Edge Cases

```json
[
  {
    "description": "edge cases for string length",
    "schema": {
      "type": "string",
      "minLength": 1,
      "maxLength": 10
    },
    "tests": [
      { "description": "empty string is invalid", "data": "", "valid": false },
      { "description": "exactly min length", "data": "a", "valid": true },
      { "description": "exactly max length", "data": "0123456789", "valid": true },
      { "description": "one char over max", "data": "01234567890", "valid": false }
    ]
  }
]
```

### Complex Nested Structures

```json
[
  {
    "description": "nested object validation",
    "schema": {
      "type": "object",
      "properties": {
        "user": {
          "type": "object",
          "properties": {
            "name": { "type": "string" },
            "address": {
              "type": "object",
              "properties": {
                "street": { "type": "string" },
                "city": { "type": "string" }
              },
              "required": ["city"]
            }
          },
          "required": ["name"]
        }
      }
    },
    "tests": [
      {
        "description": "valid nested structure",
        "data": {
          "user": {
            "name": "John",
            "address": {
              "street": "123 Main St",
              "city": "Boston"
            }
          }
        },
        "valid": true
      },
      {
        "description": "missing nested required field",
        "data": {
          "user": {
            "name": "John",
            "address": {
              "street": "123 Main St"
            }
          }
        },
        "valid": false
      }
    ]
  }
]
```

## Organizing Test Files

### Directory Structure

Organize tests by feature or category:

```
tests/compliance/
├── basic/
│   ├── types.json
│   ├── objects.json
│   └── arrays.json
├── validation/
│   ├── string-validation.json
│   ├── number-validation.json
│   └── format-validation.json
└── advanced/
    ├── nested-objects.json
    ├── references.json
    └── composition.json
```

### File Naming Conventions

Use descriptive names that indicate what's being tested:

- `types.json` - Basic type validation
- `required-properties.json` - Required field validation
- `string-length.json` - String length constraints
- `number-range.json` - Numeric range validation
- `array-items.json` - Array item validation

## Test Coverage

### What to Test

**Core Functionality:**
- Valid data passes validation
- Invalid data fails validation
- Edge cases (empty values, boundaries, null)
- Type mismatches

**Constraints:**
- Minimum/maximum values
- String length limits
- Array size constraints
- Pattern matching

**Complex Scenarios:**
- Nested structures
- Optional vs required fields
- Multiple validation rules
- Conditional validation

### Example Comprehensive Test

```json
[
  {
    "description": "comprehensive user validation",
    "schema": {
      "type": "object",
      "properties": {
        "id": { "type": "integer", "minimum": 1 },
        "email": { "type": "string", "format": "email" },
        "age": { "type": "integer", "minimum": 0, "maximum": 150 },
        "tags": {
          "type": "array",
          "items": { "type": "string" },
          "minItems": 0,
          "maxItems": 5
        }
      },
      "required": ["id", "email"]
    },
    "tests": [
      {
        "description": "valid complete user",
        "data": {
          "id": 1,
          "email": "user@example.com",
          "age": 30,
          "tags": ["developer", "author"]
        },
        "valid": true
      },
      {
        "description": "valid minimal user",
        "data": { "id": 1, "email": "user@example.com" },
        "valid": true
      },
      {
        "description": "invalid - missing required id",
        "data": { "email": "user@example.com" },
        "valid": false
      },
      {
        "description": "invalid - id is zero",
        "data": { "id": 0, "email": "user@example.com" },
        "valid": false
      },
      {
        "description": "invalid - email format",
        "data": { "id": 1, "email": "not-an-email" },
        "valid": false
      },
      {
        "description": "invalid - age too high",
        "data": { "id": 1, "email": "user@example.com", "age": 200 },
        "valid": false
      },
      {
        "description": "invalid - too many tags",
        "data": {
          "id": 1,
          "email": "user@example.com",
          "tags": ["a", "b", "c", "d", "e", "f"]
        },
        "valid": false
      }
    ]
  }
]
```

## Best Practices

### Descriptive Test Names

Use clear, specific descriptions:

**Good:**
- `"valid email address"`
- `"invalid - age below minimum"`
- `"missing required field 'name'"`

**Avoid:**
- `"test 1"`
- `"invalid"`
- `"fails"`

### One Concept Per Test

Each test should validate a single concept:

**Good:**
```json
{ "description": "string too short", "data": "", "valid": false },
{ "description": "string too long", "data": "...", "valid": false }
```

**Avoid:**
```json
{
  "description": "various string failures",
  "data": "...",
  "valid": false
}
```

### Test Both Positive and Negative Cases

Always include both valid and invalid examples:

```json
[
  {
    "description": "minimum value validation",
    "schema": { "type": "integer", "minimum": 10 },
    "tests": [
      { "description": "exactly at minimum", "data": 10, "valid": true },
      { "description": "above minimum", "data": 11, "valid": true },
      { "description": "below minimum", "data": 9, "valid": false }
    ]
  }
]
```

## Running Tests

Run all tests:

```bash
vendor/bin/compliance test
```

Run with failure details:

```bash
vendor/bin/compliance test --failures
```

Run specific draft only:

```bash
vendor/bin/compliance test --draft="Draft 04"
```

CI mode (no Termwind formatting):

```bash
vendor/bin/compliance test --ci
```

## Next Steps

- [Configuration](configuration.md) - Configure test suites
- [API Reference](api-reference.md) - Complete API documentation
