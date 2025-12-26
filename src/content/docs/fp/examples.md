---
title: Examples
description: Practical examples demonstrating how to use pipe functions for real-world scenarios including file processing and lazy generators.
---

It may not be entirely obvious how all of the above fit together. To help make it clear, here's some examples of pipes and their functions in action.

## File Processing Example

The following example will take an input file name `$inputFile`, load its content into memory, trim whitespace, split it by lines into an array, call a custom `pairUp()` function on that array, filter the resulting array, and then count the remaining values. All of that in one simple statement.

```php
use function Cline\fp\{pipe, explode, afilter};

$result = pipe($inputFile,
    file_get_contents(...),
    trim(...),
    explode(PHP_EOL),
    pairUp(...),
    afilter(static fn($v): bool => $v[0] > $v[1]),
    count(...),
);
```

## Lazy Generator Example

This example uses a custom function to read lines from a file lazily. That produces a lazy generator, which is then passed to `itmap()` which will apply `parse()` to each item in the generator, but is itself a generator so is also lazy. `parse()` produces a `Step` object from each line of input. `reduce()` therefore receives an iterable of `Step` objects, and applies `move()` to each one in turn, from a starting point. Each time, `move()` returns a new `Position`.

In other words, these few lines of code constitute a complete script parsing engine, albeit a simple one. Because each step is lazy, only one `Step` is loaded into memory at once.

```php
use function Cline\fp\{pipe, itmap, reduce};

function lines(string $file): iterable
{
    $fp = fopen($file, 'rb');

    while ($line = fgets($fp)) {
        yield trim($line);
    }

    fclose($fp);
}

function parse(string $line): Step
{
    [$cmd, $size] = \explode(' ', $line);
    return new Step(cmd: Command::from($cmd), size: (int)$size);
}

$end = pipe($inputFile,
    lines(...),
    itmap(parse(...)),
    reduce(new Position(0, 0), move(...)),
);
```

## Greedy Array Version

The alternate version below uses the greedy `amap()` instead. That will produce an already-computed array of `Step` objects rather than a generator that produces `Step` objects.

```php
use function Cline\fp\{pipe, amap, reduce};

$end = pipe($inputFile,
    lines(...),
    amap(parse(...)),
    reduce(new Position(0, 0), move(...)),
);
```
