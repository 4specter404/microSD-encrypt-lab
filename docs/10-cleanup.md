# 10 — Cleanup and Safe Disconnection

## Objective

Remove temporary encryption artifacts, safely disconnect encrypted storage, and maintain the desired encryption configuration without unnecessarily exposing or destroying important information.

## 1. Cleanup Options

| Operation | Effect |
|---|---|
| Unmount encrypted volume | Locks the volume without deleting files |
| Delete files inside encrypted volume | Removes files while retaining volume encryption |
| Delete nested container | Removes the additional encrypted volume and its contents |
| Change volume password | Changes authentication without replacing master encryption keys |
| Recreate encrypted volume | Generates new encryption keys and destroys existing contents |
| Delete temporary backup | Removes the accessible backup copy |

These operations serve different purposes and should not be treated as interchangeable.

## 2. Inspect Active Encrypted Volumes

```bash
veracrypt -t --list
```

Inspect connected storage:

```bash
lsblk -e 7 -o NAME,SIZE,FSTYPE,MOUNTPOINTS,MODEL
```

Verify which encrypted volumes are mounted before removing any files.

## 3. Remove Nested Encryption While Retaining Outer Encryption

To remove an inner VeraCrypt container without affecting the outer encrypted device:

1. Preserve any required information from the inner container.
2. Unmount the inner container.
3. Keep the outer volume mounted.
4. Verify the container filename and location.
5. Delete the inner container.

Example:

```bash
rm -- /media/veracrypt1/Vault.hc
```

Verify the remaining contents:

```bash
ls -lah /media/veracrypt1
df -h /media/veracrypt1
```

The outer volume remains encrypted using its existing master encryption keys and authentication credentials.

Deleting the container removes access to its files through normal filesystem operations. It does not guarantee secure erasure of the underlying flash memory.

## 4. Remove Test Files

Inspect the test directory:

```bash
ls -lah /media/veracrypt1/FileEncryptionLab
```

If the directory contains only disposable test information, remove it:

```bash
rm -r -- /media/veracrypt1/FileEncryptionLab
```

Verify the remaining contents:

```bash
ls -lah /media/veracrypt1
```

Confirm the actual mount point before executing these commands.

Never use recursive deletion against an unverified path.

## 5. Close Temporary LUKS2 Backups

Check the encrypted backup status:

```bash
sudo cryptsetup status cryptolab_backup
```

If the backup is mounted, unmount and close it:

```bash
sudo umount /mnt/cryptolab-backup && \
  sudo cryptsetup close cryptolab_backup
```

Verify:

```bash
sudo cryptsetup status cryptolab_backup
```

Expected result:

```text
/dev/mapper/cryptolab_backup is inactive.
```

Do not delete an encrypted disk image while it remains mounted or actively mapped.

## 6. Remove Temporary Backup and Practice Files

**WARNING: Deleting a backup may permanently remove the last accessible copy of important information.**

Inspect the private project directory:

```bash
ls -lah ~/CryptoLab/private/
```

Only remove files after confirming they are no longer required.

Example:

```bash
rm -- ~/CryptoLab/private/backup.img
```

Remove disposable test keys and containers individually:

```bash
rm -- ~/CryptoLab/private/age-test-key.txt
rm -- ~/CryptoLab/private/age-public.txt
```

Verify the remaining files:

```bash
ls -lah ~/CryptoLab/private/
```

Do not delete production encryption keys, required recovery credentials or the only backup of important information.

## 7. Ordinary Deletion vs. Secure Erasure

Ordinary file deletion removes filesystem references to stored information.

It does not necessarily overwrite the underlying data.

Flash-based storage introduces additional complications:

- Wear leveling.
- Remapped storage cells.
- Spare storage capacity.
- Internal controller behavior.
- Inaccessible historical data.

Tools such as `rm` and `shred` cannot guarantee physical erasure of all historical data from SSDs or microSD cards.

Encrypting previously unused storage before writing sensitive information is preferable to relying on subsequent deletion.

## 8. Cryptographic Erasure

Cryptographic erasure involves rendering encrypted information inaccessible by eliminating the necessary encryption keys.

Its effectiveness depends on factors including:

- Whether all relevant information was encrypted.
- Whether all recoverable copies of the keys have been eliminated.
- Whether old headers or backups remain accessible.
- Whether plaintext copies exist elsewhere.
- Whether the encryption implementation is trustworthy.

Changing a VeraCrypt password alone is not cryptographic erasure because the underlying master encryption keys remain unchanged.

Deleting encrypted files or recreating a volume should not automatically be described as guaranteed cryptographic erasure.

## 9. Safely Lock Nested Volumes

When nested encryption is configured:

1. Close files and applications accessing the inner volume.
2. Move terminal sessions outside the mounted directories.

```bash
cd ~
```

3. Unmount the inner VeraCrypt container.
4. Unmount the outer VeraCrypt volume.
5. Clear cached passwords where applicable.

Verify:

```bash
veracrypt -t --list
```

Expected result when no encrypted volumes remain mounted:

```text
Error: No volumes mounted.
```

This message is expected when all volumes are locked.

## 10. Safely Disconnect Physical Storage on Linux

Identify the removable device:

```bash
lsblk -e 7 -o NAME,SIZE,FSTYPE,MOUNTPOINTS,MODEL,TRAN
```

Confirm that the encrypted volume is unmounted and that the physical device identifier is correct.

Safely power off the storage device:

```bash
udisksctl power-off -b /dev/sdX
```

Replace `/dev/sdX` with the verified device identifier.

This requests safe removal of the device and may power off the connected USB card reader.

Disconnect the reader after the command succeeds.

If the operation fails, investigate mounted filesystems or active processes before physically disconnecting the device.

## 11. Safely Disconnect Physical Storage on Windows

1. Close applications accessing the encrypted volume.
2. Unmount nested VeraCrypt containers first.
3. Unmount the outer VeraCrypt volume.
4. Use the Windows safe hardware removal function.
5. Disconnect the removable storage.

This procedure should be verified on the intended Windows system.

## 12. Repository Privacy and Cleanup

Technical documentation should contain reproducible instructions, not sensitive operational information.

Example `.gitignore`:

```gitignore
# Private information
private/
logs/
*.key
*.pem
*.header
.env

# Storage images and encrypted containers
*.img
*.hc
*.inc
*.iso
*.tar
*.age

# Downloaded software
software/
*.deb

# Temporary files
*.tmp
*.bak
*~
```

Inspect repository status:

```bash
git status --short --ignored
```

Review staged files before committing:

```bash
git diff --cached
git diff --cached --name-only
```

A `.gitignore` file does not automatically remove files already tracked by Git.

Never publish:

- Encryption passphrases.
- Private cryptographic keys.
- Production volume headers.
- Sensitive disk images.
- Confidential files.
- Unsanitized diagnostic logs.
- Personal information.

## Completion Criteria

- Required data remains recoverable.
- Unnecessary test files are removed.
- Temporary encrypted backups are closed.
- Desired encryption configuration remains intact.
- All removable volumes are locked.
- Physical storage is safely disconnected.
- Repository contents are reviewed for sensitive information.

## References

- [VeraCrypt Documentation](https://veracrypt.io/en/Documentation.html)
- [VeraCrypt Wear Leveling](https://veracrypt.io/en/Wear-Leveling.html)
- [Cryptsetup](https://gitlab.com/cryptsetup/cryptsetup)
- [Git Ignore Documentation](https://git-scm.com/docs/gitignore)
- [UDisks Documentation](https://storaged.org/doc/udisks2-api/latest/)

---

[Previous: Troubleshooting](09-troubleshooting.md) | [Return to README](../README.md) | [Cryptography Research](../research/cryptography.md)
