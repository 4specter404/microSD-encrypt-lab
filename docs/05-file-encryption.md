# 05 — File and Directory Encryption

## Objective

Encrypt individual files and directories independently of their storage location using symmetric and asymmetric cryptography.

Unlike device-level encryption, file-level encryption protects information even when the encrypted file is transferred to another storage device or computer.

## 1. Encryption Methods

| Method | Authentication | Recommended use |
|---|---|---|
| Symmetric | Shared passphrase | Personal files and archives |
| Asymmetric | Public/private key pair | Secure information transfer |
| Digital signatures | Signing and verification keys | Authenticity and integrity verification |

## 2. Cryptographic Technologies

| Technology | Function |
|---|---|
| age | Modern file encryption |
| ChaCha20-Poly1305 | Authenticated encryption |
| X25519 | Public-key agreement |
| scrypt | Passphrase-based key derivation in age |
| GnuPG | OpenPGP encryption and digital signatures |
| SHA-256 | File integrity verification |

## 3. Install Required Software

```bash
sudo apt install age gnupg
```

Verify installation:

```bash
age --version
gpg --version
```

## 4. Establish a Testing Environment

Use non-sensitive information during experimentation.

The following example uses a mounted VeraCrypt volume.

```bash
mkdir -p /media/veracrypt1/FileEncryptionLab
cd /media/veracrypt1/FileEncryptionLab

echo "Confidential test information" > test.txt
```

Verify the actual mount point before creating files.

## 5. Symmetric Encryption Using age

Encrypt an individual file:

```bash
age -p -o test.txt.age test.txt
```

Enter and confirm a strong passphrase.

### Command Parameters

| Parameter | Purpose |
|---|---|
| `-p` | Enable passphrase-based encryption |
| `-o` | Specify the output file |
| `test.txt` | Original plaintext |
| `test.txt.age` | Encrypted output |

The original plaintext file remains unchanged.

### Inspect the Encrypted File

```bash
ls -lh
xxd -l 256 test.txt.age
```

The age format identifier and key derivation metadata remain readable by design. The payload is encrypted.

## 6. Decrypt and Verify Integrity

```bash
age -d -o recovered.txt test.txt.age
```

Enter the correct encryption passphrase.

Compare the original and recovered files:

```bash
sha256sum test.txt recovered.txt
```

Identical SHA-256 hashes confirm that the recovered file matches the original.

A direct comparison can also be performed:

```bash
cmp test.txt recovered.txt
```

No output and a successful exit status indicate that the files are identical.

## 7. Authenticated Encryption and Tampering Detection

Age uses authenticated encryption to detect unauthorized modifications to encrypted information.

Create a disposable copy:

```bash
cp test.txt.age tampered.txt.age
```

Modify one byte:

```bash
python3 - <<'PYCODE'
with open("tampered.txt.age", "r+b") as f:
    f.seek(-1, 2)
    original = f.read(1)
    f.seek(-1, 2)
    f.write(bytes([original[0] ^ 0x01]))
PYCODE
```

This procedure flips one bit in the final byte of a non-empty encrypted file.

Attempt decryption:

```bash
age -d -o tampered-recovered.txt tampered.txt.age
```

Expected result:

```text
age: error: failed to decrypt and authenticate payload chunk
```

The exact error may differ depending on which byte was modified.

A failed authentication check demonstrates that the modified ciphertext was rejected.

Do not trust or use partial output from a failed decryption.

Unlike age, AES-XTS storage encryption does not independently authenticate encrypted sectors.

## 8. Asymmetric Encryption Using age

Asymmetric encryption allows information to be encrypted for a recipient without exchanging a shared password.

| Key | Function |
|---|---|
| Public key | Used to encrypt information for the recipient |
| Private key | Used by the recipient to decrypt information |

### Generate a Practice Key Pair

```bash
mkdir -p ~/CryptoLab/private
chmod 700 ~/CryptoLab/private

age-keygen -o ~/CryptoLab/private/age-test-key.txt

chmod 600 ~/CryptoLab/private/age-test-key.txt
```

Export the public key:

```bash
age-keygen -y ~/CryptoLab/private/age-test-key.txt \
  > ~/CryptoLab/private/age-public.txt
```

Display the public key:

```bash
cat ~/CryptoLab/private/age-public.txt
```

The public key should begin with `age1`.

Never display, publish or distribute the private key.

### Encrypt Using the Public Key

```bash
age -R ~/CryptoLab/private/age-public.txt \
  -o asymmetric.txt.age test.txt
```

No shared passphrase is required.

### Decrypt Using the Private Key

```bash
age -d -i ~/CryptoLab/private/age-test-key.txt \
  -o asymmetric-recovered.txt asymmetric.txt.age
```

Verify the result:

```bash
sha256sum test.txt asymmetric-recovered.txt
```

### Recipient Key Management

For actual information transfers:

1. The recipient generates their own key pair.
2. The recipient retains their private key.
3. The recipient provides their public key to the sender.
4. The sender independently verifies the public key's authenticity.
5. The sender encrypts information using the recipient's public key.
6. The recipient decrypts using their private key.

The sender does not require access to the recipient's private key.

Public-key encryption alone does not establish the identity of the sender.

## 9. Encrypt an Entire Directory

Create sample information:

```bash
mkdir -p FolderTest

echo "Confidential document 1" > FolderTest/document1.txt
echo "Confidential document 2" > FolderTest/document2.txt
```

Package and encrypt the directory directly:

```bash
set -o pipefail

tar -cf - FolderTest | \
  age -R ~/CryptoLab/private/age-public.txt \
  -o FolderTest.tar.age
```

This avoids creating a separate plaintext TAR archive on disk.

For passphrase-based directory encryption, replace the recipient option with `-p`.

### Decrypt and Recover the Directory

```bash
mkdir -p Recovered

age -d -i ~/CryptoLab/private/age-test-key.txt \
  FolderTest.tar.age | tar -xf - -C Recovered
```

Verify the recovered files:

```bash
diff -r FolderTest Recovered/FolderTest
```

No reported differences and a successful exit status indicate that the compared files match.

Only extract archives from trusted sources. If decryption fails, treat extracted files as incomplete or untrusted.

## 10. Protect the Private Key

A native age private key is not necessarily encrypted at rest.

Filesystem permissions provide access control but do not replace encryption.

Protect a private key using an independent passphrase:

```bash
age -p \
  -o ~/CryptoLab/private/age-test-key.txt.age \
  ~/CryptoLab/private/age-test-key.txt
```

Test the protected identity:

```bash
age -d \
  -i ~/CryptoLab/private/age-test-key.txt.age \
  -o protected-key-recovery.txt asymmetric.txt.age
```

Verify that recovery succeeds before removing the original private key.

Maintain a secure backup of the private key. Losing it may permanently prevent decryption.

Previously stored plaintext key material may remain recoverable from the storage medium.

## 11. Optional: GnuPG Symmetric Encryption

The following GnuPG procedures are additional learning exercises and should be independently tested before production use.

Encrypt a file using AES-256:

```bash
gpg --symmetric --cipher-algo AES256 \
  --output test.txt.gpg test.txt
```

Decrypt the file:

```bash
gpg --output gpg-recovered.txt \
  --decrypt test.txt.gpg
```

Verify:

```bash
sha256sum test.txt gpg-recovered.txt
```

## 12. Optional: GnuPG Public-Key Encryption

Generate a key pair:

```bash
gpg --full-generate-key
```

Select suitable signing and encryption algorithms supported by both communicating systems.

List public keys and fingerprints:

```bash
gpg --list-keys --fingerprint
```

Export a public key:

```bash
gpg --armor --export YOUR_FINGERPRINT \
  > public-key.asc
```

Import a recipient's public key:

```bash
gpg --import recipient-public-key.asc
```

Independently verify the recipient's fingerprint.

Encrypt a file:

```bash
gpg --encrypt \
  --recipient RECIPIENT_FINGERPRINT \
  --output recipient-file.gpg test.txt
```

The recipient decrypts using their corresponding private key:

```bash
gpg --output recovered.txt \
  --decrypt recipient-file.gpg
```

Replace fingerprint placeholders with the actual verified fingerprints.

## 13. Optional: Digital Signatures

Digital signatures provide authenticity and integrity verification but do not provide confidentiality.

Create a detached signature:

```bash
gpg --armor --detach-sign \
  --local-user YOUR_FINGERPRINT test.txt
```

Verify the signature:

```bash
gpg --verify test.txt.asc test.txt
```

The recipient must possess and independently authenticate the signer's public key.

Encryption and digital signatures can be combined when both confidentiality and sender authentication are required.

## 14. Combining Encryption Layers

```text
Physical Storage Device
└── Outer VeraCrypt Volume
    └── Inner VeraCrypt Container
        └── Encrypted File or Directory
```

Each layer serves a different purpose:

- Outer encryption protects the entire storage device.
- Nested encryption creates an independent access boundary.
- File encryption protects data independently of its storage location.
- Authenticated encryption detects ciphertext modifications.
- Digital signatures establish integrity and authenticity relative to a verified signing key.

Not every deployment requires all layers.

## 15. Security Considerations

- Never reuse important encryption passphrases.
- Never publish private keys or credentials.
- Verify recipient public keys before encrypting sensitive information.
- Keep recoverable backups of important keys.
- Encryption creates protected copies but does not automatically remove plaintext originals.
- Avoid writing decrypted information to unencrypted storage.
- Consider exposure through swap, temporary files and compromised hosts.
- File-level encryption may expose filenames, file sizes and other external metadata.
- Ordinary deletion does not guarantee secure erasure from flash storage.
- Verify interoperability before transferring files between Windows and Linux.

## Completion Criteria

- Symmetric file encryption and decryption verified.
- Asymmetric encryption and decryption verified.
- Original and recovered file contents compared.
- Unauthorized ciphertext modification detected.
- Directory encryption and recovery verified.
- Private-key protection understood.
- Optional GnuPG and signature procedures independently tested before deployment.

## References

- [age — Official Repository](https://github.com/FiloSottile/age)
- [age Encryption Format Specification](https://age-encryption.org/v1)
- [GnuPG Documentation](https://gnupg.org/documentation/)
- [ChaCha20-Poly1305 — RFC 8439](https://www.rfc-editor.org/rfc/rfc8439.html)
- [scrypt — RFC 7914](https://www.rfc-editor.org/rfc/rfc7914.html)
- [OpenPGP — RFC 9580](https://www.rfc-editor.org/rfc/rfc9580.html)

---

[Previous: Nested and Selective Encryption](04-nested-encryption.md) | [Return to README](../README.md) | [Next: Access and Passwords](06-access-and-passwords.md)
