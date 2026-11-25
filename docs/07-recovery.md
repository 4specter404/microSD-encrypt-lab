# 07 — Encryption Recovery and Header Backups

## Objective

Establish recovery procedures for encrypted storage in the event of header corruption, hardware failure, accidental modification or credential loss.

## 1. Understand the VeraCrypt Volume Header

The volume header contains protected cryptographic information required to access encrypted storage.

This includes:

- Master encryption keys.
- Encryption parameters.
- Volume configuration.
- Information required to interpret the encrypted storage.

The header is protected using keys derived from the configured authentication credentials.

Damage to the header can prevent access even when the correct password is available.

## 2. Recovery Architecture

```text
Standard VeraCrypt Volume
├── Primary Volume Header
├── Encrypted Storage
└── Embedded Backup Header

Separate Secure Storage
└── External Header Backup
```

VeraCrypt volumes contain an embedded backup header. An external header backup provides an additional recovery option.

## 3. Types of Backups

| Backup | Purpose |
|---|---|
| Embedded header | Recover from primary header corruption |
| External header | Recover from header corruption affecting the original device |
| Encrypted file backup | Recover individual files |
| Complete encrypted disk image | Preserve the entire encrypted device |
| Credential backup | Prevent loss of required authentication credentials |

A header backup does not contain the original files and cannot recover destroyed encrypted data.

## 4. Create an External Header Backup

Before creating a backup:

1. Verify that the encrypted volume mounts successfully.
2. Unmount the volume.
3. Identify the correct storage device or container.
4. Prepare a separate, secure backup location.

In VeraCrypt:

1. Select Device or Select File.
2. Select the encrypted volume.
3. Open the Tools menu.
4. Select Backup Volume Header.
5. Enter the required authentication credentials.
6. Follow the randomness collection instructions if prompted.
7. Save the header backup to separate secure storage.

Example filename:

```text
portable-storage.header
```

Do not store the only header backup on the encrypted device it protects.

Never upload header backups to public repositories.

## 5. Protect External Header Backups

Although VeraCrypt header backups are encrypted, they contain important cryptographic material.

Recommended precautions:

- Store backups on independently encrypted storage.
- Restrict access to authorized individuals.
- Maintain more than one recoverable copy where appropriate.
- Keep credentials separate from backup media.
- Record which volume each backup belongs to.
- Replace outdated backups after authentication changes.
- Never publish header backups or encryption keys.

Optional Linux file permissions:

```bash
chmod 600 portable-storage.header
```

Filesystem permissions restrict local access but do not replace encryption.

## 6. Recover Using the Embedded Backup Header

If a volume fails to mount, first verify:

- Correct storage device.
- Correct passphrase.
- Correct key derivation function.
- Correct PIM.
- Required keyfiles.
- Compatible VeraCrypt version.
- Storage hardware functionality.

To attempt recovery without immediately modifying the header:

1. Select the encrypted volume.
2. Select Mount.
3. Open Mount Options.
4. Enable the option to use the embedded backup header.
5. Enable read-only mounting where appropriate.
6. Enter the correct credentials.
7. Attempt to mount the volume.

If successful, immediately preserve important files in independently encrypted storage.

Do not assume the primary header is the only damaged component.

## 7. Restore a Damaged Volume Header

**WARNING: Header restoration modifies the target volume. Incorrect recovery operations may cause further data loss.**

Create a complete backup or disk image of the affected volume before attempting repairs.

In VeraCrypt:

1. Unmount the encrypted volume.
2. Select the correct device or container.
3. Open Tools.
4. Select Restore Volume Header.
5. Select the appropriate recovery source:
   - Embedded backup header.
   - External header backup.
6. Enter the credentials associated with the selected backup.
7. Complete the restoration procedure.
8. Attempt to mount the restored volume.
9. Verify the filesystem and important files.

These recovery procedures should first be practiced on disposable test volumes.

## 8. Header Backups and Password Changes

Changing a VeraCrypt password changes the protection applied to the master encryption keys. It does not generate new master keys.

Restoring an older external header may restore the authentication requirements that existed when that backup was created.

Consequently, outdated headers remain security-sensitive even after password changes.

After changing credentials:

1. Verify the new credentials.
2. Confirm the encrypted data remains accessible.
3. Create an updated header backup.
4. Securely manage superseded backups.

If master encryption keys may have been exposed, create a completely new encrypted volume and migrate the required files.

## 9. Create a Complete Encrypted Disk Image

A complete image preserves the encrypted contents and volume headers.

This is an optional Linux recovery procedure.

Identify and unmount the removable storage device.

Verify the destination has sufficient available capacity:

```bash
df -h /path/to/backup-storage
```

Create a restricted backup directory on trusted storage:

```bash
mkdir -p ~/CryptoLab/private
chmod 700 ~/CryptoLab/private
```

Create an image:

```bash
sudo dd if=/dev/sdX \
  of="$HOME/CryptoLab/private/encrypted-device.img" \
  bs=4M status=progress conv=fsync
```

Replace `/dev/sdX` with the verified source device.

The destination must not be located on the source device. Ensure sufficient capacity for the entire image.

### Verify Image Integrity

```bash
sudo sha256sum ~/CryptoLab/private/encrypted-device.img
```

For a healthy, unchanged source device, compare its checksum with the image:

```bash
sudo sha256sum /dev/sdX
```

Matching checksums establish that the image matches the readable source contents at the time of verification.

Keep the recorded checksum in a trusted location.

For failing storage devices, GNU ddrescue is preferable to ordinary dd because it is designed to recover readable data while managing read errors.

## 10. Verify Recovery From an Image

Keep the original physical storage unchanged.

Create a separate working copy of the image.

In VeraCrypt:

1. Select File.
2. Select the working image.
3. Attempt to mount it using the original credentials.
4. Use read-only mounting initially.
5. Verify the expected files are accessible.
6. Unmount the image.

A whole-device VeraCrypt image may be directly mountable as a file. Images containing partition tables may require different handling.

Always preserve the original backup image during recovery experiments.

## 11. Restore a Complete Disk Image

**WARNING: This operation overwrites the destination device. Verify all device identifiers before execution.**

The replacement device must provide sufficient capacity and compatible storage geometry.

Unmount the destination device.

Example:

```bash
sudo dd if="$HOME/CryptoLab/private/encrypted-device.img" \
  of=/dev/sdX \
  bs=4M status=progress conv=fsync
```

Do not reverse the input and output parameters.

For straightforward restoration, use a destination matching the original device size. Larger devices may require additional consideration because embedded backup headers depend on volume layout.

After restoration:

1. Safely disconnect and reconnect the device.
2. Verify the device identifier.
3. Mount the restored volume.
4. Verify encryption credentials.
5. Verify important file contents.

This restores the original encrypted volume rather than creating independent master encryption keys.

## 12. Filesystem Recovery

A volume may decrypt successfully while its internal filesystem is damaged.

In this situation:

1. Preserve the entire encrypted volume before attempting repair.
2. Attempt read-only access.
3. Recover important readable files.
4. Use filesystem-specific repair tools on a working copy.

The appropriate repair utility depends on the internal filesystem.

| Filesystem | Example recovery utility |
|---|---|
| exFAT | fsck.exfat |
| ext4 | e2fsck |
| NTFS | Windows chkdsk |

Filesystem repair must target the decrypted filesystem, not the raw encrypted device.

Do not run repair utilities against a mounted filesystem unless the specific utility explicitly supports that operation.

## 13. Forgotten Passwords and Lost Keys

VeraCrypt does not provide a universal password recovery mechanism.

| Situation | Recovery possibility |
|---|---|
| Primary header corrupted | Embedded or external header may permit recovery |
| Both embedded headers corrupted | External header may permit recovery |
| Forgotten password | Requires valid alternative credentials or a compatible older header with known credentials |
| Lost required keyfile | Requires a recoverable copy or compatible alternative credentials |
| Destroyed encrypted data | Requires a separate data backup |
| Lost age private key | Requires another authorized recipient identity or a private-key backup |

Strong encryption intentionally prevents recovery without the necessary authentication material.

## 14. Recovery Testing

Use disposable encrypted containers before testing recovery on important storage.

Recommended exercises:

- Create and verify an external header backup.
- Mount using the embedded backup header.
- Restore a header on a disposable test volume.
- Create and mount a complete encrypted disk image.
- Verify recovered files against original checksums.
- Test recovery after a password change.
- Verify encrypted backup accessibility.

Never intentionally corrupt the only copy of important information.

## 15. Security Limitations

- Header backups cannot replace file backups.
- Password changes do not replace master encryption keys.
- Older headers may remain usable with previous credentials.
- Physical storage failure may destroy both embedded headers.
- Flash-memory wear leveling complicates secure deletion.
- Recovery operations may modify or destroy existing data.
- Encryption cannot compensate for missing recovery credentials.

## Completion Criteria

- External header backup created and protected.
- Recovery credentials securely maintained.
- Embedded header recovery understood.
- Complete encrypted backup verified.
- Recovery procedures successfully tested on disposable media.

## Testing Status

The procedures in this guide are additional recovery exercises. They should be independently verified before use on production storage.

## References

- [VeraCrypt Documentation](https://veracrypt.io/en/Documentation.html)
- [VeraCrypt Program Menu](https://veracrypt.io/en/Program%20Menu.html)
- [Mounting VeraCrypt Volumes](https://veracrypt.io/en/Mounting%20VeraCrypt%20Volumes.html)
- [VeraCrypt Volume Format Specification](https://veracrypt.io/en/VeraCrypt%20Volume%20Format%20Specification.html)
- [Changing Passwords and Keyfiles](https://veracrypt.io/en/Changing%20Passwords%20and%20Keyfiles.html)
- [GNU ddrescue Manual](https://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html)

---

[Previous: Access and Passwords](06-access-and-passwords.md) | [Return to README](../README.md) | [Next: Security Testing](08-security-testing.md)
