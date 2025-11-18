
# Architecture

## System Architecture

tpm-fido is a Linux FIDO/U2F/WebAuthn token implementation that emulates a USB HID authenticator device. The system consists of a main event loop that routes authentication requests to either a TPM-backed or in-memory backend, with user confirmation via pinentry GUI.

### High-Level Flow

```
Browser Request
    ↓
UHID Device (fidohid) - Receives HID frames
    ↓
FIDO Auth Parser (fidoauth) - Parses U2F/CTAP protocol
    ↓
Main Event Loop (tpmfido.go) - Routes to handler
    ↓
Handler (Register/Authenticate/Version)
    ↓
Backend (TPM or Memory) - Performs cryptographic operations
    ↓
Pinentry - User confirmation (if needed)
    ↓
Response sent back to browser via UHID
```

## Core Components

### 1. Main Entry Point: [`tpmfido.go`](../../tpmfido.go)

**Purpose**: Main event loop and request routing

**Key Responsibilities**:
- Initialize backend (TPM or memory) based on `-backend` flag
- Create UHID device for HID emulation
- Listen for authentication events from UHID
- Route requests to appropriate handlers (Register/Authenticate/Version)
- Manage user confirmation via pinentry

**Key Functions**:
- `main()` - Entry point
- `newServer()` - Initialize server with selected backend
- `run()` - Main event loop
- `handleRegister()` - Process registration requests
- `handleAuthenticate()` - Process authentication requests
- `handleVersion()` - Return U2F version string

**Key Data Structures**:
- `server` - Contains pinentry instance and signer interface
- `Signer` interface - Abstraction for TPM and memory backends

### 2. TPM Backend: [`tpm/tpm.go`](../../tpm/tpm.go)

**Purpose**: TPM 2.0 integration for secure key storage and signing

**Key Responsibilities**:
- Create per-registration primary keys using HKDF-seeded templates
- Derive signing child keys from primary keys
- Perform ECDSA signing operations with TPM
- Ensure keys are bound to TPM + origin via HKDF

**Key Functions**:
- `New(devicePath)` - Initialize TPM connection
- `RegisterKey(applicationParam)` - Generate new key pair for registration
- `SignASN1(keyHandle, applicationParam, digest)` - Sign digest with loaded key
- `Counter()` - Return monotonic counter for signature
- `primaryKeyTmpl(seed, applicationParam)` - Generate primary key template

**Key Technical Details**:
- Uses 20-byte random seed per registration
- HKDF with SHA256 derives unique primary key template from seed + application parameter
- Primary key is restricted (FlagRestricted) and uses AES-128-CFB symmetric encryption
- Child key is signing-only (FlagSign) with ECDSA-SHA256
- Key handle encodes: private key blob + public key blob + 20-byte seed
- On authentication, primary key is recreated from seed and app parameter, then child key is loaded

**Security Model**:
- Keys never leave TPM in plaintext
- Primary key template uniqueness ensures keys are bound to TPM + origin
- Seed is stored in key handle (not secret) but seed alone cannot recreate keys without correct app parameter
- Mutex protects concurrent TPM access

### 3. Memory Backend: [`memory/memory.go`](../../memory/memory.go)

**Purpose**: In-memory key storage for testing (not for production)

**Key Responsibilities**:
- Generate and store P256 key pairs in memory
- Encrypt child private keys using ChaCha20-Poly1305
- Perform ECDSA signing with in-memory keys

**Key Functions**:
- `New()` - Initialize memory backend with random master key
- `RegisterKey(applicationParam)` - Generate and encrypt new key pair
- `SignASN1(keyHandle, applicationParam, digest)` - Decrypt and sign with key
- `Counter()` - Return incrementing counter

**Key Technical Details**:
- Master key is 32-byte random ChaCha20-Poly1305 key
- Each registration generates random P256 key pair
- Child private key is encrypted with ChaCha20-Poly1305 using nonce + AAD (metadata + app parameter)
- Key handle = nonce + encrypted private key
- On authentication, key is decrypted and used for signing

### 4. HID Emulation: [`fidohid/fidohid.go`](../../fidohid/fidohid.go)

**Purpose**: USB HID device emulation via UHID

**Key Responsibilities**:
- Create virtual USB HID device
- Handle HID initialization and feature requests
- Parse incoming HID frames (init and continuation frames)
- Assemble fragmented messages
- Send responses back to browser

**Key Technical Details**:
- Uses psanford/uhid library for UHID device creation
- Implements U2F HID framing protocol
- Frame types: init (0x80) and continuation (0x00)
- Max packet size: 64 bytes
- Handles CmdInit, CmdMsg, CmdError, CmdKeepAlive
- Maintains channel ID for request/response correlation

### 5. FIDO Auth Protocol: [`fidoauth/fidoauth.go`](../../fidoauth/fidoauth.go)

**Purpose**: U2F/CTAP protocol parsing and definitions

**Key Responsibilities**:
- Parse incoming U2F/CTAP commands
- Define command types and structures
- Extract registration and authentication parameters

**Key Commands**:
- `CmdRegister` (0x01) - Registration request
- `CmdAuthenticate` (0x02) - Authentication request
- `CmdVersion` (0x03) - Version query

### 6. User Confirmation: [`pinentry/pinentry.go`](../../pinentry/pinentry.go)

**Purpose**: GUI-based user presence confirmation

**Key Responsibilities**:
- Launch pinentry GUI dialog
- Display challenge and application parameter
- Return user confirmation result

**Key Functions**:
- `ConfirmPresence()` - Launch pinentry dialog and return result channel

### 7. Attestation: [`attestation/attestation.go`](../../attestation/attestation.go)

**Purpose**: Attestation certificate and private key for registration responses

**Key Details**:
- Uses GitHub's SoftU2F attestation certificate (for privacy/anonymity)
- Contains embedded attestation private key (SECURITY CONCERN - should be externalized)
- Used to sign registration responses

### 8. Supporting Utilities

- [`sitesignatures/sitesignatures.go`](../../sitesignatures/sitesignatures.go) - Derive site signatures from application parameters
- [`statuscode/statuscode.go`](../../statuscode/statuscode.go) - U2F status codes
- [`internal/lencode/lencode.go`](../../internal/lencode/lencode.go) - Length-prefixed encoding for key handles

## Key Technical Decisions

### 1. Per-Registration Uniqueness via HKDF

**Decision**: Use HKDF with 20-byte seed + application parameter to derive unique primary key templates

**Rationale**:
- Ensures keys are bound to both TPM and origin
- Seed is stored in key handle (not secret)
- Seed alone cannot recreate keys without correct application parameter
- Prevents key reuse across sites

### 2. Restricted Primary Keys

**Decision**: Primary keys use TPM FlagRestricted with symmetric encryption

**Rationale**:
- Restricted keys cannot be used for general-purpose encryption
- Ensures keys can only be used for their intended purpose
- Provides additional security boundary

### 3. Mutex-Protected TPM Access

**Decision**: Single mutex protects all TPM operations

**Rationale**:
- TPM device is single-threaded
- Prevents concurrent access issues
- Serializes all cryptographic operations

### 4. Attestation Certificate Reuse

**Decision**: Use GitHub's SoftU2F attestation certificate

**Rationale**:
- Reduces ability to track individual users
- Provides privacy through certificate sharing
- Simplifies deployment (no per-device certificates)

## Design Patterns

### 1. Signer Interface

```go
type Signer interface {
    RegisterKey(applicationParam []byte) ([]byte, *big.Int, *big.Int, error)
    SignASN1(keyHandle, applicationParam, digest []byte) ([]byte, error)
    Counter() uint32
}
```

**Purpose**: Abstraction allowing multiple backend implementations (TPM, memory)

**Implementations**:
- `tpm.TPM` - TPM 2.0 backend
- `memory.Mem` - In-memory backend

### 2. Event-Driven Architecture

**Pattern**: Main loop receives events from UHID device and dispatches to handlers

**Benefits**:
- Asynchronous request handling
- Clean separation of concerns
- Easy to add new command types

### 3. Context-Based Timeouts

**Pattern**: Use Go context with timeouts for user confirmation

**Benefits**:
- Prevents indefinite blocking on pinentry
- Graceful timeout handling
- Cancellation propagation

## Critical Implementation Paths

### Registration Flow

1. Browser sends registration request via UHID
2. `handleRegister()` launches pinentry for user confirmation
3. User confirms in GUI dialog
4. `registerSite()` calls `signer.RegisterKey(appParam)`
5. TPM backend:
   - Generates 20-byte random seed
   - Creates primary key template using HKDF(seed, appParam)
   - Creates primary key in TPM
   - Creates signing child key under primary key
   - Returns key handle (encoded with seed + key blobs)
6. Registration response signed with attestation private key
7. Response sent back to browser

### Authentication Flow

1. Browser sends authentication request with key handle via UHID
2. `handleAuthenticate()` validates key handle by attempting dummy signature
3. If valid, launches pinentry for user confirmation
4. User confirms in GUI dialog
5. `handleAuthenticate()` constructs data to sign (appParam + userPresent + counter + challenge)
6. TPM backend:
   - Decodes key handle to extract seed and key blobs
   - Recreates primary key template using HKDF(seed, appParam)
   - Creates primary key in TPM
   - Loads child key under primary key
   - Signs digest with child key
7. Response sent back to browser with signature

## Deployment Architecture

### Runtime Requirements

- **TPM Device**: `/dev/tpmrm0` (TPM 2.0 resource manager)
- **UHID Device**: `/dev/uhid` (for HID emulation)
- **Pinentry**: GUI pinentry binary in PATH
- **Permissions**: User must be in `tss` group (for TPM) and have UHID access

### Udev Configuration

```
KERNEL=="uhid", SUBSYSTEM=="misc", GROUP="SOME_UHID_GROUP_MY_USER_BELONGS_TO", MODE="0660"
```

### Module Loading

Add to `/etc/modules-load.d/uhid.conf`:
```
uhid
```

## Data Flow Diagrams

### Registration Data Flow

```
Browser
  ↓ (WebAuthn Register)
UHID Device
  ↓ (HID frames)
fidohid (Frame assembly)
  ↓ (U2F Register command)
fidoauth (Protocol parsing)
  ↓ (Register request)
tpmfido.handleRegister()
  ↓ (User confirmation)
pinentry (GUI dialog)
  ↓ (User confirms)
tpm.RegisterKey()
  ↓ (TPM operations)
TPM Device
  ↓ (Primary + child keys)
Attestation signing
  ↓ (Sign with attestation key)
Response assembly
  ↓ (HID frames)
UHID Device
  ↓ (USB HID)
Browser
```

### Authentication Data Flow

```
Browser
  ↓ (WebAuthn Authenticate)
UHID Device
  ↓ (HID frames)
fidohid (Frame assembly)
  ↓ (U2F Authenticate command)
fidoauth (Protocol parsing)
  ↓ (Authenticate request)
tpmfido.handleAuthenticate()
  ↓ (Validate key handle)
tpm.SignASN1() [dummy signature]
  ↓ (User confirmation)
pinentry (GUI dialog)
  ↓ (User confirms)
tpm.SignASN1() [real signature]
  ↓ (TPM operations)
TPM Device
  ↓ (Load and sign)
Response assembly
  ↓ (HID frames)
UHID Device
  ↓ (USB HID)
Browser
```
