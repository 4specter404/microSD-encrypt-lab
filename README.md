# CryptoLab — Secure Portable Storage

## Overview

A practical technical guide for implementing, managing, and testing encrypted portable storage on Linux and Windows.

The project focuses on protecting data against unauthorized physical access, offline password attacks, disk imaging, forensic recovery, and unauthorized modification.

## Supported Encryption Methods

| Method | Technology |
|---|---|
| Full-device encryption | VeraCrypt |
| Selective storage encryption | VeraCrypt containers |
| Nested encryption | Independent VeraCrypt volumes |
| Symmetric file encryption | age / GnuPG |
| Asymmetric file encryption | age / GnuPG |
| Encrypted backup storage | LUKS2 |

## Cryptographic Technologies

- AES-256-XTS
- Argon2id
- PBKDF2-HMAC-SHA-512
- ChaCha20-Poly1305
- X25519
- scrypt
- SHA-256
- OpenPGP

## Documentation

| Guide | Description |
|---|---|
| [01 — Environment Setup](docs/01-environment-setup.md) | Software installation and authenticity verification |
| [02 — Backup and Migration](docs/02-backup-and-migration.md) | Preserve existing data before encryption |
| [03 — Full-Device Encryption](docs/03-full-device-encryption.md) | Encrypt an entire storage device |
| [04 — Nested and Selective Encryption](docs/04-nested-encryption.md) | Protect selected data using independent encrypted volumes |
| [05 — File Encryption](docs/05-file-encryption.md) | Encrypt individual files and directories |
| [06 — Access and Passwords](docs/06-access-and-passwords.md) | Mounting, credential management and transferring ownership |
| [07 — Recovery](docs/07-recovery.md) | Encryption header backups and recovery procedures |
| [08 — Security Testing](docs/08-security-testing.md) | Confidentiality, integrity and forensic resistance testing |
| [09 — Troubleshooting](docs/09-troubleshooting.md) | Common errors and potential solutions |
| [10 — Cleanup](docs/10-cleanup.md) | Removing temporary data and safely disconnecting devices |
| [Cryptography Research](research/cryptography.md) | Technical concepts and reference material |

## Requirements

### Hardware

- Computer running Linux or Windows
- Removable storage device
- Compatible storage reader

### Software

- VeraCrypt
- Cryptsetup (Linux)
- age
- GnuPG

## Security Principles

- Use established cryptographic implementations.
- Generate strong, independent passphrases.
- Verify backups before destructive operations.
- Verify software signatures before installation.
- Never publish private keys or authentication credentials.
- Understand the limitations of encryption and flash-memory sanitization.

## Limitations

Encryption does not guarantee protection against compromised host computers, exposed credentials, unlocked volumes, or previously stored plaintext remaining in inaccessible flash-memory cells.

Advanced and unverified procedures are identified separately throughout the documentation.

## Disclaimer

This repository is intended for educational purposes and authorized data protection. Always verify device identifiers and maintain recoverable backups before performing destructive storage operations.
