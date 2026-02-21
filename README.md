# FERMAT

**Format for Electronic Registry Manifest Automation and Transfer**

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/your-org/fermat)
[![License](https://img.shields.io/badge/license-Open%20Standard-green.svg)](LICENSE)
[![Specification](https://img.shields.io/badge/spec-stable-brightgreen.svg)](docs/fermat-specification-v1.0.0.md)

> A standardized manifest format for tracking classified document deliveries across multiple classification authorities (NATO, EU, national systems).

Named after Pierre de Fermat (1607-1665), mathematician from Toulouse, France.

---

## Overview

FERMAT enables secure, auditable transfer of classified documents between organizations with:

- **Authority-aggnostic**: NATO, EU, US, France, and other national classification systems
- **Structured classification metadata**: Machine-readable classification objects with authority, level, marking, and caveats
- **Automation-ready**: Designed for integration with Document Management Systems (DMS)
- **Human-readable**: YAML format allows manual editing for simple transfers
- **Integrity verification**: Built-in checksums and HMAC signatures
- **Compliance**: Aligned with EU, NATO, and national security regulations

## Quick Start

### For Simple Document Transfers

For small deliveries (1-5 documents), you can create a FERMAT manifest manually using any text editor:

1. **Create a file** named `manifest.yaml` or `manifest.fermat.yaml`

2. **Edit with Notepad/TextEdit/vim** - the YAML format is human-readable. Here is a minimal example:

```yaml

manifest:
  format: fermat-manifest

delivery:
  reference: DP-2026-001
  sender: Your Organization
  recipient: Receiving Organization
  title: Quarterly Activity Report
  classification: NATO RESTRICTED

documents:
  - id: 2026-arp-001-nr
    filename: report_q1_2026.pdf
    title: Q1 2026 Activity Report
    classification: NATO RESTRICTED
  - id: 2026-inv-002
    filename: invoice_2026-01.pdf
    title: Q1 2026 Activity Report Invoice
    classification: ~

```

3. **Save and transmit** with your documents according to approved procedures

### For Automated/Bulk Processing

For large deliveries (10+ documents), use automation tools and libraries (see [Tools & Libraries](#tools--libraries)). Such tools could be more verbose to avoid any ambiguity:

```yaml

manifest:
  schema_version: 1.0.0
  id: MAN-20260214-A1B2C3D4
  created: 2026-02-14T10:30:00Z
  format: fermat-manifest
  encoding: utf-8
  classification:
    authority: NATO
    level: unclassified
    marking: NATO UNCLASSIFIED
    caveats: null

delivery:
  sender: Your Organization
  recipient: Receiving Organization
  reference: DP-2026-001
  title: Quarterly Security Report
  classification:
    authority: NATO
    level: restricted
    marking: NATO RESTRICTED
    caveats: null
  transmission_method: secure_network
  acknowledgment_required: false

documents:
  - id: reference of the first document
    filename: report_q1_2026.pdf
    title: Q1 2026 Security Assessment
    classification:
      authority: NATO
      level: restricted
      marking: NATO RESTRICTED
    mime_type: application/pdf
    size_bytes: 1048576
    checksum_algorithm: sha256
    checksum: a3b2c1d4e5f6...your_checksum_here
  - id: reference of the second document
    filename: communication_q1.pdf
    title: Q1 2026 Communication flyer
    classification: null
    mime_type: application/pdf
    size_bytes: 10485
    checksum_algorithm: sha256
    checksum: 0226a3b2c1d4e5f6...your_checksum_here

```

## Key Features

### Authority-agnostic, Multi-Authority Classification and Multi Classification 

Support for multiple classification in a single delivery (each document has its own).

### Security & Compliance

- Built-in integrity verification (SHA-256 checksums)

### Automation-Friendly

- Machine-readable JSON/YAML format
- Schema validation support
- Designed for DMS integration
- Auto-computed summary statistics

## Documentation

Detailed specifications are available in the [`docs/`](docs/) directory:

| Document | Description |
|----------|-------------|
| [FERMAT Specification v1.0.0](docs/fermat-specification.md) | Complete FERMAT manifest format specification |
| [Classification Metadata Format v1.0.0](docs/classification-metadata-format-v1.0.0.md) | Referenced classification structure specification |

## Use Cases

### 1. Transmission Workflow

```
┌─────────────────┐
│ Prepare Delivery│
│  - Collect docs │
│  - Verify class.│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Create Manifest │ ◄─── Manual (text editor)
│  - FERMAT YAML  │      or Automated (DMS)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Transmit Package│
│  - Manifest +   │
│    Documents    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Recipient       │
│ - Verify        │
│ - Register      │ ◄─── Manual (text editor) or Automated (DMS)
│ - Acknowledge   │
└─────────────────┘
```

## File Formats

FERMAT supports two serialization formats:

### YAML (Recommended for manual editing)

Automation tools SHALL accept YAML when receiving.

```yaml
# manifest.fermat.yaml
manifest:
  schema_version: "1.0.0"
  # ...
```

**Advantages:**
- Human-readable and editable
- Supports comments
- Less verbose than JSON
- Perfect for simple/manual workflows

### JSON (Recommended for automation)

Automation tools SHOULD accept JSON and YAML when receiving.
Automation tools SHOULD use JSON for known recipient having coompatible tools.

```json
{
  "manifest": {
    "schema_version": "1.0.0"
  }
}
```

**Advantages:**
- Universal parser support
- Strict schema validation
- Better for automated systems

## Tools & Libraries

Community contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md).

## Conformance Levels

### Minimum Conformance

A conforming manifest MUST:
- Include all required fields (manifest, delivery, documents)
- Use valid classification objects
- Use RFC 3339 datetime format
- Set `format` to `"fermat-manifest"`
- Ensure marking fields contain only base **official** markings (no embedded caveats)

### Full Conformance

A fully conforming implementation SHOULD:
- Support both JSON and YAML formats
- Support all classification object forms (canonical, shorthand, simplified)
- Compute checksums for all documents
- Support HMAC signing and verification
- Auto-compute summary section
- Validate against JSON Schema
- Validate classification objects against authority registries

## Security Considerations

### When Using FERMAT

1. **Physical Security**: Manifests may contain classified metadata - handle according to highest classification level present
2. **Transmission**: When manifest is classified, use approved secure channels (encrypted networks, courier, diplomatic pouch)
3. **Integrity**: Always verify checksums and signatures on receipt
4. **Audit Trail**: Preserve manifests for compliance audits

### Checksum Generation

```bash
# SHA-256 checksum
sha256sum document.pdf
# Output: a3b2c1d4e5f6... document.pdf
```

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for:

- Reporting issues
- Proposing enhancements
- Submitting pull requests
- Adding support for new classification authorities
- Creating tools and libraries

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0   | February 2026 | Initial release |

## License

FERMAT is an open standard. See [LICENSE](LICENSE) for details.

## References

### Normative References

- **ISO 3166-1** - Country codes

## Support

- **Issues**: [GitHub Issues](https://github.com/your-org/fermat/issues)

