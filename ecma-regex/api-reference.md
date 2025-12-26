# API Reference

Complete API documentation for ECMA-262 Regex package.

## Facade API

The `EcmaRegex` facade provides the simplest interface for pattern matching.

### `test(string $pattern, string $input): bool`

Test if a pattern matches the input string.

**Parameters:**
- `$pattern` - Regex pattern (e.g., `/^[a-z]+$/`)
- `$input` - String to test against

**Returns:** `true` if pattern matches, `false` otherwise

**Example:**
```php
use Cline\EcmaRegex\Facades\EcmaRegex;

EcmaRegex::test('/^[a-z]+$/', 'hello');  // true
EcmaRegex::test('/\d+/', 'abc');         // false
```

### `match(string $pattern, string $input): MatchResult`

Execute pattern and return detailed match information.

**Parameters:**
- `$pattern` - Regex pattern
- `$input` - String to match against

**Returns:** `MatchResult` object with match details

**Example:**
```php
$result = EcmaRegex::match('/h(ello)/', 'hello world');

$result->matched;   // bool: true
$result->match;     // string: 'hello'
$result->index;     // int: 0
$result->captures;  // array: ['ello']
$result->input;     // string: 'hello world'
```

### `matchAll(string $pattern, string $input): array`

Find all matches of a pattern in the input.

**Parameters:**
- `$pattern` - Regex pattern
- `$input` - String to search

**Returns:** Array of `MatchResult` objects

**Example:**
```php
$matches = EcmaRegex::matchAll('/\d+/', '123 abc 456');

foreach ($matches as $match) {
    echo "{$match->match} at {$match->index}\n";
}
// Output:
// 123 at 0
// 456 at 8
```

### `compile(string $pattern, string $flags = ''): Pattern`

Compile a pattern for reuse.

**Parameters:**
- `$pattern` - Regex pattern
- `$flags` - Optional flags: `g`, `i`, `m`, `s`, `u`, `y`

**Returns:** `Pattern` instance

**Example:**
```php
$pattern = EcmaRegex::compile('/[a-z]+/i');

$pattern->test('Hello');      // true
$pattern->exec('Hello World'); // MatchResult
```

### `flushCache(): void`

Clear the compiled pattern cache.

**Example:**
```php
EcmaRegex::flushCache();
```

## Pattern API

The `Pattern` class represents a compiled regex pattern.

### `compile(string $source, string $flags, LexerInterface $lexer, ParserInterface $parser): self`

Static factory method to compile a pattern.

**Parameters:**
- `$source` - Pattern source string
- `$flags` - Flag string (empty or combination of `gimsuy`)
- `$lexer` - Lexer instance for tokenization
- `$parser` - Parser instance for AST generation

**Returns:** Compiled `Pattern` instance

**Example:**
```php
use Cline\EcmaRegex\Pattern;
use Cline\EcmaRegex\Contracts\LexerInterface;
use Cline\EcmaRegex\Contracts\ParserInterface;

$pattern = Pattern::compile(
    source: '/^test$/',
    flags: 'i',
    lexer: app(LexerInterface::class),
    parser: app(ParserInterface::class)
);
```

### `test(string $input): bool`

Test if pattern matches the input.

**Parameters:**
- `$input` - String to test

**Returns:** `true` if matched, `false` otherwise

**Example:**
```php
$pattern->test('hello');  // bool
```

### `exec(string $input): MatchResult`

Execute pattern and return match details.

**Parameters:**
- `$input` - String to match

**Returns:** `MatchResult` object

**Example:**
```php
$result = $pattern->exec('hello world');

if ($result->matched) {
    echo $result->match;
}
```

### `matchAll(string $input): array`

Find all pattern matches.

**Parameters:**
- `$input` - String to search

**Returns:** Array of `MatchResult` objects

**Example:**
```php
$matches = $pattern->matchAll('test input');
```

### Property Accessors

- `source(): string` - Get original pattern source
- `flags(): string` - Get pattern flags
- `ast(): NodeInterface` - Get compiled AST root node

## MatchResult

Immutable value object representing a match result.

### Static Constructors

#### `success(int $index, string $match, array $captures, string $input): self`

Create a successful match result.

**Parameters:**
- `$index` - Match start position (0-based)
- `$match` - Matched substring
- `$captures` - Array of captured groups
- `$input` - Original input string

**Example:**
```php
$result = MatchResult::success(0, 'hello', ['ello'], 'hello world');
```

#### `failed(): self`

Create a failed match result.

**Example:**
```php
$result = MatchResult::failed();
$result->matched;  // false
```

### Properties

- `matched: bool` - Whether pattern matched
- `index: int|null` - Match start position (null if no match)
- `match: string|null` - Matched substring (null if no match)
- `captures: array` - Captured groups (empty if no match)
- `input: string|null` - Original input (null if no match)

## Manager API

The `EcmaRegexManager` coordinates pattern compilation and caching.

### `__construct(CacheInterface $cache, LexerInterface $lexer, ParserInterface $parser, bool $cacheEnabled, int $cacheTtl)`

**Parameters:**
- `$cache` - Cache implementation
- `$lexer` - Lexer for tokenization
- `$parser` - Parser for AST generation
- `$cacheEnabled` - Enable/disable caching
- `$cacheTtl` - Cache TTL in seconds

### `compile(string $pattern, string $flags = ''): Pattern`

Compile and cache a pattern.

**Example:**
```php
$manager = app(EcmaRegexManager::class);
$pattern = $manager->compile('/test/', 'i');
```

### `test(string $pattern, string $input): bool`

Quick pattern test.

**Example:**
```php
$manager->test('/\d+/', '123');  // true
```

### `match(string $pattern, string $input): MatchResult`

Quick pattern match.

**Example:**
```php
$result = $manager->match('/(\d+)/', '123');
```

### `flushCache(): void`

Clear the pattern cache.

## Contracts

### `PatternInterface`

```php
interface PatternInterface
{
    public function test(string $input): bool;
    public function exec(string $input): MatchResult;
    public function matchAll(string $input): array;
    public function source(): string;
    public function flags(): string;
    public function ast(): NodeInterface;
}
```

### `LexerInterface`

```php
interface LexerInterface
{
    /**
     * @return array<Token>
     */
    public function tokenize(string $source): array;
}
```

### `ParserInterface`

```php
interface ParserInterface
{
    public function parse(array $tokens): NodeInterface;
}
```

### `MatcherInterface`

```php
interface MatcherInterface
{
    public function match(NodeInterface $ast, string $input): MatchResult;
}
```

### `NodeInterface`

```php
interface NodeInterface
{
    public function type(): NodeType;
    public function toArray(): array;
}
```

## Exceptions

All exceptions implement `EcmaRegexException` marker interface.

### Parsing Exceptions

- `IncompleteEscapeSequenceException` - Incomplete escape sequence
- `IncompleteGroupConstructException` - Unclosed group
- `IncompleteLookbehindConstructException` - Unclosed lookbehind
- `InvalidGroupConstructException` - Invalid group syntax
- `InvalidLookbehindConstructException` - Invalid lookbehind syntax
- `MissingTokenException` - Expected token not found
- `UnclosedCharacterClassException` - Unclosed character class `[`
- `UnexpectedTokenException` - Unexpected token encountered
- `UnexpectedTokenInCharacterClassException` - Invalid token in character class

### Runtime Exceptions

- `UnsupportedAssertionKindException` - Unknown assertion type
- `UnsupportedNodeTypeException` - Unknown AST node type

**Example:**
```php
use Cline\EcmaRegex\Facades\EcmaRegex;
use Cline\EcmaRegex\Exceptions\EcmaRegexException;

try {
    EcmaRegex::test('/[unclosed/', 'test');
} catch (EcmaRegexException $e) {
    echo "Invalid pattern: {$e->getMessage()}";
}
```

## Enums

### `NodeType`

AST node types:

- `NodeType::Root` - Root pattern node
- `NodeType::Alternation` - Alternation (`|`)
- `NodeType::Group` - Capturing/non-capturing group
- `NodeType::Assertion` - Lookahead/lookbehind
- `NodeType::Quantifier` - Repetition operator
- `NodeType::CharacterClass` - Character class `[...]`
- `NodeType::Literal` - Literal character
- `NodeType::Backreference` - Backreference `\1`, `\2`, etc.

### `TokenType`

Lexer token types (internal use):

- `TokenType::Literal` - Literal character
- `TokenType::Dot` - Metacharacter `.`
- `TokenType::Caret` - Anchor `^`
- `TokenType::Dollar` - Anchor `$`
- `TokenType::LeftBracket` - Character class start `[`
- `TokenType::RightBracket` - Character class end `]`
- `TokenType::LeftParen` - Group start `(`
- `TokenType::RightParen` - Group end `)`
- `TokenType::Pipe` - Alternation `|`
- `TokenType::Asterisk` - Quantifier `*`
- `TokenType::Plus` - Quantifier `+`
- `TokenType::Question` - Quantifier `?`
- `TokenType::LeftBrace` - Quantifier start `{`
- `TokenType::RightBrace` - Quantifier end `}`
- `TokenType::Comma` - Quantifier separator `,`
- `TokenType::Hyphen` - Range separator `-`
- `TokenType::Backslash` - Escape character `\`
- `TokenType::Eof` - End of input

## Service Provider

### `EcmaRegexServiceProvider`

Registers package services and bindings.

**Registered Services:**
- `EcmaRegexManager::class` - Manager singleton
- `PatternInterface::class` → `Pattern::class`
- `LexerInterface::class` → `Lexer::class`
- `ParserInterface::class` → `Parser::class`
- `MatcherInterface::class` → `Matcher::class`

**Publishable Assets:**
- Config: `php artisan vendor:publish --tag="ecma-regex-config"`

## Configuration

### `config/ecma-regex.php`

```php
return [
    // Enable pattern caching
    'cache_enabled' => env('ECMA_REGEX_CACHE_ENABLED', true),

    // Cache TTL in seconds (1 hour default)
    'cache_ttl' => env('ECMA_REGEX_CACHE_TTL', 3600),
];
```

### Environment Variables

- `ECMA_REGEX_CACHE_ENABLED` - Enable/disable caching (default: `true`)
- `ECMA_REGEX_CACHE_TTL` - Cache lifetime in seconds (default: `3600`)
