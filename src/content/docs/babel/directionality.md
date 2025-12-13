---
title: Directionality
description: Detect text direction (RTL/LTR) for proper rendering and layout.
---

## Overview

Some languages read right-to-left (RTL), such as Arabic, Hebrew, and Persian. Babel provides methods to detect text direction for proper UI rendering and layout decisions.

## Check RTL Dominance

Determine if text is predominantly right-to-left:

```php
use Cline\Babel\Babel;

// Arabic text
Babel::from('مرحبا بالعالم')->isRtl();      // true

// Hebrew text
Babel::from('שלום עולם')->isRtl();          // true

// English text
Babel::from('Hello World')->isRtl();        // false

// Mixed (depends on dominant direction)
Babel::from('Hello مرحبا World')->isRtl();  // false (more LTR chars)
```

## Check for RTL Presence

Check if any RTL characters exist (regardless of dominance):

```php
// Pure RTL
Babel::from('مرحبا')->containsRtl();         // true

// Mixed content
Babel::from('Hello مرحبا')->containsRtl();   // true

// Pure LTR
Babel::from('Hello World')->containsRtl();   // false

// Numbers only (neutral)
Babel::from('12345')->containsRtl();         // false
```

## Get Text Direction

Get the dominant direction as a string value:

```php
// Left-to-right
Babel::from('Hello World')->direction();     // "ltr"

// Right-to-left
Babel::from('مرحبا بالعالم')->direction();    // "rtl"

// Mixed directions
Babel::from('Hello مرحبا World مرحبا')->direction();  // "mixed"

// Neutral (numbers, punctuation only)
Babel::from('12345')->direction();           // "neutral"
Babel::from('!!!')->direction();             // "neutral"

// Empty
Babel::from('')->direction();                // "neutral"
Babel::from(null)->direction();              // "neutral"
```

## Return Values

The `direction()` method returns one of four values:

| Value | Description |
|-------|-------------|
| `ltr` | Predominantly left-to-right text |
| `rtl` | Predominantly right-to-left text |
| `mixed` | Significant characters in both directions |
| `neutral` | Only neutral characters (numbers, punctuation, whitespace) |

## Use Cases

### HTML Direction Attribute

```php
function getHtmlDir(string $content): string
{
    return Babel::from($content)->direction() === 'rtl' ? 'rtl' : 'ltr';
}

// Usage
$dir = getHtmlDir($userComment);
echo "<p dir=\"{$dir}\">{$userComment}</p>";
```

### Dynamic Layout

```php
function getTextAlignment(string $text): string
{
    $direction = Babel::from($text)->direction();

    return match ($direction) {
        'rtl' => 'right',
        'ltr' => 'left',
        default => 'left',  // Default for mixed/neutral
    };
}
```

### Bidirectional Text Warning

```php
function hasBidiContent(string $text): bool
{
    return Babel::from($text)->direction() === 'mixed';
}

// Alert users about potential rendering issues
if (hasBidiContent($message)) {
    $warning = 'This message contains mixed-direction text.';
}
```

### Form Input Validation

```php
function validateArabicInput(string $input): bool
{
    $babel = Babel::from($input);

    // Must be Arabic and RTL dominant
    return $babel->containsArabic() && $babel->isRtl();
}
```

## RTL Scripts

The following Unicode scripts are considered right-to-left:

- Arabic
- Hebrew
- Syriac
- Thaana (Maldivian)
- N'Ko
- Samaritan
- Mandaic

## Notes

- Numbers and punctuation are considered **neutral** and don't affect direction
- Common characters (spaces, basic punctuation) are also neutral
- The `mixed` direction is returned when both RTL and LTR characters are present in significant amounts
- Empty strings and null values return `neutral`
