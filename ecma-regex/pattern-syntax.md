# Pattern Syntax

Complete reference for ECMA-262 regex pattern syntax supported by this package.

## Literals

Match exact characters:

```php
EcmaRegex::test('/abc/', 'abc');     // true
EcmaRegex::test('/123/', '123');     // true
EcmaRegex::test('/hello/', 'hello'); // true
```

## Metacharacters

### Dot (.)

Matches any single character except newline:

```php
EcmaRegex::test('/./', 'a');     // true
EcmaRegex::test('/a.c/', 'abc'); // true
EcmaRegex::test('/a.c/', 'a\nc'); // false (newline)
```

### Anchors

**Start of string** (`^`):

```php
EcmaRegex::test('/^start/', 'start test');  // true
EcmaRegex::test('/^start/', 'test start');  // false
```

**End of string** (`$`):

```php
EcmaRegex::test('/end$/', 'test end');  // true
EcmaRegex::test('/end$/', 'end test');  // false
```

**Word boundaries** (`\b`, `\B`):

```php
EcmaRegex::test('/\bword\b/', 'a word here');  // true
EcmaRegex::test('/\bword\b/', 'password');     // false
EcmaRegex::test('/\Bword\B/', 'password');     // true
```

## Character Classes

### Basic Classes

Match any character from a set:

```php
EcmaRegex::test('/[abc]/', 'a');      // true
EcmaRegex::test('/[abc]/', 'b');      // true
EcmaRegex::test('/[abc]/', 'd');      // false
```

### Ranges

Use hyphens for character ranges:

```php
EcmaRegex::test('/[a-z]/', 'x');      // true
EcmaRegex::test('/[A-Z]/', 'M');      // true
EcmaRegex::test('/[0-9]/', '5');      // true
EcmaRegex::test('/[a-zA-Z0-9]/', 'A'); // true
```

### Negated Classes

Use `^` to match anything NOT in the set:

```php
EcmaRegex::test('/[^0-9]/', 'a');     // true (not a digit)
EcmaRegex::test('/[^0-9]/', '5');     // false (is a digit)
EcmaRegex::test('/[^abc]/', 'd');     // true
```

### Predefined Classes

**Digits** (`\d` = `[0-9]`, `\D` = `[^0-9]`):

```php
EcmaRegex::test('/\d+/', '123');      // true
EcmaRegex::test('/\D/', 'a');         // true
```

**Word characters** (`\w` = `[a-zA-Z0-9_]`, `\W` = `[^a-zA-Z0-9_]`):

```php
EcmaRegex::test('/\w+/', 'hello');    // true
EcmaRegex::test('/\W/', '!');         // true
```

**Whitespace** (`\s` = spaces/tabs/newlines, `\S` = non-whitespace):

```php
EcmaRegex::test('/\s+/', '   ');      // true
EcmaRegex::test('/\S/', 'a');         // true
```

## Quantifiers

### Zero or More (`*`)

```php
EcmaRegex::test('/ab*c/', 'ac');      // true (zero b's)
EcmaRegex::test('/ab*c/', 'abc');     // true (one b)
EcmaRegex::test('/ab*c/', 'abbbbc');  // true (many b's)
```

### One or More (`+`)

```php
EcmaRegex::test('/ab+c/', 'ac');      // false (needs at least one b)
EcmaRegex::test('/ab+c/', 'abc');     // true
EcmaRegex::test('/ab+c/', 'abbbbc');  // true
```

### Zero or One (`?`)

```php
EcmaRegex::test('/colou?r/', 'color');   // true
EcmaRegex::test('/colou?r/', 'colour');  // true
```

### Exact Count (`{n}`)

```php
EcmaRegex::test('/a{3}/', 'aaa');     // true
EcmaRegex::test('/a{3}/', 'aa');      // false
EcmaRegex::test('/\d{4}/', '2024');   // true
```

### Range Count (`{n,m}`)

```php
EcmaRegex::test('/a{2,4}/', 'aa');    // true
EcmaRegex::test('/a{2,4}/', 'aaa');   // true
EcmaRegex::test('/a{2,4}/', 'aaaaa'); // false
```

### Minimum Count (`{n,}`)

```php
EcmaRegex::test('/a{2,}/', 'aa');     // true
EcmaRegex::test('/a{2,}/', 'aaaaa');  // true
EcmaRegex::test('/a{2,}/', 'a');      // false
```

### Greedy vs. Lazy

By default, quantifiers are greedy (match as much as possible). Add `?` for lazy (match as little as possible):

```php
// Greedy
$result = EcmaRegex::match('/".+"/', '"first" "second"');
echo $result->match;  // "first" "second" (matches entire string)

// Lazy
$result = EcmaRegex::match('/".+?"/', '"first" "second"');
echo $result->match;  // "first" (matches minimally)
```

## Groups

### Capturing Groups

Parentheses create capturing groups:

```php
$result = EcmaRegex::match('/(hello) (world)/', 'hello world');
print_r($result->captures);  // ['hello', 'world']

$result = EcmaRegex::match('/(\d{3})-(\d{3})-(\d{4})/', '555-123-4567');
print_r($result->captures);  // ['555', '123', '4567']
```

### Non-Capturing Groups

Use `(?:...)` to group without capturing:

```php
EcmaRegex::test('/(?:abc)+/', 'abcabc');  // true

$result = EcmaRegex::match('/(?:ab)+/', 'abab');
print_r($result->captures);  // [] (no captures)
```

### Backreferences

Reference earlier captures with `\1`, `\2`, etc.:

```php
// Match repeated words
EcmaRegex::test('/(\w+) \1/', 'hello hello');  // true
EcmaRegex::test('/(\w+) \1/', 'hello world');  // false

// Match HTML tags
EcmaRegex::test('/<(\w+)>.*<\/\1>/', '<div>content</div>');  // true
```

## Assertions

### Lookahead

**Positive lookahead** `(?=...)` - matches if followed by pattern:

```php
// Password must contain at least one digit
EcmaRegex::test('/^(?=.*\d)/', 'abc123');  // true
EcmaRegex::test('/^(?=.*\d)/', 'abcdef');  // false

// Find 'q' not followed by 'u'
EcmaRegex::test('/q(?!u)/', 'Iraq');  // true
```

**Negative lookahead** `(?!...)` - matches if NOT followed by pattern:

```php
EcmaRegex::test('/\d+(?!\.)\b/', '123');     // true
EcmaRegex::test('/\d+(?!\.)\b/', '123.45');  // false
```

### Lookbehind

**Positive lookbehind** `(?<=...)` - matches if preceded by pattern:

```php
EcmaRegex::test('/(?<=@)\w+/', '@test');     // true (matches 'test')
EcmaRegex::test('/(?<=\$)\d+/', '$100');     // true (matches '100')
```

**Negative lookbehind** `(?<!...)` - matches if NOT preceded by pattern:

```php
EcmaRegex::test('/(?<!@)\w+/', 'test');      // true
EcmaRegex::test('/(?<!@)\w+/', '@test');     // false
```

## Alternation

Use `|` for "or" logic:

```php
EcmaRegex::test('/cat|dog/', 'cat');         // true
EcmaRegex::test('/cat|dog/', 'dog');         // true
EcmaRegex::test('/cat|dog/', 'bird');        // false

EcmaRegex::test('/red|blue|green/', 'blue'); // true
```

## Escape Sequences

Escape special characters with backslash:

```php
EcmaRegex::test('/\./', '.');       // literal dot
EcmaRegex::test('/\*/', '*');       // literal asterisk
EcmaRegex::test('/\(/', '(');       // literal parenthesis
EcmaRegex::test('/\[/', '[');       // literal bracket
EcmaRegex::test('/\$/', '$');       // literal dollar
EcmaRegex::test('/\^/', '^');       // literal caret
```

Special escape sequences:

```php
EcmaRegex::test('/\n/', "\n");      // newline
EcmaRegex::test('/\t/', "\t");      // tab
EcmaRegex::test('/\r/', "\r");      // carriage return
```

## Flags

### Case-Insensitive (`i`)

```php
$pattern = EcmaRegex::compile('/hello/', 'i');
$pattern->test('HELLO');  // true
$pattern->test('HeLLo');  // true
```

### Global (`g`)

Find all matches (used internally by `matchAll`):

```php
$pattern = EcmaRegex::compile('/\d+/', 'g');
$matches = $pattern->matchAll('123 abc 456');
// Returns all numeric matches
```

### Multiline (`m`)

Changes `^` and `$` to match line boundaries:

```php
$pattern = EcmaRegex::compile('/^start/m');
$pattern->test("line1\nstart of line2");  // true
```

### Dotall (`s`)

Makes `.` match newlines:

```php
$pattern = EcmaRegex::compile('/a.b/', 's');
$pattern->test("a\nb");  // true
```

### Unicode (`u`)

Enables full Unicode support:

```php
$pattern = EcmaRegex::compile('/\w+/', 'u');
$pattern->test('café');  // true
```

## Examples

### Validate Email

```php
$pattern = '/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/';
EcmaRegex::test($pattern, 'user@example.com');
```

### Validate Phone

```php
$pattern = '/^\d{3}-\d{3}-\d{4}$/';
EcmaRegex::test($pattern, '555-123-4567');
```

### Extract URLs

```php
$pattern = '/https?:\/\/[^\s]+/g';
$matches = EcmaRegex::matchAll($pattern, 'Visit https://example.com or http://test.org');
```

### Password Validation

Requires 8+ chars, 1 uppercase, 1 lowercase, 1 digit:

```php
$pattern = '/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/';
EcmaRegex::test($pattern, 'SecurePass1');  // true
```
