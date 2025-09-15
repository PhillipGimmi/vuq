# VUQ Format Specification

## Verified Unified Qualified Documents

The **VUQ** format (`.vuq`) is a secure, responsive, interactive document container that executes in modern web browsers via WebAssembly. VUQ documents package content, code and policy into a single binary file and are designed to deliver rich multimedia experiences across devices while supporting strong integrity, authenticity and policy enforcement.

> **Status:** Draft 0.1 — media type registration submitted as `application/vnd.vuq`.

---

## Overview

VUQ addresses limitations in current document/container formats:

* **Security** — integrity and authenticity using SHA‑256 and Ed25519; optional confidentiality using AES‑256‑GCM.
* **Responsiveness** — constraint‑based layout engine for mobile through desktop.
* **Interactivity** — embedded WebAssembly modules with a policy‑constrained JS environment.
* **Protection** — optional copy‑tracking signals and content obfuscation at rest.
* **Portability** — a single file bundles assets, code, and configuration.

### Scope & Non‑Goals

* **In scope:** packaging, verification, sandboxed execution, responsive rendering, and optional policy controls (network, licensing, basic copy‑tracking).
* **Not in scope:** preventing all forms of screen capture or out‑of‑band reproduction; guaranteeing confidentiality without keys; replacing DRM or legal/licensing frameworks.

---

## Technical Architecture

### File Structure

```
.vuq Binary Container
├── Magic Header (8 bytes)           # 56 55 51 01 00 00 00 56 (hex)
├── Security Envelope                # Encrypted keys and policies
├── Content Manifest                 # Asset inventory and metadata
├── Encrypted Content Chunks         # Text, images, multimedia
├── WebAssembly Runtime (~400KB)     # Browser execution engine
├── JavaScript Sandbox               # Policy-scoped user code (obfuscated)
├── Responsive Layout Rules          # Adaptive design constraints
└── Integrity Hash Tree              # SHA-256 Merkle verification
```

### Cryptographic Primitives (Normative)

* **Hash**: SHA‑256 (FIPS 180‑4) for content hashing and Merkle trees.
* **AEAD**: AES‑256‑GCM (NIST SP 800‑38D) for optional content encryption.
* **Signatures**: Ed25519 (RFC 8032) for publisher/document authenticity.

> Implementations **MUST** use constant‑time operations for key material where applicable and securely clear sensitive buffers.

### Browser Execution Model

```
Browser → WebAssembly Sandbox → Rust Core Engine → Content Rendering
                ↓
        JavaScript Execution Environment
        (Isolated & Policy‑Enforced)
```

* WebAssembly engine provides memory isolation.
* JS is limited to an allow‑listed API surface with no direct filesystem access; network access is denied by default and can be explicitly allow‑listed per policy.

---

## Security Considerations (Normative)

1. **Active content**: VUQ may embed executable WebAssembly and JavaScript. Viewers **MUST** execute content in a least‑privilege sandbox with CPU/memory/time limits and integrity checks before execution.
2. **Integrity & authenticity**: A Merkle tree over content combined with a file‑level SHA‑256 hash and an Ed25519 signature allows tamper detection and source verification.
3. **Confidentiality (optional)**: When used, AES‑256‑GCM provides encryption with integrity. Key management is out of scope for this spec.
4. **Networking**: Default **DENY**. When enabled, only explicitly allow‑listed origins may be contacted; implementations **SHOULD** apply rate limiting.
5. **Container & compression safety**: Implementations **MUST** validate sizes/offsets before allocation/decompression to mitigate zip‑bomb/DoS classes and reject malformed structures.
6. **Side‑channels**: Use constant‑time crypto; avoid observable branching on secrets; clear keys after use.
7. **Links/URIs**: Link following is disabled unless policy enables it; fragment identifiers are undefined for this media type (see below).

**References:** FIPS 180‑4; NIST SP 800‑38D; RFC 8032; RFC 6234; RFC 6838 §4.6.

---

## Interoperability Considerations

* **Endianness**: All multi‑byte integers are little‑endian.
* **Versioning**: Header carries `major.minor`. Reject higher **major**; accept higher **minor** with unknown‑chunk ignore.
* **Unknown chunks**: Ignore unknown **optional** chunks; fail on unknown **mandatory** chunks with a diagnostic.
* **Magic**: Verify 8‑octet magic `56 55 51 01 00 00 00 56` prior to parsing.
* **SHA‑256**: Use canonical padding/byte order per FIPS 180‑4.
* **Default policy**: Network **off**; copy‑tracking **on** (if implemented by the viewer).

---

## Fragment Identifier Considerations

**None.** This media type does not define fragment identifiers; processors **MUST NOT** assign semantics to URI `#fragments` for `.vuq` resources.

---

## Encoding Considerations

**binary** — NUL may appear; CR/LF may appear outside CRLF; lines may exceed 998 octets. See RFC 2045 §6 and RFC 6838 §4.8.

---

## Format Specification (Informative)

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
0x10    | 32   | SHA‑256 hash of entire file content
0x30    | ...  | Security envelope (encrypted)
```

### Chunk Structure

All content is organized in per‑chunk records; chunks may be individually encrypted.

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

---

## IANA Considerations — Media Type Registration Template

**Type name:** application

**Subtype name:** vnd.vuq

**Required parameters:** N/A

**Optional parameters:** N/A

**Encoding considerations:** binary (see above)

**Security considerations:** See *Security Considerations* section.

**Interoperability considerations:** See *Interoperability Considerations* section.

**Published specification:** [https://github.com/PhillipGimmi/vuq/blob/main/README.md](https://github.com/PhillipGimmi/vuq/blob/main/README.md)

**Applications that use this media type:** VUQ Creator/Packager, VUQ Viewer (browser/WebAssembly and/or native helper), Enterprise Admin/Dashboard; third‑party software treats `.vuq` as an opaque binary unless integrated with the VUQ runtime.

**Fragment identifier considerations:** None.

**Restrictions on usage:** None (use binary‑safe transfer where required).

**Additional information:**

* File extension(s): `.vuq`
* Macintosh file type code(s): None
* Magic number(s): `56 55 51 01 00 00 00 56`
* Object Identifiers (OIDs): None
* Deprecated alias names: None

**Intended usage:** LIMITED USE (primarily with VUQ software; not a general‑purpose interchange format)

**Author/Change controller:** Phillip Gimmi — [phillip.gimmi@gmail.com](mailto:phillip.gimmi@gmail.com)

---

## Implementation Status

This repository hosts the **format specification** for registration and interop. Reference code will be released separately.

### Available Documentation

* Format specification (this document)
* Security model description
* Cryptographic implementation notes
* Interoperability requirements

### Implementation Roadmap (Informative)

* **Phase 1** — Core Rust engine compiled to WebAssembly
* **Phase 2** — Browser viewer & creator tools
* **Phase 3** — Enterprise features (policy/audit) and compliance tooling

---

## Conformance Keywords

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all capitals.

---

## Contact

**Format Owner:** Phillip Gimmi
**Email:** [phillip.gimmi@gmail.com](mailto:phillip.gimmi@gmail.com)
**Repository:** [https://github.com/PhillipGimmi/vuq](https://github.com/PhillipGimmi/vuq)

---

## License

The VUQ format specification is proprietary. Reference implementations and additional materials may be released under separate terms.
