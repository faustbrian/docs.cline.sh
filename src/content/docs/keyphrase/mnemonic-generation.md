---
title: Mnemonic Generation
description: The MnemonicGenerator creates BIP39-compliant mnemonic phrases for cryptocurrency wallets and secure key derivation.
---

The `MnemonicGenerator` creates BIP39-compliant mnemonic phrases for cryptocurrency wallets and secure key derivation.

## Basic Usage

```php
use Cline\Keyphrase\Generators\MnemonicGenerator;

// Using static factory method
$mnemonic = MnemonicGenerator::create()->generate();

// Using constructor
$generator = new MnemonicGenerator();
$mnemonic = $generator->generate();
```

## Word Count

BIP39 specifies valid word counts that correspond to specific entropy levels:

| Words | Entropy Bits | Checksum Bits | Total Bits |
|-------|-------------|---------------|------------|
| 12 | 128 | 4 | 132 |
| 15 | 160 | 5 | 165 |
| 18 | 192 | 6 | 198 |
| 21 | 224 | 7 | 231 |
| 24 | 256 | 8 | 264 |

```php
// Default is 12 words (128 bits entropy)
$mnemonic = MnemonicGenerator::create()->generate();

// 24 words for maximum security (256 bits entropy)
$mnemonic = MnemonicGenerator::create()
    ->words(24)
    ->generate();
```

Invalid word counts will throw an `InvalidWordCountException`:

```php
// Throws InvalidWordCountException
$mnemonic = MnemonicGenerator::create()->words(10)->generate();
```

## Languages

BIP39 defines wordlists for 9 languages:

### English (Default)

```php
$mnemonic = MnemonicGenerator::create()
    ->english()
    ->generate();
```

### Spanish

```php
$mnemonic = MnemonicGenerator::create()
    ->spanish()
    ->generate();
```

### French

```php
$mnemonic = MnemonicGenerator::create()
    ->french()
    ->generate();
```

### Italian

```php
$mnemonic = MnemonicGenerator::create()
    ->italian()
    ->generate();
```

### Japanese

Japanese uses ideographic space (U+3000) as the separator:

```php
$mnemonic = MnemonicGenerator::create()
    ->japanese()
    ->generate();

// Words separated by ideographic space: "\u{3000}"
```

### Korean

```php
$mnemonic = MnemonicGenerator::create()
    ->korean()
    ->generate();
```

### Czech

```php
$mnemonic = MnemonicGenerator::create()
    ->czech()
    ->generate();
```

### Chinese (Simplified)

```php
$mnemonic = MnemonicGenerator::create()
    ->chineseSimplified()
    ->generate();
```

### Chinese (Traditional)

```php
$mnemonic = MnemonicGenerator::create()
    ->chineseTraditional()
    ->generate();
```

### Using Enum Directly

```php
use Cline\Keyphrase\Enums\BIP39Language;

$mnemonic = MnemonicGenerator::create()
    ->useLanguage(BIP39Language::Spanish)
    ->generate();
```

## Custom Separator

Override the default separator (space for most languages):

```php
$mnemonic = MnemonicGenerator::create()
    ->separator('-')
    ->generate();

// Example: "abandon-ability-able-about-above-absent-absorb-abstract-absurd-abuse-access-accident"
```

## Entropy Methods

### Total Entropy (Including Checksum)

```php
$generator = MnemonicGenerator::create()->words(12);

$entropy = $generator->entropy();
// Returns: 132.0 (128 bits + 4 checksum bits)
```

### Raw Entropy Bits

```php
$generator = MnemonicGenerator::create()->words(12);

$bits = $generator->entropyBits();
// Returns: 128
```

## Generating Multiple Mnemonics

```php
$mnemonics = MnemonicGenerator::create()
    ->words(12)
    ->english()
    ->generateMany(5);

// Returns array of 5 unique mnemonics
```

## Accessing Generator State

```php
$generator = MnemonicGenerator::create()
    ->words(24)
    ->spanish();

$wordCount = $generator->getWordCount();
// Returns: 24

$language = $generator->getLanguage();
// Returns: BIP39Language::Spanish
```

## Full Example

```php
use Cline\Keyphrase\Generators\MnemonicGenerator;
use Cline\Keyphrase\Enums\BIP39Language;

// Create a mnemonic generator for a cryptocurrency wallet
$generator = MnemonicGenerator::create()
    ->words(24)
    ->english();

// Generate the recovery phrase
$mnemonic = $generator->generate();

// Check entropy
$entropy = $generator->entropy();
echo "Mnemonic entropy: {$entropy} bits";

// Verify word count
$words = explode(' ', $mnemonic);
echo "Word count: " . count($words);

// Generate for different language
$spanishMnemonic = $generator
    ->useLanguage(BIP39Language::Spanish)
    ->generate();
```

## BIP39 Compliance

All generated mnemonics are fully BIP39 compliant:

1. **Wordlist**: Uses official BIP39 wordlists (2048 words each)
2. **Entropy**: Cryptographically secure random entropy
3. **Checksum**: Proper SHA-256 checksum appended
4. **Validation**: Word counts match BIP39 specification

### Wordlist Properties

Each BIP39 wordlist has these properties:
- Exactly 2048 words
- Words are sorted for binary search
- First 4 characters uniquely identify each word
- No word is a prefix of another word

## Security Recommendations

| Use Case | Words | Entropy |
|----------|-------|---------|
| Standard wallet | 12 | 128 bits |
| High-value wallet | 24 | 256 bits |
| Cold storage | 24 | 256 bits |
| Hardware wallet | 24 | 256 bits |

For maximum security:
- Always use 24 words for significant cryptocurrency holdings
- Store the mnemonic securely (metal backup, safety deposit box)
- Never store digitally or share online
- Test recovery before depositing funds
