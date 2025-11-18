# Repository Brief

## Purpose

tpm-fido is a Linux FIDO/U2F/WebAuthn token implementation that stores and protects private keys inside the system TPM and exposes a virtual USB HID authenticator via UHID so browsers can use it. See [Readme.md](../../../Readme.md).

## High Level Architecture

### Core Components

- **Core binary**: `tpmfido.go:1` - main event loop; selects backend (TPM or memory), handles requests, and routes register/authenticate/version flows.
- **TPM backend**: `tpm/tpm.go:1` - creates per-registration primary key using HKDF-seeded template, derives a signing child key, and performs signing with tpm2.
- **HID emulation**: `fidohid/fidohid.go:1` - implements U2F/CTAP framing and exposes a UHID device so browsers detect it as an authenticator.
- **Auth protocol parsing/defs**: `fidoauth/fidoauth.go:1`
- **User confirmation**: `pinentry/pinentry.go:1` - launches pinentry GUI to confirm user presence.
- **Attestation**: `attestation/attestation.go:1` - contains attestation cert and private key (see Security notes).

## Design Highlights

### Key Features

- **Per-registration uniqueness**: generates a 20-byte seed for each registration and uses HKDF with the browser’s application parameter to create unique primary key templates so keys are bound to TPM + origin.
- **Key handles**: key handle returned to the browser encodes the child key handles plus the 20-byte seed so the token can reload keys only on the same TPM.
- **UHID integration**: presents as a HID authenticator for browser compatibility (Chrome/Firefox supported per README).
- **Modes**: supports an in-memory backend for testing (memory/) and a TPM-backed backend for production.

## Build / Run / Dependencies

### Quick Reference

- **Go project**: build with `go build` in repo root (see [Readme.md](../../../Readme.md)).
- **Runtime requirements**: access to `/dev/tpmrm0` and `/dev/uhid` (udev rule recommended). Requires pinentry GUI binary present in PATH. README provides udev guidance. See [Readme.md](../../../Readme.md) and `tpmfido.go:69`.

## Current Status & Notable Findings

### Project Status

- Project is active and intended to work with Chrome and Firefox on Linux (README).

### Security Notes

- **Security concern**: attestation private key is embedded in plaintext inside `attestation/attestation.go:14`. This is a sensitive secret and should not be committed to the repo.
- **Good cryptographic practice**: the TPM code uses HKDF, tpm2 APIs, and creates restricted primary templates (see `tpm/tpm.go:48` and signing flow `tpm/tpm.go:211`).

### Implementation Details

- UHID descriptor and U2F framing are implemented in `fidohid/fidohid.go:212` and framing constants at `fidohid/fidohid.go:142`.
