# Cryptography Research and Technical Foundations

## 1. Objective

Explain the cryptographic algorithms, key derivation functions, encryption architectures and security principles used throughout CryptoLab.

The project uses established cryptographic implementations rather than developing custom encryption algorithms.

## 2. Cryptographic Technologies

| Technology | Classification | Application |
|---|---|---|
| AES-256 | Symmetric block cipher | VeraCrypt and LUKS2 |
| XTS | Storage encryption mode | Device and container encryption |
| Argon2id | Memory-hard key derivation | Password protection |
| PBKDF2 | Password-based key derivation | Alternative volume configuration |
| HMAC-SHA-512 | Keyed hash construction | PBKDF2 pseudorandom function |
| ChaCha20 | Symmetric stream cipher | age file encryption |
| Poly1305 | Message authentication | Ciphertext integrity |
| X25519 | Elliptic-curve key agreement | Recipient-based file encryption |
| scrypt | Memory-hard key derivation | age passphrase encryption |
| SHA-256 | Cryptographic hash function | File integrity verification |
| OpenPGP | Cryptographic message format | GnuPG encryption and signatures |

## 3. Symmetric Cryptography

Symmetric encryption uses secret key material to encrypt and decrypt information.

Examples include AES and ChaCha20.

### Advantages

- High performance.
- Suitable for large files and storage devices.
- Efficient encryption and decryption.

### Limitations

- Secret keys must remain protected.
- Shared credentials require secure distribution.
- Compromised keys can expose protected information.

## 4. AES-256

The Advanced Encryption Standard is a symmetric block cipher standardized by NIST.

AES supports three key sizes:

| Variant | Key size | Encryption rounds |
|---|---|---|
| AES-128 | 128 bits | 10 |
| AES-192 | 192 bits | 12 |
| AES-256 | 256 bits | 14 |

All three variants use a 128-bit block size.

AES-256 was selected for device encryption because it is widely supported, extensively studied and efficient on modern hardware.

A 256-bit key does not mean that a password protecting that key automatically possesses 256 bits of entropy.

## 5. XTS Storage Encryption

XTS is a block cipher mode designed for storage encryption.

AES-256-XTS uses two independent 256-bit AES keys:

- One key encrypts the data.
- One key contributes to the position-dependent tweak.

The combined key material is 512 bits. This does not establish an equivalent 512-bit security strength.

The tweak ensures that identical plaintext blocks at different storage positions do not necessarily produce identical ciphertext.

### Advantages

- Supports random-access storage.
- Avoids conventional ciphertext expansion.
- Suitable for sector-based encryption.

### Limitations

- Does not provide independent authentication of encrypted sectors.
- Cannot inherently prevent ciphertext modification.
- Does not protect against deletion or rollback of encrypted data.

## 6. VeraCrypt Encryption Architecture

```text
User Passphrase + Salt + Configured Parameters
                      |
                      v
            Key Derivation Function
                      |
                      v
             Header Encryption Keys
                      |
                      v
              Decrypted Header
                      |
                      v
             Master Encryption Keys
                      |
                      v
              AES-XTS Data Access
```

The user's passphrase is not directly used as the storage encryption key.

Instead, the key derivation function produces key material used to unlock the protected volume header.

The header provides access to the independently generated master encryption keys.

This architecture allows a password to be changed without re-encrypting every storage sector.

## 7. Password-Based Key Derivation

Human-readable passwords are generally less unpredictable than randomly generated cryptographic keys.

A password-based key derivation function transforms authentication credentials into cryptographic key material while increasing the cost of password guessing.

### Salt

A salt is a random, non-secret value incorporated into key derivation.

Salts help prevent:

- Reuse of precomputed password tables.
- Identical derived keys across independently salted volumes.
- Efficient reuse of password computations across multiple targets.

Salts do not increase the inherent entropy of a user's password.

## 8. Argon2id

Argon2 is a password hashing and key derivation algorithm.

Its primary variants are:

| Variant | Design characteristic |
|---|---|
| Argon2d | Data-dependent memory access |
| Argon2i | Data-independent memory access |
| Argon2id | Hybrid memory access |

Argon2id combines properties of Argon2i and Argon2d.

Its configurable parameters include:

- Memory consumption.
- Computational passes.
- Parallelism.
- Output length.

Memory-hard key derivation increases the resource requirements associated with large-scale password guessing.

It does not make weak passwords secure or eliminate hardware-accelerated attacks.

Argon2id was selected for the recommended volume configuration because of its memory-hard design.

## 9. PBKDF2 and HMAC-SHA-512

PBKDF2 derives cryptographic key material by repeatedly evaluating a pseudorandom function.

HMAC-SHA-512 is one possible pseudorandom function used by PBKDF2.

PBKDF2 increases computational costs through repeated iterations but does not impose the same configurable memory-hard requirements as Argon2id.

PBKDF2 remains an established key derivation technology when appropriately configured.

An implementation-specific error involving PBKDF2 parameters does not demonstrate a cryptographic weakness in PBKDF2 or SHA-512.

## 10. Passphrase Entropy

Entropy describes uncertainty associated with a randomly generated secret.

For uniformly selected independent words:

```text
Entropy = Number of words × log2(Wordlist size)
```

Using a 7,776-word Diceware list:

| Random words | Approximate entropy |
|---|---|
| 4 | 51.7 bits |
| 6 | 77.5 bits |
| 8 | 103.4 bits |
| 10 | 129.2 bits |

These estimates assume independent, uniformly random word selection.

Human-selected words, quotations and predictable substitutions do not necessarily provide equivalent entropy.

### Offline Password Guessing

An attacker possessing an encrypted volume can potentially evaluate password guesses offline.

Unlike an online authentication service, encrypted storage cannot reliably enforce account lockouts against an attacker working with a copied volume.

Resistance depends on both password entropy and key derivation costs.

## 11. Asymmetric Cryptography

Asymmetric cryptography uses related public and private keys.

The public key can be distributed. The private key must remain protected.

Public-key encryption enables information to be encrypted for a recipient without exchanging a shared secret beforehand.

### X25519

X25519 is an elliptic-curve Diffie-Hellman key agreement function based on Curve25519.

It enables parties to establish shared cryptographic key material.

X25519 is not itself a general-purpose file encryption algorithm. In age, it contributes to protecting the randomly generated file encryption key.

## 12. Hybrid Encryption

Hybrid encryption combines asymmetric cryptography with efficient symmetric encryption.

```text
Recipient Public Key
         |
         v
Public-Key Agreement / Key Protection
         |
         v
Protected Random File Key
         |
         v
Symmetric Authenticated Encryption
         |
         v
Encrypted File
```

The recipient uses the corresponding private key to recover the file key and decrypt the payload.

This approach combines convenient public-key distribution with efficient bulk encryption.

## 13. ChaCha20-Poly1305

ChaCha20 is a symmetric stream cipher.

Poly1305 is a message authentication algorithm.

Together, they provide authenticated encryption with associated data, commonly abbreviated AEAD.

Authenticated encryption provides:

- Confidentiality.
- Integrity verification.
- Authentication of ciphertext relative to the encryption key.

Unauthorized changes to protected ciphertext should cause authentication to fail.

This does not independently establish the identity of the human sender.

Age uses ChaCha20-Poly1305 in its encryption format.

## 14. scrypt

scrypt is a memory-hard password-based key derivation function.

Age uses scrypt for passphrase-based encryption.

Its purpose is to increase the computational and memory requirements associated with password guessing.

Strong randomly generated passphrases remain necessary.

## 15. Cryptographic Hash Functions

A cryptographic hash function transforms input information into a fixed-length digest.

SHA-256 produces a 256-bit digest.

Common applications include:

- File integrity verification.
- Backup validation.
- Digital signatures.
- Cryptographic protocol construction.

Example:

```bash
sha256sum original.txt recovered.txt
```

Matching hashes provide strong evidence that the compared file contents are identical.

An ordinary hash does not provide confidentiality or establish authenticity against an adversary who can replace both the file and its recorded hash.

## 16. Digital Signatures

Digital signatures provide a mechanism for verifying integrity and authenticity relative to a signing key.

```text
Original Data + Private Signing Key
                 |
                 v
           Digital Signature

Original Data + Signature + Public Key
                 |
                 v
         Signature Verification
```

A valid signature establishes that the signature corresponds to the signed data and the associated signing key.

The verifier must independently establish trust in the public key to associate it with the intended signer.

Digital signatures do not encrypt information.

## 17. Encryption Layer Comparison

| Property | AES-XTS volume | age encrypted file | Digital signature |
|---|---|---|---|
| Confidentiality | Yes | Yes | No |
| Ciphertext authentication | No, not inherently | Yes | Not applicable |
| Protects entire filesystem | Yes | No | No |
| Portable protection for copied files | No, if copied as plaintext | Yes | No |
| Sender identity verification | No | No, not by itself | Yes, with a trusted public key |

Different technologies address different security requirements.

## 18. Nested Encryption

Nested encryption establishes multiple independently authenticated storage boundaries.

```text
Outer Encrypted Volume
└── Independently Encrypted Container
    └── Authenticated Encrypted File
```

Potential benefits include:

- Independent access control.
- Reduced exposure when only the outer volume is unlocked.
- Separate credentials for selected information.
- Additional protection for files transferred outside the encrypted device.

Nested encryption does not automatically multiply the effective security strength of the underlying algorithms.

Its value primarily comes from independent keys, separate authentication boundaries and different protection properties.

## 19. Confidentiality, Integrity and Availability

### Confidentiality

Prevents unauthorized disclosure of information.

Examples include AES-XTS and ChaCha20 encryption.

### Integrity

Provides mechanisms for identifying unauthorized or accidental modification.

Examples include authenticated encryption, trusted hashes and digital signatures.

### Availability

Ensures authorized users can access information when needed.

Examples include verified encrypted backups, protected recovery credentials and tested restoration procedures.

Encryption without effective recovery planning can negatively affect availability.

## 20. Threat Model and Security Boundaries

The primary threat model assumes an unauthorized individual obtains a locked storage device.

The selected encryption technologies are intended to protect stored information from unauthorized disclosure.

The following remain outside or only partially within that protection boundary:

- Compromised host computers.
- Malware.
- Credential theft.
- Unlocked encrypted volumes.
- Memory extraction.
- Historical plaintext on flash storage.
- Physical destruction.
- Ciphertext modification and rollback.
- Loss of required authentication credentials.

Security claims must remain consistent with the actual threat model.

## 21. Flash-Memory Limitations

SSDs and microSD cards use internal controllers that may relocate data between physical storage cells.

Wear leveling and remapped sectors can prevent software from directly accessing every physical location that previously contained information.

Consequently:

- Ordinary deletion is not guaranteed sanitization.
- Full formatting does not guarantee erasure of inaccessible cells.
- Encryption should ideally be established before sensitive information is written.
- Recovery and disposal procedures must account for storage technology.

## 22. Recommended Cryptographic Configuration

| Requirement | Recommended technology |
|---|---|
| Cross-platform portable storage | VeraCrypt |
| Linux encrypted backup storage | LUKS2 |
| Storage encryption | AES-256-XTS |
| Volume password derivation | Argon2id |
| Volume passphrase | Eight independently randomized Diceware words |
| Selective additional protection | Independent VeraCrypt container |
| Authenticated file encryption | age |
| Passphrase-based file encryption | age with scrypt |
| Recipient-based file encryption | age with X25519 identities |
| Digital signatures | GnuPG / OpenPGP |
| File comparison | SHA-256 |

These recommendations assume compatible implementations, strong credential management and a trusted operating environment.

## 23. Implementation and Testing Status

The practical project validated the following on Linux:

- LUKS2 encrypted backup creation and reopening.
- Backup migration and file comparison.
- VeraCrypt whole-device encryption using AES and Argon2.
- Nested VeraCrypt container creation and mounting.
- Symmetric age encryption and decryption.
- Recipient-based age encryption and decryption.
- Directory encryption and recovery.
- SHA-256 comparison.
- Rejection of modified age ciphertext.
- Safe volume unmounting and physical device removal.

The following are additional procedures requiring independent validation:

- Windows interoperability.
- VeraCrypt password changes.
- External header backup and restoration.
- Comprehensive forensic recovery testing.
- Complete encrypted disk image restoration.
- GnuPG file encryption and digital signatures.
- Passphrase-protected age identity recovery.

Successful functional testing does not constitute a formal cryptographic audit.

## 24. Technical References

### Cryptographic Standards

- [AES — NIST FIPS 197](https://csrc.nist.gov/pubs/fips/197/final)
- [XTS-AES — NIST SP 800-38E](https://csrc.nist.gov/pubs/sp/800/38/e/final)
- [Argon2 — RFC 9106](https://www.rfc-editor.org/rfc/rfc9106.html)
- [PBKDF2 — RFC 8018](https://www.rfc-editor.org/rfc/rfc8018.html)
- [ChaCha20-Poly1305 — RFC 8439](https://www.rfc-editor.org/rfc/rfc8439.html)
- [X25519 — RFC 7748](https://www.rfc-editor.org/rfc/rfc7748.html)
- [scrypt — RFC 7914](https://www.rfc-editor.org/rfc/rfc7914.html)
- [OpenPGP — RFC 9580](https://www.rfc-editor.org/rfc/rfc9580.html)
- [Secure Hash Standard — NIST FIPS 180-4](https://csrc.nist.gov/pubs/fips/180-4/upd1/final)

### Implementation Documentation

- [VeraCrypt](https://veracrypt.io/en/Documentation.html)
- [Cryptsetup](https://gitlab.com/cryptsetup/cryptsetup)
- [age](https://github.com/FiloSottile/age)
- [age Format Specification](https://age-encryption.org/v1)
- [GnuPG](https://gnupg.org/documentation/)
- [EFF Diceware](https://www.eff.org/dice)

---

[Return to README](../README.md)
