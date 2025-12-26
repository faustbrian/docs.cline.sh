---
title: Language & Locale Value Objects
description: Working with languages and locales using the Language and Locale value objects in Cline Intl
---

The `Language` and `Locale` value objects provide type-safe handling of language codes and locale identifiers, with localized names powered by Symfony Intl. These work together to enable proper internationalization support in your Laravel application.

## Language Value Object

The `Language` value object represents a language using ISO 639-1 language codes.

### Creating Language Instances

Create a language instance from an ISO 639-1 language code:

```php
use Cline\Intl\ValueObjects\Language;

$language = Language::createFromString('en');
```

The factory method accepts 2-letter ISO 639-1 language codes (e.g., `en`, `fr`, `de`, `es`).

### Language Properties

```php
$language = Language::createFromString('en');

// Language code
echo $language->value;  // "en"

// Localized language name
echo $language->localized;  // "English"
```

When you change your application's locale, the `localized` property will reflect the language name in that locale:

```php
app()->setLocale('fr');
$language = Language::createFromString('en');
echo $language->localized;  // "anglais"

app()->setLocale('de');
$language = Language::createFromString('en');
echo $language->localized;  // "Englisch"
```

### String Representation

The `Language` object implements `Stringable` and returns the language code:

```php
$language = Language::createFromString('fr');

echo (string) $language;  // "fr"
echo $language->toString();  // "fr"
```

### Equality Comparison

```php
$language1 = Language::createFromString('en');
$language2 = Language::createFromString('en');
$language3 = Language::createFromString('fr');

$language1->isEqualTo($language2);  // true
$language1->isEqualTo($language3);  // false
```

## Locale Value Object

The `Locale` value object represents a full locale identifier including language, region, script, and variant.

### Creating Locale Instances

Create a locale instance from a locale identifier:

```php
use Cline\Intl\ValueObjects\Locale;

$locale = Locale::createFromString('en_US');
```

The factory method accepts locale identifiers in various formats:
- `en_US` (language + region)
- `fr_FR`
- `de_DE`
- `pt_BR`
- `zh_CN`

### Locale Properties

```php
$locale = Locale::createFromString('en_US');

// Locale identifier
echo $locale->value;  // "en_US"

// Localized locale name
echo $locale->localized;  // "English (United States)"
```

The `localized` property respects your application's locale:

```php
app()->setLocale('fr');
$locale = Locale::createFromString('en_US');
echo $locale->localized;  // "anglais (États-Unis)"
```

### String Representation

```php
$locale = Locale::createFromString('fr_FR');

echo (string) $locale;  // "fr_FR"
echo $locale->toString();  // "fr_FR"
```

### Equality Comparison

```php
$locale1 = Locale::createFromString('en_US');
$locale2 = Locale::createFromString('en_US');
$locale3 = Locale::createFromString('en_GB');

$locale1->isEqualTo($locale2);  // true
$locale1->isEqualTo($locale3);  // false
```

## Eloquent Model Integration

### Using Language and Locale Casts

```php
use Illuminate\Database\Eloquent\Model;
use Cline\Intl\Data\Cast\LanguageCast;
use Cline\Intl\Data\Cast\LocaleCast;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'language' => LanguageCast::class,
            'locale' => LocaleCast::class,
        ];
    }
}
```

### Working with the Casts

```php
// Store as strings, retrieve as value objects
$user = new User();
$user->language = 'en';
$user->locale = 'en_US';
$user->save();

// Automatically cast to value objects
echo $user->language->localized;  // "English"
echo $user->locale->localized;    // "English (United States)"

// Update with strings
$user->language = 'fr';
$user->locale = 'fr_FR';
$user->save();
```

### Database Schema

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('language', 2);  // ISO 639-1 code
    $table->string('locale', 10);   // Locale identifier
    $table->timestamps();
});
```

## Validation

### Language Validation

```php
use Cline\Intl\Rules\LanguageRule;

class UpdateProfileRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'language' => ['required', 'string', new LanguageRule()],
        ];
    }
}
```

### Locale Validation

```php
use Cline\Intl\Rules\LocaleRule;

class UpdatePreferencesRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'locale' => ['required', 'string', new LocaleRule()],
        ];
    }
}
```

## Common Use Cases

### User Language Preference

```php
use Cline\Intl\ValueObjects\Language;
use Cline\Intl\Data\Cast\LanguageCast;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'language' => LanguageCast::class,
        ];
    }

    public function getPreferredLanguage(): string
    {
        return $this->language->value;
    }

    public function setAppLocale(): void
    {
        app()->setLocale($this->language->value);
    }
}

// Usage
$user = auth()->user();
$user->setAppLocale();  // Set application locale to user's preference
```

### Content Localization

```php
use Cline\Intl\ValueObjects\Language;
use Cline\Intl\Data\Cast\LanguageCast;

class Article extends Model
{
    protected function casts(): array
    {
        return [
            'language' => LanguageCast::class,
        ];
    }

    public function scopeInLanguage($query, Language $language)
    {
        return $query->where('language', $language->value);
    }

    public function hasTranslation(Language $language): bool
    {
        return $this->translations()
            ->where('language', $language->value)
            ->exists();
    }
}

// Usage
$language = Language::createFromString('fr');
$articles = Article::inLanguage($language)->get();
```

### Locale-Based Formatting

```php
use Cline\Intl\ValueObjects\Locale;
use Cline\Intl\Data\Cast\LocaleCast;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'locale' => LocaleCast::class,
        ];
    }

    public function formatDate(\DateTime $date): string
    {
        $formatter = new \IntlDateFormatter(
            $this->locale->value,
            \IntlDateFormatter::LONG,
            \IntlDateFormatter::NONE
        );

        return $formatter->format($date);
    }

    public function formatNumber(float $number): string
    {
        $formatter = new \NumberFormatter(
            $this->locale->value,
            \NumberFormatter::DECIMAL
        );

        return $formatter->format($number);
    }
}

// Usage
$user = User::find(1);
$user->locale = 'fr_FR';

echo $user->formatDate(new \DateTime());  // "15 décembre 2025"
echo $user->formatNumber(1234.56);        // "1 234,56"
```

### Language Selector

```php
use Symfony\Component\Intl\Languages;

class LanguageController extends Controller
{
    public function index()
    {
        // Get all languages for a dropdown
        $languages = collect(Languages::getNames())
            ->map(fn($name, $code) => [
                'code' => $code,
                'name' => $name,
            ])
            ->sortBy('name')
            ->values();

        return view('languages.index', compact('languages'));
    }
}
```

### Multi-Language Application

```php
use Cline\Intl\ValueObjects\Language;

class LocalizationMiddleware
{
    public function handle($request, Closure $next)
    {
        // Set locale from authenticated user
        if ($user = auth()->user()) {
            app()->setLocale($user->language->value);
        }
        // Or from session
        elseif ($language = session('language')) {
            app()->setLocale($language);
        }
        // Or from Accept-Language header
        else {
            $language = $request->getPreferredLanguage(['en', 'fr', 'de', 'es']);
            app()->setLocale($language);
        }

        return $next($request);
    }
}
```

### Translation Management

```php
use Cline\Intl\ValueObjects\Language;
use Cline\Intl\Data\Cast\LanguageCast;

class Translation extends Model
{
    protected function casts(): array
    {
        return [
            'language' => LanguageCast::class,
        ];
    }

    public function scopeForLanguage($query, Language $language)
    {
        return $query->where('language', $language->value);
    }
}

class TranslatableModel extends Model
{
    public function translations()
    {
        return $this->hasMany(Translation::class);
    }

    public function getTranslation(Language $language): ?string
    {
        return $this->translations()
            ->forLanguage($language)
            ->first()
            ?->content;
    }

    public function translate(Language $language, string $content): Translation
    {
        return $this->translations()->create([
            'language' => $language->value,
            'content' => $content,
        ]);
    }
}
```

## Language vs. Locale: When to Use Which

### Use Language When:
- You only need to identify the language without regional specifics
- Managing content translations
- Simple language preferences
- Language-specific content filtering

```php
// Simple language preference
$user->language = 'en';  // Just English, no regional variant
```

### Use Locale When:
- You need regional formatting (dates, numbers, currency)
- Full internationalization with regional differences
- Supporting multiple variants of the same language (en_US vs. en_GB)
- Using PHP's Intl extension for formatting

```php
// Full locale with regional formatting
$user->locale = 'en_US';  // English with US formatting
$user->locale = 'en_GB';  // English with UK formatting
```

## Exception Handling

Both value objects throw `MissingResourceException` for invalid codes:

```php
use Symfony\Component\Intl\Exception\MissingResourceException;
use Cline\Intl\ValueObjects\Language;
use Cline\Intl\ValueObjects\Locale;

try {
    $language = Language::createFromString('invalid');
} catch (MissingResourceException $e) {
    // Handle invalid language code
}

try {
    $locale = Locale::createFromString('invalid_LOCALE');
} catch (MissingResourceException $e) {
    // Handle invalid locale identifier
}
```

Always validate user input using validation rules to prevent exceptions.

## Best Practices

1. **Validate input** - Use `LanguageRule` and `LocaleRule` in form requests
2. **Store codes, not names** - Store language codes and locale identifiers, not localized names
3. **Use middleware** - Set application locale based on user preferences or request headers
4. **Consider fallbacks** - Have a default language/locale for users who haven't set preferences
5. **Respect regional differences** - Use full locales when regional formatting matters
6. **Cache language lists** - Cache the list of available languages for dropdowns
7. **Test with RTL languages** - If supporting Arabic or Hebrew, test right-to-left layouts

## Related Resources

- [Symfony Intl Languages Component](https://symfony.com/doc/current/components/intl.html)
- [Symfony Intl Locales Component](https://symfony.com/doc/current/components/intl.html)
- [ISO 639-1 Language Codes](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)
- [Validation Rules](/intl/casts-and-rules)
