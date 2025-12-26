---
title: API Reference
description: Complete API reference for cline/shamir - Shamir's Secret Sharing implementation.
---

Complete API documentation for the Shamir package.

## Shamir Facade

Static facade providing convenient access to ShamirManager functionality.

### `split()`

Split a secret into shares (static API).

```php
public static function split(
    string $secret,
    int $threshold,
    int $shares,
): ShareCollection
```

**Parameters:**
- `$secret` - The secret to split (any binary string)
- `$threshold` - Minimum shares needed to reconstruct (k)
- `$shares` - Total number of shares to generate (n)

**Returns:** `ShareCollection` - Collection of generated shares

**Throws:**
- `InvalidArgumentException` - If threshold > shares or threshold < 2

**Example:**
```php
$shares = Shamir::split('my-secret', threshold: 3, shares: 5);
```

### `for()`

Begin fluent split operation.

```php
public static function for(string $secret): SplitConductor
```

**Parameters:**
- `$secret` - The secret to split

**Returns:** `SplitConductor` - Fluent conductor for configuration

**Example:**
```php
$shares = Shamir::for('my-secret')
    ->threshold(3)
    ->shares(5)
    ->split();
```

### `combine()`

Reconstruct a secret from shares (static API).

```php
public static function combine(iterable $shares): string
```

**Parameters:**
- `$shares` - Iterable of Share objects or share strings

**Returns:** `string` - The reconstructed secret

**Throws:**
- `InsufficientSharesException` - If fewer shares than threshold
- `InvalidShareException` - If share format is invalid or checksum fails
- `IncompatibleSharesException` - If shares are from different splits

**Example:**
```php
$secret = Shamir::combine([$share1, $share2, $share3]);
```

### `from()`

Begin fluent combine operation.

```php
public static function from(iterable $shares): CombineConductor
```

**Parameters:**
- `$shares` - Iterable of Share objects or share strings

**Returns:** `CombineConductor` - Fluent conductor for combination

**Example:**
```php
$secret = Shamir::from([$share1, $share2, $share3])->combine();
```

### `withConfig()`

Create manager with different configuration.

```php
public static function withConfig(Config $config): ShamirManager
```

**Parameters:**
- `$config` - Configuration instance

**Returns:** `ShamirManager` - New manager with config

**Example:**
```php
$config = new Config(prime: Config::PRIME_512, encoding: 'hex');
$manager = Shamir::withConfig($config);
$shares = $manager->split('secret', 3, 5);
```

### `areCompatible()`

Check if shares are from the same split operation.

```php
public static function areCompatible(Share ...$shares): bool
```

**Parameters:**
- `...$shares` - Variable number of Share objects

**Returns:** `bool` - True if shares have the same threshold

**Example:**
```php
if (Shamir::areCompatible($share1, $share2, $share3)) {
    // Safe to combine
}
```

## ShamirManager

Central manager for orchestrating secret sharing operations.

### Constructor

```php
public function __construct(private Config $config = new Config())
```

### `split()`

Split a secret into shares.

```php
public function split(
    string $secret,
    int $threshold,
    int $shares,
): ShareCollection
```

### `combine()`

Reconstruct a secret from shares.

```php
public function combine(iterable $shares): string
```

### `for()`

Begin fluent split operation.

```php
public function for(string $secret): SplitConductor
```

### `from()`

Begin fluent combine operation.

```php
public function from(iterable $shares): CombineConductor
```

### `withConfig()`

Create new manager with different config.

```php
public function withConfig(Config $config): self
```

### `areCompatible()`

Verify shares are compatible.

```php
public function areCompatible(Share ...$shares): bool
```

### `getConfig()`

Get current configuration.

```php
public function getConfig(): Config
```

## Conductors

### SplitConductor

Fluent API for configuring split operations.

#### `threshold()`

Set minimum shares needed.

```php
public function threshold(int $threshold): self
```

#### `shares()`

Set total shares to generate.

```php
public function shares(int $shares): self
```

#### `split()`

Execute the split operation.

```php
public function split(): ShareCollection
```

**Example:**
```php
$shares = $manager->for($secret)
    ->threshold(3)
    ->shares(5)
    ->split();
```

### CombineConductor

Fluent API for combine operations.

#### `combine()`

Execute the combine operation.

```php
public function combine(): string
```

**Example:**
```php
$secret = $manager->from($shares)->combine();
```

## Share Class

Pure value object representing a single share in the secret sharing scheme.

### Constructor

```php
public function __construct(
    private int $index,
    private string $value,
    private int $threshold,
    private string $checksum,
)
```

### Methods

#### `getIndex()`

```php
public function getIndex(): int
```

Get the share's index (1-based).

#### `getValue()`

```php
public function getValue(): string
```

Get the encoded share value.

#### `getThreshold()`

```php
public function getThreshold(): int
```

Get the threshold required to reconstruct the secret.

#### `getChecksum()`

```php
public function getChecksum(): string
```

Get the share's checksum for validation.

#### `toString()`

```php
public function toString(): string
```

Serialize to portable string format: `"index:threshold:checksum:value"`

Delegates to ShareSerializer internally.

**Example:**
```php
$shareString = $share->toString();
// "1:3:abc123...:encoded_value"
```

#### `fromString()`

```php
public static function fromString(string $encoded): self
```

Parse a share from string format.

Delegates to ShareSerializer internally.

**Throws:** `InvalidShareException` - If format is invalid

**Example:**
```php
$share = Share::fromString('1:3:checksum:value');
```

#### `fromArray()`

```php
public static function fromArray(array $data): self
```

Create share from array (JSON deserialization).

Delegates to ShareSerializer internally.

**Throws:** `InvalidShareException` - If required fields are missing

**Example:**
```php
$data = json_decode($json, true);
$share = Share::fromArray($data);
```

#### `jsonSerialize()`

```php
public function jsonSerialize(): array
```

Serialize to array for JSON encoding.

Delegates to ShareSerializer internally.

**Example:**
```php
$json = json_encode($share);
```

## ShareSerializer

Service for serializing and deserializing Share objects.

### `toString()`

Serialize a share to string format.

```php
public function toString(Share $share): string
```

### `fromString()`

Deserialize a share from string format.

```php
public function fromString(string $encoded): Share
```

**Throws:** `InvalidShareException`

### `toArray()`

Serialize a share to array format.

```php
public function toArray(Share $share): array
```

### `fromArray()`

Deserialize a share from array format.

```php
public function fromArray(array $data): Share
```

**Throws:** `InvalidShareException`

**Example:**
```php
$serializer = new ShareSerializer();

// String serialization
$string = $serializer->toString($share);
$share = $serializer->fromString($string);

// Array serialization
$array = $serializer->toArray($share);
$share = $serializer->fromArray($array);
```

## ShareCollection Class

Collection of shares with utility methods.

### Constructor

```php
public function __construct(private array $shares)
```

### Methods

#### `get()`

```php
public function get(int $index): Share
```

Get share by index (1-based).

**Throws:** `InvalidShareException` - If share not found

#### `take()`

```php
public function take(int $count): self
```

Take first N shares.

#### `random()`

```php
public function random(int $count): self
```

Get N random shares.

**Throws:** `InvalidArgumentException` - If count > available shares

#### `count()`

```php
public function count(): int
```

Get total number of shares.

#### `toArray()`

```php
public function toArray(): array
```

Convert to plain array of Share objects.

#### `forDistribution()`

```php
public function forDistribution(): array
```

Get shares in randomized order, indexed by share number.

**Returns:** `array<int, Share>` - Shares indexed by their index

**Example:**
```php
foreach ($shares->forDistribution() as $index => $share) {
    sendToKeyHolder($index, $share);
}
```

## Config Class

Configuration options for Shamir operations.

### Constructor

```php
public function __construct(
    public string $prime = Config::PRIME_256,
    public string $encoding = 'base64',
)
```

### Constants

#### Prime Field Sizes

```php
// 128-bit prime (secrets up to ~16 bytes per chunk)
Config::PRIME_128

// 256-bit prime (secrets up to ~32 bytes per chunk) - DEFAULT
Config::PRIME_256

// 512-bit prime (larger secrets)
Config::PRIME_512
```

### Properties

#### `$prime`

The prime number defining the Galois field size.

#### `$encoding`

Encoding format for share values: `'base64'` (default) or `'hex'`.

### Example

```php
$config = new Config(
    prime: Config::PRIME_512,
    encoding: 'hex',
);

$shares = Shamir::split($secret, 3, 5, $config);
```

## Exceptions

### `ShamirException`

Base exception for all Shamir-related errors.

### `InsufficientSharesException`

Thrown when fewer shares than threshold are provided.

**Static Constructor:**
```php
InsufficientSharesException::notEnoughShares(int $provided, int $required)
```

### `InvalidShareException`

Thrown when share format is invalid or checksum fails.

**Static Constructors:**
```php
InvalidShareException::invalidFormat(string $encoded)
InvalidShareException::missingRequiredFields()
InvalidShareException::shareNotFound(int $index)
InvalidShareException::checksumMismatch()
```

### `IncompatibleSharesException`

Thrown when shares are from different split operations.

**Static Constructors:**
```php
IncompatibleSharesException::differentThresholds()
IncompatibleSharesException::differentChecksums()
```

### `SecretTooLargeException`

Thrown when secret exceeds field size (rarely occurs due to automatic chunking).

**Static Constructor:**
```php
SecretTooLargeException::exceedsFieldSize(int $secretSize, int $maxSize)
```

## Type Reference

### Iterables

The `combine()` method accepts any iterable of shares:

```php
// Array of Share objects
Shamir::combine([$share1, $share2, $share3]);

// ShareCollection
Shamir::combine($shares->take(3));

// Array of strings
Shamir::combine([
    '1:3:checksum:value1',
    '2:3:checksum:value2',
    '3:3:checksum:value3',
]);

// Mixed
Shamir::combine([
    $shareObject,
    '2:3:checksum:value',
    Share::fromString('3:3:checksum:value'),
]);
```
