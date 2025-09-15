# VUQ Format Specification
## Verified Unified Qualified Documents

The VUQ format (.vuq) is a secure, responsive, interactive document container that executes natively in web browsers using WebAssembly. VUQ documents provide Bitcoin-level cryptographic security while delivering rich multimedia experiences across all devices.

## Overview

VUQ addresses fundamental problems with current document formats:
- **Security**: Bitcoin-level SHA-256 cryptographic protection
- **Responsiveness**: Perfect mobile experience with constraint-based layouts  
- **Interactivity**: Full application capabilities within documents
- **Protection**: Advanced copy detection and content obfuscation
- **Portability**: Single file contains all assets and dependencies

## Technical Architecture

### File Structure
```
.vuq Binary Container
├── Magic Header (8 bytes)           # 56 55 51 01 00 00 00 56 (hex)
├── Security Envelope                # Encrypted keys and policies  
├── Content Manifest                 # Asset inventory and metadata
├── Encrypted Content Chunks         # Text, images, multimedia
├── WebAssembly Runtime (~400KB)     # Browser execution engine
├── JavaScript Sandbox              # User code (obfuscated)
├── Responsive Layout Rules          # Adaptive design constraints
└── Integrity Hash Tree              # SHA-256 Merkle verification
```

### Cryptographic Security

VUQ uses identical cryptographic primitives as the Bitcoin blockchain:

- **Hash Algorithm**: SHA-256 (same as Bitcoin)
- **Encryption**: AES-256-GCM for content protection
- **Signatures**: Ed25519 for authenticity verification
- **Integrity**: Merkle tree verification (Ethereum-style)

**Security Guarantee**: If Bitcoin's cryptographic security holds, VUQ documents are equally secure.

### Browser Execution Model

VUQ documents execute directly in web browsers through WebAssembly:

```
Browser → WebAssembly Sandbox → Rust Core Engine → Content Rendering
                ↓
        JavaScript Execution Environment
        (Isolated, Policy-Enforced)
```

## Security Features

### Content Protection
- **Complete Obfuscation**: JavaScript, games, multimedia invisible in file explorers
- **Copy Detection**: Timestamp-based ledger detects unauthorized copies
- **Policy Enforcement**: Configurable network access, sharing rules, licensing
- **Sandbox Isolation**: WebAssembly prevents system access

### Copy Detection Modes
1. **Kill Original**: Original self-destructs when copy detected
2. **Kill Copy**: Copies become non-functional when opened
3. **Time Transfer**: Delayed ownership transfer to copy
4. **Tracked Copies**: Allow copying but log all access

## Configuration Options

### Content Types
- **Document**: Responsive text and media
- **Game**: Interactive applications with full WebAssembly
- **Interactive Media**: Rich multimedia experiences
- **Enterprise**: Compliance-focused with audit trails

### Network Policies
- **Isolated**: No network access (read-only walled garden)
- **Outbound Only**: API calls permitted to allowlisted domains
- **Bidirectional**: Full network with content filtering

### Protection Levels
- **Unrestricted**: Standard document behavior
- **Tracked**: Copy attempts logged and reported
- **Licensed**: Requires valid license to function
- **Prevented**: Advanced copy protection with self-destruct

## MIME Type Registration

- **Media Type**: `application/vnd.vuq`
- **File Extension**: `.vuq`
- **Magic Number**: `56 55 51 01 00 00 00 56` (hex)
- **Encoding**: Binary
- **Status**: IANA registration submitted

## Implementation Status

This repository contains the format specification for IANA registration and standardization purposes. 

### Available Documentation
- Format specification (this document)
- Security model description
- Cryptographic implementation details
- Interoperability requirements

### Implementation Roadmap
- **Phase 1**: Core Rust engine with WebAssembly compilation
- **Phase 2**: Browser viewer and creator tools
- **Phase 3**: Enterprise features and compliance tools

## Security Considerations

VUQ documents contain active content and require specific security measures:

1. **Sandbox Execution**: All content runs in WebAssembly isolation
2. **Resource Limits**: Maximum 512MB memory, 30-second timeouts
3. **Network Restrictions**: Default deny with explicit allowlisting
4. **Integrity Verification**: SHA-256 hash chains prevent tampering
5. **Content Validation**: Input sanitization prevents injection attacks

## Interoperability Requirements

### Cross-Platform Compatibility
- **Endianness**: Little-endian encoding for all multi-byte integers
- **Version Handling**: Graceful handling of minor version differences
- **Hash Computation**: Canonical SHA-256 implementation per FIPS 180-4
- **Policy Defaults**: Standardized security and sharing policies

### Browser Support
VUQ requires modern browsers with:
- WebAssembly support (all major browsers since 2017)
- Canvas API for rendering
- Web Audio API for multimedia (optional)
- Crypto API for verification

## Format Specification

### File Header
```
Offset  | Size | Description
--------|------|------------------------------------------
0x00    | 3    | Magic signature "VUQ" (0x56, 0x55, 0x51)
0x03    | 1    | Format version (currently 0x01)
0x04    | 3    | Reserved (0x00, 0x00, 0x00)
0x07    | 1    | Verification byte (0x56)
0x08    | 4    | Security envelope length
0x0C    | 4    | Content manifest offset
0x10    | 32   | SHA-256 hash of entire file content
0x30    | ...  | Security envelope (encrypted)
```

### Chunk Structure
All content is organized in chunks with individual encryption:
```
Chunk Header (12 bytes):
- Chunk Type (4 bytes)
- Chunk Length (4 bytes) 
- Encryption Flag (1 byte)
- Reserved (3 bytes)

Chunk Data (variable):
- Encrypted content (if encryption enabled)
- Raw content (if unencrypted)
```

## Contact

**Format Owner**: Phillip Gimmi  
**Email**: phillip.gimmi@gmail.com  
**Repository**: https://github.com/PhillipGimmi/vuq

## License

The VUQ format specification is proprietary. Implementation details and reference software will be made available under appropriate licensing terms.

---

*VUQ: Verified, Unified, Qualified - The future of secure digital documents.*