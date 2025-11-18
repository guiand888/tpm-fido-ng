# This is a community-driven fork of the original WebAuthn/U2F token project, now actively maintained and updated. All dependencies are patched to their latest secure versions, outstanding pull requests are integrated, and the codebase is regularly reviewed for security and compatibility. The goal is to provide a reliable, up-to-date implementation of a TPM-protected WebAuthn/U2F token for modern Linux systems.

# tpm-fido

tpm-fido is FIDO token implementation for Linux that protects the token keys by using your system's TPM. tpm-fido uses Linux's [uhid](https://github.com/psanford/uhid) facility to emulate a USB HID device so that it is properly detected by browsers.

##  Implementation details

tpm-fido uses the TPM 2.0 API. The overall design is as follows:

On registration tpm-fido generates a new P256 primary key under the Owner hierarchy on the TPM. To ensure that the key is unique per site and registration, tpm-fido generates a random 20 byte seed for each registration. The primary key template is populated with unique values from a sha256 hkdf of the 20 byte random seed and the application parameter provided by the browser.

A signing child key is then generated from that primary key. The key handle returned to the caller is a concatenation of the child key's public and private key handles and the 20 byte seed.

On an authentication request, tpm-fido will attempt to load the primary key by initializing the hkdf in the same manner as above. It will then attempt to load the child key from the provided key handle. Any incorrect values or values created by a different TPM will fail to load.

## Status

tpm-fido has been tested to work with Chrome and Firefox on Linux.

## Building

```
# in the root directory of tpm-fido run:
go build
```

**Requirements**: Go 1.22 or later

## Running

In order to run `tpm-fido` you will need permission to access `/dev/tpmrm0`. On Ubuntu and Arch, you can add your user to the `tss` group.

Your user also needs permission to access `/dev/uhid` so that `tpm-fido` can appear to be a USB device.
I use the following udev rule to set the appropriate `uhid` permissions:

```
KERNEL=="uhid", SUBSYSTEM=="misc", GROUP="SOME_UHID_GROUP_MY_USER_BELONGS_TO", MODE="0660"
```

To ensure the above udev rule gets triggered, I also add the `uhid` module to `/etc/modules-load.d/uhid.conf` so that it loads at boot.

To run:

```
# as a user that has permission to read and write to /dev/tpmrm0:
./tpm-fido
```
Note: do not run with `sudo` or as root, as it will not work.

## Dependencies

tpm-fido requires `pinentry` to be available on the system. If you have gpg installed you most likely already have `pinentry`.

## Cloning with Internal Standards

This repository includes a private `dev/` Git submodule containing internal development standards and AI rules. To clone the full repository including this submodule:

```bash
git clone --recurse-submodules <repository-url>
```

If you've already cloned without the submodule, initialize it with:

```bash
git submodule update --init --recursive
```

For more information about the dev submodule, see [`dev/README.md`](dev/README.md) and [`dev/SETUP.md`](dev/SETUP.md).
