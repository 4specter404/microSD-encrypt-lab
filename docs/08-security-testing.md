# 08 — Security and Forensic Testing

## Objective

Evaluate encrypted portable storage against unauthorized access, offline password guessing, disk imaging, file recovery and ciphertext modification.

Testing must be performed only on owned or explicitly authorized storage devices.

## 1. Threat Model

Assume an unauthorized individual obtains physical possession of the locked storage device.

| Threat | Security mechanism |
|---|---|
| Unauthorized mounting | VeraCrypt authentication |
| Offline password guessing | High-entropy passphrase and memory-hard key derivation |
| Raw disk inspection | AES-256-XTS |
| Disk imaging | Encrypted storage contents |
| File recovery software | Storage encryption |
| Unauthorized file modification | Authenticated file-level encryption |
| Exposure of the outer password | Independent nested encryption |
| Primary header corruption | Embedded and external header backups |

Encryption does not protect against every possible attack. Host compromise, exposed credentials and historical plaintext remain important considerations.

## 2. Testing Tools

| Tool | Purpose |
|---|---|
| VeraCrypt | Authentication and volume inspection |
| lsblk | Block device identification |
| xxd | Hexadecimal inspection |
| strings | Search for readable data |
| file | File format identification |
| SHA-256 | Integrity verification |
| TestDisk / PhotoRec | Filesystem and file recovery testing |
| age | Authenticated file encryption |
| dd | Disk imaging |
| Cryptsetup | Cryptographic performance benchmarking |

Install additional Linux testing utilities:

```bash
sudo apt update
sudo apt install testdisk binutils xxd file
```

## 3. Establish a Controlled Testing Environment

Use disposable test data rather than genuinely sensitive information.

Create a test file inside an unlocked encrypted volume:

```bash
echo "CRYPTOLAB-VERIFICATION-SAMPLE" \
  > /media/veracrypt1/verification.txt
```

Calculate its checksum:

```bash
sha256sum /media/veracrypt1/verification.txt
```

Record the checksum for subsequent verification.

Unmount the encrypted volume before performing raw storage inspection.

## 4. Authentication Testing

Attempt to mount the encrypted volume using incorrect credentials.

Test the following conditions where applicable:

- Incorrect password.
- Incorrect custom PIM.
- Missing required keyfile.
- Incorrect explicitly selected key derivation function.

Verify that invalid credentials are rejected.

Then mount the volume using the correct credentials and verify that the original test file remains accessible.

These tests demonstrate authentication behavior but do not establish resistance against every possible password attack.

## 5. Inspect the Locked Device

Identify the physical storage device:

```bash
lsblk -e 7 -o NAME,SIZE,FSTYPE,MOUNTPOINTS,MODEL
```

Replace `/dev/sdX` in subsequent commands with the verified test device.

Inspect the first 256 bytes:

```bash
sudo xxd -l 256 /dev/sdX
```

Check for recognizable file format information:

```bash
sudo file -s /dev/sdX
```

Search the first 64 MiB for readable strings:

```bash
sudo dd if=/dev/sdX bs=1M count=64 \
  status=none | strings -n 12 | head -50
```

Encrypted storage should not expose the decrypted filesystem or its plaintext files through ordinary raw inspection.

Random-looking hexadecimal output alone does not prove that encryption is secure.

## 6. Optional: Disk Imaging

Create a complete image of the locked encrypted storage device using the recovery guide.

The resulting image should contain the encrypted volume rather than an automatically decrypted filesystem.

Verify that:

1. The image matches the readable source device.
2. The image cannot be mounted as an ordinary plaintext filesystem.
3. VeraCrypt can access the image using valid credentials, where the volume layout permits.
4. Incorrect credentials are rejected.

Disk imaging does not inherently bypass encryption.

However, an attacker possessing an image can perform offline password guesses without interacting with the original storage device.

## 7. Optional: File Recovery Testing

PhotoRec attempts to recover recognizable files from storage without relying entirely on filesystem metadata.

Use a disposable encrypted disk image rather than important physical storage.

Launch PhotoRec against the working image:

```bash
photorec /path/to/disposable-test.img
```

Select a separate recovery destination with sufficient capacity.

Do not select the original storage device as the recovery destination.

Examine recovered output for recognizable plaintext test files.

A correctly encrypted volume should not expose its encrypted plaintext files through ordinary file-carving methods.

False positives are possible because ciphertext may coincidentally resemble recognizable file signatures.

Failure to recover plaintext does not prove that every historical copy has been removed from physical flash memory.

## 8. Password Resistance

Storage encryption allows an attacker possessing the relevant volume header to attempt password guesses offline.

Resistance depends on:

- Passphrase entropy.
- Key derivation algorithm.
- Key derivation parameters.
- Available computational resources.
- Implementation quality.

### Diceware Passphrase Entropy

For independently selected words from a 7,776-word list:

| Words | Approximate entropy |
|---|---|
| 4 | 52 bits |
| 6 | 78 bits |
| 8 | 103 bits |
| 10 | 129 bits |

These estimates require genuinely random word selection.

Human-selected words and predictable phrases do not necessarily provide equivalent entropy.

### Key Derivation Benchmarking

On Linux:

```bash
cryptsetup benchmark
```

This measures supported cryptographic operations and key derivation performance on the local system.

Results are specific to Cryptsetup and the local hardware. They do not directly establish VeraCrypt's Argon2id performance or an attacker's password-guessing rate.

Use VeraCrypt's available benchmark functionality and measure actual volume unlocking behavior when evaluating its configuration.

## 9. Test Independent Encryption Layers

For nested VeraCrypt volumes:

1. Mount the outer encrypted device.
2. Mount the inner encrypted container.
3. Create non-sensitive test files.
4. Unmount the inner container.
5. Confirm that the outer filesystem remains accessible.
6. Confirm that the inner filesystem is no longer accessible.
7. Unlock the inner container using its independent credentials.
8. Verify the original test files.

Repeat the procedure with both volumes locked.

The objective is to demonstrate independent authentication boundaries.

## 10. Authenticated Encryption Testing

VeraCrypt's AES-XTS encryption provides confidentiality but does not independently authenticate each encrypted sector.

Age provides authenticated file encryption.

Create a disposable encrypted test file:

```bash
echo "Authenticated encryption verification sample" > test.txt

age -p -o test.txt.age test.txt
```

Copy the encrypted file:

```bash
cp test.txt.age tampered.txt.age
```

Modify a byte in the encrypted payload:

```bash
python3 - <<'PYCODE'
with open("tampered.txt.age", "r+b") as f:
    f.seek(-1, 2)
    original = f.read(1)
    f.seek(-1, 2)
    f.write(bytes([original[0] ^ 0x01]))
PYCODE
```

This procedure requires a non-empty encrypted file. Confirm that the modified copy differs from the original.

```bash
cmp test.txt.age tampered.txt.age
```

Attempt to decrypt the modified file:

```bash
age -d -o tampered-recovered.txt tampered.txt.age
```

Expected behavior: age rejects the modified ciphertext.

Do not use partial output from a failed decryption.

Verify that the unmodified encrypted file can still be decrypted:

```bash
age -d -o recovered.txt test.txt.age

sha256sum test.txt recovered.txt
```

Matching checksums confirm successful recovery of the original file.

## 11. Cross-Platform Testing

Test the same encrypted device on compatible Windows and Linux systems.

Verify:

- Device identification.
- Successful authentication.
- Filesystem accessibility.
- File reading and writing.
- Integrity checksum consistency.
- Nested container compatibility.
- Safe unmounting and removal.

Linux checksum command:

```bash
sha256sum verification.txt
```

Windows PowerShell equivalent:

```powershell
Get-FileHash .\verification.txt -Algorithm SHA256
```

The SHA-256 values should match.

Never accept an operating system request to format an encrypted device simply because its locked filesystem is not recognized.

## 12. Host Security Testing

Investigate potential exposure outside encrypted storage.

On Linux, inspect active swap:

```bash
swapon --show
```

Inspect mounted filesystems:

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS
```

Review the following potential exposure points:

- Unencrypted swap.
- Hibernation images.
- Temporary files.
- Application caches.
- Password caching.
- Automatic backups.
- Cloud synchronization.
- Clipboard history.
- Malware and unauthorized processes.

The absence of obvious plaintext exposure does not prove that a host is uncompromised.

## 13. Testing Record

Use the following template to document results without including sensitive information.

| Test | Expected result | Actual result |
|---|---|---|
| Correct password | Access granted | Pending |
| Incorrect password | Access rejected | Pending |
| Raw storage inspection | No recognizable test plaintext | Pending |
| Disk image authentication | Valid credentials required | Pending |
| File recovery | No recovery of encrypted test plaintext | Pending |
| Nested encryption | Independent authentication | Pending |
| Ciphertext modification | Authentication failure | Pending |
| SHA-256 comparison | Matching checksums | Pending |
| Windows compatibility | Successful access | Pending |
| Linux compatibility | Successful access | Pending |
| Header recovery | Successful recovery | Pending |

Record software versions, test conditions and observed limitations.

Never publish actual passwords, private keys, production volume headers, sensitive recovered files or confidential disk images.

## 14. Security Limitations

- No finite set of tests proves absolute security.
- Strong encryption cannot compensate for compromised credentials.
- Unlocked volumes expose accessible plaintext to the host.
- AES-XTS does not independently authenticate stored sectors.
- Flash-memory wear leveling may retain historical plaintext.
- Encryption does not prevent deletion, corruption or physical destruction.
- Password guessing resistance depends on actual passphrase entropy and key derivation parameters.

## Completion Criteria

- Correct and incorrect authentication behavior verified.
- Locked storage inspected.
- Original and recovered file integrity verified.
- Authenticated encryption tested against modified ciphertext.
- Independent encryption layers evaluated where applicable.
- Intended operating system compatibility verified.
- Recovery procedures independently tested.

## Testing Status

This guide provides a repeatable security assessment methodology. Results must be established through actual testing rather than assumed from the selected encryption algorithms.

## References

- [VeraCrypt Documentation](https://veracrypt.io/en/Documentation.html)
- [PhotoRec Documentation](https://www.cgsecurity.org/wiki/PhotoRec)
- [Cryptsetup](https://gitlab.com/cryptsetup/cryptsetup)
- [age Encryption](https://github.com/FiloSottile/age)
- [Argon2 Specification — RFC 9106](https://www.rfc-editor.org/rfc/rfc9106.html)
- [EFF Diceware Wordlists](https://www.eff.org/dice)

---

[Previous: Recovery](07-recovery.md) | [Return to README](../README.md) | [Next: Troubleshooting](09-troubleshooting.md)
