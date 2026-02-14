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



3. Simplified Representation (Compatibility Mode)

The simplified representation is a string containing the official marking:

```YAML
classification: NATO RESTRICTED

```


or in a list:

```YAML
authorized_classifications:
  - NATO RESTRICTED
  - RESTREINT UE/EU/RESTRICTED
```


This form is permitted for compatibility but MUST be normalized into canonical form before any policy, authorization, or filtering logic is applied.







## 4. Authority / Organization Codes (Normative)

The organization field identifies the authority defining the classification scheme.

Two categories of authorities are supported:

### 4.1 Sovereign States

For sovereign states, the organization value MUST be the ISO 3166-1 alpha-2 country code.

Examples (non-exhaustive):

| Code | Country       |
| ---- | ------------- |
| FR   | France        |
| DE   | Germany       |
| US   | United States |
| IT   | Italy         |
| ES   | Spain         |
| NL   | Netherlands   |

These codes are language-neutral and MUST be used regardless of the language of the document.

### 4.2 International and Supranational Organizations

For international organizations, the organization value MUST be:
- The official English acronym of the organization
- Uppercase ASCII
- Stable over time
- Documented in a public registry

Localized acronyms MUST NOT be used.

Examples (recommended registry entries)

| Code | Official English Name              | Disallowed localized forms |
| ---- | ---------------------------------- | -------------------------- |
| NATO | North Atlantic Treaty Organization | OTAN                       |
| EU   | European Union                     | UE                         |
| ESA  | European Space Agency              | ASE                        |
| UN   | United Nations                     | ONU                        |


### 4.3 Validation Rules (Normative)

The organization value MUST satisfy one of the following:
- Match ISO 3166-1 alpha-2 (for countries), OR
- Match a registered international organization code

Additionally:
- The value MUST be uppercase
- The value MUST be language-neutral
- Localized acronyms (e.g. OTAN, UE, ASE, ONU) MUST NOT be used

Formal validation pattern:

```Plain text
organization ∈ ISO_3166_1_ALPHA_2 ∪ REGISTERED_ORG_CODES

```

### 4.4 Registry of Organization Codes (Recommended)

Implementations SHOULD maintain or reference a public registry:

```YAML
organizations:
  FR:
    type: country
    name: France

  DE:
    type: country
    name: Germany

  NATO:
    type: organization
    name: North Atlantic Treaty Organization

  EU:
    type: organization
    name: European Union

  ESA:
    type: organization
    name: European Space Agency

  UN:
    type: organization
    name: United Nations
```

This registry SHOULD be extendable without breaking compatibility.

### 4.5 Normalization Rules (Normative)

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

### 4.6 Rationale (Non-Normative)

Using English canonical acronyms ensures:

- Cross-language interoperability
- Stable identifiers in code and configuration
- Avoidance of ambiguous or localized variants
- Alignment with common international technical standards

### Examples (Canonical)

```
classification:
  organization: NATO
  level: restricted
  marking: NATO RESTRICTED
```


```
classification:
  organization: FR
  level: confidential
  marking: CONFIDENTIEL DÉFENSE
```

## 5. Normalized Levels

This specification defines a common abstract level set for interoperability:


| Level code     | Semantics             |
| -------------- | --------------------- |
| `unclassified` | Not classified        |
| `restricted`   | Low sensitivity       |
| `confidential` | Medium sensitivity    |
| `secret`       | High sensitivity      |
| `top_secret`   | Very high sensitivity |



Organizations MAY map their internal levels to these normalized levels.

## 6. Marking Registry (Recommended)

Implementations SHOULD maintain a registry mapping official markings to normalized form.

Example registry entries:



| Marking                       | Organization | Level        |
| ----------------------------- | ------------ | ------------ |
| NATO RESTRICTED               | NATO         | restricted   |
| RESTREINT UE/EU/RESTRICTED    | EU           | restricted   |
| CONFIDENTIEL DÉFENSE          | FR           | confidential |
| VS-NUR FÜR DEN DIENSTGEBRAUCH | DE           | restricted   |

## 7. Normalization Rules (Normative)

When a simplified form is used, the implementation MUST:
- Look up the marking in the registry
- Replace it with the canonical structured form
- Reject unknown markings unless explicitly configured otherwise

Example:

```YAML
classification: NATO RESTRICTED

```

Normalized:


```YAML
classification:
  organization: NATO
  level: restricted
  marking: NATO RESTRICTED
```



## 8. Validation Rules (Normative)

- organization MUST be uppercase ASCII
- level MUST be one of the normalized levels
- marking MUST be a non-empty string
- (organization, level) MUST match the official meaning of marking if a registry is available
- Authorization logic MUST rely on (organization, level) and NOT on raw marking strings

## 9. Forward Compatibility

Implementations SHOULD:
- Ignore unknown fields
- Allow new organization codes
- Allow new markings without schema change
