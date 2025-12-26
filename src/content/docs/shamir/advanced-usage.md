---
title: Advanced Usage
description: Advanced patterns and techniques for using Shamir's Secret Sharing.
---

Advanced usage patterns, integration strategies, and optimization techniques.

## Progressive Secret Reconstruction

Build a system that progressively collects shares until threshold is reached:

```php
use Cline\Shamir\Shamir;
use Cline\Shamir\Share;

class ProgressiveReconstruction
{
    private array $collectedShares = [];
    private int $threshold;

    public function addShare(string $shareString): array
    {
        $share = Share::fromString($shareString);

        // Store threshold from first share
        if (empty($this->collectedShares)) {
            $this->threshold = $share->getThreshold();
        }

        // Validate compatibility
        if ($share->getThreshold() !== $this->threshold) {
            throw new \RuntimeException('Incompatible share threshold');
        }

        // Check for duplicates
        foreach ($this->collectedShares as $existing) {
            if ($existing->getIndex() === $share->getIndex()) {
                throw new \RuntimeException('Duplicate share');
            }
        }

        $this->collectedShares[] = $share;

        return [
            'progress' => count($this->collectedShares),
            'threshold' => $this->threshold,
            'complete' => count($this->collectedShares) >= $this->threshold,
        ];
    }

    public function canReconstruct(): bool
    {
        return count($this->collectedShares) >= $this->threshold;
    }

    public function reconstruct(): ?string
    {
        if (!$this->canReconstruct()) {
            return null;
        }

        // Static API
        return Shamir::combine($this->collectedShares);

        // Or fluent API
        return Shamir::from($this->collectedShares)->combine();
    }

    public function reset(): void
    {
        $this->collectedShares = [];
        $this->threshold = 0;
    }
}

// Usage
$reconstructor = new ProgressiveReconstruction();

// Operators submit shares one by one
$status = $reconstructor->addShare($operatorShare1);
// ['progress' => 1, 'threshold' => 3, 'complete' => false]

$status = $reconstructor->addShare($operatorShare2);
// ['progress' => 2, 'threshold' => 3, 'complete' => false]

$status = $reconstructor->addShare($operatorShare3);
// ['progress' => 3, 'threshold' => 3, 'complete' => true]

if ($reconstructor->canReconstruct()) {
    $secret = $reconstructor->reconstruct();
}
```

## Time-Based Share Activation

Implement shares that become active only after a certain time:

```php
use Cline\Shamir\Shamir;

class TimeLockedShares
{
    public function createTimeLockedShares(
        string $secret,
        int $threshold,
        int $shares,
        \DateTimeInterface $activationTime
    ): array {
        // Create shares - static API
        $secretShares = Shamir::split($secret, $threshold, $shares);

        // Or fluent API
        $secretShares = Shamir::for($secret)
            ->threshold($threshold)
            ->shares($shares)
            ->split();

        $timeLockedShares = [];
        foreach ($secretShares as $share) {
            $timeLockedShares[] = [
                'share' => $share->toString(),
                'activation_time' => $activationTime->format('c'),
                'activated' => false,
            ];
        }

        return $timeLockedShares;
    }

    public function attemptReconstruction(array $timeLockedShares): string
    {
        $now = new \DateTimeImmutable();
        $activeShares = [];

        foreach ($timeLockedShares as $locked) {
            $activationTime = new \DateTimeImmutable($locked['activation_time']);

            if ($now < $activationTime) {
                throw new \RuntimeException(
                    "Share not yet active. Available at {$locked['activation_time']}"
                );
            }

            $activeShares[] = Share::fromString($locked['share']);
        }

        // Static API
        return Shamir::combine($activeShares);

        // Or fluent API
        return Shamir::from($activeShares)->combine();
    }
}

// Create shares that activate in 24 hours
$timeLocked = new TimeLockedShares();
$shares = $timeLocked->createTimeLockedShares(
    secret: $masterKey,
    threshold: 3,
    shares: 5,
    activationTime: new \DateTimeImmutable('+24 hours')
);

// Will fail if attempted before activation time
try {
    $secret = $timeLocked->attemptReconstruction($shares);
} catch (\RuntimeException $e) {
    echo "Shares not yet active\n";
}
```

## Hierarchical Secret Sharing

Create a hierarchy where different groups have different access levels:

```php
use Cline\Shamir\Shamir;

class HierarchicalSharing
{
    public function createHierarchy(string $secret): array
    {
        // Level 1: Executives (2 of 3 needed) - static API
        $execShares = Shamir::split($secret, threshold: 2, shares: 3);

        // Or fluent API for more expressive configuration
        $execShares = Shamir::for($secret)->threshold(2)->shares(3)->split();

        // Level 2: Management (3 of 5 needed)
        $mgmtShares = Shamir::for($secret)->threshold(3)->shares(5)->split();

        // Level 3: Board (5 of 7 needed)
        $boardShares = Shamir::for($secret)->threshold(5)->shares(7)->split();

        return [
            'executives' => $execShares,
            'management' => $mgmtShares,
            'board' => $boardShares,
        ];
    }
}

// Any group can reconstruct independently
$hierarchy = new HierarchicalSharing();
$levels = $hierarchy->createHierarchy($masterKey);

// Executives can unlock quickly (only need 2) - static API
$secret = Shamir::combine($levels['executives']->take(2));

// Or fluent API
$secret = Shamir::from($levels['executives']->take(2))->combine();

// Management can unlock (need 3)
$secret = Shamir::from($levels['management']->take(3))->combine();

// Board can unlock (need 5)
$secret = Shamir::from($levels['board']->take(5))->combine();
```

## Share Refresh Without Reconstructing

Update shares without reconstructing the secret (advanced):

```php
use Cline\Shamir\Shamir;

class ShareRefreshService
{
    /**
     * Refresh shares by adding a zero-polynomial.
     * This changes all share values while keeping the secret the same.
     */
    public function refreshShares(array $oldShares): array
    {
        // This requires reconstructing the secret
        // For true proactive secret sharing without reconstruction,
        // you'd need a more complex protocol

        // For this implementation, reconstruct and re-split - static API
        $secret = Shamir::combine($oldShares);
        $newShares = Shamir::split(
            $secret,
            threshold: $oldShares[0]->getThreshold(),
            shares: count($oldShares)
        );

        // Or using fluent API
        $secret = Shamir::from($oldShares)->combine();
        $newShares = Shamir::for($secret)
            ->threshold($oldShares[0]->getThreshold())
            ->shares(count($oldShares))
            ->split();

        // Clear secret from memory
        sodium_memzero($secret);

        return $newShares;
    }
}
```

## Integration with Laravel Queue

Process share distribution asynchronously:

```php
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Cline\Shamir\Shamir;

class DistributeSharesJob implements ShouldQueue
{
    use Queueable;

    public function __construct(
        private string $secret,
        private array $recipients,
        private int $threshold,
    ) {}

    public function handle(): void
    {
        // Static API
        $shares = Shamir::split(
            $this->secret,
            $this->threshold,
            count($this->recipients)
        );

        // Or fluent API
        $shares = Shamir::for($this->secret)
            ->threshold($this->threshold)
            ->shares(count($this->recipients))
            ->split();

        foreach ($shares as $i => $share) {
            $recipient = $this->recipients[$i];

            // Send via secure channel
            $this->sendSecurely($recipient, $share);

            // Log distribution
            Log::info('Share distributed', [
                'recipient' => $recipient['id'],
                'share_index' => $share->getIndex(),
            ]);
        }
    }

    private function sendSecurely($recipient, $share): void
    {
        // Encrypt with recipient's public key
        $encrypted = $this->encryptForRecipient($share, $recipient);

        // Send via secure channel
        Mail::to($recipient['email'])->send(
            new ShareDistributionMail($encrypted)
        );
    }
}

// Dispatch job
DistributeSharesJob::dispatch($masterKey, $recipients, 3);
```

## Database Storage Pattern

Store encrypted shares in database:

```php
use Illuminate\Database\Eloquent\Model;
use Cline\Shamir\Share;

class SecretShare extends Model
{
    protected $fillable = ['user_id', 'share_index', 'encrypted_share', 'threshold'];

    protected $casts = [
        'share_index' => 'integer',
        'threshold' => 'integer',
    ];

    public function setShareAttribute(Share $share): void
    {
        $this->attributes['share_index'] = $share->getIndex();
        $this->attributes['threshold'] = $share->getThreshold();
        $this->attributes['encrypted_share'] = encrypt($share->toString());
    }

    public function getShareAttribute(): Share
    {
        $decrypted = decrypt($this->attributes['encrypted_share']);
        return Share::fromString($decrypted);
    }
}

// Store shares - static API
$shares = Shamir::split($secret, 3, 5);

// Or fluent API
$shares = Shamir::for($secret)->threshold(3)->shares(5)->split();

foreach ($shares as $share) {
    SecretShare::create([
        'user_id' => $userId,
        'share' => $share,
    ]);
}

// Reconstruct - static API
$shares = SecretShare::where('user_id', $userId)->get();
$secret = Shamir::combine($shares->pluck('share'));

// Or fluent API
$secret = Shamir::from($shares->pluck('share'))->combine();
```

## Custom Encoding

Use custom encoding for specialized requirements:

```php
use Cline\Shamir\Config;
use Cline\Shamir\Shamir;

// Use hex encoding for compatibility with systems that don't support base64
$config = new Config(
    prime: Config::PRIME_256,
    encoding: 'hex'
);

// Using manager with custom config
$manager = Shamir::withConfig($config);
$shares = $manager->split($secret, 3, 5);

// Or fluent API with custom manager
$shares = $manager->for($secret)->threshold(3)->shares(5)->split();

// Shares will use hex encoding instead of base64
```

## Testing Reconstruction Before Distribution

Verify shares can reconstruct before distributing:

```php
use Cline\Shamir\Shamir;

class ShareVerificationService
{
    public function createAndVerifyShares(string $secret, int $threshold, int $shares): array
    {
        // Create shares - fluent API
        $shareCollection = Shamir::for($secret)
            ->threshold($threshold)
            ->shares($shares)
            ->split();

        // Test reconstruction with exactly threshold shares
        $testReconstructed = Shamir::from($shareCollection->take($threshold))->combine();

        if ($testReconstructed !== $secret) {
            throw new \RuntimeException('Share reconstruction test failed');
        }

        // Test all possible combinations (for small share counts)
        if ($shares <= 7) {
            $this->verifyAllCombinations($secret, $shareCollection, $threshold);
        }

        return $shareCollection->toArray();
    }

    private function verifyAllCombinations($secret, $shares, $threshold): void
    {
        $allShares = $shares->toArray();
        $combinations = $this->generateCombinations($allShares, $threshold);

        foreach ($combinations as $combo) {
            $reconstructed = Shamir::combine($combo);
            if ($reconstructed !== $secret) {
                throw new \RuntimeException('Combination failed verification');
            }
        }
    }
}
```

## Audit Trail

Implement comprehensive audit logging:

```php
class AuditedShamirService
{
    public function split(string $secret, int $threshold, int $shares): ShareCollection
    {
        $this->log('share_split_initiated', [
            'threshold' => $threshold,
            'total_shares' => $shares,
        ]);

        // Using fluent API for better readability
        $shareCollection = Shamir::for($secret)
            ->threshold($threshold)
            ->shares($shares)
            ->split();

        $this->log('share_split_completed', [
            'share_count' => $shareCollection->count(),
        ]);

        return $shareCollection;
    }

    public function combine(array $shares): string
    {
        $this->log('reconstruction_initiated', [
            'share_count' => count($shares),
            'share_indices' => array_map(fn($s) => $s->getIndex(), $shares),
        ]);

        try {
            // Using fluent API
            $secret = Shamir::from($shares)->combine();

            $this->log('reconstruction_successful', [
                'share_count' => count($shares),
            ]);

            return $secret;
        } catch (\Exception $e) {
            $this->log('reconstruction_failed', [
                'error' => $e->getMessage(),
                'share_count' => count($shares),
            ]);

            throw $e;
        }
    }

    private function log(string $event, array $context): void
    {
        Log::channel('security')->info($event, array_merge($context, [
            'user_id' => auth()->id(),
            'ip' => request()->ip(),
            'timestamp' => now(),
        ]));
    }
}
```

## Performance Optimization

For large-scale operations:

```php
class OptimizedShamirService
{
    private array $shareCache = [];

    public function splitWithCaching(string $secretId, string $secret): ShareCollection
    {
        // Check cache first
        if (isset($this->shareCache[$secretId])) {
            return $this->shareCache[$secretId];
        }

        // Generate shares - fluent API
        $shares = Shamir::for($secret)->threshold(3)->shares(5)->split();

        // Cache result
        $this->shareCache[$secretId] = $shares;

        return $shares;
    }

    public function batchSplit(array $secrets): array
    {
        $results = [];

        foreach ($secrets as $id => $secret) {
            // Fluent API for expressive batch operations
            $results[$id] = Shamir::for($secret)->threshold(3)->shares(5)->split();
        }

        return $results;
    }
}
```

## Best Practices Summary

1. **Always verify shares** before distribution
2. **Log all operations** for audit trail
3. **Clear secrets** from memory after use
4. **Test reconstruction** in disaster recovery drills
5. **Version shares** for rotation tracking
6. **Encrypt shares** at rest and in transit
7. **Rate limit** reconstruction attempts
8. **Monitor** for unusual access patterns
