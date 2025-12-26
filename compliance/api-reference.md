# API Reference

Complete API documentation for the Compliance testing framework.

## ComplianceTestInterface

The main interface for implementing compliance test adapters.

**Namespace:** `Cline\Compliance\Contracts\ComplianceTestInterface`

### Methods

#### `getName(): string`

Returns the human-readable name of the test suite.

**Returns:** `string` - The test suite name

**Example:**
```php
public function getName(): string
{
    return 'JSON Schema Draft 04';
}
```

---

#### `getValidatorClass(): string`

Returns the fully-qualified class name of the validator.

**Returns:** `class-string` - The validator class name

**Example:**
```php
public function getValidatorClass(): string
{
    return Draft04Validator::class;
}
```

---

#### `getTestDirectory(): string`

Returns the absolute path to the directory containing test files.

**Returns:** `string` - The test directory path

**Example:**
```php
public function getTestDirectory(): string
{
    return __DIR__.'/tests/compliance/draft4';
}
```

---

#### `validate(mixed $data, mixed $schema): ValidationResult`

Validates data against a schema.

**Parameters:**
- `$data` (mixed) - The data to validate
- `$schema` (mixed) - The schema to validate against

**Returns:** `ValidationResult` - The validation result

**Example:**
```php
public function validate(mixed $data, mixed $schema): ValidationResult
{
    $validator = new MyValidator();

    return $validator->validate($data, $schema);
}
```

---

#### `getTestFilePatterns(): array`

Returns glob patterns for finding test files.

**Returns:** `array<int, string>` - Array of glob patterns

**Example:**
```php
public function getTestFilePatterns(): array
{
    return ['*.json', '**/*.json'];
}
```

---

#### `decodeJson(string $json): mixed`

Decodes JSON strings while preserving type distinctions.

**Parameters:**
- `$json` (string) - The JSON string to decode

**Returns:** `mixed` - The decoded data

**Example:**
```php
public function decodeJson(string $json): mixed
{
    return json_decode($json, true, 512, JSON_THROW_ON_ERROR);
}
```

## ValidationResult

Interface for validation result objects.

**Namespace:** `Cline\Compliance\Contracts\ValidationResult`

### Methods

#### `isValid(): bool`

Checks if validation passed.

**Returns:** `bool` - True if valid, false otherwise

**Example:**
```php
if ($result->isValid()) {
    echo 'Validation passed';
}
```

---

#### `getError(): ?string`

Returns the validation error message.

**Returns:** `?string` - Error message or null if valid

**Example:**
```php
if (!$result->isValid()) {
    echo $result->getError();
}
```

## ComplianceRunner

Service class for running compliance tests.

**Namespace:** `Cline\Compliance\Services\ComplianceRunner`

### Methods

#### `run(ComplianceTestInterface $compliance): TestSuite`

Executes all tests for a compliance adapter.

**Parameters:**
- `$compliance` (ComplianceTestInterface) - The compliance test adapter

**Returns:** `TestSuite` - Test results with statistics

**Example:**
```php
use Cline\Compliance\Services\ComplianceRunner;

$runner = new ComplianceRunner();
$suite = $runner->run($complianceAdapter);

echo "Total: {$suite->totalTests()}";
echo "Passed: {$suite->passedTests()}";
echo "Failed: {$suite->failedTests()}";
```

## ConfigLoader

Service class for loading compliance configuration.

**Namespace:** `Cline\Compliance\Services\ConfigLoader`

### Methods

#### `load(): array`

Loads compliance test adapters from `compliance.php`.

**Returns:** `array<int, ComplianceTestInterface>` - Array of compliance test adapters

**Throws:** `ConfigurationFileNotFoundException` - If config file not found

**Example:**
```php
use Cline\Compliance\Services\ConfigLoader;

$loader = new ConfigLoader();
$tests = $loader->load();

foreach ($tests as $test) {
    echo $test->getName();
}
```

## TestSuite

Immutable value object representing test suite results.

**Namespace:** `Cline\Compliance\ValueObjects\TestSuite`

### Properties

- `string $name` - The test suite name
- `array $results` - Array of TestResult objects
- `float $duration` - Execution time in seconds

### Constructor

```php
public function __construct(
    public string $name,
    public array $results,
    public float $duration,
)
```

### Methods

#### `totalTests(): int`

Returns the total number of tests.

**Returns:** `int` - Total test count

**Example:**
```php
$total = $suite->totalTests();
```

---

#### `passedTests(): int`

Returns the number of passed tests.

**Returns:** `int` - Passed test count

**Example:**
```php
$passed = $suite->passedTests();
```

---

#### `failedTests(): int`

Returns the number of failed tests.

**Returns:** `int` - Failed test count

**Example:**
```php
$failed = $suite->failedTests();
```

---

#### `passRate(): float`

Returns the pass rate as a percentage.

**Returns:** `float` - Pass rate (0-100)

**Example:**
```php
$rate = $suite->passRate(); // 95.5
echo sprintf('%.1f%% passed', $rate);
```

---

#### `failures(): array`

Returns only the failed test results.

**Returns:** `array<int, TestResult>` - Array of failed TestResult objects

**Example:**
```php
foreach ($suite->failures() as $failure) {
    echo $failure->description;
}
```

## TestResult

Immutable value object representing a single test result.

**Namespace:** `Cline\Compliance\ValueObjects\TestResult`

### Properties

- `string $id` - Unique test identifier
- `string $file` - Test file path (relative to test directory)
- `string $group` - Test group description
- `string $description` - Test description
- `mixed $data` - The data that was validated
- `bool $expectedValid` - Expected validation result
- `bool $actualValid` - Actual validation result
- `bool $passed` - Whether the test passed
- `?string $error` - Error message (null if no error)

### Static Factory Methods

#### `TestResult::pass(...): TestResult`

Creates a passing test result.

**Parameters:**
- `string $id` - Test identifier
- `string $file` - Test file path
- `string $group` - Group description
- `string $description` - Test description
- `mixed $data` - Validated data
- `bool $expectedValid` - Expected result

**Returns:** `TestResult`

**Example:**
```php
$result = TestResult::pass(
    id: 'draft4:basic:0:1',
    file: 'basic.json',
    group: 'string validation',
    description: 'valid string',
    data: 'hello',
    expectedValid: true,
);
```

---

#### `TestResult::fail(...): TestResult`

Creates a failing test result.

**Parameters:**
- `string $id` - Test identifier
- `string $file` - Test file path
- `string $group` - Group description
- `string $description` - Test description
- `mixed $data` - Validated data
- `bool $expectedValid` - Expected result
- `bool $actualValid` - Actual result
- `?string $error` - Optional error message

**Returns:** `TestResult`

**Example:**
```php
$result = TestResult::fail(
    id: 'draft4:basic:0:1',
    file: 'basic.json',
    group: 'string validation',
    description: 'invalid number',
    data: 123,
    expectedValid: true,
    actualValid: false,
    error: 'Expected string, got integer',
);
```

## Output Renderers

Classes for rendering test results to the console.

### SummaryRenderer

Renders beautiful summary output using Termwind.

**Namespace:** `Cline\Compliance\Output\SummaryRenderer`

#### `render(array $suites): void`

Displays a summary of all test suites.

**Parameters:**
- `$suites` (array<TestSuite>) - Array of test suites

**Example:**
```php
use Cline\Compliance\Output\SummaryRenderer;

$renderer = new SummaryRenderer();
$renderer->render($suites);
```

---

### DetailRenderer

Renders detailed failure information using Termwind.

**Namespace:** `Cline\Compliance\Output\DetailRenderer`

#### `render(TestSuite $suite): void`

Displays detailed failure information for a test suite.

**Parameters:**
- `$suite` (TestSuite) - The test suite with failures

**Example:**
```php
use Cline\Compliance\Output\DetailRenderer;

$renderer = new DetailRenderer();
$renderer->render($suite);
```

---

### CiRenderer

Renders CI-friendly output without Termwind formatting.

**Namespace:** `Cline\Compliance\Output\CiRenderer`

#### `render(array $suites): void`

Displays summary output for CI environments.

**Parameters:**
- `$suites` (array<TestSuite>) - Array of test suites

**Example:**
```php
use Cline\Compliance\Output\CiRenderer;
use Symfony\Component\Console\Output\ConsoleOutput;

$output = new ConsoleOutput();
$renderer = new CiRenderer($output);
$renderer->render($suites);
```

---

#### `renderFailures(TestSuite $suite): void`

Displays failure details for CI environments.

**Parameters:**
- `$suite` (TestSuite) - The test suite with failures

**Example:**
```php
$renderer->renderFailures($suite);
```

## TestCommand

Symfony Console command for running compliance tests.

**Namespace:** `Cline\Compliance\Commands\TestCommand`

### Options

#### `--ci`

Use CI-friendly output without Termwind formatting.

**Example:**
```bash
vendor/bin/compliance test --ci
```

---

#### `--failures`

Show detailed failure information.

**Example:**
```bash
vendor/bin/compliance test --failures
```

---

#### `--draft=<name>`

Run tests for a specific draft/version only.

**Example:**
```bash
vendor/bin/compliance test --draft="Draft 04"
```

## Exceptions

### ComplianceException

Base exception for all compliance-related errors.

**Namespace:** `Cline\Compliance\Exceptions\ComplianceException`

**Extends:** `Exception`

---

### ConfigurationFileNotFoundException

Thrown when `compliance.php` cannot be found.

**Namespace:** `Cline\Compliance\Exceptions\ConfigurationFileNotFoundException`

**Extends:** `ComplianceException`

**Example:**
```php
try {
    $loader = new ConfigLoader();
    $tests = $loader->load();
} catch (ConfigurationFileNotFoundException $e) {
    echo "Config file not found: {$e->getMessage()}";
}
```

## Application

Main application class for the CLI tool.

**Namespace:** `Cline\Compliance\Application`

**Extends:** `Symfony\Component\Console\Application`

### Constructor

```php
public function __construct()
```

Automatically registers the `TestCommand` as the default command.

**Example:**
```php
use Cline\Compliance\Application;

$app = new Application();
$app->run();
```

## Next Steps

- [Getting Started](getting-started.md) - Installation and quick start
- [Configuration](configuration.md) - Configure test suites
- [Writing Tests](writing-tests.md) - Create compliance test suites
