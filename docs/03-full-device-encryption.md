# 03 — Full-Device Encryption

## Objective

Encrypt an entire removable storage device using VeraCrypt to protect its contents against unauthorized physical access and offline attacks.

## 1. Recommended Configuration

| Parameter | Recommended setting |
|---|---|
| Encryption software | VeraCrypt |
| Volume type | Standard |
| Encryption algorithm | AES-256 |
| Encryption mode | XTS |
| Key derivation function | Argon2id |
| Passphrase | Eight randomly generated Diceware words |
| PIM | Default |
| Keyfiles | Optional |
| Filesystem | exFAT |
| Quick Format | Disabled for previously used devices |

## 2. Cryptographic Architecture

VeraCrypt generates independent cryptographic keys to encrypt the storage volume.

The user's passphrase is processed by a key derivation function to generate the keys necessary to decrypt the volume header.

The decrypted header provides access to the master encryption keys used to encrypt and decrypt the storage sectors.

### AES-256-XTS

AES-XTS is designed for storage encryption. It uses two independent AES keys and incorporates the sector position into encryption.

AES-XTS provides confidentiality but does not independently authenticate stored data against tampering.

### Argon2id

Argon2id is a memory-hard key derivation function designed to increase the computational and memory costs of password guessing.

A strong passphrase remains essential because possession of an encrypted device permits offline password-guessing attempts.

## 3. Identify the Target Device

Run before and after connecting the storage device:

```bash
lsblk -e 7 -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS,MODEL,TRAN
```

Identify the removable device using its capacity, model and connection type.

The following examples use `/dev/sdX` as a placeholder.

Never assume that a device identifier remains unchanged after reconnecting hardware or restarting the operating system.

## 4. Prepare the Storage Device

For a device containing existing information, complete the backup and migration procedure before proceeding.

Unmount all mounted partitions.

Example:

```bash
udisksctl unmount -b /dev/sdX1
```

Verify:

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS /dev/sdX
```

### Remove Existing Partition Signatures

VeraCrypt may refuse whole-device encryption when existing partitions are present.

**WARNING: The following commands remove partition and filesystem signatures. Verify the target device and backup before execution.**

```bash
sudo wipefs --all /dev/sdX
sudo partprobe /dev/sdX
```

Verify:

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS /dev/sdX
```

The device should no longer display its previous partitions.

Removing filesystem signatures does not securely erase existing data.

## 5. Create the Encrypted Volume

Launch VeraCrypt:

```bash
veracrypt
```

Complete the volume creation wizard:

1. Select Create Volume.
2. Select Encrypt a non-system partition/drive.
3. Select Standard VeraCrypt volume.
4. Select Device.
5. Select the verified removable storage device.
6. Confirm the whole-device encryption warning.
7. Select AES as the encryption algorithm.
8. Select Argon2 as the key derivation function, using a compatible VeraCrypt release.
9. Enter and confirm a strong, randomly generated passphrase.
10. Leave PIM at its default unless custom parameters are required.
11. Leave keyfiles disabled unless deliberately implementing separate keyfile management.
12. Select Yes when asked about storing files larger than 4 GiB.
13. Select exFAT.
14. Select cross-platform support if prompted.
15. Disable Quick Format for previously used storage.
16. Move the mouse randomly within the wizard to contribute entropy.
17. Select Format and confirm the destructive operation.

Do not disconnect the device or interrupt the formatting process.

## 6. Formatting Considerations

Full formatting writes encrypted data across the accessible volume.

Quick formatting initializes the encrypted volume without overwriting all previously accessible storage sectors.

For previously used media, full formatting is preferable to reduce exposure of historical plaintext.

Neither method guarantees sanitization of inaccessible flash-memory cells because of wear leveling, remapped sectors and internal controller behavior.

For highly sensitive information, use previously unused storage and encrypt it before storing plaintext.

## 7. Mount the Encrypted Device

In VeraCrypt:

1. Select an available slot.
2. Select Device.
3. Select the encrypted removable device.
4. Select Mount.
5. Enter the encryption passphrase.
6. Select the corresponding key derivation function or use autodetection.
7. Enter the configured PIM if applicable.
8. Confirm the operation.

The decrypted filesystem should become accessible.

## 8. Verify Encryption

List mounted VeraCrypt volumes:

```bash
veracrypt -t --list
```

Inspect the block device:

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS /dev/sdX
```

Check available storage, using the actual mount point:

```bash
df -h /media/veracrypt1
```

Open Volume Properties in VeraCrypt and verify the configured encryption parameters.

## 9. Verify Data Persistence

Create a non-sensitive test file inside the mounted volume:

```bash
echo "Encryption verification test" > /media/veracrypt1/test.txt
sha256sum /media/veracrypt1/test.txt
```

Unmount the volume through VeraCrypt.

Mount it again using the correct credentials.

Verify the file:

```bash
cat /media/veracrypt1/test.txt
sha256sum /media/veracrypt1/test.txt
```

The checksums should match.

## 10. Lock and Safely Disconnect

Close applications accessing the encrypted filesystem.

Unmount the volume using
