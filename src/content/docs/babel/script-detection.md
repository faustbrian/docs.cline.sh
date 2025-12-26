---
title: Script Detection
description: Detect Unicode scripts and character sets in strings.
---

## Contains Methods

Check if a string contains characters from specific scripts:

### Asian Scripts

```php
use Cline\Babel\Babel;

// Any Asian script (Han, Hiragana, Katakana, Hangul)
Babel::from('Hello 世界')->containsAsian();      // true
Babel::from('Hello World')->containsAsian();     // false

// Chinese (Han script)
Babel::from('北京欢迎你')->containsChinese();     // true
Babel::from('こんにちは')->containsChinese();     // false (Japanese only)

// Japanese (Hiragana, Katakana, or Han)
Babel::from('こんにちは')->containsJapanese();    // true (Hiragana)
Babel::from('カタカナ')->containsJapanese();      // true (Katakana)
Babel::from('日本')->containsJapanese();          // true (Han/Kanji)

// Korean (Hangul)
Babel::from('안녕하세요')->containsKorean();       // true
Babel::from('한글')->containsKorean();            // true
```

### European Scripts

```php
// Cyrillic
Babel::from('Привет мир')->containsCyrillic();    // true
Babel::from('Москва')->containsCyrillic();        // true

// Greek
Babel::from('Αθήνα')->containsGreek();            // true
Babel::from('Ελλάδα')->containsGreek();           // true

// Latin
Babel::from('Hello')->containsLatin();            // true
Babel::from('Café')->containsLatin();             // true
Babel::from('北京')->containsLatin();              // false
```

### Middle Eastern Scripts

```php
// Arabic
Babel::from('مرحبا بالعالم')->containsArabic();    // true
Babel::from('السلام')->containsArabic();          // true

// Hebrew
Babel::from('שלום עולם')->containsHebrew();       // true
Babel::from('ישראל')->containsHebrew();           // true
```

### South Asian Scripts

```php
// Devanagari (Hindi, Sanskrit, Marathi)
Babel::from('नमस्ते')->containsDevanagari();       // true
Babel::from('Hello नमस्ते')->containsDevanagari(); // true

// Bengali
Babel::from('বাংলা')->containsBengali();           // true

// Tamil
Babel::from('தமிழ்')->containsTamil();             // true
```

### Southeast Asian Scripts

```php
// Thai
Babel::from('สวัสดี')->containsThai();             // true
Babel::from('ประเทศไทย')->containsThai();          // true

// Vietnamese (Latin with diacritics)
Babel::from('Việt Nam')->containsVietnamese();    // true
Babel::from('Xin chào')->containsVietnamese();    // true
```

### Caucasian Scripts

```php
// Armenian
Babel::from('Հայdelays')->containsArmenian();     // true

// Georgian
Babel::from('საქართველო')->containsGeorgian();     // true
```

### Other Scripts

```php
// Emoji
Babel::from('Hello 👋 World')->containsEmoji();   // true
Babel::from('Hello World')->containsEmoji();      // false
```

### Generic Script Detection

Check for any Unicode script by name:

```php
Babel::from('Здравствуйте')->containsScript('Cyrillic');  // true
Babel::from('Γειά σου')->containsScript('Greek');         // true
Babel::from('नमस्ते')->containsScript('Devanagari');      // true
```

## Exclusive Methods

Check if a string contains **only** characters from specific sets:

### Latin Only

```php
Babel::from('Hello World')->isLatin();     // true
Babel::from('Hello 123')->isLatin();       // true (includes common chars)
Babel::from('Héllo')->isLatin();           // true (includes accented)
Babel::from('Hello 世界')->isLatin();       // false (mixed)
```

### Numeric Only

```php
Babel::from('12345')->isNumeric();         // true
Babel::from('123.45')->isNumeric();        // false (period)
Babel::from('123abc')->isNumeric();        // false (letters)
```

### Alphanumeric Only

```php
Babel::from('Hello123')->isAlphanumeric();      // true
Babel::from('Hello World')->isAlphanumeric();   // false (space)
Babel::from('Hello_123')->isAlphanumeric();     // false (underscore)
```

### Single Script Only

Check if string contains only one specific script:

```php
Babel::from('Привет')->isScript('Cyrillic');           // true
Babel::from('Hello Привет')->isScript('Cyrillic');     // false (mixed)
Babel::from('12345')->isScript('Common');              // true (numbers are Common)
```

## Encoding Detection

Detect and validate string encodings:

```php
// Detect encoding
Babel::from('Hello')->detect();              // "ASCII"
Babel::from('Héllo')->detect();              // "UTF-8"
Babel::from(null)->detect();                 // null

// Check specific encodings
Babel::from('Hello')->isUtf8();              // true
Babel::from('Hello')->isAscii();             // true
Babel::from('Héllo')->isAscii();             // false

// Validate encoding
Babel::from($text)->isValidEncoding('UTF-8');         // true/false
Babel::from($text)->isValidEncoding('ISO-8859-1');    // true/false
```

## Use Cases

### Content Moderation

```php
function requiresTranslation(string $content): bool
{
    $babel = Babel::from($content);

    return $babel->containsChinese()
        || $babel->containsJapanese()
        || $babel->containsKorean()
        || $babel->containsCyrillic()
        || $babel->containsArabic();
}
```

### Form Validation

```php
function validateUsername(string $username): bool
{
    $babel = Babel::from($username);

    // Only allow Latin alphanumeric
    return $babel->isAlphanumeric() && $babel->isLatin();
}
```

### Localization Detection

```php
function detectLanguageHint(string $text): ?string
{
    $babel = Babel::from($text);

    return match (true) {
        $babel->containsJapanese() => 'ja',
        $babel->containsKorean() => 'ko',
        $babel->containsChinese() => 'zh',
        $babel->containsArabic() => 'ar',
        $babel->containsHebrew() => 'he',
        $babel->containsCyrillic() => 'ru',
        $babel->containsGreek() => 'el',
        $babel->containsThai() => 'th',
        $babel->containsVietnamese() => 'vi',
        $babel->containsDevanagari() => 'hi',
        $babel->containsBengali() => 'bn',
        $babel->containsTamil() => 'ta',
        $babel->containsArmenian() => 'hy',
        $babel->containsGeorgian() => 'ka',
        default => null,
    };
}
```
