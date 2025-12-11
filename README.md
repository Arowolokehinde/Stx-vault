# Stx-analytics - Time-Locked Asset Vault

A Clarity 4 smart contract demonstrating advanced features including time-based vault locks, passkey authentication, and contract verification.

## Overview

This contract implements a secure vault system that allows users to create time-locked STX token vaults. It showcases multiple Clarity 4 features including:

- **Time-based logic** using `stacks-block-time`
- **Passkey authentication** using `secp256r1-verify`
- **ASCII conversion** using `to-ascii?`
- **Contract verification** using `contract-hash?`

## Features

### 1. Time-Locked Vaults
Create vaults that lock STX tokens for a specified duration. Funds can only be released after the unlock time has passed.

### 2. Passkey Authentication (secp256r1)
Users can register hardware wallet public keys and release vaults using biometric signatures.

### 3. Contract Verification
Verify and whitelist external contracts before interaction using on-chain hash verification.

### 4. External Contract Callbacks
Safely call external contracts when releasing vaults.

## Contract Functions

### Public Functions

#### `register-passkey`
Register a secp256r1 public key for passkey authentication.

```clarity
(register-passkey (public-key (buff 33)))
```

**Parameters:**
- `public-key`: 33-byte secp256r1 public key

**Returns:** `(ok true)` on success

---

#### `create-vault`
Create a time-locked vault.

```clarity
(create-vault (amount uint) (lock-duration uint))
```

**Parameters:**
- `amount`: Amount of STX to lock (in microSTX)
- `lock-duration`: Lock duration in seconds (minimum 86400 = 1 day)

**Returns:** `(ok vault-id)` on success

**Example:**
```clarity
;; Lock 1000 STX for 7 days
(create-vault u1000000000 u604800)
```

---

### Read-Only Functions

All functions are currently commented out. Uncomment them to enable:

#### `get-vault-info`
Get detailed vault information with ASCII status.

```clarity
(get-vault-info (vault-id uint))
```

**Returns:**
```clarity
{
  vault-data: {...},
  is-unlocked: bool,
  current-time: uint,
  time-remaining: uint,
  status-message: (string-ascii 5)
}
```

---

#### `get-current-time`
Get the current block timestamp.

```clarity
(get-current-time)
```

**Returns:** `(ok stacks-block-time)`

---

#### `get-vault-count`
Get total number of vaults created.

```clarity
(get-vault-count)
```

**Returns:** `(ok uint)`

---

#### `has-passkey`
Check if a user has registered a passkey.

```clarity
(has-passkey (user principal))
```

**Returns:** `(ok bool)`

---

#### `is-contract-verified`
Check if a contract has been verified and whitelisted.

```clarity
(is-contract-verified (contract-principal principal))
```

**Returns:** `(ok bool)`

---

#### `get-contract-hash`
Get the hash of a contract's code body using Clarity 4's `contract-hash?`.

```clarity
(get-contract-hash (contract-principal principal))
```

**Returns:** `(ok (optional (buff 32)))`

---

## Error Codes

| Code | Constant | Description |
|------|----------|-------------|
| u100 | ERR_UNAUTHORIZED | Caller is not authorized |
| u101 | ERR_VAULT_NOT_FOUND | Vault does not exist or already released |
| u102 | ERR_STILL_LOCKED | Vault unlock time has not passed |
| u103 | ERR_INVALID_AMOUNT | Invalid amount or duration |
| u104 | ERR_INVALID_CONTRACT | Contract verification failed |
| u105 | ERR_INVALID_SIGNATURE | Passkey signature verification failed |
| u106 | ERR_ASSET_RESTRICTION_FAILED | Asset restriction check failed |

## Clarity 4 Features Demonstrated

### 1. `stacks-block-time`
Get the timestamp of the current block for time-based logic.

**Usage in contract:**
- Used in `create-vault` to calculate unlock times
- Used in vault release functions to verify time has passed

**Example:**
```clarity
(define-public (create-vault (amount uint) (lock-duration uint))
  (let
    (
      (current-time stacks-block-time)
      (unlock-time (+ current-time lock-duration))
    )
    ;; ... vault creation logic
  )
)
```

### 2. `secp256r1-verify`
Verify secp256r1 signatures for passkey authentication.

**Usage in contract:**
- Used in `release-vault-with-passkey` to verify hardware wallet signatures

**Example:**
```clarity
(secp256r1-verify message-hash signature public-key)
```

### 3. `to-ascii?`
Convert simple values (booleans, principals) to ASCII strings.

**Usage in contract:**
- Used in `get-vault-status-ascii` to create readable status messages

**Example:**
```clarity
(to-ascii? true)  ;; Returns (some "true")
(to-ascii? false) ;; Returns (some "false")
```

### 4. `contract-hash?`
Fetch the hash of another contract's code body.

**Usage in contract:**
- Used in `get-contract-hash` read-only function
- Can be used to verify contracts before interaction

**Example:**
```clarity
(contract-hash? 'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM.some-contract)
```

## Development

### Prerequisites
- [Clarinet](https://github.com/hirosystems/clarinet) v3.11.0 or higher
- Node.js (for testing)

### Installation

1. Clone the repository
2. Install dependencies:
```bash
npm install
```

### Testing

Run contract checks:
```bash
clarinet check
```

Run tests:
```bash
npm test
```

### Deployment

Deploy to Devnet:
```bash
clarinet integrate
```

## Usage Examples

### Creating a Vault

```clarity
;; Create a vault locking 5000 STX for 30 days
(contract-call? .Stx-analytics create-vault u5000000000 u2592000)
```

### Registering a Passkey

```clarity
;; Register a secp256r1 public key (33 bytes compressed)
(contract-call? .Stx-analytics register-passkey 0x02a1b2c3d4e5f6...)
```

### Checking Vault Status

```clarity
;; Get vault info for vault ID 0
(contract-call? .Stx-analytics get-vault-info u0)
```

### Verifying a Contract

```clarity
;; Verify and whitelist a contract
(contract-call? .Stx-analytics verify-contract
  'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM.trusted-contract
  0x1234567890abcdef...)
```

## Security Considerations

1. **Time-based locks**: Ensure lock durations are sufficient for your use case (minimum 1 day)
2. **Passkey security**: Store secp256r1 private keys securely in hardware wallets
3. **Contract verification**: Always verify contract hashes before whitelisting
4. **Input validation**: The contract includes basic validation but additional checks may be needed

## Data Structures

### Vault
```clarity
{
  owner: principal,
  amount: uint,
  unlock-time: uint,
  created-at: uint,
  released: bool
}
```

### User Passkey
```clarity
{
  public-key: (buff 33)
}
```

### Verified Contract
```clarity
{
  verified: bool,
  hash: (buff 32)
}
```

## Constants

- `MIN_LOCK_DURATION`: u86400 (1 day in seconds)
- `CONTRACT_OWNER`: Principal who deployed the contract

## License

MIT

## Contributing

Contributions are welcome! Please submit issues and pull requests.

## Resources

- [Clarity Documentation](https://docs.stacks.co/clarity)
- [Clarity 4 Features](https://docs.stacks.co/clarity/new-features)
- [Clarinet Documentation](https://docs.hiro.so/stacks/clarinet)
- [Stacks Blockchain](https://www.stacks.co)

## Author

Built with Clarity 4 for the Stacks blockchain.
