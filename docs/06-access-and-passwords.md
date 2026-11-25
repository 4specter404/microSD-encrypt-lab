# 06 — Access and Password Management

## Objective

Access encrypted portable storage securely, manage authentication credentials, change encryption passwords, and transfer encrypted storage between authorized users and computers.

## 1. Access Requirements

| Component | Required |
|---|---|
| VeraCrypt software | Yes |
| Compatible VeraCrypt version | Yes |
| Encryption passphrase | Yes |
| Original computer | No |
| Original operating system | No |
| Internet connection | No |
| Hardware security key | No |
| Keyfile | Only if configured |
| Custom PIM | Only if configured |

Encrypted storage is not tied to the computer or operating system used to create it.

## 2. Access Encrypted Storage on Linux

Connect the encrypted storage device.

Identify the device:

```bash
lsblk -e 7 -o NAME,SIZE,FSTYPE,MOUNTPOINTS,MODEL,TRAN
```

Launch VeraCrypt:

```bash
veracrypt
```

Complete the following:

1. Select an available slot.
2. Select Device.
3. Identify and select the encrypted storage device.
4. Select Mount.
5. Enter the encryption passphrase.
6. Select Argon2 or the corresponding configured key derivation function.
7. Enter a custom PIM if one was configured.
8. Provide keyfiles if required.
9. Confirm the operation.

Verify the mounted volume:

```bash
veracrypt -t --list
```

Access the filesystem using the displayed mount point.

Example:

```bash
cd /media/veracrypt1
ls -lah
```

Slot numbers and mount locations may vary.

## 3. Access Encrypted Storage on Windows

This cross-platform procedure should be independently verified before production deployment.

1. Install a compatible version of VeraCrypt.
2. Connect the encrypted storage device.
3. Cancel any Windows prompts requesting that the device be formatted or initialized.
4. Launch VeraCrypt.
5. Select an available drive letter.
6. Select Device.
7. Identify the encrypted removable storage device.
8. Select Mount.
9. Enter the required authentication credentials.
10. Open the mounted drive through File Explorer.

The exFAT filesystem allows the decrypted contents to be accessed on Windows and Linux.

## 4. Access Nested Encrypted Containers

When independent nested encryption is configured:

1. Mount the outer encrypted device.
2. Select another available slot or drive letter.
3. Select File.
4. Locate the inner VeraCrypt container.
5. Enter the inner container's independent passphrase.
6. Mount the inner volume.

The outer password does not automatically grant access to the inner volume.

## 5. Understand Encryption Credentials

| Credential | Function |
|---|---|
| Passphrase | Protects access to the encrypted volume header |
| PIM | Configures key derivation costs |
| Keyfile | Optional additional authentication material |
| Header encryption key | Protects the volume header |
| Master encryption keys | Encrypt the actual storage sectors |

The passphrase and master encryption keys are different cryptographic elements.

VeraCrypt derives header encryption keys from the configured credentials. Unlocking the header provides access to the master encryption keys.

## 6. Generate Strong Passphrases

Use randomly generated passphrases instead of predictable passwords.

Recommended approach:

- Use eight independently and randomly selected Diceware words.
- Generate passphrases using a trusted random source.
- Use different passphrases for independent encryption layers.
- Avoid personal information, common phrases and predictable patterns.
- Store recovery credentials securely.

The EFF large wordlist contains 7,776 words. Eight independently selected words provide approximately 103 bits of theoretical entropy.

This assumes genuinely random selection rather than manually choosing preferred words.

Reference:

https://www.eff.org/dice

## 7. Change an Existing VeraCrypt Password

Changing a volume password does not require re-encrypting the entire storage device.

Before proceeding:

- Maintain a verified backup of important files.
- Confirm possession of the existing credentials.
- Unmount the target encrypted volume.
- Verify the target device or container path.

In VeraCrypt:

1. Select Device or Select File.
2. Select the intended encrypted volume.
3. Open the Volumes menu.
4. Select Change Volume Password.
5. Enter the existing authentication credentials.
6. Enter and confirm the new passphrase.
7. Retain the intended key derivation function.
8. Retain or deliberately update the PIM and keyfile settings.
9. Confirm the password change.

### Verify the Password Change

1. Attempt to mount the volume using the new credentials.
2. Confirm the existing data remains accessible.
3. Unmount the volume.
4. Clear any cached credentials.
5. Verify that the previous credentials no longer unlock the current volume.
6. Create an updated external header backup if required.

Changing the password updates header protection without replacing the master encryption keys.

This procedure should be verified on disposable test media before changing credentials on important storage.

## 8. Password Change vs. Complete Re-encryption

| Operation | Password change | Recreate encrypted volume |
|---|---|---|
| Changes passphrase | Yes | Yes |
| Generates new master encryption keys | No | Yes |
| Preserves existing files automatically | Yes | No |
| Requires complete formatting | No | Yes |
| Requires backup and restoration | Recommended | Required to preserve data |

A password change is appropriate for routine credential management.

Creating a new encrypted volume is preferable when the existing master encryption keys may have been compromised.

## 9. Recreate Encryption With New Master Keys

**WARNING: This procedure destroys the existing volume contents.**

1. Back up all required data to independently encrypted storage.
2. Verify backup integrity and recovery.
3. Unmount the existing encrypted volume.
4. Positively identify the target device.
5. Create a new standard VeraCrypt volume.
6. Configure the desired encryption algorithm and key derivation function.
7. Generate a new independent passphrase.
8. Perform a full format.
9. Mount the new encrypted volume.
10. Restore the required data.
11. Verify the restored files.
12. Create a new header backup.

The new volume receives independently generated master encryption keys.

Recreating the volume does not guarantee that historical data has been removed from inaccessible flash-memory cells.

## 10. Transfer Encrypted Storage to Another Person

### Routine Transfer

1. Confirm the recipient has compatible encryption software.
2. Verify that the storage device can be unlocked.
3. Lock and safely disconnect the device.
4. Provide the encrypted storage to the recipient.
5. Communicate authentication credentials through a separate secure channel.
6. Have the recipient verify access.
7. Allow the recipient to change the passphrase.

Never include the passphrase in an unencrypted file stored alongside the encrypted volume.

### Transfer With Independent Encryption Keys

If the recipient requires exclusive cryptographic control, changing the password alone is insufficient.

The previous owner may have retained copies of the volume header or encryption keys.

For stronger separation, the recipient should create a new encrypted volume using credentials and master encryption keys under their exclusive control.

For higher-assurance deployments, consider using previously unused storage.

## 11. Old Header and Credential Exposure

External volume header backups contain protected master encryption key material.

An older header backup may still be accessible using the credentials that existed when the backup was created.

Consequently:

- Protect all header backups.
- Replace outdated header backups after password changes.
- Do not assume a password change revokes access through previously copied headers.
- Account for flash-memory wear leveling.
- Generate new master encryption keys when compromise is suspected.

Previously copied encrypted data cannot be retroactively protected by changing the password on the original device.

## 12. Password Caching and Volume History

For security-sensitive usage:

- Disable unnecessary password caching.
- Clear cached credentials after completing operations.
- Enable the option preventing unnecessary volume history retention.
- Avoid automatic mounting where unattended access is not required.
- Unmount volumes before leaving the computer unattended.

Volume history settings affect convenience and metadata retention. They do not change encryption strength.

## 13. Safely Disconnect Encrypted Storage

Close files, applications and terminals accessing the mounted volumes.

Unmount nested volumes before unmounting the outer encrypted device.

Verify:

```bash
veracrypt -t --list
lsblk -e 7 -o NAME,SIZE,FSTYPE,MOUNTPOINTS
```

A message indicating that no VeraCrypt volumes are mounted is expected when all volumes have been locked.

Identify the current physical device path.

Safely power off the removable storage:

```bash
udisksctl power-off -b /dev/sdX
```

Replace `/dev/sdX` with the verified device identifier.

Disconnect the device after the command succeeds.

On Windows, unmount through VeraCrypt and use the operating system's safe removal function.

Closing the VeraCrypt application alone does not necessarily unmount encrypted volumes.

## 14. Security Considerations

- Never disclose private keys or passwords unnecessarily.
- Do not mount sensitive storage on untrusted computers.
- Use independently generated passphrases for nested encryption.
- Protect external header backups.
- Verify compatibility before transferring storage.
- Avoid unnecessary plaintext copies.
- Understand that encryption does not protect information while it is accessible to a compromised host.
- Maintain verified backups of irreplaceable information.

## Completion Criteria

- Encrypted storage can be accessed independently of the original computer.
- Authentication requirements are understood.
- Password changes can be performed and verified.
- Differences between password changes and complete re-encryption are understood.
- Secure ownership transfer procedures are established.
- Storage can be locked and safely disconnected.

## References

- [VeraCrypt Documentation](https://veracrypt.io/en/Documentation.html)
- [VeraCrypt Program Menu](https://veracrypt.io/en/Program%20Menu.html)
- [Changing Passwords and Keyfiles](https://veracrypt.io/en/Changing%20Passwords%20and%20Keyfiles.html)
- [Personal Iterations Multiplier](https://veracrypt.io/en/Personal%20Iterations%20Multiplier%20%28PIM%29.html)
- [Flash-Memory Wear Leveling](https://veracrypt.io/en/Wear-Leveling.html)
- [EFF Diceware Wordlists](https://www.eff.org/dice)

---

[Previous: File Encryption](05-file-encryption.md) | [Return to README](../README.md) | [Next: Recovery](07-recovery.md)
