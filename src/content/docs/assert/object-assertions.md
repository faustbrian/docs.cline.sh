---
title: Object Assertions
description: Validate objects, classes, interfaces, and their properties.
---

Object assertions validate objects, classes, interfaces, and their properties/methods.

## Available Assertions

### isInstanceOf()

Assert that a value is an instance of a given class.

```php
use Cline\Assert\Assertions\Assertion;

Assertion::isInstanceOf($user, User::class);
Assertion::isInstanceOf($model, Model::class, 'Expected Model instance');
```

### notIsInstanceOf()

Assert that a value is NOT an instance of a given class.

```php
Assertion::notIsInstanceOf($value, LegacyUser::class);
```

### classExists()

Assert that a class exists.

```php
Assertion::classExists(User::class);
Assertion::classExists($className, 'Class does not exist');
```

### interfaceExists()

Assert that an interface exists.

```php
Assertion::interfaceExists(UserInterface::class);
```

### subclassOf()

Assert that a class is a subclass of another.

```php
Assertion::subclassOf(AdminUser::class, User::class);
```

### implementsInterface()

Assert that a class implements an interface.

```php
Assertion::implementsInterface(User::class, UserInterface::class);
```

### methodExists()

Assert that a method exists on an object.

```php
Assertion::methodExists('save', $model);
Assertion::methodExists('handle', $handler, 'Handler must have handle method');
```

### propertyExists()

Assert that a property exists on an object or class.

```php
Assertion::propertyExists($user, 'email');
```

### propertiesExist()

Assert that multiple properties exist.

```php
$required = ['id', 'name', 'email'];
Assertion::propertiesExist($user, $required);
```

## Chaining Object Assertions

```php
use Cline\Assert\Assert;

Assert::that($user)
    ->isObject()
    ->isInstanceOf(User::class);

Assert::that(User::class)
    ->classExists()
    ->implementsInterface(UserInterface::class);
```

## Common Patterns

### Dependency Injection Validation

```php
Assert::that($logger)
    ->isObject()
    ->implementsInterface(LoggerInterface::class);
```

### Model Validation

```php
Assert::that($model)
    ->isObject()
    ->isInstanceOf(Model::class)
    ->propertyExists('id')
    ->propertyExists('created_at');
```

### Plugin Validation

```php
Assert::that($plugin)
    ->isObject()
    ->implementsInterface(PluginInterface::class)
    ->methodExists('register')
    ->methodExists('boot');
```

### Factory Pattern Validation

```php
public function make(string $class)
{
    Assertion::classExists($class);
    Assertion::subclassOf($class, BaseService::class);

    return new $class();
}
```

## Working with Interfaces

```php
Assertion::implementsInterface(JsonSerializer::class, Serializer::class);

Assert::that($serializer)
    ->isObject()
    ->isInstanceOf(Serializer::class);
```

## Best Practices

### Interface Over Implementation

```php
// Tightly coupled
Assert::that($logger)->isInstanceOf(MonologLogger::class);

// Depends on interface
Assert::that($logger)->isInstanceOf(LoggerInterface::class);
```

### Combine with Method Checks

```php
Assert::that($handler)
    ->isObject()
    ->isInstanceOf(HandlerInterface::class)
    ->methodExists('handle');
```

## Next Steps

- [Type Assertions](/assert/type-assertions/) - Basic type checking
- [Custom Assertions](/assert/custom-assertions/) - Custom object validation
- [Lazy Assertions](/assert/lazy-assertions/) - Validate multiple properties
