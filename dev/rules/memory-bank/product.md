
# Product

## Why This Project Exists

tpm-fido is a community-driven fork of the original WebAuthn/U2F token project, actively maintained with all dependencies patched to latest secure versions. It provides a reliable, up-to-date implementation of a TPM-protected WebAuthn/U2F token for modern Linux systems.

## Problems It Solves

1. **Key Protection**: Protects FIDO token private keys by storing them inside the system TPM (Trusted Platform Module), preventing key extraction even if the system is compromised.
2. **Browser Compatibility**: Emulates a USB HID authenticator device so browsers (Chrome, Firefox) properly detect and use it for WebAuthn/U2F authentication.
3. **Per-Site Uniqueness**: Generates unique keys per registration and site, binding keys to both the TPM and the origin, preventing key reuse across sites.
4. **Dependency Security**: Maintains current versions of all dependencies with security patches applied.

## How It Should Work

### User Flow

1. **Registration**: User initiates WebAuthn registration on a website
   - Browser detects tpm-fido as a USB HID authenticator
   - User confirms presence via pinentry GUI dialog
   - tpm-fido generates a unique P256 key pair in the TPM
   - Returns public key and attestation certificate to browser
   - Browser stores the key handle for future authentication

2. **Authentication**: User initiates WebAuthn authentication
   - Browser sends challenge and key handle to tpm-fido
   - User confirms presence via pinentry GUI dialog
   - tpm-fido loads the key from TPM using the key handle
   - Signs the challenge with the private key
   - Returns signature to browser
   - Browser verifies signature and completes authentication

### Technical Flow

- **Registration**: Generate 20-byte seed → HKDF with app parameter → Create primary key in TPM → Create signing child key → Return key handle (encoded with seed and key handles)
- **Authentication**: Decode key handle → Recreate primary key using seed and app parameter → Load child key → Sign challenge → Return signature

## User Experience Goals

1. **Seamless Integration**: Works transparently with standard WebAuthn APIs in browsers
2. **User Confirmation**: Clear, simple pinentry dialogs for user presence confirmation
3. **Security by Default**: No manual key management; keys are protected by TPM automatically
4. **Reliability**: Works consistently with Chrome and Firefox on Linux
5. **No Root Required**: Runs as regular user (not sudo/root) with proper udev permissions
