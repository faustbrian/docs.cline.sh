---
title: Entropy Guide
description: Understanding entropy helps you make informed decisions about password and passphrase security.
---

Understanding entropy helps you make informed decisions about password and passphrase security.

## What is Entropy?

Entropy measures the randomness or unpredictability of a password. It's expressed in bits - higher bits mean more possible combinations and harder to crack.

```php
use Cline\Keyphrase\Support\Entropy;

// Calculate entropy for a password
$entropy = Entropy::forPassword(16, 62);
// 16 characters from 62-character set (a-z, A-Z, 0-9)
// Returns: ~95.27 bits
```

## Entropy Calculation

### Password Entropy

```
Entropy = Length × log₂(CharacterSetSize)
```

```php
use Cline\Keyphrase\Generators\PasswordGenerator;

// Alphanumeric (62 characters: a-z, A-Z, 0-9)
$entropy = PasswordGenerator::create()
    ->alphanumeric()
    ->length(16)
    ->entropy();
// Returns: 95.27 bits

// With symbols (~95 characters)
$entropy = PasswordGenerator::create()
    ->withSymbols()
    ->length(16)
    ->entropy();
// Returns: ~105 bits
```

### Passphrase Entropy

```
Entropy = WordCount × log₂(WordlistSize)
```

```php
use Cline\Keyphrase\Generators\PassphraseGenerator;

// Large wordlist (7776 words)
$entropy = PassphraseGenerator::create()
    ->large()
    ->words(6)
    ->entropy();
// Returns: 77.55 bits

// With number included
$entropy = PassphraseGenerator::create()
    ->large()
    ->words(6)
    ->includeNumber()
    ->entropy();
// Returns: ~81 bits
```

### Mnemonic Entropy

BIP39 mnemonics have fixed entropy based on word count:

```php
use Cline\Keyphrase\Generators\MnemonicGenerator;

$generator = MnemonicGenerator::create()->words(12);

// Total entropy (including checksum)
$totalEntropy = $generator->entropy();
// Returns: 132.0 bits

// Raw entropy (without checksum)
$rawBits = $generator->entropyBits();
// Returns: 128
```

## Strength Labels

```php
use Cline\Keyphrase\Support\Entropy;

$label = Entropy::strengthLabel(50);
// Returns: "reasonable"
```

| Entropy (bits) | Label | Crack Time (1B guesses/sec) |
|---------------|-------|----------------------------|
| < 28 | Very Weak | Instant |
| 28-35 | Weak | Minutes |
| 36-59 | Reasonable | Days to Years |
| 60-127 | Strong | Centuries |
| >= 128 | Very Strong | Heat death of universe |

## Time to Crack

Estimate cracking time at various attack speeds:

```php
use Cline\Keyphrase\Support\Entropy;

$result = Entropy::timeToCrack(80);
// Returns: ['seconds' => 1.9e+16, 'human' => 'centuries']
```

The calculation assumes:
- 1 billion (10^9) guesses per second
- Average case (50% of keyspace)

### Human-Readable Output

```php
$result = Entropy::timeToCrack(20);
// ['seconds' => 524.3, 'human' => 'minutes']

$result = Entropy::timeToCrack(40);
// ['seconds' => 5.5e+5, 'human' => 'days']

$result = Entropy::timeToCrack(60);
// ['seconds' => 5.8e+10, 'human' => 'centuries']

$result = Entropy::timeToCrack(100);
// ['seconds' => 6.3e+22, 'human' => 'centuries']
```

## Using the Manager

The `KeyphraseManager` provides convenient access to entropy utilities:

```php
use Cline\Keyphrase\KeyphraseManager;

$manager = new KeyphraseManager();

// Get strength label
$label = $manager->strengthLabel(80);
// Returns: "strong"

// Get time to crack
$result = $manager->timeToCrack(80);
// Returns: ['seconds' => ..., 'human' => 'centuries']
```

## Security Recommendations

### Minimum Entropy by Use Case

| Use Case | Minimum Entropy | Recommendation |
|----------|----------------|----------------|
| Low-value accounts | 36 bits | 8-char alphanumeric |
| Standard accounts | 60 bits | 12-char with symbols |
| Financial/Email | 80 bits | 16-char with symbols or 6-word passphrase |
| Cryptocurrency | 128 bits | 24-word mnemonic |
| Master passwords | 100+ bits | 8+ word passphrase or 20+ char password |

### Entropy Comparison

```php
use Cline\Keyphrase\Generators\PasswordGenerator;
use Cline\Keyphrase\Generators\PassphraseGenerator;
use Cline\Keyphrase\Generators\MnemonicGenerator;

// ~95 bits
PasswordGenerator::create()->length(16)->alphanumeric()->entropy();

// ~77 bits
PassphraseGenerator::create()->words(6)->large()->entropy();

// 132 bits (128 + checksum)
MnemonicGenerator::create()->words(12)->entropy();

// ~105 bits
PasswordGenerator::create()->length(16)->withSymbols()->entropy();

// ~103 bits
PassphraseGenerator::create()->words(8)->large()->entropy();

// 264 bits (256 + checksum)
MnemonicGenerator::create()->words(24)->entropy();
```

## Attack Scenarios

### Online Attacks

- Speed: 100-1000 guesses/second (rate limited)
- 40 bits entropy = centuries to crack
- Most services have lockouts after failed attempts

### Offline Attacks

- Speed: Billions of guesses/second (GPU clusters)
- 60+ bits entropy recommended
- Hashed passwords can be attacked in parallel

### Nation-State Attacks

- Speed: Trillions of guesses/second (specialized hardware)
- 80+ bits entropy recommended
- 128+ bits for long-term secrets

## Full Example

```php
use Cline\Keyphrase\Generators\PasswordGenerator;
use Cline\Keyphrase\Generators\PassphraseGenerator;
use Cline\Keyphrase\Support\Entropy;

// Compare options for a master password

$password = PasswordGenerator::create()
    ->length(20)
    ->withSymbols();

$passphrase = PassphraseGenerator::create()
    ->words(6)
    ->large();

echo "Password entropy: " . $password->entropy() . " bits\n";
echo "Passphrase entropy: " . $passphrase->entropy() . " bits\n";

echo "Password strength: " . Entropy::strengthLabel($password->entropy()) . "\n";
echo "Passphrase strength: " . Entropy::strengthLabel($passphrase->entropy()) . "\n";

$pwdCrack = Entropy::timeToCrack($password->entropy());
$ppCrack = Entropy::timeToCrack($passphrase->entropy());

echo "Password crack time: " . $pwdCrack['human'] . "\n";
echo "Passphrase crack time: " . $ppCrack['human'] . "\n";
```
