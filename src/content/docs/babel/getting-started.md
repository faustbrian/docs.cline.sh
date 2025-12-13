---
title: Getting Started
description: Babel provides Unicode-aware string encoding, conversion, and analysis for PHP applications with a fluent API.
---

## Requirements

Babel requires PHP 8.2+ with the following extensions:
- `ext-intl` (ICU transliteration)
- `ext-mbstring` (multibyte string handling)
- `ext-iconv` (encoding conversion)

## Installation

Install Babel with composer:

```bash
composer require cline/babel
```

## Basic Usage

Create a Babel instance from any string:

```php
use Cline\Babel\Babel;

$babel = Babel::from('Héllo Wörld');
```

### Fluent Transformations

Chain methods for complex transformations:

```php
$slug = Babel::from('Héllo Wörld!')
    ->toAscii();  // "Hello World!"
```

### Null Safety

Babel handles null values gracefully:

```php
$babel = Babel::from(null);
$babel->isEmpty();     // true
$babel->toAscii();     // null
$babel->isUtf8();      // true (empty is valid UTF-8)
```

### Immutability

All transformation methods return new instances:

```php
$original = Babel::from('Café');
$ascii = $original->toAscii();

$original->value();  // "Café" (unchanged)
$ascii;              // "Cafe"
```

## Quick Examples

### Convert to ASCII

```php
Babel::from('Żółć')->toAscii();           // "Zolc"
Babel::from('北京')->toAscii();            // "bei jing"
Babel::from('Привет')->toAscii();          // "Privet"
```

### Detect Scripts

```php
Babel::from('Hello 世界')->containsChinese();   // true
Babel::from('Привет мир')->containsCyrillic();  // true
Babel::from('مرحبا')->isRtl();                  // true
```

### Clean Strings

```php
Babel::from("Hello\x00World")->removeNonPrintable()->value();  // "HelloWorld"
Babel::from('Hello 👋')->removeEmoji()->value();                // "Hello "
```

### Grapheme Operations

```php
// Split into grapheme clusters
Babel::from('Hello')->graphemes();  // ['H', 'e', 'l', 'l', 'o']
Babel::from('café')->graphemes();   // ['c', 'a', 'f', 'é']

// Reverse preserving graphemes
Babel::from('café')->reverse()->value();  // "éfac"
```

### Create Slugs

```php
Babel::from('Héllo Wörld!')->toSlug();  // "hello-world"
```

## Next Steps

- **[Conversion](/babel/conversion/)** - Encoding conversion methods
- **[Script Detection](/babel/script-detection/)** - Detect scripts and character sets
- **[Directionality](/babel/directionality/)** - RTL/LTR detection
- **[Character Analysis](/babel/character-analysis/)** - Analyze string contents
- **[Normalization](/babel/normalization/)** - Clean and normalize strings
