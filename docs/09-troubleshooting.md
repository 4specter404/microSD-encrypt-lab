# 09 — Troubleshooting and Error Resolution

## Objective

Provide a systematic troubleshooting methodology for encrypted portable storage without unnecessarily risking existing data or repeating destructive operations.

## 1. Troubleshooting Methodology

1. Identify the symptom.
2. Collect the exact error message.
3. Identify the affected component.
4. Test possible causes individually.
5. Apply the appropriate solution.
6. Verify the result.

Avoid reformatting encrypted storage until the problem has been identified and important data has been backed up.

## 2. Storage Device Not Detected

List connected storage devices:

```bash
lsblk -e 7 -o NAME,SIZE,FSTYPE,MOUNTPOINTS,MODEL,TRAN
```

List USB devices:

```bash
lsusb
```

Inspect recent kernel messages:

```bash
sudo journalctl -k -b --no-pager | tail -50
```

Possible causes:

- Defective card reader.
- Loose USB connection.
- Damaged storage device.
- Insufficient USB power.
- Unsupported hardware.
- Device identifier changed.

Try another USB port or compatible reader before concluding that the storage device has failed.

## 3. Device Contains Existing Partitions

VeraCrypt may refuse whole-device encryption when existing partitions are present.

Inspect the device:

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS /dev/sdX
```

If the entire device is intended for encryption, preserve and verify its existing data before removing partition signatures.

**WARNING: The following commands are destructive to the existing partition layout.**

```bash
sudo wipefs --all /dev/sdX
sudo partprobe /dev/sdX
```

Replace the placeholder with the positively identified removable device.

Refer to the backup and migration guide before performing this operation.

## 4. VeraCrypt Volume Fails to Mount

Possible causes include:

- Incorrect passphrase.
- Incorrect PIM.
- Missing keyfile.
- Incorrect key derivation function.
- Incompatible VeraCrypt version.
- Corrupted volume header.
- Storage hardware problems.
- Incorrect device selection.

Verify each parameter individually.

If autodetection fails, explicitly select the key derivation function used during volume creation.

Attempt read-only mounting with the embedded backup header if appropriate.

Do not immediately reformat the device.

## 5. VeraCrypt Key Derivation Error

Example error:

```text
ParameterIncorrect at VeraCrypt::Pkcs5Kdf::ValidateParameters:97
```

This indicates that the implementation rejected a key derivation parameter.

It does not establish that SHA-512 or PBKDF2 is cryptographically broken.

### Isolation Procedure

1. Leave the physical storage device unchanged.
2. Create a small disposable VeraCrypt file container.
3. Configure AES and PBKDF2-HMAC-SHA-512.
4. Attempt to format and mount the container.
5. Create another disposable container using Argon2id.
6. Compare the results.

If one configuration succeeds while another fails, investigate software compatibility and parameter handling.

A working Argon2id configuration may provide an alternative when supported by the intended recipient's VeraCrypt version.

The precise cause of this error may depend on the software release and configuration. Do not assume that a successful workaround establishes the underlying cause.

## 6. VeraCrypt Volume Does Not Appear in a Slot

Creating a volume does not automatically mount it.

Select the encrypted device or container and mount it using the correct credentials.

List active volumes:

```bash
veracrypt -t --list
```

An empty slot indicates that no volume is mounted in that slot.

The Never Save History setting controls retention of previously selected paths. It does not prevent currently mounted volumes from appearing.

## 7. Incorrect Container Capacity

Verify the selected size unit during volume creation.

For example:

- 10 MiB is approximately 10 megabytes.
- 10 GiB is 10,240 MiB.

Inspect mounted capacity:

```bash
df -h /media/veracrypt2
```

If a container is too small, create a correctly sized replacement and migrate any required files.

Verify the replacement before removing the original container.

## 8. Device Is Busy or Cannot Be Unmounted

Close applications accessing the mounted filesystem.

Move any terminal sessions outside the mount directory:

```bash
cd ~
```

Identify processes accessing the filesystem:

```bash
sudo fuser -vm /media/veracrypt1
```

Close the relevant applications normally.

For nested encryption, unmount the inner volume before the outer volume.

Avoid forced unmounting during active write operations.

## 9. Storage Hardware Errors

Inspect relevant kernel messages:

```bash
sudo journalctl -k -b --no-pager | \
  grep -iE 'I/O error|reset.*USB|write protect|sdX' | tail -50
```

Replace `sdX` with the actual device name.

Potential indicators include:

- Input/output errors.
- Repeated USB resets.
- Unexpected device disconnections.
- Write protection.
- Reported capacity changes.

The absence of kernel errors does not conclusively establish that the storage hardware is healthy.

If hardware failure is suspected, prioritize recovery and avoid unnecessary write operations.

## 10. Windows Requests to Format the Device

Windows may not recognize a locked VeraCrypt volume as an ordinary filesystem.

Cancel any formatting or initialization prompts.

Open the device through a compatible version of VeraCrypt.

The decrypted filesystem should become accessible after successful authentication.

## 11. Cryptsetup Authentication Errors

Example:

```text
No key available with this passphrase.
```

Verify that the correct LUKS passphrase is being entered.

The Ubuntu administrator password and the LUKS encryption passphrase are separate credentials.

Check administrator authentication:

```bash
sudo -v
```

Then attempt to unlock the encrypted backup:

```bash
sudo cryptsetup open --readonly \
  private/backup.img cryptolab_backup
```

## 12. Cryptsetup Device Already Exists

Example:

```text
Device cryptolab_backup already exists.
```

Inspect its status:

```bash
sudo cryptsetup status cryptolab_backup
```

An active mapping may indicate that the previous unlock operation succeeded.

Do not create another encrypted filesystem over an existing mapping.

Inspect its mount status:

```bash
findmnt /dev/mapper/cryptolab_backup
```

If the filesystem is not mounted, mount it using the appropriate options.

## 13. age Passphrase Errors

Possible error messages:

```text
age: error: incorrect passphrase
age: error: passphrase can't be empty
```

Verify the original encryption passphrase.

Age can generate a passphrase during encryption if the user leaves the passphrase prompt empty. That generated passphrase must be retained to decrypt the file.

For disposable test data, repeat the encryption and decryption procedure using a new output filename.

Do not overwrite the only copy of important encrypted information.

## 14. age Authentication Failure

Example:

```text
age: error: failed to decrypt and authenticate payload chunk
```

Possible causes:

- Modified ciphertext.
- Corrupted file.
- Incomplete file transfer.

Do not trust partially recovered output.

Restore a verified copy of the encrypted file and attempt decryption again.

Authenticated encryption is designed to reject unauthorized modifications.

## 15. Optional: VirtualBox Kernel Driver Problems

VirtualBox may require external kernel modules to operate on Linux.

Example error:

```text
The vboxdrv kernel module is not loaded.
```

Inspect the environment:

```bash
VBoxManage --version
uname -r
lsmod | grep vbox
```

For Oracle VirtualBox installations providing the configuration utility:

```bash
sudo /sbin/vboxconfig
```

Inspect compilation errors:

```bash
sudo grep -inE 'error:|fatal:|failed' \
  /var/log/vbox-setup.log
```

Potential causes include missing kernel headers, incompatible kernel interfaces and Secure Boot module-signing requirements.

Do not disable Secure Boot unnecessarily.

Native Linux or KVM/QEMU can provide alternative environments for encryption experiments.

## 16. Optional: Virtual Machine Manager Python Error

Example:

```text
ModuleNotFoundError: No module named 'gi'
```

Inspect the Python environment:

```bash
python3 --version
ls -l /usr/bin/python3
head -n 1 /usr/bin/virt-manager
/usr/bin/python3 -c "import gi"
```

If the system Python can import the required library but the default interpreter cannot, launch Virtual Machine Manager explicitly:

```bash
/usr/bin/python3 /usr/bin/virt-manager
```

This avoids modifying the system-wide Python configuration.

## 17. Safe Recovery Principles

- Preserve existing data before attempting destructive repairs.
- Use disposable containers for troubleshooting experiments.
- Record software versions and exact error messages.
- Do not repeatedly format physical storage to diagnose software problems.
- Change one configuration variable at a time.
- Verify encryption and decryption after resolving an issue.
- Distinguish confirmed root causes from suspected causes.
- Never disclose credentials in diagnostic logs or screenshots.

## References

- [VeraCrypt Troubleshooting](https://veracrypt.io/en/Troubleshooting.html)
- [VeraCrypt Documentation](https://veracrypt.io/en/Documentation.html)
- [Cryptsetup](https://gitlab.com/cryptsetup/cryptsetup)
- [age](https://github.com/FiloSottile/age)
- [VirtualBox Documentation](https://www.virtualbox.org/manual/)

---

[Previous: Security Testing](08-security-testing.md) | [Return to README](../README.md) | [Next: Cleanup](10-cleanup.md)
