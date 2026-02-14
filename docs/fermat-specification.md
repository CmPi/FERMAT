# FERMAT Specification

**Format for Electronic Registry Manifest Automation and Transfer**

*Version 1.0.0 - February 2026*

---

## 1. Introduction

### 1.1 Purpose

FERMAT defines a standardized manifest format for tracking classified document deliveries between organizations. It enables:
- Automated document registration by Registry Control Officers (RCOs)
- Interoperability between Document Management Systems (DMS)
- Integrity verification of transmitted documents
- Audit trail compliance with international security regulations
- Support for multiple classification authorities (NATO, EU, national systems)

### 1.2 Scope

This specification covers:
- Manifest structure and required fields
- Multi-authority classification handling using structured classification objects
- Document metadata requirements
- Integrity verification mechanisms
- Serialization formats (JSON, YAML)

### 1.3 Normative References

- **Classification Metadata Format v1.0.0** - Referenced classification structure specification
- **EU Council Decision 2013/488/EU** - Security rules for protecting EU classified information
- **EC Decision 2015/444** - Security rules for protecting EU classified information
- **NATO Security Policy (C-M(2002)49)** - NATO security regulations
- **JSON Schema draft-07** - Schema definition format
- **RFC 3339** - Date and time format
- **RFC 4648** - Base16 (hex) encoding for checksums
- **ISO 3166-1** - Country codes

### 1.4 Naming

**FERMAT** - Format for Electronic Registry Manifest Automation and Transfer

Named after Pierre de Fermat (1607-1665), mathematician from Toulouse, France.

The name emphasizes:
- **Electronic**: Digital, machine-readable format
- **Registry**: Document registration and tracking
- **Manifest**: Structured metadata describing transmitted documents
- **Automation**: Support for automated processing by DMS systems
- **Transfer**: Cross-border, multi-authority document exchange

---

## 2. Terminology

| Term | Definition |
|------|------------|
| **Datapack** | A collection of documents transmitted as a unit |
| **Manifest** | Machine-readable metadata file describing a datapack |
| **RCO** | Registry Control Officer - responsible for classified document registration |
| **DMS** | Document Management System |
| **Authority** | Classification authority (e.g., NATO, EU, national government) |
| **Originator** | The entity (person, office, or organization) that creates or originally classifies information and retains control over its dissemination (see ORCON caveat) |
| **Sender** | The organization physically transmitting the datapack; may differ from the document originator |
| **Recipient** | The organization receiving the datapack |
| **Caveat** | Additional handling restriction or dissemination control |

### 2.1 Originator vs Sender

**Important distinction:**

- **Originator**: The authority that created or originally classified the information. The originator retains control rights (especially with ORCON marking) and may not be the entity physically transmitting the document.
  
- **Sender**: The entity physically transmitting the datapack in this specific transaction. The sender may be forwarding documents on behalf of the originator or may themselves be the originator.

**Example scenarios:**
1. **Sender = Originator**: French intelligence directorate creates and sends a report directly to NATO
2. **Sender ≠ Originator**: NATO forwards a US-originated document (marked US ORCON) to EU member states - NATO is the sender, but US remains the originator with control rights

---

## 3. Manifest Structure

A FERMAT manifest consists of five top-level sections:

```
manifest/           # Manifest metadata
header/             # Delivery information
documents[]         # Array of document entries
summary/            # Computed statistics
integrity/          # Signature data
```

### 3.1 Required Sections

- `manifest` - REQUIRED
- `header` - REQUIRED
- `documents` - REQUIRED (may be empty array)

### 3.2 Optional Sections

- `summary` - OPTIONAL (auto-computed)
- `integrity` - OPTIONAL (recommended for classified)

---

## 4. Manifest Section

Contains metadata about the manifest itself.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schema_version` | string | Yes | Schema version (semver format, current: "1.0.0") |
| `id` | string | Yes | Unique manifest identifier |
| `created` | datetime | Yes | Creation timestamp (RFC 3339) |
| `modified` | datetime | No | Last modification timestamp |
| `format` | string | Yes | Must be `"fermat-manifest"` |

### 4.1 Manifest ID Format

Recommended format: `MAN-YYYYMMDD-XXXXXXXX`

Where:
- `MAN` - Fixed prefix
- `YYYYMMDD` - Creation date
- `XXXXXXXX` - 8 hexadecimal characters (random or sequential)

Example: `MAN-20260128-A1B2C3D4`

---

## 5. Header Section

Contains information about the delivery.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sender` | string | Yes | Organization physically transmitting the datapack |
| `recipient` | string | Yes | Organization receiving the datapack |
| `datapack_reference` | string | Yes | Unique datapack reference |
| `title` | string | No | Human-readable title |
| `description` | string | No | Detailed description |
| `classification` | object/string/null | No | Structured classification object (see 5.1) |
| `transmission_method` | string | No | Method of transmission |
| `acknowledgment_required` | boolean | No | Default: false (see 5.3) |

### 5.1 Classification Object

The `classification` field follows the **Classification Metadata Format v1.0.0** specification and can be:

1. **Structured object (canonical form)** - RECOMMENDED:

```yaml
classification:
  authority:
    code: EU
    type: organization
    name: European Union
  level: confidential
  marking: CONFIDENTIEL UE/EU CONFIDENTIAL
  caveats: [ATOMAL]
```

2. **Shorthand authority notation**:

```yaml
classification:
  authority: NATO
  level: secret
  marking: NATO SECRET
  caveats: [CCI]
```

3. **Simplified string (compatibility mode)**:

```yaml
classification: "NATO SECRET"
```

4. **Null (non-classified)**:

```yaml
classification: null
```

5. **Unknown classification**:

```yaml
classification: "unknown"
```

**IMPORTANT**: The `marking` field contains ONLY the official base classification marking. Caveats are specified separately in the `caveats` array and MUST NOT appear in the marking string.

### 5.2 Classification Examples by Authority

#### NATO Classification

```yaml
classification:
  authority:
    code: NATO
    type: organization
    name: North Atlantic Treaty Organization
  level: restricted
  marking: NATO RESTRICTED
  caveats: null
```

#### EU Classification

```yaml
classification:
  authority:
    code: EU
    type: organization
    name: European Union
  level: secret
  marking: SECRET UE/EU SECRET
  caveats: [ATOMAL, BOHEMIA]
```

#### French National Classification

```yaml
classification:
  authority:
    code: FR
    type: country
    name: France
  level: secret
  marking: SECRET
  caveats: [CRYPTO]
```

#### US Classification

```yaml
classification:
  authority:
    code: US
    type: country
    name: United States
  level: top_secret
  marking: TOP SECRET
  caveats: ["REL TO USA, GBR, CAN, AUS, NZL"]
```

#### EU Non-Classified

EU does not have an official "EU UNCLASSIFIED" marking, so use `null`:

```yaml
classification: null
```

### 5.3 Acknowledgment Rules

Acknowledgment requirements vary by authority and classification level:

#### EU Regulations

| Classification | Acknowledgment |
|----------------|----------------|
| Unclassified (null) | Not required |
| RESTREINT UE/EU RESTRICTED | Typically not required |
| CONFIDENTIEL UE/EU CONFIDENTIAL | Counter-signed receipt required |
| SECRET UE/EU SECRET | Counter-signed receipt required |
| TRÈS SECRET UE/EU TOP SECRET | Counter-signed receipt required |

#### NATO Regulations

| Classification | Acknowledgment |
|----------------|----------------|
| NATO UNCLASSIFIED | Not required |
| NATO RESTRICTED | Typically not required |
| NATO CONFIDENTIAL | Receipt required |
| NATO SECRET | Counter-signed receipt required |
| NATO COSMIC TOP SECRET | Hand receipt required |

#### General Guidance

- Default value for `acknowledgment_required` is `false`
- Implementations SHOULD set this field based on the classification authority and level
- National regulations may impose additional requirements

### 5.4 Transmission Methods

Suggested values (not exhaustive):
- `secure_courier`
- `encrypted_transfer`
- `diplomatic_pouch`
- `approved_network`
- `hand_delivery`

### 5.5 Common Caveats Reference

Common caveats across classification authorities:

| Caveat      | Authority     | Description |
|-------------|---------------|-------------|
| `CRYPTO`    | NATO, US, FR, many | Cryptographic information/material |
| `CCI`       | NATO, US      | Controlled Cryptographic Item |
| `COMSEC`    | NATO, US      | Communications Security material |
| `ATOMAL`    | NATO, EU      | Atomic/nuclear information |
| `BOHEMIA`   | NATO, EU      | Signals intelligence |
| `ORCON`     | US, NATO      | **Originator Controlled** - Requires originator's consent for further dissemination |
| `NOFORN`    | US            | No foreign nationals |
| `PROPIN`    | US            | Proprietary information involved |
| `REL TO`    | US, NATO      | Releasable to specific countries |
| `PRS-USE`, `PRS-RCV`, `PRS-SUP` | EU | Galileo PRS/PSI need-to-know |

**ORCON (Originator Controlled):**
This critical caveat means the document originator retains control over further dissemination. Recipients cannot share ORCON-marked information with third parties without explicit permission from the originator. The `originator` and `originator_organization` fields in the document metadata identify who controls dissemination rights.

**Example:** If a US document marked `SECRET//ORCON` is forwarded by NATO to the EU, NATO is the sender but the US remains the originator with dissemination control. EU SATCEN cannot further share this document without US permission.

**References:**
- ATOMAL, BOHEMIA: See [NATO AAP-15](https://nso.nato.int/nso/zPublic/ap/aap15-ed(2019).pdf) and [NATO Security Policy](https://www.nato.int/cps/en/natohq/topics_110496.htm)
- ORCON: See [US Intelligence Community Policy Guidance 710.1](https://www.odni.gov/files/documents/ICPG/ICPG-710-1.pdf)
- PRS caveats: See [Galileo PRS Decision 1104/2011/EU](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32011D1104) and [EU GNSS Security Accreditation Board](https://www.gsa.europa.eu/gnss-security-accreditation-board)

---

## 6. Documents Section

An array of document entries. Each entry describes one document in the datapack.

### 6.1 Document Fields

#### Identification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique ID within manifest |
| `filename` | string | Yes | Original filename |
| `path` | string | No | Relative path within datapack |
| `title` | string | Yes | Document title |
| `description` | string | No | Document description |
| `version` | string | No | Document version |
| `revision` | string | No | Revision identifier |

#### Classification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `classification` | object/string/null | Yes | Document classification (follows Classification Metadata Format v1.0.0) |

**Note:** The `classification` field for documents follows the same structure as the header classification (see Section 5.1). It can be:
- A structured classification object (canonical form - RECOMMENDED)
- A shorthand form with authority code
- A simplified string (compatibility)
- `null` (non-classified)
- `"unknown"` (indeterminate status)

Documents MAY have different classifications from the overall datapack classification.

#### Originator & Approval

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `originator` | string | No | Person or office that created/originally classified the document |
| `originator_organization` | string | No | Organization of the originator (may differ from sender) |
| `approval_status` | string | No | pending/approved/rejected/withdrawn |
| `approval_authority` | string | No | Approving person/entity |
| `approval_date` | date | No | Date of approval |
| `approved` | boolean | No | Is approved for release |

**Note on originator field:** This identifies who created or originally classified the document, which determines dissemination control rights (especially for ORCON-marked documents). This may differ from the datapack sender - for example, when NATO (sender) forwards a US-originated document marked with US ORCON.

#### File Information

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mime_type` | string | No | MIME type |
| `document_type` | string | No | Business document type |
| `size_bytes` | integer | No | File size in bytes |
| `created` | datetime | No | File creation time |
| `modified` | datetime | No | File modification time |

#### Integrity

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `checksum_algorithm` | string | No | Hash algorithm (default: sha256) |
| `checksum` | string | No | File checksum (hex) |

#### References

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `reference_number` | string | No | External reference number |
| `supersedes` | string | No | Reference to superseded document |
| `related_documents` | array | No | Related document references |

### 6.2 Document ID Format

Recommended format: `DOC-NNNN`

Where `NNNN` is a zero-padded sequence number.

Example: `DOC-0001`, `DOC-0002`

---

## 7. Summary Section

Auto-computed statistics about the delivery.

| Field | Type | Description |
|-------|------|-------------|
| `total_documents` | integer | Number of documents |
| `total_size_bytes` | integer | Total size of all documents |
| `classification_breakdown` | object | Count per classification level |

This section SHOULD be computed automatically when saving.

---

## 8. Integrity Section

Contains signature data for tamper detection.

| Field | Type | Description |
|-------|------|-------------|
| `algorithm` | string | Signature algorithm |
| `signature` | string | HMAC signature (hex) |
| `signed_at` | datetime | Signature timestamp |

### 8.1 Signature Algorithm

Default: `sha256-hmac`

The signature is computed over the JSON-encoded manifest, header, and documents sections (excluding summary and integrity).

### 8.2 Signature Computation

```
data = JSON.stringify({manifest, header, documents})
signature = HMAC-SHA256(data, shared_secret)
```

---

## 9. Serialization

### 9.1 JSON Format

- UTF-8 encoding
- Pretty-printed for readability (optional)
- File extension: `.json` or `.fermat.json`

### 9.2 YAML Format

- UTF-8 encoding
- Comments allowed (ignored by parsers)
- File extension: `.yaml`, `.yml`, or `.fermat.yaml`

### 9.3 Format Detection

Implementations SHOULD detect format from file extension:
- `.json` → JSON
- `.yaml`, `.yml` → YAML

---

## 10. Conformance

### 10.1 Minimum Conformance

A conforming manifest MUST:
- Include all required fields in manifest, header, and documents sections
- Use valid classification objects conforming to Classification Metadata Format v1.0.0
- Use RFC 3339 datetime format
- Set `format` to `"fermat-manifest"`
- Ensure `marking` fields contain only base classification markings (no caveats)

### 10.2 Full Conformance

A fully conforming implementation SHOULD:
- Support both JSON and YAML formats
- Support all classification object forms (canonical, shorthand, simplified)
- Compute checksums for all documents
- Support HMAC signing and verification
- Auto-compute summary section
- Validate against JSON Schema
- Validate classification objects against authority registries

---

## 11. Complete Example

### 11.1 Multi-Authority Datapack

```yaml
manifest:
  schema_version: "1.0.0"
  id: MAN-20260214-A1B2C3D4
  created: "2026-02-14T10:30:00Z"
  format: fermat-manifest

header:
  sender: "NATO ACT"
  recipient: "EU SATCEN"
  datapack_reference: "DP-2026-001"
  title: "Joint Intelligence Assessment"
  description: "Collaborative intelligence sharing under NATO-EU cooperation"
  classification:
    authority:
      code: NATO
      type: organization
      name: North Atlantic Treaty Organization
    level: secret
    marking: NATO SECRET
    caveats: [ATOMAL, BOHEMIA]
  transmission_method: secure_courier
  acknowledgment_required: true

documents:
  - id: DOC-0001
    filename: "assessment_report.pdf"
    title: "Intelligence Assessment Report"
    originator: "NATO Intelligence Division"
    originator_organization: "NATO"
    classification:
      authority: NATO
      level: secret
      marking: NATO SECRET
      caveats: [ATOMAL]
    mime_type: application/pdf
    size_bytes: 2457600
    checksum_algorithm: sha256
    checksum: "a3b2c1d4e5f6789012345678901234567890abcdef1234567890abcdef123456"
    
  - id: DOC-0002
    filename: "french_annex.pdf"
    title: "French National Contribution"
    originator: "Direction du Renseignement Militaire"
    originator_organization: "French Ministry of Armed Forces"
    classification:
      authority:
        code: FR
        type: country
        name: France
      level: secret
      marking: SECRET
      caveats: [CRYPTO]
    mime_type: application/pdf
    size_bytes: 1048576
    checksum_algorithm: sha256
    checksum: "b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2a1b0c9d8e7f6a5b4c3"
    
  - id: DOC-0003
    filename: "us_technical_data.xlsx"
    title: "US Technical Analysis"
    originator: "Defense Intelligence Agency"
    originator_organization: "United States Department of Defense"
    classification:
      authority:
        code: US
        type: country
        name: United States
      level: secret
      marking: SECRET
      caveats: [ORCON, "REL TO USA, GBR, CAN, AUS, NZL"]
    mime_type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
    size_bytes: 524288
    checksum_algorithm: sha256
    checksum: "c5d4e3f2a1b0c9d8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4"

summary:
  total_documents: 3
  total_size_bytes: 4030464
  classification_breakdown:
    NATO_secret: 1
    FR_secret: 1
    US_secret: 1

integrity:
  algorithm: sha256-hmac
  signature: "d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2a1b0c9d8e7f6a5b4c3d2e1f0a9b8c7d6e5"
  signed_at: "2026-02-14T10:35:00Z"
```

**Note:** In this example, NATO ACT is the sender (physically transmitting the datapack), but DOC-0003 is US-originated and marked with ORCON, meaning further dissemination requires US approval. The originator fields clarify control rights.

---

*FERMAT - Format for Electronic Registry Manifest Automation and Transfer*
*Version 1.0.0*
*© 2026 - Open Standard*
