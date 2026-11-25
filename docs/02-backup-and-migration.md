# 02 — Backup and Data Migration

## Objective

Preserve existing data before performing destructive storage encryption operations.

## 1. Available Methods

| Method | Description |
|---|---|
| Backup and restore | Preserve files in encrypted temporary storage, encrypt the target device, and restore the files. |
| In-place encryption | Encrypt existing storage without first removing all files, where supported. |

Backup and restore is the recommended method because it reduces the risk of permanent data loss.

In-place encryption depends on operating system, filesystem and encryption software support. A verified backup is still strongly recommended.

## 2. Identify the Source Device

```bash
lsblk -e 7 -o NAME,SIZE,FSTYPE,MOUNTPOINTS,MODEL,TRAN
```

The following examples assume:

- `/dev/sdX` is the removable storage device.
- `/dev/sdX1` is a FAT boot partition.
- `/dev/sdX2` is an ext4 data partition.

Replace these identifiers with the actual device paths.

Never execute destructive commands against an unverified device.

## 3. Mount Existing Filesystems

Unmount any automatically mounted source partitions before continuing.

```bash
udisksctl unmount -b /dev/sdX1
```

Create temporary mount points:

```bash
sudo mkdir -p /mnt/cryptolab-source
sudo mkdir -p /mnt/cryptolab-boot
sudo mkdir -p /mnt/cryptolab-backup
```

Mount the original filesystems in read-only mode:

```bash
sudo mount -o ro,noload /dev/sdX2 /mnt/cryptolab-source
sudo mount -o ro /dev/sdX1 /mnt/cryptolab-boot
```

The `noload` option prevents ext4 journal replay. It is specific to compatible filesystems and should not be applied universally.
### Read-Only Mounting Limitations

A read-only filesystem mount is not equivalent to forensic-grade write protection.

Although the `ro,noload` options restrict filesystem writes and prevent ext4 journal replay, they should not be treated as a guarantee that the underlying device remains completely unchanged.

For forensic preservation, use a compatible hardware write blocker where possible. Block-device-level read-only protection provides an additional software-based precaution but is not equivalent to a hardware write blocker.

The backup procedure documented here is intended for data migration, not forensic evidence acquisition.
## 4. Calculate Backup Capacity

```bash
sudo du -sh /mnt/cryptolab-source
sudo du -sh /mnt/cryptolab-boot
df -h ~
```

Select a backup capacity sufficient for the existing data, filesystem overhead and additional working space.

The following example uses a 10 GiB backup image.

## 5. Create Encrypted Backup Storage

```bash
mkdir -p ~/CryptoLab/private
chmod 700 ~/CryptoLab/private
cd ~/CryptoLab

fallocate -l 10G private/backup.img
chmod 600 private/backup.img
```

Initialize LUKS2 encryption:

```bash
sudo cryptsetup luksFormat \
  --type luks2 \
  --cipher aes-xts-plain64 \
  --key-size 512 \
  --pbkdf argon2id \
  private/backup.img
```

Enter a strong backup passphrase.

The 512-bit XTS key consists of two 256-bit AES keys.

## 6. Unlock and Format the Backup

```bash
sudo cryptsetup open private/backup.img cryptolab_backup

sudo cryptsetup status cryptolab_backup
```

Create an ext4 filesystem:

```bash
sudo mkfs.ext4 -L CryptoBackup \
  /dev/mapper/cryptolab_backup
```

Mount the encrypted backup:

```bash
sudo mount /dev/mapper/cryptolab_backup \
  /mnt/cryptolab-backup
```

Verify capacity:

```bash
df -h /mnt/cryptolab-backup
```

## 7. Copy the Existing Data

```bash
sudo mkdir -p /mnt/cryptolab-backup/{boot,rootfs}
```

Copy Linux files while preserving filesystem metadata:

```bash
sudo rsync -aHAX --numeric-ids --info=progress2 \
  /mnt/cryptolab-source/ \
  /mnt/cryptolab-backup/rootfs/
```

Copy the FAT boot partition:

```bash
sudo rsync -rt --info=progress2 \
  /mnt/cryptolab-boot/ \
  /mnt/cryptolab-backup/boot/
```

### rsync Parameters

| Parameter | Purpose |
|---|---|
| `-a` | Archive mode |
| `-H` | Preserve hard links |
| `-A` | Preserve access control lists |
| `-X` | Preserve extended attributes |
| `--numeric-ids` | Preserve numeric ownership identifiers |
| `--info=progress2` | Display overall transfer progress |

## 8. Verify Backup Integrity

```bash
sudo rsync -aHAXnc --numeric-ids --itemize-changes \
  /mnt/cryptolab-source/ \
  /mnt/cryptolab-backup/rootfs/
```

```bash
sudo rsync -rnc --itemize-changes \
  /mnt/cryptolab-boot/ \
  /mnt/cryptolab-backup/boot/
```

The `-n` parameter performs a dry run.

The `-c` parameter compares file contents using checksums.

A successful command with no reported differences indicates that the compared files match. Investigate any errors or differences before continuing.

## 9. Lock and Verify Backup Recovery

```bash
sync
sudo umount /mnt/cryptolab-source
sudo umount /mnt/cryptolab-boot
sudo umount /mnt/cryptolab-backup
sudo cryptsetup close cryptolab_backup
```

Test whether the backup can be unlocked again:

```bash
sudo cryptsetup open --readonly \
  private/backup.img cryptolab_backup

sudo mount -o ro,noload \
  /dev/mapper/cryptolab_backup \
  /mnt/cryptolab-backup

sudo du -sh /mnt/cryptolab-backup/{boot,rootfs}
```

After verifying accessibility:

```bash
sudo umount /mnt/cryptolab-backup
sudo cryptsetup close cryptolab_backup
```

## 10. Restore Data After Device Encryption

Mount the newly encrypted destination using VeraCrypt.

Unlock and mount the backup as described previously.

For ordinary documents and media, files can be copied directly into the encrypted destination.

For Linux system files, use an archive to preserve permissions, ownership, symbolic links and other metadata when the destination uses exFAT.

Example destination:

```bash
sudo tar --acls --xattrs --xattrs-include='*' \
  --numeric-owner --sparse \
  -cpf /media/veracrypt1/original-data.tar \
  -C /mnt/cryptolab-backup boot rootfs
```

Verify archive contents against the backup:

```bash
sudo tar --compare \
  --file=/media/veracrypt1/original-data.tar \
  -C /mnt/cryptolab-backup
```

Record an integrity checksum:

```bash
sha256sum /media/veracrypt1/original-data.tar
```

An archived operating system is not equivalent to a bootable disk image.

## 11. Security Limitations

- Never delete the original data before verifying the backup.
- Keep the backup until destination encryption and recovery have been verified.
- File-level backups do not preserve partition tables or unallocated sectors.
- Flash-memory wear leveling can retain historical plaintext in inaccessible cells.
- Ordinary file deletion does not guarantee secure erasure.
- Avoid exposing sensitive data through unencrypted swap, temporary files or compromised host systems.

## Completion Criteria

- Existing data preserved in encrypted storage.
- Backup contents verified.
- Backup successfully locked and reopened.
- Original files restored into encrypted destination.
- Restored data integrity verified.

---

[Previous: Environment Setup](01-environment-setup.md) | [Return to README](../README.md) | [Next: Full-Device Encryption](03-full-device-encryption.md)
