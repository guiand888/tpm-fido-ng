
# Technology Stack

## Languages & Runtimes

- **Go 1.22** - Primary language (upgraded from 1.16 in Phase 2)
- **Linux** - Target OS (UHID and TPM 2.0 are Linux-specific)

## Core Dependencies

### Cryptography & TPM

- **`github.com/google/go-tpm v0.3.2`** - TPM 2.0 API bindings
  - Provides tpm2 package for TPM operations
  - Used for primary key creation, child key creation, and signing
  - Communicates with `/dev/tpmrm0` (TPM 2.0 resource manager)
  - **Status**: Pinned to v0.3.2 (from 2021); newer versions available

- **`golang.org/x/crypto v0.0.0-20210513164829-c07d793c2f9a`** - Standard cryptography library
  - HKDF for key derivation
  - ChaCha20-Poly1305 for memory backend encryption
  - ECDSA signing utilities
  - **Status**: Pinned to May 2021 snapshot; should be updated to latest

### HID & Device Emulation

- **`github.com/psanford/uhid v0.0.0-20210426002309-4864eff247db`** - UHID device emulation
  - Creates virtual USB HID device via `/dev/uhid`
  - Handles HID frame assembly/disassembly
  - Manages device initialization and feature requests
  - **Status**: Pinned to April 2021 snapshot; check for updates

### User Interaction

- **`github.com/foxcpp/go-assuan v1.0.0`** - Assuan protocol client
  - Communicates with pinentry for user confirmation dialogs
  - Sends challenge and application parameter to GUI
  - **Status**: v1.0.0 (stable); check for newer versions

## System Dependencies

### Runtime Requirements

- **TPM 2.0 Device** - `/dev/tpmrm0` (TPM 2.0 resource manager)
  - Must be accessible by user (typically via `tss` group membership)
  - Provides secure key storage and cryptographic operations

- **UHID Module** - Linux kernel module for HID emulation
  - Provides `/dev/uhid` device
  - Must be loaded: `modprobe uhid`
  - Requires udev rule for proper permissions

- **Pinentry** - GUI dialog for user confirmation
  - Must be available in PATH
  - Typically installed with GnuPG
  - Used for WebAuthn user presence confirmation

### Build Requirements

- Go 1.22+ compiler
- Standard C build tools (for cgo if needed)
- Linux kernel headers (for UHID support)

## Development Setup

### Prerequisites

1. Linux system with TPM 2.0 support
2. Go 1.22+ installed
3. User in `tss` group for TPM access
4. Pinentry GUI binary in PATH
5. UHID module loaded

### Build

```bash
cd /home/guillaume/Development/tpm-fido-ng
go build
```

### Run

```bash
# TPM backend (default)
./tpm-fido

# Memory backend (for testing)
./tpm-fido -backend memory

# Custom TPM device path
./tpm-fido -device /dev/tpmrm0
```

### Udev Configuration

Add to `/etc/udev/rules.d/99-uhid.rules`:
```
KERNEL=="uhid", SUBSYSTEM=="misc", GROUP="YOUR_GROUP", MODE="0660"
```

Add to `/etc/modules-load.d/uhid.conf`:
```
uhid
```

## Technical Constraints

### TPM Constraints

- **Single-threaded**: TPM device access is serialized via mutex
- **Key size**: P256 elliptic curve (256-bit keys)
- **Signing algorithm**: ECDSA with SHA256
- **Primary key template**: Restricted, uses AES-128-CFB symmetric encryption
- **Child key**: Signing-only, cannot be used for encryption

### HID Constraints

- **Max packet size**: 64 bytes per HID frame
- **Frame types**: Init (0x80) and continuation (0x00)
- **Channel ID**: 4-byte identifier for request/response correlation
- **Timeout**: 750ms for user confirmation dialogs

### Key Handle Constraints

- **Max size**: 255 bytes (limited by U2F protocol)
- **Format**: Length-prefixed encoding with separator
- **Contents**: Private key blob + public key blob + 20-byte seed

### Attestation Constraints

- **Certificate**: GitHub SoftU2F certificate (shared across all instances)
- **Private key**: Embedded in source code (SECURITY CONCERN)
- **Signature**: ECDSA-SHA256 over registration response

## Dependency Management

### Current Status

- Go 1.22 (released Dec 2023) - **Current**
- Dependencies pinned to specific versions in go.mod
- Most dependencies are from 2021 and need updating for security patches
- No automatic updates configured

### Recommended Actions

1. **Update dependencies** to latest secure versions (Phase 3):
   - `github.com/google/go-tpm` - Update from v0.3.2 to latest
   - `golang.org/x/crypto` - Update from May 2021 snapshot to latest
   - `github.com/psanford/uhid` - Update from April 2021 snapshot to latest
   - `github.com/foxcpp/go-assuan` - Check for newer versions beyond v1.0.0

2. **Set up dependency scanning** for security vulnerabilities
3. **Document upgrade path** for future maintenance

## Testing Infrastructure

### Current State

- Unit tests exist for some components (e.g., `attestation_test.go`, `lencode_test.go`)
- No integration tests documented
- No CI/CD pipeline visible in repository

### Recommended Additions

- Integration tests for registration/authentication flows
- TPM backend tests (may require TPM device)
- Memory backend tests (can run without TPM)
- HID frame assembly/disassembly tests
- End-to-end browser compatibility tests

## Security Considerations

### Current Issues

1. **Attestation private key in source**: Embedded plaintext in [`attestation/attestation.go:14`](../../attestation/attestation.go:14)
   - Should be externalized using SOPS or similar
   - Never commit private keys to repository

2. **Shared attestation certificate**: All instances use same certificate
   - Reduces tracking ability but may be privacy concern
   - Consider per-instance certificates for production

### Best Practices Implemented

- HKDF for key derivation (per-site uniqueness)
- Restricted TPM primary keys (security boundary)
- ChaCha20-Poly1305 for memory backend (authenticated encryption)
- Mutex protection for TPM access (thread safety)
- Context-based timeouts (prevents indefinite blocking)

## Performance Characteristics

### TPM Operations

- **RegisterKey**: ~100-200ms (TPM key creation + loading)
- **SignASN1**: ~50-100ms (TPM key loading + signing)
- **Counter**: <1ms (time-based counter)

### Memory Backend

- **RegisterKey**: <1ms (in-memory key generation)
- **SignASN1**: <1ms (in-memory signing)
- **Counter**: <1ms (counter increment)

### HID Communication

- **Frame assembly**: <1ms per frame
- **User confirmation timeout**: 750ms
- **Total registration**: ~1-2 seconds (including user interaction)
- **Total authentication**: ~1-2 seconds (including user interaction)

## Deployment Considerations

### Production Deployment

- Use TPM backend (not memory backend)
- Ensure TPM 2.0 device is available
- Configure udev rules for UHID access
- Load UHID kernel module at boot
- Add user to `tss` group for TPM access
- Ensure pinentry GUI is available in PATH

### Testing Deployment

- Can use memory backend for testing without TPM
- Useful for CI/CD pipelines
- Not suitable for production (keys stored in memory)

### Container Deployment

- Requires access to host TPM device (`/dev/tpmrm0`)
- Requires access to host UHID device (`/dev/uhid`)
- Requires pinentry binary in container
- May need special Docker/Podman configuration for device access
