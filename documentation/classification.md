# Public Specification – Classification Metadata Format

## 1. Scope & Purpose

This specification defines a portable, machine-readable format for expressing security classification metadata attached to resources such as documents, persons, datasets, or messages.

It supports:
* A canonical structured form (normative)
* A simplified shorthand form (compatibility / human-friendly)
* Deterministic normalization between both forms

This specification is organization-agnostic and extensible.

## 2. Canonical Representation (Normative)

The canonical representation of a classification SHALL be an object with the following fields:

```YAML
classification:
  organization: <ORG_CODE>
  level: <LEVEL_CODE>
  marking: <OFFICIAL_MARKING>
```

### 2.1 Fields


| Field          | Type   | Normative rules                                |
| -------------- | ------ | ---------------------------------------------- |
| `organization` | string | Uppercase short code identifying the authority |
| `level`        | string | Lowercase normalized classification level      |
| `marking`      | string | Official marking as it appears on media        |


4.5 Normalization Rules (Normative)

If localized acronyms are encountered during ingestion (e.g. OTAN, UE, ASE), implementations:
- MAY normalize them to their English canonical form (NATO, EU, ESA)
- MUST emit a warning or error
- MUST output only canonical English codes

Example normalization:

```
OTAN  → NATO
UE    → EU
ASE   → ESA
ONU   → UN
```

4.6 Rationale (Non-Normative)

Using English canonical acronyms ensures:

- Cross-language interoperability
- Stable identifiers in code and configuration
- Avoidance of ambiguous or localized variants
- Alignment with common international technical standards

Examples (Canonical)

classification:
  organization: NATO
  level: restricted
  marking: NATO RESTRICTED


classification:
  organization: FR
  level: confidential
  marking: CONFIDENTIEL DÉFENSE


