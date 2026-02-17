# Classification Metadata Format

## 1. Scope & Purpose

This specification defines a portable, machine-readable format for expressing security classification metadata attached to resources such as documents, persons, datasets, or messages.

It supports:
* A canonical structured form (normative)
* A simplified shorthand form (compatibility / human-friendly)
* Special values for non-classified and unknown classification states
* Deterministic normalization between both forms

This specification is **authority-agnostic** and extensible.

## 2. Special Classification Values

In addition to structured classification objects, two special values are supported:

### 2.1 Null Classification

The value `null` indicates that a resource is **not classified** within the context of an authority that does not have an explicit "unclassified" level.

```yaml
classification: null
```

**Use cases:**
- EU documents (EU has no official "EU UNCLASSIFIED" marking)
- Resources that are public or non-sensitive but originate from an authority without an unclassified designation
- Distinguishing "no classification" from "classification unknown"

**Semantics:**
- `null` means "this resource is not subject to classification controls"
- It is distinct from `level: unclassified` which is an explicit classification marking (e.g., "NATO UNCLASSIFIED")

**Comparison: `null` vs `unclassified`**

| Aspect                | `classification: null`          | `level: unclassified`                    |
| --------------------- | ------------------------------- | ---------------------------------------- |
| Meaning               | Not subject to classification   | Explicitly marked as unclassified        |
| Authority required    | No authority specified          | Authority must be specified              |
| Official marking      | No marking                      | Authority has official marking (e.g., "NATO UNCLASSIFIED") |
| Use case              | EU documents, public data       | NATO, US, and other authorities with unclassified levels |
| Access control        | Public/open access              | May still have handling restrictions     |

### 2.2 Unknown Classification

The string value `"unknown"` indicates that the classification status cannot be determined.

```yaml
classification: unknown
```

Document elements without classification key shall be assumed as unknown classification

**Use cases:**
- Legacy documents without machine-readable classification metadata
- Documents where classification markings are illegible or ambiguous
- Data ingestion pipelines that cannot reliably parse classification information
- Placeholder value during document processing

**Semantics:**
- `"unknown"` means "classification status is indeterminate"
- Access control systems SHOULD treat unknown classifications conservatively (e.g., deny by default)
- Systems MAY prompt for manual classification review

## 3. Canonical Representation (Normative)

The canonical representation of a classification SHALL be an object with the following fields:

```yaml
classification:
  authority:
    code: <AUTHORITY_CODE>
    type: <AUTHORITY_TYPE>
    name: <AUTHORITY_NAME>
  level: <LEVEL_CODE>
  marking: <OFFICIAL_MARKING>
  caveats: [<CAVEAT1>, <CAVEAT2>, ...]
```

### 3.1 Fields

| Field       | Type         | Normative rules                                           |
| ----------- | ------------ | --------------------------------------------------------- |
| `authority` | object       | Structured object identifying the classifying authority   |
| `level`     | string       | Lowercase normalized classification level                 |
| `marking`   | string       | Official marking as it appears on media                   |
| `caveats`   | array or null | Optional list of handling restrictions and dissemination controls |

### 3.2 Authority Object

The `authority` field is a structured object with the following fields:

| Field  | Type   | Normative rules                                    |
| ------ | ------ | -------------------------------------------------- |
| `code` | string | Uppercase short code identifying the authority     |
| `type` | string | Authority type: `country` or `organization`        |
| `name` | string | Full official name of the authority in English     |

### 3.3 Shorthand Authority Notation

For convenience, `authority` MAY be specified as a string containing only the code:

```yaml
classification:
  authority: FR
  level: confidential
  marking: CONFIDENTIEL DÉFENSE
```

This shorthand form MUST be normalized to the full structured form before any policy, authorization, or filtering logic is applied, using the authority registry.

### 3.4 Caveats Field

The `caveats` field is an optional array containing handling restrictions and dissemination controls:

| Field     | Type         | Normative rules                                    |
| --------- | ------------ | -------------------------------------------------- |
| `caveats` | array or null | List of caveat codes; MAY be null or omitted if no caveats apply |

**Caveat values:**
- MUST be uppercase strings
- MUST represent official handling restrictions recognized by the authority
- MUST NOT be included in the `marking` field
- Examples: `CRYPTO`, `CCI`, `NOFORN`, `ORCON`, `ATOMAL`, `COMSEC`

**Example with caveats:**

```yaml
classification:
  authority:
    code: NATO
    type: organization
    name: North Atlantic Treaty Organization
  level: secret
  marking: NATO SECRET
  caveats: [ATOMAL, CCI]
```

**Example without caveats:**

```yaml
classification:
  authority:
    code: US
    type: country
    name: United States
  level: confidential
  marking: CONFIDENTIAL
  caveats: null
```

Or simply omit the field:

```yaml
classification:
  authority: US
  level: confidential
  marking: CONFIDENTIAL
```

## 4. Simplified Representation (Compatibility Mode)

The simplified representation is a string containing the official marking:

```yaml
classification: NATO RESTRICTED
```

or in a list:

```yaml
authorized_classifications:
  - NATO RESTRICTED
  - RESTREINT UE/EU RESTRICTED
```

This form is permitted for compatibility but MUST be normalized into canonical form before any policy, authorization, or filtering logic is applied.

## 5. Authority Codes and Types (Normative)

The authority object identifies the entity defining the classification scheme.

### 5.1 Authority Types

Two authority types are supported:

| Type           | Description                                    |
| -------------- | ---------------------------------------------- |
| `country`      | Sovereign state using ISO 3166-1 alpha-2 code  |
| `organization` | International or supranational organization    |

### 5.2 Country Authorities

For sovereign states:
- `code` MUST be the ISO 3166-1 alpha-2 country code
- `type` MUST be `country`
- `name` SHOULD be the official English short name

Examples (non-exhaustive):

| Code | Type    | Name           |
| ---- | ------- | -------------- |
| FR   | country | France         |
| DE   | country | Germany        |
| US   | country | United States  |
| GB   | country | United Kingdom |
| IT   | country | Italy          |
| ES   | country | Spain          |
| NL   | country | Netherlands    |

**Note:** France reformed its classification system on July 1, 2021, replacing the three-level system (Confidentiel Défense, Secret Défense, Très Secret Défense) with a two-level system: **SECRET** and **TRÈS SECRET** (without "Défense" suffix).

These codes are language-neutral and MUST be used regardless of the language of the document.

### 5.3 Organization Authorities

For international organizations:
- `code` MUST be the official English acronym
- `type` MUST be `organization`
- `name` MUST be the official English name
- Code MUST be uppercase ASCII
- Code MUST be stable over time
- Code MUST be documented in a public registry

Localized acronyms MUST NOT be used.

Examples (recommended registry entries):

| Code | Type         | Name                                   | Disallowed localized forms |
| ---- | ------------ | -------------------------------------- | -------------------------- |
| NATO | organization | North Atlantic Treaty Organization     | OTAN                       |
| EU   | organization | European Union                         | UE                         |
| ESA  | organization | European Space Agency                  | ASE                        |
| UN   | organization | United Nations                         | ONU                        |

### 5.4 Validation Rules (Normative)

The authority code value MUST satisfy one of the following:
- Match ISO 3166-1 alpha-2 (for countries), OR
- Match a registered international organization code

Additionally:
- `code` MUST be uppercase ASCII
- `code` MUST be language-neutral
- Localized acronyms (e.g. OTAN, UE, ASE, ONU) MUST NOT be used
- `type` MUST be either `country` or `organization`
- `name` MUST match the registered name for the given code

Formal validation pattern:

```
authority.code ∈ ISO_3166_1_ALPHA_2 ∪ REGISTERED_ORG_CODES
authority.type ∈ {"country", "organization"}
authority.name = REGISTRY[authority.code].name
```

### 5.5 Authority Registry (Recommended)

Implementations SHOULD maintain or reference a public registry with full authority structures:

```yaml
authorities:
  FR:
    type: country
    name: France
    aliases: []

  DE:
    type: country
    name: Germany
    aliases: []

  US:
    type: country
    name: United States
    aliases: []

  NATO:
    type: organization
    name: North Atlantic Treaty Organization
    aliases: [OTAN]

  EU:
    type: organization
    name: European Union
    aliases: [UE]

  ESA:
    type: organization
    name: European Space Agency
    aliases: [ASE]

  UN:
    type: organization
    name: United Nations
    aliases: [ONU]
```

This registry SHOULD be extendable without breaking compatibility.

### 5.6 Normalization Rules (Normative)

#### 5.6.1 Shorthand Authority Expansion

When authority is specified as a string, implementations MUST:
- Look up the code in the authority registry
- Replace it with the full structured form
- Reject unknown codes unless explicitly configured otherwise

Example:

```yaml
# Input (shorthand)
classification:
  authority: NATO
  level: restricted
  marking: NATO RESTRICTED
```

Normalized:

```yaml
# Output (canonical)
classification:
  authority:
    code: NATO
    type: organization
    name: North Atlantic Treaty Organization
  level: restricted
  marking: NATO RESTRICTED
```

#### 5.6.2 Localized Acronym Normalization

If localized acronyms are encountered during ingestion (e.g. OTAN, UE, ASE), implementations:
- MAY normalize them to their English canonical code (NATO, EU, ESA)
- MUST emit a warning or error
- MUST output only canonical English codes in the normalized form

Example normalization:

```
OTAN  → NATO
UE    → EU
ASE   → ESA
ONU   → UN
```

### 5.7 Rationale (Non-Normative)

Using structured authority objects with English canonical codes ensures:

- Cross-language interoperability
- Stable identifiers in code and configuration
- Avoidance of ambiguous or localized variants
- Alignment with common international technical standards
- Clear distinction between authority types
- Extensibility for new authority types in the future

### 5.8 Examples (Canonical)

NATO classification:

```yaml
classification:
  authority:
    code: NATO
    type: organization
    name: North Atlantic Treaty Organization
  level: restricted
  marking: NATO RESTRICTED
```

French national classification (current system as of 2021):

```yaml
classification:
  authority:
    code: FR
    type: country
    name: France
  level: secret
  marking: SECRET
```

US classification:

```yaml
classification:
  authority:
    code: US
    type: country
    name: United States
  level: top_secret
  marking: TOP SECRET
```

EU classification:

```yaml
classification:
  authority:
    code: EU
    type: organization
    name: European Union
  level: restricted
  marking: RESTREINT UE/EU RESTRICTED
```

## 6. Normalized Levels

This specification defines a common abstract level set for interoperability:

| Level code     | Semantics             |
| -------------- | --------------------- |
| `unclassified` | Not classified        |
| `restricted`   | Low sensitivity       |
| `confidential` | Medium sensitivity    |
| `secret`       | High sensitivity      |
| `top_secret`   | Very high sensitivity |

Authorities MAY map their internal levels to these normalized levels.

## 7. Marking Registry (Recommended)

Implementations SHOULD maintain a registry mapping official markings to normalized form.

**Important:** The `marking` field contains ONLY the official base classification marking. Caveats such as NOFORN, CCI, CRYPTO, ATOMAL are specified separately in the `caveats` field and MUST NOT be included in the marking string.

Example registry entries:

| Marking                         | Authority Code | Level        | Notes                                |
| ------------------------------- | -------------- | ------------ | ------------------------------------ |
| NATO UNCLASSIFIED               | NATO           | unclassified |                                      |
| NATO RESTRICTED                 | NATO           | restricted   |                                      |
| NATO SECRET                     | NATO           | secret       |                                      |
| NATO COSMIC TOP SECRET          | NATO           | top_secret   |                                      |
| RESTREINT UE/EU RESTRICTED      | EU             | restricted   |                                      |
| CONFIDENTIEL UE/EU CONFIDENTIAL | EU             | confidential |                                      |
| SECRET UE/EU SECRET             | EU             | secret       |                                      |
| TRÈS SECRET UE/EU TOP SECRET    | EU             | top_secret   |                                      |
| SECRET                          | FR             | secret       | France post-2021                     |
| TRÈS SECRET                     | FR             | top_secret   | France post-2021                     |
| CONFIDENTIEL DÉFENSE            | FR             | confidential | France pre-2021 (deprecated)         |
| SECRET DÉFENSE                  | FR             | secret       | France pre-2021 (deprecated)         |
| TRÈS SECRET DÉFENSE             | FR             | top_secret   | France pre-2021 (deprecated)         |
| VS-NUR FÜR DEN DIENSTGEBRAUCH   | DE             | restricted   |                                      |
| VS-VERTRAULICH                  | DE             | confidential |                                      |
| GEHEIM                          | DE             | secret       |                                      |
| STRENG GEHEIM                   | DE             | top_secret   |                                      |
| CONFIDENTIAL                    | US             | confidential |                                      |
| SECRET                          | US             | secret       |                                      |
| TOP SECRET                      | US             | top_secret   |                                      |

## 8. Caveats Registry (Recommended)

Implementations SHOULD maintain a registry of recognized caveats for each authority.

### 8.1 Common Caveats

| Caveat   | Authority     | Meaning                                    |
| -------- | ------------- | ------------------------------------------ |
| CRYPTO   | NATO, US, FR, many | Cryptographic information/material    |
| CCI      | NATO, US      | Controlled Cryptographic Item              |
| COMSEC   | NATO, US      | Communications Security material           |
| ATOMAL   | NATO          | Atomic/nuclear information                 |
| BOHEMIA  | NATO          | Releasable to specific nations             |
| BALK     | NATO          | Special handling required                  |
| NOFORN   | US            | No foreign nationals                       |
| ORCON    | US            | Originator controlled dissemination        |
| PROPIN   | US            | Proprietary information involved           |
| FGI      | US            | Foreign Government Information             |
| LES      | US            | Law Enforcement Sensitive                  |
| FOUO     | US            | For Official Use Only                      |

### 8.2 Releasability Caveats

Some caveats specify permitted recipients:

**NATO REL TO format:**
```yaml
caveats: ["REL NATO BEL FRA DEU"]
```

**US REL TO format (Five Eyes example):**
```yaml
caveats: ["REL TO USA, GBR, CAN, AUS, NZL"]
```

### 8.3 Caveat Validation

Implementations SHOULD:
- Validate caveats against the registry for the specified authority
- Reject or warn on unrecognized caveats
- Check for mutually exclusive caveats (e.g., NOFORN and REL TO)
- Preserve caveat order as specified (may be significant)

## 9. Normalization Rules (Normative)

When a simplified form is used, the implementation MUST:
- Look up the marking in the marking registry
- Retrieve the associated authority code and level
- Expand the authority code using the authority registry
- Replace the simplified form with the canonical structured form
- Reject unknown markings unless explicitly configured otherwise

Example:

```yaml
# Input (simplified)
classification: NATO RESTRICTED
```

Normalized:

```yaml
# Output (canonical)
classification:
  authority:
    code: NATO
    type: organization
    name: North Atlantic Treaty Organization
  level: restricted
  marking: NATO RESTRICTED
```

## 10. Validation Rules (Normative)

### 10.1 Structured Classification Objects

When `classification` is an object:

- `authority.code` MUST be uppercase ASCII
- `authority.type` MUST be one of: `country`, `organization`
- `authority.name` MUST be a non-empty string
- `authority` structure MUST match the registered authority for the given code
- `level` MUST be one of the normalized levels
- `marking` MUST be a non-empty string
- `marking` MUST NOT contain caveats (caveats go in the `caveats` field)
- (`authority.code`, `level`) MUST match the official meaning of `marking` if a registry is available
- Authorization logic MUST rely on (`authority.code`, `level`) and NOT on raw marking strings

### 10.2 Caveats Validation

When `caveats` field is present:

- `caveats` MUST be either `null`, an empty array `[]`, or an array of strings
- Each caveat string MUST be uppercase
- Each caveat SHOULD be recognized in the caveats registry for the specified authority
- Implementations SHOULD warn on unrecognized caveats
- Implementations SHOULD check for mutually exclusive caveats
- `caveats` MUST NOT be duplicated in the `marking` field

### 10.3 Special Values

When `classification` is a special value:

- `null` is valid and indicates non-classified status
- `"unknown"` (string) is valid and indicates indeterminate classification
- Both `null` and `"unknown"` MUST NOT be used within structured objects
- Access control systems SHOULD treat `null` as "no restrictions" and `"unknown"` conservatively

### 10.4 Type Safety

The `classification` field MUST be one of:
- A structured object (canonical form)
- A string (simplified marking or `"unknown"`)
- `null` (non-classified)

Any other type (number, boolean, array) is invalid.

## 11. Implementation Recommendations

This section provides non-normative guidance for implementing this specification.

### 11.1 Handling Marking Ambiguity

When multiple authorities use identical or similar markings (e.g., "SECRET" used by both France and the United States):

- Implementations SHOULD use document metadata (source system, creator organization, document origin) to disambiguate
- If ambiguity cannot be resolved, implementations SHOULD:
  - Prompt for manual classification
  - Log the ambiguity for review
  - Default to the most restrictive interpretation when access control is involved

### 11.2 Migration from Legacy Systems

For systems migrating to this specification:

1. **Audit existing classifications**: Identify all unique markings currently in use
2. **Build mapping tables**: Create authority registry and marking registry entries
3. **Handle deprecated markings**: Maintain mappings for historical classifications (e.g., French pre-2021 system)
4. **Extract caveats**: Parse existing combined markings (e.g., "SECRET//NOFORN") into separate `marking` and `caveats` fields
5. **Phased migration**: Support both legacy and canonical forms during transition
6. **Validation logging**: Track normalization failures for manual review

### 11.3 Versioning Strategy

Implementations SHOULD include a `spec_version` field in their metadata:

```yaml
classification:
  spec_version: "1.0.0"
  authority:
    code: NATO
    type: organization
    name: North Atlantic Treaty Organization
  level: restricted
  marking: NATO RESTRICTED
  caveats: [CCI]
```

This enables forward compatibility when the specification evolves.

### 11.4 Performance Considerations

- **Cache normalized forms**: Store both simplified and canonical forms to avoid repeated lookups
- **Index by authority code**: Optimize queries by authority using `authority.code`
- **Precompute equivalencies**: Build lookup tables for cross-authority classification equivalence
- **Index caveats**: For access control, index documents by caveat presence for fast filtering

### 11.5 API Design Guidance

When exposing classification metadata via APIs:

- **Input**: Accept both simplified and canonical forms
- **Output**: Always return canonical form unless explicitly requested otherwise
- **Validation**: Reject invalid classifications at ingestion time
- **Documentation**: Clearly indicate which form is expected/returned

### 11.6 Audit and Compliance

Implementations handling classified information SHOULD:

- Log all classification assignments and changes with timestamps
- Track who assigned/modified classifications (user, system, or automated process)
- Maintain chain of custody for classification decisions
- Support export of audit trails for compliance review

### 11.7 Parsing Legacy Markings with Embedded Caveats

When migrating from legacy systems where caveats were embedded in marking strings (e.g., "SECRET//NOFORN", "NATO SECRET ATOMAL"), implementations need to extract them:

**Common patterns:**
- US format: `LEVEL//CAVEAT1/CAVEAT2` (e.g., "SECRET//NOFORN")
- NATO format: `NATO LEVEL CAVEAT1 CAVEAT2` (e.g., "NATO SECRET ATOMAL")
- Format varies by authority

**Implementation approach:**

```python
# Pseudocode example
def normalize_legacy_marking(combined_marking: str, authority_code: str) -> dict:
    """Extract base marking and caveats from combined string."""
    if authority_code == "US":
        parts = combined_marking.split("//")
        marking = parts[0]  # e.g., "SECRET"
        caveats = parts[1].split("/") if len(parts) > 1 else []
    elif authority_code == "NATO":
        parts = combined_marking.split()
        # NATO markings are at least 2 words: "NATO SECRET"
        marking = " ".join(parts[:2])
        caveats = parts[2:] if len(parts) > 2 else []
    # ... other authorities
    
    return {
        "marking": marking,
        "caveats": caveats
    }

# Example usage
result = normalize_legacy_marking("SECRET//NOFORN", "US")
# Returns: {"marking": "SECRET", "caveats": ["NOFORN"]}
```

**Recommendations:**
- Maintain a registry of known caveats per authority
- Validate parsed caveats against the registry
- Log unrecognized caveats for manual review
- Store in separate fields: `marking` contains only the base marking, `caveats` contains the extracted list

## 12. Forward Compatibility

Implementations SHOULD:
- Ignore unknown fields in the classification object
- Allow new authority codes via registry extension
- Allow new authority types (e.g. `agency`, `coalition`) in future versions
- Allow new markings without schema change
- Preserve unknown fields during normalization
- Allow new caveat codes via caveats registry extension

## 13. Complete Examples

### 13.1 NATO Document (Canonical Form)

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

### 13.2 French Document (Shorthand Authority)

```yaml
classification:
  authority: FR
  level: secret
  marking: SECRET
```

### 13.3 US Document with Caveats (Canonical Form)

```yaml
classification:
  authority:
    code: US
    type: country
    name: United States
  level: secret
  marking: SECRET
  caveats: [NOFORN]
```

### 13.4 NATO Document with Multiple Caveats

```yaml
classification:
  authority:
    code: NATO
    type: organization
    name: North Atlantic Treaty Organization
  level: secret
  marking: NATO SECRET
  caveats: [ATOMAL, CCI]
```

### 13.5 EU Non-Classified Document

EU does not have an official "EU UNCLASSIFIED" marking, so use `null`:

```yaml
classification: null
```

### 13.6 NATO Unclassified Document

NATO has an explicit unclassified level:

```yaml
classification:
  authority:
    code: NATO
    type: organization
    name: North Atlantic Treaty Organization
  level: unclassified
  marking: NATO UNCLASSIFIED
  caveats: null
```

### 13.7 Legacy Document with Unknown Classification

```yaml
classification: unknown
```

### 13.8 Multiple Authorized Classifications

```yaml
authorized_classifications:
  - authority:
      code: NATO
      type: organization
      name: North Atlantic Treaty Organization
    level: restricted
    marking: NATO RESTRICTED
    caveats: null
  - authority:
      code: EU
      type: organization
      name: European Union
    level: restricted
    marking: RESTREINT UE/EU RESTRICTED
    caveats: null
```

### 13.9 US Document with Releasability Caveat

```yaml
classification:
  authority:
    code: US
    type: country
    name: United States
  level: secret
  marking: SECRET
  caveats: ["REL TO USA, GBR, CAN, AUS, NZL"]
```

### 13.10 Simplified Form (Compatibility)

```yaml
classification: SECRET
```

This MUST be normalized to:

```yaml
classification:
  authority:
    code: FR
    type: country
    name: France
  level: secret
  marking: SECRET
```

**Note:** In cases where multiple authorities use the same marking (e.g., both France and US use "SECRET"), implementations MUST use additional context (document source, metadata, etc.) to determine the correct authority.

## 14. Future Considerations

This section outlines potential enhancements for future versions of this specification.

### 14.1 Composite Classifications

Some documents may carry multiple classifications from different authorities simultaneously (e.g., a NATO document that also contains US national information). Future versions could support:

```yaml
classification:
  primary:
    authority: NATO
    level: secret
    marking: NATO SECRET
    caveats: [ATOMAL]
  additional:
    - authority: US
      level: secret
      marking: SECRET
      caveats: [NOFORN]
```

### 14.2 Temporal Classification

Support for time-based automatic declassification:

```yaml
classification:
  authority: US
  level: secret
  marking: SECRET
  caveats: null
  declassify_on: "2030-12-31"
  declassify_review_date: "2028-12-31"
```

### 14.3 Classification Provenance

Track classification decision history:

```yaml
classification:
  authority: FR
  level: secret
  marking: SECRET
  caveats: [CRYPTO]
  provenance:
    classified_by: "Jean Dupont"
    classified_date: "2025-03-15"
    classification_reason: "1.4(c)"
    derived_from: "SOURCE-DOC-2025-001"
```

### 14.4 Caveat and Compartment Support Enhancement

For special access programs and compartmented information, future versions could add structured compartment support beyond the simple caveats array:

```yaml
classification:
  authority: US
  level: top_secret
  marking: TOP SECRET
  caveats: null
  compartments:
    - type: SCI
      programs: [TK, G]
  sap: "SPECIAL-ACCESS-PROGRAM-NAME"
```

### 14.5 Extended Authority Types

Future versions may support additional authority types:

- `agency`: Intelligence agencies, national security services
- `coalition`: Ad-hoc multinational coalitions (e.g., "Combined Joint Task Force")
- `bilateral`: Bilateral agreements between two countries
- `corporate`: Private sector entities with classified contracts

### 14.6 Localization Support

While authority codes remain English, implementations may benefit from localized display:

```yaml
classification:
  authority:
    code: NATO
    type: organization
    name: North Atlantic Treaty Organization
    name_localized:
      fr: Organisation du traité de l'Atlantique nord
      de: Nordatlantikpakt-Organisation
      es: Organización del Tratado del Atlántico Norte
  level: restricted
  marking: NATO RESTRICTED
  caveats: null
```

### 14.7 Machine-Readable Access Control Policies

Express who can access classified information:

```yaml
classification:
  authority: US
  level: secret
  marking: SECRET
  caveats: [NOFORN]
  access_control:
    clearance_required: secret
    citizenship_required: [US]
    need_to_know: true
    organization_restriction: [DOD, IC]
```

## 15. Acknowledgments

This specification builds upon the classification systems of NATO, the European Union, and member states including France, Germany, the United States, and others. It aims to provide interoperability while respecting each authority's sovereignty over their classification systems.

## 16. References

### Normative References
- ISO 3166-1: Codes for the representation of names of countries and their subdivisions – Part 1: Country codes
- YAML 1.2: YAML Ain't Markup Language Version 1.2

### Informative References
- NATO Security Policy (C-M(2002)49)
- Council of the European Union Security Regulations
- French IGI 1300 (Instruction Générale Interministérielle n° 1300)
- US Executive Order 13526: Classified National Security Information
- German VSA (Verschlusssachenanweisung)

## 17. Appendix: Suggested JSON Schema

For implementations requiring formal validation, here is a suggested JSON Schema:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://example.org/classification-metadata-v1.0.0.schema.json",
  "title": "Classification Metadata",
  "description": "Security classification metadata format v1.0.0",
  "oneOf": [
    {"type": "null"},
    {"type": "string", "const": "unknown"},
    {"type": "string", "minLength": 1},
    {
      "type": "object",
      "required": ["authority", "level", "marking"],
      "properties": {
        "spec_version": {
          "type": "string",
          "pattern": "^\\d+\\.\\d+\\.\\d+$"
        },
        "authority": {
          "oneOf": [
            {"type": "string", "pattern": "^[A-Z]{2,10}$"},
            {
              "type": "object",
              "required": ["code", "type", "name"],
              "properties": {
                "code": {"type": "string", "pattern": "^[A-Z]{2,10}$"},
                "type": {"type": "string", "enum": ["country", "organization"]},
                "name": {"type": "string", "minLength": 1}
              }
            }
          ]
        },
        "level": {
          "type": "string",
          "enum": ["unclassified", "restricted", "confidential", "secret", "top_secret"]
        },
        "marking": {
          "type": "string",
          "minLength": 1
        },
        "caveats": {
          "oneOf": [
            {"type": "null"},
            {
              "type": "array",
              "items": {
                "type": "string",
                "pattern": "^[A-Z][A-Z0-9\\s,]*$"
              }
            }
          ]
        }
      }
    }
  ]
}
```
