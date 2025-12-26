---
title: Formatting
description: Master Numerus formatting capabilities with internationalization support for currencies, file sizes, percentages, ordinals, and more.
---

Numerus provides rich formatting capabilities inspired by Laravel's Number class, supporting internationalization and various number representations.

## Abbreviation

Format large numbers with unit abbreviations (K, M, B, T):

```php
use function Cline\Numerus\numerus;

numerus(1000)->abbreviate();           // "1K"
numerus(1500)->abbreviate();           // "2K"
numerus(1000000)->abbreviate();        // "1M"
numerus(1230000)->abbreviate(2);       // "1.23M"
numerus(1000000000)->abbreviate();     // "1B"
numerus(1000000000000)->abbreviate();  // "1T"
```

## Human-Readable Format

Convert numbers to word-based representations:

```php
numerus(1000)->forHumans();        // "1 thousand"
numerus(1500)->forHumans();        // "2 thousand"
numerus(1000000)->forHumans();     // "1 million"
numerus(1230000)->forHumans(2);    // "1.23 million"
numerus(1000000000)->forHumans();  // "1 billion"
```

With locale support:

```php
numerus(1000000)->forHumans(locale: 'fr');  // "1 million" (French)
numerus(1000000)->forHumans(locale: 'de');  // "1 Million" (German)
```

## File Size Formatting

Format byte values as readable file sizes:

```php
numerus(0)->fileSize();           // "0 B"
numerus(512)->fileSize();         // "512 B"
numerus(1024)->fileSize();        // "1 KB"
numerus(1024 * 1024)->fileSize(); // "1 MB"
numerus(1073741824)->fileSize();  // "1 GB"

// With precision
numerus(1536)->fileSize(2);       // "1.50 KB"
numerus(1048576 * 1.5)->fileSize(1);  // "1.5 MB"
```

## Number Formatting

Format numbers with thousands separators and decimal precision:

```php
numerus(1000)->format();          // "1,000"
numerus(100000)->format();        // "100,000"
numerus(1234.56)->format(2);      // "1,234.56"

// With max precision
numerus(1234.5)->format(2, 2);    // "1,234.50"
```

Locale-specific formatting:

```php
numerus(1000)->format(locale: 'de');     // "1.000" (German)
numerus(1234.56)->format(2, locale: 'fr');  // "1 234,56" (French)
```

## Currency Formatting

Format values as currency:

```php
numerus(1000)->currency();                    // "$1,000.00"
numerus(1000)->currency(in: 'EUR');           // "€1,000.00"
numerus(1000)->currency(in: 'GBP');           // "£1,000.00"
numerus(1234.56)->currency(precision: 2);     // "$1,234.56"

// With locale
numerus(1000)->currency(in: 'EUR', locale: 'de');  // "1.000,00 €"
numerus(1000)->currency(in: 'EUR', locale: 'fr');  // "1 000,00 €"
```

Configure default currency:

```php
Numerus::useCurrency('EUR');
numerus(1000)->currency();  // Uses EUR by default

Numerus::defaultCurrency();  // "EUR"
```

Temporary currency:

```php
$formatted = Numerus::withCurrency('GBP', function () {
    return numerus(1000)->currency();
});
// Uses GBP, then restores previous default
```

## Percentage Formatting

Format values as percentages:

```php
numerus(10)->percentage();          // "10%"
numerus(50)->percentage();          // "50%"
numerus(10.123)->percentage(2);     // "10.12%"
numerus(99.999)->percentage(1);     // "100.0%"

// With max precision
numerus(10.12)->percentage(0, 2);   // "10%"
```

## Ordinal Formatting

Format numbers with ordinal suffixes:

```php
numerus(1)->ordinal();    // "1st"
numerus(2)->ordinal();    // "2nd"
numerus(3)->ordinal();    // "3rd"
numerus(4)->ordinal();    // "4th"
numerus(11)->ordinal();   // "11th"
numerus(21)->ordinal();   // "21st"
numerus(101)->ordinal();  // "101st"
numerus(1000)->ordinal(); // "1,000th"
```

## Spelling Numbers

Convert numbers to word representations:

```php
numerus(1)->spell();      // "one"
numerus(10)->spell();     // "ten"
numerus(21)->spell();     // "twenty-one"
numerus(99)->spell();     // "ninety-nine"
numerus(100)->spell();    // "one hundred"
numerus(102)->spell();    // "one hundred two"

// With locale
numerus(42)->spell(locale: 'fr');  // "quarante-deux"
numerus(88)->spell(locale: 'de');  // "achtundachtzig"
```

Spell ordinals:

```php
numerus(1)->spellOrdinal();   // "first"
numerus(2)->spellOrdinal();   // "second"
numerus(3)->spellOrdinal();   // "third"
numerus(11)->spellOrdinal();  // "eleventh"
numerus(21)->spellOrdinal();  // "twenty-first"
```

## Trimming

Remove trailing zeros from decimal numbers:

```php
$num = numerus(12.0);
$trimmed = $num->trim();
// $trimmed->value() === 12.0

numerus(12.30)->trim()->value();   // 12.3
numerus(12.300)->trim()->value();  // 12.3
numerus(12.34)->trim()->value();   // 12.34
```

## Parsing

Parse locale-aware strings to Numerus instances:

```php
// Parse integers
$num = Numerus::parseInt('42');       // 42
$num = Numerus::parseInt('1,234');    // 1234

// Parse floats
$num = Numerus::parseFloat('42.5');      // 42.5
$num = Numerus::parseFloat('1,234.56');  // 1234.56

// With locale
$num = Numerus::parseFloat('1.234,56', locale: 'de');  // 1234.56
$num = Numerus::parseFloat('1 234,56', locale: 'fr');  // 1234.56
```

## Number Pairs

Generate arrays of number ranges:

```php
$pairs = Numerus::pairs(25, 10);
// [[1, 10], [11, 20], [21, 25]]

$pairs = Numerus::pairs(25, 10, offset: 0);
// [[0, 9], [10, 19], [20, 25]]

$pairs = Numerus::pairs(100, 25);
// [[1, 25], [26, 50], [51, 75], [76, 100]]
```

Useful for pagination:

```php
$totalRecords = 247;
$perPage = 50;
$ranges = Numerus::pairs($totalRecords, $perPage, offset: 0);
// [[0, 49], [50, 99], [100, 149], [150, 199], [200, 247]]

foreach ($ranges as [$start, $end]) {
    // Fetch records from $start to $end
}
```

## Locale Configuration

Set default locale for all formatting operations:

```php
Numerus::useLocale('fr');
numerus(1000)->format();  // "1 000"

Numerus::defaultLocale();  // "fr"
```

Temporarily use a different locale:

```php
$formatted = Numerus::withLocale('de', function () {
    return numerus(1234.56)->format(2);
});
// "1.234,56" (German format)

Numerus::defaultLocale();  // Returns to previous locale
```

## Practical Examples

### Format Price with Tax

```php
function formatPriceWithTax(float $price, float $taxRate): string
{
    return numerus($price)
        ->multiplyBy(1 + $taxRate)
        ->currency(in: 'USD', precision: 2);
}

$total = formatPriceWithTax(99.99, 0.08);  // "$107.99"
```

### Display File Upload Size

```php
function formatUploadSize(int $bytes): string
{
    return numerus($bytes)->fileSize(2);
}

$size = formatUploadSize(2048576);  // "1.95 MB"
```

### Format Statistics

```php
function formatStat(int $value): string
{
    if ($value >= 1000000) {
        return numerus($value)->abbreviate(1);
    }

    return numerus($value)->format();
}

$views = formatStat(1234567);   // "1.2M"
$likes = formatStat(999);        // "999"
```

### Progress Percentage

```php
function formatProgress(int $completed, int $total): string
{
    return numerus($completed)
        ->divideBy($total)
        ->multiplyBy(100)
        ->percentage(1);
}

$progress = formatProgress(75, 100);  // "75.0%"
```

### Format Contest Position

```php
function formatPosition(int $rank): string
{
    return numerus($rank)->spellOrdinal();
}

$position = formatPosition(1);   // "first"
$position = formatPosition(42);  // "forty-second"
```
