---
title: Getting Started
description: Install and use Shamir's Secret Sharing for splitting and reconstructing secrets in PHP applications.
---

A pure PHP implementation of Shamir's Secret Sharing scheme, allowing secrets to be split into N shares where any M shares can reconstruct the original secret.

## Overview

Shamir's Secret Sharing is a cryptographic algorithm that divides a secret into multiple parts (shares), where a minimum threshold of shares is required to reconstruct the original secret. Fewer than the threshold shares reveal zero information about the secret.

**Key Features:**
- Zero external dependencies (pure PHP + GMP extension)
- Split secrets into N shares, requiring K to reconstruct
- Information-theoretic security (shares below threshold reveal nothing)
- Automatic chunking for large secrets
- Built-in checksum validation
- Multiple encoding options (Base64, Hex)

## Requirements

Shamir v1.0 requires PHP 8.4+ and the GMP extension.

## Installation

Install Shamir with composer:

```bash
composer require cline/shamir
```

## Quick Start

### Basic Usage (Static API)

Split a secret into 5 shares where any 3 can reconstruct it:

```php
use Cline\Shamir\Shamir;

$secret = 'my-super-secret-encryption-key';
$shares = Shamir::split($secret, threshold: 3, shares: 5);

// Distribute shares to 5 different key holders
foreach ($shares as $share) {
    echo $share->toString() . "\n";
}
```

### Fluent API

Use the fluent conductor API for more expressive configuration:

```php
use Cline\Shamir\Shamir;

$secret = 'my-super-secret-encryption-key';

// Chain configuration methods
$shares = Shamir::for($secret)
    ->threshold(3)
    ->shares(5)
    ->split();

// Distribute shares
foreach ($shares as $share) {
    echo $share->toString() . "\n";
}
```

### Reconstructing Secrets

Collect any 3 shares and reconstruct the original secret:

```php
use Cline\Shamir\Shamir;
use Cline\Shamir\Share;

// Load shares from storage
$share1 = Share::fromString('1:3:checksum:encoded_value');
$share2 = Share::fromString('2:3:checksum:encoded_value');
$share3 = Share::fromString('3:3:checksum:encoded_value');

// Static API
$secret = Shamir::combine([$share1, $share2, $share3]);

// Or fluent API
$secret = Shamir::from([$share1, $share2, $share3])->combine();
```

### Binary Secrets

Works seamlessly with binary data like encryption keys:

```php
// Generate a 256-bit encryption key
$encryptionKey = random_bytes(32);

// Split into shares
$shares = Shamir::split($encryptionKey, threshold: 3, shares: 5);

// Reconstruct
$reconstructed = Shamir::combine($shares->take(3));

assert($reconstructed === $encryptionKey);
```

## Share Operations

### Share Serialization

Shares can be serialized for storage or transmission:

```php
// String format (compact, portable)
$shareString = $share->toString();  // "1:3:checksum:value"
$share = Share::fromString($shareString);

// JSON format
$json = json_encode($share);
$share = Share::fromArray(json_decode($json, true));
```

### Share Collections

The `ShareCollection` provides utilities for working with multiple shares:

```php
$shares = Shamir::split($secret, 3, 5);

// Take first N shares
$subset = $shares->take(3);

// Get random shares
$random = $shares->random(3);

// Access by index
$share = $shares->get(2);

// Iterate
foreach ($shares as $share) {
    // Process share
}
```

## Configuration

### Using ShamirManager

For advanced usage or dependency injection, use the manager directly:

```php
use Cline\Shamir\Config;
use Cline\Shamir\ShamirManager;

$config = new Config(
    prime: Config::PRIME_256,    // Field size (128, 256, 512)
    encoding: 'base64',          // Encoding (base64, hex)
);

$manager = new ShamirManager($config);

// Direct API
$shares = $manager->split($secret, 3, 5);

// Fluent API
$shares = $manager->for($secret)
    ->threshold(3)
    ->shares(5)
    ->split();
```

### Creating Manager with Different Config

```php
use Cline\Shamir\Config;
use Cline\Shamir\Shamir;

$hexConfig = new Config(
    prime: Config::PRIME_512,
    encoding: 'hex',
);

$manager = Shamir::withConfig($hexConfig);
$shares = $manager->split($secret, 3, 5);
```

## Common Use Cases

### 1. Master Key Protection

Split a master encryption key across multiple custodians:

```php
$masterKey = random_bytes(32);

// Static API
$shares = Shamir::split($masterKey, threshold: 3, shares: 5);

// Fluent API (more expressive)
$shares = Shamir::for($masterKey)
    ->threshold(3)
    ->shares(5)
    ->split();

// Distribute to 5 different security officers
// Require 3 to unlock critical systems
```

### 2. Backup Encryption

Require multiple parties to decrypt backups:

```php
$backupKey = generateBackupKey();

$shares = Shamir::for($backupKey)
    ->threshold(2)
    ->shares(3)
    ->split();

// Store shares in different locations
// Require 2 locations to restore backups
```

### 3. Disaster Recovery

Distribute recovery keys to trusted parties:

```php
$recoveryKey = generateRecoveryKey();

$shares = Shamir::for($recoveryKey)
    ->threshold(3)
    ->shares(7)
    ->split();

// Distribute to board members
// Require majority (3 of 7) to recover
```

## Next Steps

- [API Reference](api-reference.md) - Complete API documentation
- [Security Considerations](security.md) - Security best practices
- [Advanced Usage](advanced-usage.md) - Advanced patterns and techniques
- [Use Cases](use-cases.md) - Detailed use case examples
