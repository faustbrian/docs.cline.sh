---
title: Conversion
description: Convert strings between encodings with transliteration support.
---

## ASCII Conversion

Convert any Unicode string to ASCII with intelligent transliteration:

```php
use Cline\Babel\Babel;

// European characters
Babel::from('Żółć')->toAscii();      // "Zolc"
Babel::from('Ñoño')->toAscii();      // "Nono"
Babel::from('Ümläüt')->toAscii();    // "Umlaut"

// Asian characters (romanization)
Babel::from('北京')->toAscii();       // "bei jing"
Babel::from('東京')->toAscii();       // "dong jing"
Babel::from('서울')->toAscii();       // "seoul"

// Cyrillic
Babel::from('Москва')->toAscii();    // "Moskva"
Babel::from('Привет')->toAscii();    // "Privet"

// Greek
Babel::from('Αθήνα')->toAscii();     // "Athena"
```

## UTF-8 Conversion

Convert strings to UTF-8 from detected or specified encoding:

```php
// Auto-detect source encoding
$utf8 = Babel::from($legacyString)->toUtf8();

// Specify source encoding
$utf8 = Babel::from($latin1String)->toUtf8('ISO-8859-1');
```

## Latin-1 Conversion

Convert to ISO-8859-1 (Latin-1) with transliteration for unsupported characters:

```php
Babel::from('Café résumé')->toLatin1();  // Preserves accents
Babel::from('北京')->toLatin1();          // Transliterates to ASCII first
```

## Custom Encoding

Convert to any supported encoding:

```php
// Convert to Windows-1252
$win = Babel::from($text)->toEncoding('Windows-1252');

// Specify source encoding
$result = Babel::from($text)->toEncoding('UTF-16', 'UTF-8');
```

## HTML Entities

Convert special characters to HTML entities and back:

```php
// Encode
Babel::from('<script>"alert"</script>')
    ->toHtmlEntities();
// "&lt;script&gt;&quot;alert&quot;&lt;/script&gt;"

// Decode
Babel::from('&lt;div&gt;')
    ->fromHtmlEntities()
    ->value();
// "<div>"
```

### Custom Flags

```php
use const ENT_QUOTES;
use const ENT_HTML5;
use const ENT_XML1;

// HTML5 with quotes (default)
Babel::from($text)->toHtmlEntities(ENT_QUOTES | ENT_HTML5);

// XML compatible
Babel::from($text)->toHtmlEntities(ENT_QUOTES | ENT_XML1);
```

## URL Slugs

Create URL-safe slugs from any string:

```php
Babel::from('Hello World!')->toSlug();           // "hello-world"
Babel::from('Héllo Wörld')->toSlug();            // "hello-world"
Babel::from('  Multiple   Spaces  ')->toSlug();  // "multiple-spaces"

// Custom separator
Babel::from('Hello World')->toSlug('_');         // "hello_world"
```

## Safe Filenames

Create filesystem-safe filenames:

```php
Babel::from('My Document (Final).pdf')->toFilename();
// "my_document_final.pdf"

Babel::from('Report 2024/01/15')->toFilename();
// "report_2024_01_15"

// Custom separator
Babel::from('Hello World.txt')->toFilename('-');
// "hello-world.txt"
```

## XML-Safe Strings

Remove characters invalid in XML 1.0:

```php
Babel::from("Hello\x00World")->toXmlSafe();  // "HelloWorld"
Babel::from("Tab\tNewline\n")->toXmlSafe();  // "Tab\tNewline\n" (preserved)
```

## Error Handling

Conversion methods throw exceptions on failure:

```php
use Cline\Babel\Exceptions\EncodingException;

try {
    $result = Babel::from($text)->toEncoding('INVALID-ENCODING');
} catch (EncodingException $e) {
    // Handle invalid encoding
}
```
