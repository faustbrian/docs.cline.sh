---
title: Use Cases
description: Practical use cases and examples for Shamir's Secret Sharing in real-world applications.
---

Real-world examples demonstrating how to use Shamir's Secret Sharing to solve common security challenges.

## Master Key Protection

Split a master encryption key across multiple custodians to prevent single points of failure.

### Scenario

You have a master key that encrypts all sensitive data. If this key is compromised or lost, your entire system is at risk. By splitting it using Shamir's scheme, you ensure that:
- No single person can decrypt data alone
- The key can be recovered if some custodians are unavailable
- Compromise of individual shares reveals nothing

### Implementation

```php
use Cline\Shamir\Shamir;

// Generate master encryption key
$masterKey = random_bytes(32); // 256-bit AES key

// Split into 5 shares, requiring 3 to reconstruct - static API
$shares = Shamir::split($masterKey, threshold: 3, shares: 5);

// Or fluent API for more expressive configuration
$shares = Shamir::for($masterKey)->threshold(3)->shares(5)->split();

// Distribute to key custodians
$custodians = [
    'CEO' => $shares->get(1),
    'CTO' => $shares->get(2),
    'CFO' => $shares->get(3),
    'Security Officer' => $shares->get(4),
    'Compliance Officer' => $shares->get(5),
];

foreach ($custodians as $role => $share) {
    // Store securely (HSM, encrypted file, password manager)
    storeShareSecurely($role, $share->toString());
}

// To reconstruct the key, collect any 3 shares
$collectedShares = [
    loadShareFromStorage('CEO'),
    loadShareFromStorage('CTO'),
    loadShareFromStorage('CFO'),
];

// Static API
$reconstructedKey = Shamir::combine($collectedShares);

// Or fluent API
$reconstructedKey = Shamir::from($collectedShares)->combine();

// Use $reconstructedKey to decrypt data
```

### Best Practices

1. **Geographic Distribution** - Store shares in different physical locations
2. **Role Separation** - Distribute to people with different responsibilities
3. **Secure Storage** - Use HSMs, encrypted containers, or secure vaults
4. **Access Logging** - Log all share access and reconstruction attempts
5. **Regular Rotation** - Periodically re-split with new shares

## Backup Encryption

Require multiple parties to decrypt backup archives.

### Scenario

Your application creates encrypted backups. You want to ensure that:
- Backups can only be restored with approval from multiple parties
- A single compromised backup key doesn't expose all historical data
- Backups remain accessible even if some key holders are unavailable

### Implementation

```php
use Cline\Shamir\Shamir;

class BackupManager
{
    public function createBackup(string $data): array
    {
        // Generate unique encryption key for this backup
        $encryptionKey = random_bytes(32);

        // Encrypt the backup
        $encryptedData = sodium_crypto_secretbox(
            $data,
            $nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES),
            $encryptionKey
        );

        // Split the encryption key - fluent API
        $shares = Shamir::for($encryptionKey)->threshold(2)->shares(3)->split();

        return [
            'encrypted_data' => base64_encode($nonce . $encryptedData),
            'key_shares' => [
                'primary_admin' => $shares->get(1)->toString(),
                'backup_admin' => $shares->get(2)->toString(),
                'disaster_recovery' => $shares->get(3)->toString(),
            ],
        ];
    }

    public function restoreBackup(string $encryptedData, array $keyShares): string
    {
        // Require at least 2 shares
        if (count($keyShares) < 2) {
            throw new \RuntimeException('Need 2 administrators to restore backup');
        }

        // Reconstruct encryption key - fluent API
        $encryptionKey = Shamir::from($keyShares)->combine();

        // Decrypt backup
        $decoded = base64_decode($encryptedData);
        $nonce = substr($decoded, 0, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
        $ciphertext = substr($decoded, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);

        $data = sodium_crypto_secretbox_open($ciphertext, $nonce, $encryptionKey);

        if ($data === false) {
            throw new \RuntimeException('Backup decryption failed');
        }

        return $data;
    }
}
```

## Cryptocurrency Wallet Protection

Multi-signature-like key management for crypto wallets.

### Scenario

You're managing a cryptocurrency wallet for a DAO or organization. Requirements:
- Multiple signers must approve transactions
- Wallet remains accessible if some signers are unavailable
- Private key is never stored in a single location

### Implementation

```php
use Cline\Shamir\Shamir;

class WalletManager
{
    public function createMultiSigWallet(int $requiredSigners, int $totalSigners): array
    {
        // Generate wallet private key
        $privateKey = generatePrivateKey(); // 32 bytes

        // Split into shares - fluent API
        $shares = Shamir::for($privateKey)
            ->threshold($requiredSigners)
            ->shares($totalSigners)
            ->split();

        return [
            'public_address' => derivePublicAddress($privateKey),
            'shares' => $shares->forDistribution(),
        ];
    }

    public function signTransaction(array $signerShares, string $transaction): string
    {
        // Reconstruct private key from collected shares - fluent API
        $privateKey = Shamir::from($signerShares)->combine();

        // Sign transaction
        $signature = signTransaction($transaction, $privateKey);

        // Zero out private key from memory
        sodium_memzero($privateKey);

        return $signature;
    }
}

// Usage
$wallet = new WalletManager();

// Create 3-of-5 multisig wallet
$setup = $wallet->createMultiSigWallet(requiredSigners: 3, totalSigners: 5);

// Distribute shares to board members
distributeShares($setup['shares']);

// To sign a transaction, collect 3 shares
$signature = $wallet->signTransaction(
    signerShares: [
        loadShare('member_1'),
        loadShare('member_2'),
        loadShare('member_3'),
    ],
    transaction: $transactionData
);
```

## Vault Seal/Unseal Mechanism

Implement a HashiCorp Vault-style seal/unseal mechanism.

### Scenario

Your application has a "sealed" state where all encrypted data is inaccessible. To "unseal" (make data accessible), a quorum of operators must provide their key shares.

### Implementation

```php
use Cline\Shamir\Shamir;

class VaultManager
{
    private ?string $unsealedKey = null;

    public function initialize(int $threshold, int $totalKeys): array
    {
        // Generate master encryption key
        $masterKey = random_bytes(32);

        // Split into shares - fluent API
        $shares = Shamir::for($masterKey)->threshold($threshold)->shares($totalKeys)->split();

        // Encrypt and store master key
        $this->storeSealedKey($masterKey);

        // Return shares for distribution
        return $shares->forDistribution();
    }

    public function submitUnsealKey(string $shareString): array
    {
        $share = Share::fromString($shareString);
        $threshold = $share->getThreshold();

        // Store share temporarily
        $this->storeUnsealShare($share);

        // Get all submitted shares
        $shares = $this->getSubmittedShares();

        if (count($shares) >= $threshold) {
            // Attempt unseal - fluent API
            try {
                $this->unsealedKey = Shamir::from($shares)->combine();
                $this->clearSubmittedShares();

                return [
                    'sealed' => false,
                    'progress' => count($shares),
                    'threshold' => $threshold,
                ];
            } catch (\Exception $e) {
                return [
                    'error' => 'Invalid share combination',
                    'sealed' => true,
                ];
            }
        }

        return [
            'sealed' => true,
            'progress' => count($shares),
            'threshold' => $threshold,
        ];
    }

    public function seal(): void
    {
        // Zero out unsealed key
        if ($this->unsealedKey !== null) {
            sodium_memzero($this->unsealedKey);
            $this->unsealedKey = null;
        }

        $this->clearSubmittedShares();
    }

    public function isSealed(): bool
    {
        return $this->unsealedKey === null;
    }
}
```

## Compliance & Dual Control

Enforce dual-control requirements for sensitive operations.

### Scenario

Regulatory compliance requires two authorized personnel to approve certain operations (e.g., wire transfers, data exports, system changes).

### Implementation

```php
use Cline\Shamir\Shamir;

class DualControlSystem
{
    public function protectSensitiveOperation(callable $operation): string
    {
        // Generate one-time operation key
        $operationKey = random_bytes(32);

        // Split into 2 shares (threshold = 2, shares = 2) - fluent API
        $shares = Shamir::for($operationKey)->threshold(2)->shares(2)->split();

        // Encrypt operation data with operation key
        $encryptedOp = $this->encryptOperation($operation, $operationKey);

        // Return shares and encrypted operation
        return [
            'operation_id' => uniqid('op_'),
            'encrypted_operation' => $encryptedOp,
            'approver_1_share' => $shares->get(1)->toString(),
            'approver_2_share' => $shares->get(2)->toString(),
        ];
    }

    public function executeWithApproval(
        string $encryptedOperation,
        string $share1,
        string $share2
    ): mixed {
        // Reconstruct operation key - fluent API
        $operationKey = Shamir::from([
            Share::fromString($share1),
            Share::fromString($share2),
        ])->combine();

        // Decrypt and execute operation
        $operation = $this->decryptOperation($encryptedOperation, $operationKey);

        // Log dual approval
        $this->logApprovals($share1, $share2);

        // Execute
        return $operation();
    }
}

// Usage
$dualControl = new DualControlSystem();

// Protect a wire transfer
$protected = $dualControl->protectSensitiveOperation(function() use ($amount, $recipient) {
    return $this->bank->wireTransfer($amount, $recipient);
});

// Send shares to two different approvers
sendToApprover1($protected['approver_1_share']);
sendToApprover2($protected['approver_2_share']);

// When both approve, execute
$result = $dualControl->executeWithApproval(
    $protected['encrypted_operation'],
    $approver1Share,
    $approver2Share
);
```

## Disaster Recovery Keys

Distribute recovery keys to trusted parties for emergency access.

### Implementation

```php
use Cline\Shamir\Shamir;

class DisasterRecoveryManager
{
    public function setupRecoveryKeys(): array
    {
        // Generate recovery master key
        $recoveryKey = random_bytes(32);

        // Split into 7 shares, requiring 3 for recovery - fluent API
        $shares = Shamir::for($recoveryKey)->threshold(3)->shares(7)->split();

        // Distribute to trusted parties
        $distribution = [
            'Board Member 1' => $shares->get(1),
            'Board Member 2' => $shares->get(2),
            'Board Member 3' => $shares->get(3),
            'External Auditor' => $shares->get(4),
            'Legal Counsel' => $shares->get(5),
            'Bank Safety Deposit' => $shares->get(6),
            'Secure Backup Location' => $shares->get(7),
        ];

        // Store encrypted recovery procedures with recovery key
        $this->storeRecoveryProcedures($recoveryKey);

        return $distribution;
    }

    public function executeRecovery(array $shares): array
    {
        // Require at least 3 shares
        if (count($shares) < 3) {
            throw new \RuntimeException('Need 3 recovery key holders for disaster recovery');
        }

        // Reconstruct recovery key - fluent API
        $recoveryKey = Shamir::from($shares)->combine();

        // Decrypt recovery procedures
        $procedures = $this->decryptRecoveryProcedures($recoveryKey);

        // Execute recovery steps
        return $this->executeRecoverySteps($procedures);
    }
}
```

## Key Points

1. **Threshold Selection**: Choose threshold based on availability vs security trade-off
2. **Share Distribution**: Never store all shares in the same location or system
3. **Secure Storage**: Protect individual shares as if they were the secret itself
4. **Access Control**: Log and monitor all share access and combination attempts
5. **Regular Testing**: Periodically verify that shares can successfully reconstruct secrets
6. **Rotation**: Consider rotating shares periodically for high-security scenarios
