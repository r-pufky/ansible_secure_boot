# Secure Boot (mokutil)
Secure boot enabled systems require the MOK (Machine Owner's Key) for signed
DKMS modules to be loaded into the kernel, otherwise errors like the following
will occur:

``` bash
/sbin/modprobe zfs
> modprobe: ERROR: could not insert 'zfs': Key was rejected by service
```

Certificates must be added to secure boot certificate store before enabling
secure boot; intentionally **requires** physical presence (or pre-existing keys
to be installed during bare-metal turn-up via firmware/BIOS).

`mokutil` manages this process and generates changes for MOK Manager (loaded by
the secure boot shim) to process on next reboot.

## Manual Enable Secure Boot

``` bash
mokutil --sb-state
> SecureBoot disabled
> Platform is in Setup Mode
mokutil --test-key /var/lib/dkms/mok.pub
> /var/lib/dkms/mok.pub is not enrolled
mokutil --import /var/lib/dkms/mok.pub
mokutil --list-new
mokutil --enable-validation
reboot
```
On reboot enter the same password and confirm the new keys to be loaded. The
system will reboot again with the new keys.

DKMS will automatically use keys from `/var/lib/dkms/mok.*` with the default
Debian configuration `/etc/dkms/framework.conf`.

Reference:
* https://wiki.debian.org/SecureBoot#DKMS_and_secure_boot
* https://github.com/jiazhang0/meta-efi-secure-boot/blob/master/README.md#mokutil-and-mok-manager
* https://media.defense.gov/2023/Mar/20/2003182401/-1/-1/0/CTR-UEFI-SECURE-BOOT-CUSTOMIZATION-20230317.PDF

## Secure Boot (MOK) Password
The `mokutil` password is not stored directly as plain text -- it is stored as
a hashed value within the shim database on the system, which is accessed by the
`mokutil` utility to manage Machine Owner Keys (MOKs) used for Secure Boot.
When you enter the password, it is hashed and compared against the stored hash
in the shim database.

Password:
* Must be between **8-16** alphanumeric characters.
* Manually entered on console hardware for confirming secure boot **changes**.
* This is the password to enroll keys on the secure boot loader during boot.
* This is the password to enable/disable secure boot.
* Unique as it is stored hashed in firmware.

### Enrollment Process
After enrolling a certificate (MOK) on reboot, Secure Boot will first validate
Microsoft's shimx64.efi Secure Boot loader signature, then shimx64.efi will
detect a MOK enrollment request. Boot process is interrupted and the blue MOK
Manager screen appears. The password must be entered to completed the MOK
enrollment and store the certificate in firmware. Reboot is forced to load the
new certificate.

Validates:
* You are the user that started the process.
* You really want to complete the MOK registration process.

## Commands
Commands which change state (validation, reset, enrollment, etc) require a
reboot and manual confirmation of changes.

### --sb-state
Returns the current hardware state

``` bash
mokuil --sb-state
> SecureBoot disabled
> Platform is in Setup Mode
```
* `SecureBoot` can be either **enabled** or **disabled**; this is signature
  validation.
* `Platform ...` shows firmware state; **setup mode** means BIOS enrollment is
  in setup mode; meaning that secure boot options can be configured in the
  BIOS. This should be set to `enabled`.

### --test-key
Check if a key is enrolled
``` bash
mokutil --test-key /var/lib/dkms/mok.pub
> /var/lib/dkms/mok.pub is already enrolled
```
Exit Codes:
* `0`: not enrolled
* `1`: already enrolled
* `255`: invalid x509 format

### --generate-hash
Generate a valid password hash which may be loaded into firmware
``` bash
mokutil --generate-hash={PASS}
> $6$DtG...$6Xjc...
```

### --import
Enrolls key secure boot. The key must be loaded before secure boot is enabled;
otherwise the system may fail to boot. You can **enroll** and **enable** before
rebooting, allowing this to be completed in one step.

``` bash
mokutil --import /var/lib/dkms/mok.pub
```

### --enable-validation
Enables secure boot signature validation. System will halt when unsigned
modules are loaded.

A system can have secure boot enabled but not enforce signature verification.
``` bash
mokutil --enable-validation
mokutil --disable-validation
```
[Reference](https://github.com/lcp/mokutil/issues/45)

## Manually generate MOK
DKMS automatically generates a MOK key when secure boot is enabled and modules
require it; a key may not exist until these conditions are met.

Manually generate a key with the same parameters:
``` bash
  openssl req -nodes -new -x509 -newkey rsa:2048 -outform DER -days 36500 -subj "/CN=DKMS module signing key/" -keyout mok.key -out mok.pub
```

## List MOK certificate
Useful to determine the current auto generated MOK key constraints or verifying
keys.

```bash
openssl x509 -inform der -noout -text -in mok.pub
```

## Set timeout
Skipping enrollment will cause enrollment requests to be deleted and
potentially putting the system into an unbootable state until secure boot is
disabled and the keys are re-enrolled. Disable timeout when MOK Manager has
changes to process, preventing the system from skipping enrollment.

``` bash
mokutil --timeout -1
```
