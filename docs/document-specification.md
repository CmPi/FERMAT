# FERMAT Document Element Specification

Part of the FERMAT (Format for Electronic Registry Manifest Automation and Transfer) specification.

---

## 1. Introduction

### 1.1 Purpose

This specification defines the structure and fields for individual document entries within a FERMAT manifest. Each document element represents a single file transmitted within a datapack and contains all metadata necessary for:

- Identification and cataloging
- Classification and access control
- Integrity verification
- Audit trail and provenance tracking
- Automated processing by Document Management Systems (DMS)

### 1.2 Scope

This document covers:
- All required and optional fields for document elements
- Data types, formats, and validation rules
- Classification handling (references [Classification Metadata Format](classification-specificaion.md))
- Integrity verification methods (checksums, HMAC, digital signatures)
- File format and MIME type handling
- Localization support

### 1.3 Normative References

- **FERMAT Specification** - Parent manifest specification
- **Classification Metadata Format** - Classification structure specification
- **RFC 3339** - Date and time format
- **RFC 4648** - Base16 (hex) and Base64 encoding
- **RFC 2046** - MIME types
- **ISO 639-1** - Language codes
- **RFC 5652** - Cryptographic Message Syntax (CMS) for digital signatures

---

## 2. Document Element Structure

A document element is a structured object with the following field categories:

```yaml
documents:
  - # Identification
    id: string (required)
    filename: string (required)
    path: string (optional)
    title: string (optional)
    description: string (optional)
    
    # Classification
    classification: object/string/null
    
    # Versioning & Language
    version: string (optional)
    revision: string (optional)
    language: string (optional)
    
    # References
    reference-number: string (optional)
    supersedes: string (optional)
    related-documents: array (optional)
    
    # Authorship & Origination
    author: string (optional)
    author-organization: string (optional)
    originator: string (optional)
    originator-organization: string (optional)
    
    # Approval
    approval:
      date: date
      authority:
        person: string (optional)
        organization: string (optional)
        function: string (optional)
      status: string (optional)
    
    # File Format
    format:
      mime-type: string (optional)
      type: string (optional)
      page-count: integer (optional)
    
    # File Metadata
    size-bytes: integer (optional)
    created: datetime (optional)
    modified: datetime (optional)
    
    # Integrity & Security
    checksum:
      algorithm: string
      value: string
    hmac:
      algorithm: string
      value: string
      key-id: string (optional)
    signature:
      algorithm: string
      value: string
      certificate: string (optional)
      timestamp: datetime (optional)
    
    # Status & Keywords
    status: string (optional)
    keywords: array (optional)
    subject: string (optional)
```

---

## 3. Identification Fields

### 3.1 Document ID

