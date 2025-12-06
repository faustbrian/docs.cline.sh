---
title: Security Considerations
description: Security best practices and considerations when using Shamir's Secret Sharing.
---

Important security considerations and best practices for using Shamir's Secret Sharing in production systems.

## Information-Theoretic Security

Shamir's Secret Sharing provides **information-theoretic security**, meaning:

- Any combination of shares below the threshold reveals **zero information** about the secret
- This security doesn't rely on computational hardness (unlike RSA, AES)
- Even with unlimited computing power, shares below threshold are useless

### What This Means

```php
// With threshold = 3, shares = 5
$shares = Shamir::split($secret, 3, 5);

// An attacker with ANY 2 shares knows NOTHING about the secret
// All possible secrets are equally likely
$twoShares = [$shares->get(1), $shares->get(2)];
// ^ Completely useless to an attacker
```

## Share Protection

### Individual Share Security

**Critical**: Each share must be protected as if it were the secret itself.

❌ **Don't:**
```php
// Storing shares in plaintext
file_put_contents('/tmp/share1.txt', $share->toString());

// Logging shares
$logger->info('Share created: ' . $share->toString());

// Sending shares unencrypted
$email->send($recipient, $share->toString());
```

✅ **Do:**
```php
// Encrypt shares before storage
$encrypted = encryptShare($share->toString(), $userPassword);
$vault->store($encrypted);

// Use secure channels for distribution
$secureMessaging->sendEncrypted($recipient, $share->toString());

// Store in hardware security modules (HSMs)
$hsm->storeShare($share);
```

### Storage Best Practices

1. **Geographic Distribution** - Store shares in different physical locations
2. **Access Control** - Implement strict access controls on share storage
3. **Audit Logging** - Log all share access and reconstruction attempts
4. **Encryption at Rest** - Encrypt shares even in "secure" storage
5. **No Co-location** - Never store threshold number of shares in same system

## Checksum Validation

Shares include SHA-256 checksums to detect corruption or tampering:

```php
use Cline\Shamir\Exception\InvalidShareException;

try {
    $secret = Shamir::combine($shares);
} catch (InvalidShareException $e) {
    // Share was corrupted or tampered with
    $logger->alert('Share checksum validation failed', [
        'error' => $e->getMessage(),
        'timestamp' => now(),
    ]);
}
```

**Note**: Checksums detect **accidental corruption**, not malicious modification. An attacker with access to modify a share could recompute the checksum.

## Memory Safety

### Clearing Sensitive Data

```php
// After reconstructing a secret, clear it from memory when done
$secret = Shamir::combine($shares);
useSecret($secret);

// Clear secret from memory
sodium_memzero($secret);
```

### Avoiding Logs

```php
// Never log secrets or shares
❌ Log::info('Secret: ' . $secret);
❌ Log::debug('Share: ' . $share->toString());

// Be careful with exception messages
try {
    $secret = Shamir::combine($shares);
} catch (\Exception $e) {
    ✅ Log::error('Secret reconstruction failed', [
        'error_type' => get_class($e),
        // Don't include exception message if it might contain shares
    ]);
}
```

## Threshold Selection

Choose threshold based on security vs availability trade-off:

### Too Low Threshold

```php
// Threshold = 2, Shares = 10
// Risk: Only need to compromise 2 shares
$shares = Shamir::split($secret, threshold: 2, shares: 10);
```

**Problems:**
- Lower security bar
- Easier for attacker to gather enough shares
- Useful for: High availability scenarios

### Too High Threshold

```php
// Threshold = 9, Shares = 10
// Risk: Losing 2 shares makes secret unrecoverable
$shares = Shamir::split($secret, threshold: 9, shares: 10);
```

**Problems:**
- Low fault tolerance
- Secret lost if enough shares unavailable
- Useful for: Maximum security scenarios

### Balanced Approach

```php
// Threshold = 3, Shares = 5
// Good balance: Need 3 to reconstruct, can lose 2
$shares = Shamir::split($secret, threshold: 3, shares: 5);

// For higher security
// Threshold = 5, Shares = 7
$shares = Shamir::split($secret, threshold: 5, shares: 7);
```

## Randomness

The implementation uses PHP's `random_bytes()` for cryptographically secure randomness:

```php
// Polynomial coefficients are generated with random_bytes()
// Ensures unpredictable polynomial construction
```

**Requirements:**
- System must have access to good entropy source
- Never use `rand()` or `mt_rand()` for security purposes
- On Linux, ensure `/dev/urandom` is available

## Share Distribution

### Secure Distribution Channels

```php
// ✅ Good: Separate secure channels for each share
$share1 = $shares->get(1);
$share2 = $shares->get(2);

sendViaEncryptedEmail($recipient1, $share1);
sendViaSMS($recipient2, $share2); // Different channel
deliverInPerson($recipient3, $shares->get(3)); // Physical delivery
```

### Avoid Common Pitfalls

❌ **Don't:**
- Email all shares to same recipient
- Store shares in same database
- Send shares via same communication channel
- Keep shares in same datacenter

✅ **Do:**
- Use different distribution methods
- Geographic distribution
- Multiple authentication factors for access
- Time-delayed distribution for additional security

## Access Control

### Multi-Factor Authentication

```php
class SecureShareAccess
{
    public function retrieveShare(User $user, string $mfaToken): Share
    {
        // Require MFA
        if (!$this->verifyMFA($user, $mfaToken)) {
            throw new UnauthorizedException();
        }

        // Log access
        $this->auditLog->record([
            'user' => $user->id,
            'action' => 'share_access',
            'timestamp' => now(),
            'ip' => request()->ip(),
        ]);

        return $this->loadShare($user);
    }
}
```

### Rate Limiting

```php
class ShareReconstructionService
{
    public function combine(array $shares): string
    {
        // Rate limit reconstruction attempts
        if ($this->tooManyAttempts()) {
            throw new RateLimitException('Too many reconstruction attempts');
        }

        $this->recordAttempt();

        return Shamir::combine($shares);
    }

    private function tooManyAttempts(): bool
    {
        // Allow max 3 attempts per hour
        return Cache::get('reconstruction_attempts') >= 3;
    }
}
```

## Rotation Strategy

Periodically rotate shares for maximum security:

```php
class ShareRotationService
{
    public function rotateShares(string $secret): ShareCollection
    {
        // Generate new shares
        $newShares = Shamir::split($secret, threshold: 3, shares: 5);

        // Distribute new shares
        $this->distributeNewShares($newShares);

        // After confirming receipt, destroy old shares
        $this->revokeOldShares();

        return $newShares;
    }
}

// Rotate quarterly or after suspected compromise
$rotationService->rotateShares($masterKey);
```

## Field Size Considerations

Choose appropriate field size for your secrets:

```php
use Cline\Shamir\Config;

// Default: 256-bit field (recommended for most use cases)
$shares = Shamir::split($secret, 3, 5);

// Larger field for very large secrets
$config = new Config(prime: Config::PRIME_512);
$shares = Shamir::split($largeSecret, 3, 5, $config);
```

**Recommendations:**
- **PRIME_256** (default): Suitable for keys up to ~30 bytes per chunk
- **PRIME_512**: For very large secrets or extra security margin
- Automatic chunking handles secrets of any size

## Production Checklist

Before deploying to production:

- [ ] Shares are encrypted at rest
- [ ] Shares are distributed across different locations/systems
- [ ] Access to shares requires multi-factor authentication
- [ ] All share access is logged and monitored
- [ ] Reconstruction attempts are rate-limited
- [ ] Secrets are cleared from memory after use
- [ ] No secrets or shares are logged
- [ ] Threshold provides acceptable security/availability balance
- [ ] Disaster recovery process is documented and tested
- [ ] Share rotation schedule is established
- [ ] Audit trail is immutable and monitored

## Threat Model

### Protected Against

✅ **Single point of compromise** - Attacker needs threshold shares
✅ **Partial information leakage** - Shares below threshold reveal nothing
✅ **Accidental share loss** - Can reconstruct with remaining shares
✅ **Share corruption** - Checksums detect tampering/corruption

### Not Protected Against

❌ **Threshold shares compromised** - If attacker gets threshold shares, game over
❌ **Malicious share holders** - Threshold malicious insiders can reconstruct
❌ **Side-channel attacks** - Physical access to reconstruction process
❌ **Implementation bugs** - Use audited crypto libraries for encryption

## Additional Hardening

### Combine with Other Security Measures

```php
// Use Shamir + HSM storage
$shares = Shamir::split($masterKey, 3, 5);
foreach ($shares as $share) {
    $hsm->secureStore($share);
}

// Use Shamir + Password-based encryption
$encryptedShares = [];
foreach ($shares as $i => $share) {
    $password = $shareHolders[$i]->password;
    $encryptedShares[] = encryptWithPassword($share, $password);
}

// Use Shamir + Time-locked encryption
// Shares can only be used after specific time
foreach ($shares as $share) {
    $timeLocked = timeLockEncrypt($share, $unlockTime);
    store($timeLocked);
}
```

## Compliance

Shamir's Secret Sharing helps meet various compliance requirements:

- **PCI DSS**: Key management and dual control
- **HIPAA**: Protected health information key management
- **SOC 2**: Access controls and key custody
- **GDPR**: Data protection and encryption key management

Always consult with compliance experts for your specific requirements.
