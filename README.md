VUQ Format Specification
Verified Unified Qualified Documents

The VUQ format (.vuq) is a secure, responsive, interactive document container that executes natively in web browsers using WebAssembly. VUQ documents provide cryptographic integrity/confidentiality and rich multimedia experiences across devices.

Overview

VUQ addresses fundamental problems with current document formats:

Security: SHA-256 integrity with AES-256-GCM confidentiality

Responsiveness: Mobile-optimized, constraint-based layouts

Interactivity: Full application capabilities inside a document container

Protection: Copy detection and optional code/asset obfuscation

Portability: Single file encapsulates all assets and dependencies

Technical Architecture
File Structure
.vuq Binary Container
├── Magic Header (8 bytes)           # 56 55 51 01 00 00 00 56 (hex)
├── Security Envelope                # Encrypted keys and policies
├── Content Manifest                 # Asset inventory and metadata
├── Encrypted Content Chunks         # Text, images, multimedia
├── WebAssembly Runtime (~400KB)     # Browser execution engine
├── JavaScript Sandbox               # User code (optionally obfuscated)
├── Responsive Layout Rules          # Adaptive design constraints
└── Integrity Hash Tree              # SHA-256 Merkle verification

Cryptographic Security

VUQ uses standard, widely reviewed cryptographic primitives:

Hash Algorithm: SHA-256 (FIPS 180-4)

Encryption: AES-256-GCM (FIPS 197; NIST SP 800-38D) for content confidentiality and integrity

Digital Signatures: Ed25519 (RFC 8032) for authenticity verification

Integrity: Merkle tree for tamper detection

Browser Execution Model
Browser → WebAssembly Sandbox → Rust Core Engine → Content Rendering
                ↓
     JavaScript Execution Environment (Isolated, Policy-Enforced)

Security Features
Content Protection

Encrypted at rest: All packaged content is encrypted within the container; source code and assets may additionally be obfuscated.

Copy Detection: Instance/ledger metadata enables detection of suspected unauthorized duplication (policy-driven response).

Policy Enforcement: Configurable network access, sharing rules, and licensing.

Isolation: WebAssembly sandbox prevents direct OS/filesystem access.

Copy Detection Policies (viewer behavior)

Deny render (prior instance): When a newer instance is detected, a previous instance may be refused by the viewer.

Deny render (subsequent instance): When a duplicate is detected, the later-opened instance may be refused by the viewer.

Timed transfer: Rendering allowed after a policy-defined delay/transfer window.

Tracked copies: Rendering allowed; access is logged for audit.

Note: These are viewer/runtime behaviors. The format does not mandate external file deletion.

Configuration Options
Content Types

Document: Responsive text/media

Game: Highly interactive content using WebAssembly

Interactive Media: Rich multimedia experiences

Enterprise: Compliance/audit-focused content

Network Policies

Isolated: No network access

Outbound-only: Calls to allow-listed domains with rate limits

Bidirectional: Full network with filtering/monitoring

Protection Levels

Unrestricted · Tracked · Licensed · Prevented (policy-driven denial as above)

MIME Type Registration

Media Type: application/vnd.vuq

File Extension: .vuq

Magic Number: 56 55 51 01 00 00 00 56 (hex)

Encoding: binary

Status: IANA registration submitted

Security Considerations

This media type can contain active content (WebAssembly modules and JavaScript). Implementations MUST:

Execute in a least-privilege sandbox; no direct filesystem or OS API access.

Verify integrity and signatures prior to processing.

Deny network access by default; if enabled, limit to explicit allow-lists with rate-limiting.

Enforce resource bounds (e.g., ≤ 512 MB memory, execution timeouts).

Validate container metadata before allocation/decompression to mitigate “zip-bomb”/DoS patterns.

Use constant-time cryptographic primitives and scrub sensitive buffers after use.

Privacy/integrity services are provided by AES-256-GCM and Ed25519 signatures; transport-layer protections (e.g., TLS) are still RECOMMENDED for in-transit confidentiality.

Normative references: FIPS 197; FIPS 180-4; NIST SP 800-38D; RFC 8032; RFC 6234; RFC 6838 §4.6.

Interoperability Considerations

Endianness: All multi-byte integers are little-endian.

Versioning: Header includes major.minor. Reject unknown major; accept higher minor with unknown-chunk ignore.

Chunk processing: Unknown optional chunks → ignore; unknown mandatory chunks → fail with a clear error.

Magic: Verify the 8-octet magic 56 55 51 01 00 00 00 56 before parsing.

SHA-256: Canonical per FIPS 180-4 (byte ordering/padding as specified).

Policy defaults: Network off; copy tracking on, unless overridden.

Fragment Identifier Considerations

None. This media type does not define fragment identifiers; processors MUST NOT assign semantics to URI fragments.

Encoding Considerations

binary — NULs may appear; CR/LF may occur outside CRLF; lines may exceed 998 octets. See RFC 2045 §6; RFC 6838 §4.8.

Format Specification
File Header
Offset  | Size | Description
--------|------|------------------------------------------
0x00    | 3    | Magic "VUQ" (0x56, 0x55, 0x51)
0x03    | 1    | Format version (currently 0x01)
0x04    | 3    | Reserved (0x00, 0x00, 0x00)
0x07    | 1    | Verification byte (0x56)
0x08    | 4    | Security envelope length
0x0C    | 4    | Content manifest offset
0x10    | 32   | SHA-256 hash of entire file content
0x30    | ...  | Security envelope (encrypted)

Chunk Structure

All content is organized in individually encrypted chunks:

Chunk Header (12 bytes):
- Chunk Type (4 bytes)
- Chunk Length (4 bytes)
- Encryption Flag (1 byte)
- Reserved (3 bytes)

Chunk Data (variable):
- Encrypted content (if encryption enabled)
- Raw content (if unencrypted)

IANA Considerations (Registration Template)

Type name: application
Subtype name: vnd.vuq

Required parameters: N/A
Optional parameters: N/A

Encoding considerations: binary (see “Encoding Considerations”)

Security considerations: see “Security Considerations”

Interoperability considerations: see “Interoperability Considerations”

Published specification: https://github.com/PhillipGimmi/vuq/blob/main/README.md
 (this document)

Applications that use this media type: VUQ Creator/Packager; VUQ Viewer (browser/WebAssembly and/or native helper); Enterprise Admin/Dashboard. Third-party software treats .vuq as an opaque binary unless integrated with a VUQ runtime.

Fragment identifier considerations: None

Restrictions on usage: None (use binary-safe transfer encodings where required)

Additional information:

File extension(s): .vuq

Macintosh file type code(s): None

Magic number(s): 56 55 51 01 00 00 00 56

Object Identifiers: None

Deprecated alias names: None

Intended usage: LIMITED USE (primarily with VUQ software; not a general-purpose interchange format)

Author/Change controller: Phillip Gimmi (phillip.gimmi@gmail.com
)

Implementation Status

This repository contains the format specification for IANA registration and standardization purposes.

Available Documentation

Format specification (this document)

Security model description

Cryptographic implementation details

Interoperability requirements

Implementation Roadmap

Phase 1: Core Rust engine with WebAssembly compilation

Phase 2: Browser viewer and creator tools

Phase 3: Enterprise features and compliance tools

Contact

Format Owner: Phillip Gimmi
Email: phillip.gimmi@gmail.com

Repository: https://github.com/PhillipGimmi/vuq

License

The VUQ format specification is proprietary. Reference implementations may be released under separate license terms.