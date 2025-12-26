# Architecture

Internal architecture and design principles of the ECMA-262 Regex engine.

## Overview

The package follows a clean three-phase pipeline architecture:

```
Pattern → Lexer → Parser → Matcher → Result
  ↓         ↓        ↓        ↓         ↓
Source   Tokens    AST    Execution  Match
```

Each phase is isolated, testable, and follows SOLID principles.

## Pipeline Phases

### 1. Lexical Analysis (Lexer)

**Purpose:** Transform raw regex string into tokens

**Input:** `/^[a-z]+$/`

**Output:** Array of tokens
```php
[
    Token(Caret, '^', 1),
    Token(LeftBracket, '[', 2),
    Token(Literal, 'a', 3),
    Token(Hyphen, '-', 4),
    Token(Literal, 'z', 5),
    Token(RightBracket, ']', 6),
    Token(Plus, '+', 7),
    Token(Dollar, '$', 8),
]
```

**Implementation:** `Lexer\Lexer`

**Key methods:**
- `tokenize(string $source): array<Token>` - Main tokenization
- `scanToken(): void` - Scan next token
- `scanGroup(): void` - Handle group constructs
- `scanCharacterClass(): void` - Handle `[...]` classes
- `scanQuantifier(): void` - Handle `{n,m}` quantifiers
- `scanEscape(): void` - Handle escape sequences

### 2. Parsing (Parser)

**Purpose:** Build Abstract Syntax Tree (AST) from tokens

**Input:** Token array

**Output:** AST
```php
RootNode(
    AlternationNode([
        QuantifierNode(
            CharacterClassNode([a-z]),
            min: 1,
            max: null
        )
    ]),
    anchors: [Caret, Dollar]
)
```

**Implementation:** `Parser\Parser`

**Key methods:**
- `parse(array $tokens): NodeInterface` - Main parsing
- `parseAlternation(): NodeInterface` - Handle `|` alternation
- `parseSequence(): array` - Parse sequence of elements
- `parseAtom(): NodeInterface` - Parse single pattern element
- `parseQuantifier(): ?array` - Parse quantifier if present
- `parseGroup(): NodeInterface` - Parse groups `(...)`
- `parseCharacterClass(): NodeInterface` - Parse classes `[...]`

**Parser uses recursive descent** with operator precedence:
1. Alternation (`|`) - lowest precedence
2. Sequence (concatenation)
3. Quantifiers (`*`, `+`, `?`, `{n,m}`)
4. Atoms (literals, groups, classes) - highest precedence

### 3. Matching (Matcher)

**Purpose:** Execute AST against input string using backtracking

**Input:** AST + input string

**Output:** MatchResult

**Implementation:** `Matcher\Matcher`

**Algorithm:** Backtracking NFA (Nondeterministic Finite Automaton)

**Key methods:**
- `match(NodeInterface $ast, string $input): MatchResult`
- `matchNode(NodeInterface $node, MatchContext $ctx): bool`
- `matchAlternation(AlternationNode $node, MatchContext $ctx): bool`
- `matchQuantifier(QuantifierNode $node, MatchContext $ctx): bool`
- `matchGroup(GroupNode $node, MatchContext $ctx): bool`
- `matchCharacterClass(CharacterClassNode $node, MatchContext $ctx): bool`

**Backtracking:** Uses `BacktrackStack` to save/restore state for greedy/lazy quantifiers and alternations.

## Core Components

### Pattern

Immutable representation of a compiled regex pattern.

**Responsibilities:**
- Store compiled pattern (AST + metadata)
- Provide high-level matching interface
- Maintain thread-safety (readonly class)

**Key features:**
- Immutable and thread-safe
- Factory method pattern (`Pattern::compile()`)
- Delegates matching to Matcher

```php
final readonly class Pattern implements PatternInterface
{
    public function __construct(
        private string $source,
        private string $flags,
        private NodeInterface $ast,
    ) {}

    public function test(string $input): bool;
    public function exec(string $input): MatchResult;
    public function matchAll(string $input): array;
}
```

### AST Nodes

Hierarchy of immutable AST node classes.

**Base:** `AbstractNode implements NodeInterface`

**Node types:**
- `RootNode` - Root of pattern tree
- `AlternationNode` - Handles `|` alternation
- `GroupNode` - Capturing and non-capturing groups
- `AssertionNode` - Lookahead/lookbehind assertions
- `QuantifierNode` - Repetition (`*`, `+`, `?`, `{n,m}`)
- `CharacterClassNode` - Character sets `[...]`
- `LiteralNode` - Literal characters
- `BackreferenceNode` - Backreferences `\1`, `\2`

**Design:**
```php
abstract class AbstractNode implements NodeInterface
{
    public function __construct(
        private readonly NodeType $type,
    ) {}

    abstract public function toArray(): array;
}
```

### Match Context

Maintains matching state during execution.

**Responsibilities:**
- Track current position in input string
- Store captured groups
- Manage backtrack points

```php
final class MatchContext
{
    private int $position = 0;
    private array $captures = [];
    private BacktrackStack $backtrackStack;

    public function advance(int $count = 1): void;
    public function getPosition(): int;
    public function addCapture(int $groupIndex, string $value): void;
    public function getCaptures(): array;
}
```

### Backtrack Stack

Manages backtracking state for the matcher.

**Purpose:** Save and restore match state when trying alternative paths

**Key operations:**
- `push(BacktrackPoint $point): void` - Save state
- `pop(): ?BacktrackPoint` - Restore state
- `isEmpty(): bool` - Check if alternatives remain

```php
final class BacktrackStack
{
    private array $stack = [];

    public function push(BacktrackPoint $point): void
    {
        $this->stack[] = $point;
    }

    public function pop(): ?BacktrackPoint
    {
        return array_pop($this->stack);
    }
}

final readonly class BacktrackPoint
{
    public function __construct(
        public int $position,
        public array $captures,
        public NodeInterface $node,
    ) {}
}
```

## Design Patterns

### 1. Immutable Value Objects

All data structures are immutable:

```php
final readonly class Token { ... }
final readonly class MatchResult { ... }
final readonly class Pattern { ... }
final readonly class BacktrackPoint { ... }
```

**Benefits:**
- Thread-safe
- No side effects
- Easy to reason about
- Cacheable

### 2. Factory Methods

Named constructors for clarity:

```php
// Pattern compilation
Pattern::compile($source, $flags, $lexer, $parser);

// Match results
MatchResult::success($index, $match, $captures, $input);
MatchResult::failed();
```

### 3. Strategy Pattern

Pluggable implementations via interfaces:

```php
interface LexerInterface { ... }
interface ParserInterface { ... }
interface MatcherInterface { ... }
interface PatternInterface { ... }
```

### 4. Dependency Injection

All dependencies injected via constructor:

```php
final class EcmaRegexManager
{
    public function __construct(
        private CacheInterface $cache,
        private LexerInterface $lexer,
        private ParserInterface $parser,
        private bool $cacheEnabled,
        private int $cacheTtl,
    ) {}
}
```

### 5. Visitor Pattern

AST traversal via node types:

```php
match ($node->type()) {
    NodeType::Alternation => $this->matchAlternation($node, $ctx),
    NodeType::Quantifier => $this->matchQuantifier($node, $ctx),
    NodeType::Group => $this->matchGroup($node, $ctx),
    // ...
}
```

## SOLID Principles

### Single Responsibility

Each class has one job:
- `Lexer` - tokenization only
- `Parser` - AST generation only
- `Matcher` - pattern execution only

### Open/Closed

Extend via new node types without modifying existing code:

```php
// Add new node type
enum NodeType {
    case NewFeature;
}

// Extend matcher
private function matchNewFeature(NewFeatureNode $node, MatchContext $ctx): bool
{
    // Implementation
}
```

### Liskov Substitution

All node types are substitutable:

```php
function processNode(NodeInterface $node): void
{
    // Works with any node type
    $array = $node->toArray();
}
```

### Interface Segregation

Small, focused interfaces:

```php
interface NodeInterface {
    public function type(): NodeType;
    public function toArray(): array;
}

interface PatternInterface {
    public function test(string $input): bool;
    public function exec(string $input): MatchResult;
}
```

### Dependency Inversion

Depend on abstractions, not concretions:

```php
// Manager depends on interfaces
public function __construct(
    private LexerInterface $lexer,      // Not Lexer
    private ParserInterface $parser,    // Not Parser
) {}
```

## Caching Strategy

Two-tier cache architecture:

### L1: Memory Cache

In-process array for instant access:

```php
private array $memoryCache = [];

private function getFromMemory(string $key): ?Pattern
{
    return $this->memoryCache[$key] ?? null;
}
```

### L2: Persistent Cache

Laravel Cache for cross-request sharing:

```php
private function getFromPersistent(string $key): ?Pattern
{
    return $this->cache->get($key);
}
```

**Cache key:** Hash of pattern + flags
```php
private function cacheKey(string $pattern, string $flags): string
{
    return 'ecma-regex:' . md5($pattern . $flags);
}
```

## Error Handling

### Exception Hierarchy

```
RuntimeException
└── EcmaRegexException (marker interface)
    ├── IncompleteEscapeSequenceException
    ├── IncompleteGroupConstructException
    ├── IncompleteLookbehindConstructException
    ├── InvalidGroupConstructException
    ├── InvalidLookbehindConstructException
    ├── MissingTokenException
    ├── UnclosedCharacterClassException
    ├── UnexpectedTokenException
    ├── UnexpectedTokenInCharacterClassException
    ├── UnsupportedAssertionKindException
    └── UnsupportedNodeTypeException
```

### Error Detection

**Lexer:** Detects syntax errors during tokenization
```php
if ($this->current === null) {
    throw new IncompleteEscapeSequenceException();
}
```

**Parser:** Validates token sequences
```php
if ($token->tokenType() !== TokenType::RightParen) {
    throw new MissingTokenException(')');
}
```

**Matcher:** Validates node types
```php
default => throw new UnsupportedNodeTypeException($node->type()->value)
```

## Testing Strategy

### Unit Tests

Test each component in isolation:
- `LexerTest` - tokenization
- `ParserTest` - AST generation
- `MatcherTest` - matching logic
- `PatternTest` - pattern compilation

### Feature Tests

Test end-to-end workflows:
- `EcmaRegexFacadeTest` - facade methods
- `EcmaRegexManagerTest` - manager functionality
- `JsonSchemaCompatibilityTest` - JSON Schema compliance

### Coverage

- **294 tests passing** (100% pass rate)
- **90%+ code coverage** requirement
- **90%+ type coverage** (PHPStan)

## Performance Characteristics

### Time Complexity

**Lexer:** O(n) where n = pattern length
**Parser:** O(n) where n = token count
**Matcher (worst case):** O(2^n) due to backtracking
**Matcher (average case):** O(n*m) where n = input length, m = pattern complexity

### Space Complexity

**Compiled pattern:** O(n) where n = pattern length
**Match context:** O(m) where m = capture count
**Backtrack stack:** O(k) where k = backtrack depth

### Optimization

- Pattern compilation cached (O(1) retrieval)
- Two-tier cache (memory + persistent)
- Immutable structures (safe to share)
- Lazy quantifier support (prevents catastrophic backtracking)

## Extension Points

### Adding New Node Types

1. Define enum case:
```php
enum NodeType {
    case NewFeature;
}
```

2. Create node class:
```php
final class NewFeatureNode extends AbstractNode
{
    public function __construct(/* params */)
    {
        parent::__construct(NodeType::NewFeature);
    }
}
```

3. Update parser:
```php
private function parseNewFeature(): NodeInterface
{
    // Parsing logic
}
```

4. Update matcher:
```php
private function matchNewFeature(NewFeatureNode $node, MatchContext $ctx): bool
{
    // Matching logic
}
```

### Custom Implementations

Swap implementations via service container:

```php
// Custom lexer
app()->bind(LexerInterface::class, CustomLexer::class);

// Custom matcher
app()->bind(MatcherInterface::class, CustomMatcher::class);
```
