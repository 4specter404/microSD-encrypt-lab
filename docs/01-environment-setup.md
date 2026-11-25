# 01 — Environment Setup

## Objective

Prepare a trusted environment for implementing and testing portable storage encryption.

## 1. Required Software

| Software | Function |
|---|---|
| VeraCrypt | Device and container encryption |
| Cryptsetup | LUKS2 encrypted storage and backups |
| age | Symmetric and asymmetric file encryption |
| GnuPG | OpenPGP encryption and digital signatures |
| rsync | Data migration and backup verification |
| Git | Documentation version control |
## Tested Software Environment

The following software versions were installed in the reference Linux environment.

| Component | Version |
|---|---|
| Operating system | Ubuntu 26.04.1 LTS |
| Linux kernel | 7.0.0-44-generic |
| VeraCrypt | 1.26.31 |
| Cryptsetup | 2.8.4 |
| age | 1.2.1 |
| GnuPG | 2.4.8 |
| Libgcrypt | 1.12.0 |
| rsync | 3.4.1 |
| Git | 2.53.0 |

These versions provide a reference configuration for reproducibility and troubleshooting. They are not mandatory minimum versions.

The core encryption procedures were validated on native Linux. Windows interoperability and additional GnuPG exercises remain untested.
## 2. Environment Selection

Native Linux is recommended for direct access to removable storage devices.

Virtual machines using VirtualBox or KVM/QEMU are optional for isolated experimentation. Virtual disk images can also simulate physical storage devices without requiring a virtual machine.

## 3. Install Linux Dependencies

```bash
sudo apt update
sudo apt install cryptsetup age gnupg rsync git exfatprogs
```

Verify installation:

```bash
cryptsetup --version
age --version
gpg --version
rsync --version
git --version
```

## 4. Install VeraCrypt

Download the appropriate operating system package and corresponding PGP signature from the official website:

https://veracrypt.io/en/Downloads.html

Download the official VeraCrypt public signing key using the link provided by the developer.

### Verify the Signing Key

```bash
gpg --show-keys --with-fingerprint VeraCrypt_PGP_public_key.asc
```

Compare the fingerprint against the developer's officially published fingerprint before continuing.

### Verify the Installer

```bash
gpg --import VeraCrypt_PGP_public_key.asc
gpg --verify PACKAGE.deb.sig PACKAGE.deb
```

Replace `PACKAGE` with the downloaded package filename.

Expected result: a valid signature from the VeraCrypt signing key.

A valid signature establishes integrity relative to the signing key. Independently verifying the key fingerprint establishes confidence in its authenticity.

Do not install software if signature verification fails.

### Install the Verified Package

```bash
sudo apt install ./PACKAGE.deb
veracrypt --version
```

Launch VeraCrypt:

```bash
veracrypt
```

Use a release supporting Argon2id if following the recommended encryption configuration.

## 5. Identify the Storage Device

```bash
lsblk -e 7 -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS,MODEL,TRAN
```

Run the command before and after connecting the removable storage device.

Identify the target using its capacity, model and connection type.

Never assume that a device will always have the same identifier. Device names can change after rebooting or reconnecting hardware.

## 6. Windows Compatibility

Install VeraCrypt from the official website and verify the installer using the available signature verification procedures.

Ensure that both computers use compatible VeraCrypt versions supporting the selected encryption algorithm and key derivation function.

For cross-platform storage, exFAT is recommended when Linux-specific filesystem metadata is not required.

Windows compatibility should be verified before transferring sensitive data.

## 7. Host Security Considerations

- Keep operating systems and encryption software updated.
- Avoid untrusted computers.
- Protect private keys and passwords.
- Investigate unencrypted swap and temporary storage.
- Disable unnecessary password caching.
- Lock encrypted volumes before disconnecting storage.
- Never store production credentials in project documentation.

## Completion Criteria

- Required software installed.
- VeraCrypt installer authenticity verified.
- Storage device positively identified.
- Compatible encryption software available on recipient computers.

## Official References

- [VeraCrypt Documentation](https://veracrypt.io/en/Documentation.html)
- [Cryptsetup](https://gitlab.com/cryptsetup/cryptsetup)
- [age](https://github.com/FiloSottile/age)
- [GnuPG](https://gnupg.org/documentation/)
- [Git Documentation](https://git-scm.com/doc)

---

[Return to README](../README.md) | [Next: Backup and Migration](02-backup-and-migration.md)
