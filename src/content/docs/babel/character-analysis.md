---
title: Character Analysis
description: Analyze string contents for special, invisible, and potentially problematic characters.
---

## Non-Printable Characters

Detect characters that don't render visibly:

```php
use Cline\Babel\Babel;

// Null byte
Babel::from("Hello\x00World")->containsNonPrintable();  // true

// Bell character
Babel::from("Alert\x07!")->containsNonPrintable();      // true

// Normal text
Babel::from('Hello World')->containsNonPrintable();     // false

// Note: tabs and newlines are considered printable
Babel::from("Hello\tWorld\n")->containsNonPrintable();  // false
```

## Control Characters

Detect ASCII control characters (C0 and C1):

```php
// Null byte
Babel::from("Hello\x00World")->containsControlChars();  // true

// Escape character
Babel::from("Hello\x1BWorld")->containsControlChars();  // true

// Bell
Babel::from("Hello\x07World")->containsControlChars();  // true

// Normal text with whitespace
Babel::from("Hello\nWorld")->containsControlChars();    // true (newline is control)
```

## Whitespace Detection

Check if string contains only whitespace:

```php
// Spaces only
Babel::from('   ')->isWhitespace();            // true

// Tabs and newlines
Babel::from("\t\n\r")->isWhitespace();         // true

// Mixed content
Babel::from(' Hello ')->isWhitespace();        // false

// Empty string
Babel::from('')->isWhitespace();               // true

// Unicode whitespace
Babel::from("\u{00A0}")->isWhitespace();       // true (non-breaking space)
```

## Invisible Characters

Detect zero-width and invisible Unicode characters often used for text manipulation:

```php
// Zero-width space (U+200B)
Babel::from("Hello\u{200B}World")->containsInvisible();  // true

// Zero-width non-joiner (U+200C)
Babel::from("Hello\u{200C}World")->containsInvisible();  // true

// Zero-width joiner (U+200D)
Babel::from("Hello\u{200D}World")->containsInvisible();  // true

// Byte order mark (U+FEFF)
Babel::from("\u{FEFF}Hello")->containsInvisible();       // true

// Word joiner (U+2060)
Babel::from("Hello\u{2060}World")->containsInvisible();  // true

// Normal text
Babel::from('Hello World')->containsInvisible();         // false
```

## Homoglyph Detection

Detect characters that look similar to common Latin characters but are from different scripts (potential security issue):

```php
// Cyrillic 'а' looks like Latin 'a'
Babel::from('pаypal')->containsHomoglyphs();    // true (Cyrillic а)

// Cyrillic 'о' looks like Latin 'o'
Babel::from('gооgle')->containsHomoglyphs();    // true (Cyrillic о)

// Pure Latin
Babel::from('paypal')->containsHomoglyphs();    // false

// Pure Cyrillic (not homoglyphs, just Cyrillic)
Babel::from('Привет')->containsHomoglyphs();    // false
```

### Common Homoglyphs

| Latin | Cyrillic | Greek |
|-------|----------|-------|
| a | а (U+0430) | α (U+03B1) |
| c | с (U+0441) | - |
| e | е (U+0435) | ε (U+03B5) |
| o | о (U+043E) | ο (U+03BF) |
| p | р (U+0440) | ρ (U+03C1) |
| x | х (U+0445) | χ (U+03C7) |

## Mixed Script Detection

Detect strings containing characters from multiple scripts (potential spoofing/phishing indicator):

```php
// Mixed Latin and Cyrillic
Babel::from('Hello Привет')->containsMixedScripts();  // true

// Mixed Latin and Chinese
Babel::from('Hello 世界')->containsMixedScripts();     // true

// Mixed Latin and Arabic
Babel::from('Hello مرحبا')->containsMixedScripts();    // true

// Single script (pure Latin)
Babel::from('Hello World')->containsMixedScripts();   // false

// Single script (pure Cyrillic)
Babel::from('Привет мир')->containsMixedScripts();    // false
```

## BOM Detection

Check if string starts with a byte-order mark:

```php
// UTF-8 BOM
Babel::from("\xEF\xBB\xBFHello")->hasBom();    // true

// UTF-16 BE BOM
Babel::from("\xFE\xFFHello")->hasBom();        // true

// UTF-16 LE BOM
Babel::from("\xFF\xFEHello")->hasBom();        // true

// No BOM
Babel::from('Hello')->hasBom();                // false
```

## String Metrics

Get basic string measurements:

```php
$babel = Babel::from('Héllo 世界');

// Character count (not bytes)
$babel->length();   // 8

// Byte count
$babel->bytes();    // 13 (UTF-8 encoded)

// Check emptiness
$babel->isEmpty();     // false
$babel->isNotEmpty();  // true
```

## Use Cases

### Security Validation

```php
function isSafeUsername(string $username): bool
{
    $babel = Babel::from($username);

    return !$babel->containsHomoglyphs()
        && !$babel->containsInvisible()
        && !$babel->containsControlChars();
}
```

### Input Sanitization Check

```php
function needsSanitization(string $input): bool
{
    $babel = Babel::from($input);

    return $babel->containsNonPrintable()
        || $babel->containsInvisible()
        || $babel->containsControlChars();
}
```

### Display Validation

```php
function isDisplayable(string $text): bool
{
    $babel = Babel::from($text);

    return !$babel->containsNonPrintable()
        && !$babel->containsControlChars();
}
```

### Homoglyph Attack Detection

```php
function detectPunycodeThreat(string $domain): bool
{
    $babel = Babel::from($domain);

    // Domain contains mixed scripts with Latin lookalikes
    return $babel->containsLatin() && $babel->containsHomoglyphs();
}

// Examples
detectPunycodeThreat('google.com');    // false
detectPunycodeThreat('gооgle.com');    // true (Cyrillic о)
detectPunycodeThreat('pаypal.com');    // true (Cyrillic а)
```
