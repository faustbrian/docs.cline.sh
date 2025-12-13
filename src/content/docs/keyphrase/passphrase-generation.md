---
title: Passphrase Generation
description: The PassphraseGenerator creates memorable yet secure passphrases using the EFF diceware wordlists.
---

The `PassphraseGenerator` creates memorable yet secure passphrases using the EFF diceware wordlists.

## Basic Usage

```php
use Cline\Keyphrase\Generators\PassphraseGenerator;

// Using static factory method
$passphrase = PassphraseGenerator::create()->generate();

// Using constructor
$generator = new PassphraseGenerator();
$passphrase = $generator->generate();
```

## Word Count

```php
// Default is 6 words
$passphrase = PassphraseGenerator::create()->generate();

// Custom word count
$passphrase = PassphraseGenerator::create()
    ->words(8)
    ->generate();
```

Recommended word counts:
- **4 words**: ~51 bits entropy (basic security)
- **6 words**: ~77 bits entropy (standard security)
- **8 words**: ~103 bits entropy (high security)
- **10 words**: ~129 bits entropy (maximum security)

## Wordlists

Three EFF diceware wordlists are available:

### Large Wordlist (Default)

7,776 words, optimized for security:

```php
$passphrase = PassphraseGenerator::create()
    ->large()
    ->generate();

// Example: "correct-horse-battery-staple-cloud-mint"
```

### Short Wordlist

1,296 shorter, more memorable words:

```php
$passphrase = PassphraseGenerator::create()
    ->short()
    ->generate();

// Example: "oak-tip-red-sun-bay-ice"
```

### Unique Prefix Wordlist

1,296 words with unique 3-character prefixes (easier autocomplete):

```php
$passphrase = PassphraseGenerator::create()
    ->uniquePrefix()
    ->generate();

// Example: "abs-bea-cli-dry-elm-fig"
```

### Using Enum Directly

```php
use Cline\Keyphrase\Enums\EFFWordList;

$passphrase = PassphraseGenerator::create()
    ->useWordList(EFFWordList::Short)
    ->generate();
```

## Separators

### Default Separator

Hyphen is the default separator:

```php
$passphrase = PassphraseGenerator::create()->generate();
// Example: "word-word-word-word-word-word"
```

### Custom Separator

```php
$passphrase = PassphraseGenerator::create()
    ->separator('_')
    ->generate();
// Example: "word_word_word_word_word_word"

$passphrase = PassphraseGenerator::create()
    ->separator('.')
    ->generate();
// Example: "word.word.word.word.word.word"
```

### Space Separator

```php
$passphrase = PassphraseGenerator::create()
    ->withSpaces()
    ->generate();
// Example: "word word word word word word"
```

### No Separator

```php
$passphrase = PassphraseGenerator::create()
    ->noSeparator()
    ->generate();
// Example: "wordwordwordwordwordword"
```

## Case Transformations

### Title Case

```php
$passphrase = PassphraseGenerator::create()
    ->titleCase()
    ->generate();
// Example: "Correct-Horse-Battery-Staple-Cloud-Mint"
```

### Uppercase

```php
$passphrase = PassphraseGenerator::create()
    ->uppercase()
    ->generate();
// Example: "CORRECT-HORSE-BATTERY-STAPLE-CLOUD-MINT"
```

### Lowercase (Default)

```php
$passphrase = PassphraseGenerator::create()
    ->lowercase()
    ->generate();
// Example: "correct-horse-battery-staple-cloud-mint"
```

## Including Numbers

Add a random number to the passphrase for additional entropy:

```php
$passphrase = PassphraseGenerator::create()
    ->words(4)
    ->includeNumber()
    ->generate();
// Example: "correct-42-horse-battery-staple"
```

The number is inserted at a random position among the words.

## Generating Multiple Passphrases

```php
$passphrases = PassphraseGenerator::create()
    ->words(6)
    ->titleCase()
    ->generateMany(5);

// Returns array of 5 unique passphrases
```

## Entropy Calculation

```php
$generator = PassphraseGenerator::create()
    ->words(6)
    ->large();

$entropy = $generator->entropy();
// Returns: ~77.55 bits

// Including a number adds entropy
$entropy = PassphraseGenerator::create()
    ->words(6)
    ->includeNumber()
    ->entropy();
// Returns: ~81 bits
```

## Full Example

```php
use Cline\Keyphrase\Generators\PassphraseGenerator;

// Create a passphrase generator with common settings
$generator = PassphraseGenerator::create()
    ->words(6)
    ->large()
    ->titleCase()
    ->separator('-');

// Generate passphrases
$passphrase1 = $generator->generate();
$passphrase2 = $generator->generate();

// Check entropy
$entropy = $generator->entropy();
echo "Passphrase entropy: {$entropy} bits";

// Generate with number for extra security
$withNumber = $generator->includeNumber()->generate();
```

## Why Passphrases?

Passphrases offer several advantages over traditional passwords:

1. **Memorability**: Words are easier to remember than random characters
2. **Typing Speed**: Familiar words are faster to type
3. **Security**: Long passphrases have excellent entropy
4. **Resistance**: Harder to shoulder-surf than short passwords

### Comparison

| Type | Example | Entropy | Memorability |
|------|---------|---------|--------------|
| 8-char password | `K9#mPx$2` | ~52 bits | Hard |
| 6-word passphrase | `correct-horse-battery-staple-cloud-mint` | ~77 bits | Easy |
| 12-char password | `K9#mPx$2nQwR` | ~79 bits | Very Hard |
