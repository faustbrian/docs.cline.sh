---
title: Getting Started
description: Welcome to Keyphrase, a modern PHP library for generating secure passwords, passphrases, and BIP39 mnemonics with an immutable, fluent API.
---

Welcome to Keyphrase, a modern PHP library for generating secure passwords, passphrases, and BIP39 mnemonics with an immutable, fluent API.

## Installation

Install Keyphrase via Composer:

```bash
composer require cline/keyphrase
```

## Quick Start

### Generate a Password

```php
use Cline\Keyphrase\Generators\PasswordGenerator;

$password = PasswordGenerator::create()
    ->length(20)
    ->withSymbols()
    ->generate();

// Example: "K9#mPx$2nQwR@5vLjH8s"
```

### Generate a Passphrase

```php
use Cline\Keyphrase\Generators\PassphraseGenerator;

$passphrase = PassphraseGenerator::create()
    ->words(6)
    ->titleCase()
    ->generate();

// Example: "Correct-Horse-Battery-Staple-Cloud-Mint"
```

### Generate a BIP39 Mnemonic

```php
use Cline\Keyphrase\Generators\MnemonicGenerator;

$mnemonic = MnemonicGenerator::create()
    ->words(12)
    ->english()
    ->generate();

// Example: "abandon ability able about above absent absorb abstract absurd abuse access accident"
```

## Using the Manager

For convenient access to all generators, use the `KeyphraseManager`:

```php
use Cline\Keyphrase\KeyphraseManager;

$manager = new KeyphraseManager();

// Quick generation methods
$password = $manager->quickPassword(16);
$securePassword = $manager->quickSecurePassword(24);
$passphrase = $manager->quickPassphrase(6);
$mnemonic = $manager->quickMnemonic(12);
$pin = $manager->pin(4);

// Access generators
$password = $manager->password()
    ->length(32)
    ->withSymbols()
    ->excludeAmbiguous()
    ->generate();
```

## Laravel Integration

If you're using Laravel, the package auto-registers a service provider and facade:

```php
use Cline\Keyphrase\Facades\Keyphrase;

// Using the facade
$password = Keyphrase::password()->length(20)->generate();
$passphrase = Keyphrase::passphrase()->words(5)->generate();
$mnemonic = Keyphrase::mnemonic()->words(24)->generate();

// Quick methods
$pin = Keyphrase::pin(6);
$password = Keyphrase::quickSecurePassword(32);
```

## Immutability

All generators are immutable. Each method returns a new instance with the updated configuration:

```php
$base = PasswordGenerator::create()->length(16);
$withSymbols = $base->withSymbols();
$withoutSymbols = $base->withSymbols(false);

// $base, $withSymbols, and $withoutSymbols are all different instances
// Original configuration is never modified
```

This makes generators safe to share and reuse:

```php
$secureDefaults = PasswordGenerator::create()
    ->length(24)
    ->withSymbols()
    ->excludeAmbiguous();

// Create variations without modifying the original
$password1 = $secureDefaults->generate();
$password2 = $secureDefaults->length(32)->generate();
$password3 = $secureDefaults->withSymbols(false)->generate();
```

## Entropy Calculation

All generators can calculate the entropy (randomness) of generated values:

```php
$entropy = PasswordGenerator::create()
    ->length(16)
    ->alphanumeric()
    ->entropy();

// Returns: 95.27 (bits of entropy)
```

See the [Entropy Guide](entropy.md) for more details on security strength.

## Next Steps

- **[Password Generation](password-generation.md)** - Detailed password options
- **[Passphrase Generation](passphrase-generation.md)** - EFF diceware passphrases
- **[Mnemonic Generation](mnemonic-generation.md)** - BIP39 compliance
- **[Entropy Guide](entropy.md)** - Understanding password strength
