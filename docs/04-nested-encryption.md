# 04 — Nested and Selective Encryption

## Objective

Protect selected files and directories using independently encrypted VeraCrypt containers, with or without full-device encryption.

## 1. Encryption Configurations

| Configuration | Protection |
|---|---|
| Full-device encryption | Protects the entire storage device. |
| Selective encryption | Protects selected files within an encrypted container on otherwise unencrypted storage. |
| Nested encryption | Provides an independent encryption layer inside an already encrypted device. |
| File-level encryption | Encrypts individual files independently of their storage location. |

## 2. Nested Encryption Architecture

```text
Removable Storage Device
└── Outer VeraCrypt Volume
    ├── General Files
    └── Inner VeraCrypt Container
        └── Sensitive Files
```

The outer and inner volumes use independent encryption keys and should use different passphrases.

Unlocking the outer volume does not automatically unlock the inner container.

## 3. Recommended Configuration

| Parameter | Outer volume | Inner container |
|---|---|---|
| Encryption | AES-256-XTS | AES-256-XTS |
| Key derivation | Argon2id | Argon2id |
| Passphrase | Independent | Independent |
| PIM | Default | Default |
| Filesystem | exFAT | exFAT |
| Capacity | Entire device | Based on sensitive data size |

The same encryption algorithm can safely be used for both layers when independent keys are generated.

## 4. Calculate Required Container Capacity

Determine the size of the information requiring additional protection:

```bash
du -sh /path/to/sensitive-data
du -sb /path/to/sensitive-data
```

Check available storage:

```bash
df -h /media/veracrypt1
```

Allow approximately 20–30% additional capacity for filesystem overhead and future data.

Example:

| Existing data | Suggested container capacity |
|---|---|
| 500 MiB | 750 MiB |
| 1 GiB | 1.5 GiB |
| 4.6 GiB | 6 GiB |
| 10 GiB | 13 GiB |
| 50 GiB | 65 GiB |

These are planning recommendations, not cryptographic requirements.

## 5. Create a Nested VeraCrypt Container

Mount the outer encrypted device.

Verify its mount point:

```bash
veracrypt -t --list
```

The following example assumes the outer volume is mounted at `/media/veracrypt1`.

In VeraCrypt:

1. Select Create Volume.
2. Select Create an encrypted file container.
3. Select Standard VeraCrypt volume.
4. Specify `/media/veracrypt1/Vault.hc` as the file location.
5. Select AES.
6. Select Argon2 as the key derivation function.
7. Specify the required container capacity.
8. Verify the selected capacity unit: MB or GB.
9. Enter a new, independent passphrase.
10. Leave PIM at its default.
11. Disable keyfiles unless intentionally required.
12. Select exFAT for Windows and Linux compatibility.
13. Enable cross-platform support if prompted.
14. Collect randomness by moving the mouse.
15. Select Format.

Do not reuse the outer volume's passphrase.

## 6. Mount the Inner Container

Keep the outer volume mounted.

In VeraCrypt:

1. Select another available slot.
2. Select File.
3. Locate `Vault.hc` inside the outer volume.
4. Select Mount.
5. Enter the inner container's passphrase.
6. Select the corresponding key derivation function.

Verify both mounted volumes:

```bash
veracrypt -t --list
df -h /media/veracrypt{1,2}
```

Example:

```text
1: /dev/sdX /dev/mapper/veracrypt1 /media/veracrypt1
2: /media/veracrypt1/Vault.hc /dev/mapper/veracrypt2 /media/veracrypt2
```

Mount points and slot numbers may differ between systems.

## 7. Migrate Selected Data

Copy the information requiring additional encryption into the inner volume:

```bash
rsync -rt --info=progress2 \
  /path/to/sensitive-data/ \
  /media/veracrypt2/
```

Verify the copied files:

```bash
rsync -rnc --itemize-changes \
  /path/to/sensitive-data/ \
  /media/veracrypt2/
```

A successful comparison with no reported differences indicates that the compared file contents match.

For individual files, compare SHA-256 checksums:

```bash
sha256sum /path/to/original-file
sha256sum /media/veracrypt2/copied-file
```

Both checksums should match.

For Linux system files requiring preservation of permissions, symbolic links and extended attributes, use the TAR archive procedure described in the backup and migration guide.

Do not remove the original data until the encrypted copy and recovery process have been verified.

## 8. Verify Independent Encryption

1. Create a non-sensitive test file inside the inner container.
2. Unmount the inner container only.
3. Confirm the outer volume remains accessible.
4. Confirm the inner filesystem is no longer mounted.
5. Attempt to unlock the inner container using incorrect credentials.
6. Confirm access is rejected.
7. Unlock using the correct credentials.
8. Verify the test file remains intact.

Check mounted volumes:

```bash
veracrypt -t --list
```

This demonstrates that the two encryption boundaries operate independently.

## 9. Selective Encryption Without Full-Device Encryption

A VeraCrypt container can also be created on an ordinary, unencrypted storage device.

Example architecture:

```text
Unencrypted MicroSD
├── Public Documents
├── General Media
└── Vault.hc
    └── Encrypted Sensitive Information
```

Procedure:

1. Connect an ordinarily formatted storage device.
2. Calculate the size of the sensitive information.
3. Create a standard VeraCrypt file container on the device.
4. Configure AES and Argon2.
5. Select an appropriate capacity.
6. Format and mount the container.
7. Copy sensitive files into it.
8. Verify the copied information.
9. Unmount the container.

Other files on the device remain unencrypted.

This configuration does not require formatting the entire storage device, provided sufficient free space exists.

## 10. Unmount Nested Volumes

Always unmount the inner container before the outer device.

Verify that applications and terminals are no longer accessing either volume.

After unmounting both volumes:

```bash
veracrypt -t --list
lsblk -e 7 -o NAME,SIZE,FSTYPE,MOUNTPOINTS
```

Verify the physical device identifier before safely disconnecting it:

```bash
udisksctl power-off -b /dev/sdX
```

## 11. Security Considerations

- Use independent passphrases for each encryption layer.
- Additional encryption layers do not compensate for weak credentials.
- Do not cache passwords unnecessarily.
- Maintain verified encrypted backups.
- Container capacity is fixed at creation unless a supported expansion procedure is used.
- Deleted plaintext may remain recoverable from previously used storage.
- AES-XTS does not independently authenticate encrypted storage against modification.
- A compromised host may access information when its corresponding volume is unlocked.
- Nested encryption does not automatically double the effective cryptographic security strength.

For authenticated encryption of individual files, use age or GnuPG as described in the file encryption guide.

## Completion Criteria

- Appropriately sized encrypted container created.
- Independent passphrase configured.
- Container successfully mounted and unmounted.
- Sensitive files migrated and verified.
- Independent encryption boundaries tested.
- Recovery procedures documented.

## References

- [VeraCrypt Documentation](https://veracrypt.io/en/Documentation.html)
- [VeraCrypt Beginner's Tutorial](https://veracrypt.io/en/Beginner%27s%20Tutorial.html)
- [Argon2 Specification — RFC 9106](https://www.rfc-editor.org/rfc/rfc9106.html)

---

[Previous: Full-Device Encryption](03-full-device-encryption.md) | [Return to README](../README.md) | [Next: File Encryption](05-file-encryption.md)
