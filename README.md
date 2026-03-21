# jhcontext Protocol Specification (v0.3)

## Abstract

The **jhcontext protocol** (PAC-AI — Protocol for Auditable Context in AI) is a research protocol for managing semantic context in multi-agent AI systems with explicit support for provenance, lifecycle management, and auditability. It encapsulates existing context models without constraining their internal semantics, enabling traceable and verifiable context usage aligned with the EU AI Act.

## Status

- **Version:** v0.3
- **Stability:** Research specification (targeting AIS 2026)
- **Reference implementation:** [jhcontext-sdk](https://github.com/jhdarosa/jhcontext-sdk) (Python, v0.2.x)

## Non-Goals

- Defining new context ontologies
- Mandating a specific agent architecture
- Standardizing inference mechanisms

---

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Envelope** | Top-level context container. Carries semantic payload, artifact metadata, privacy/compliance blocks, provenance reference, and cryptographic proof. |
| **Artifact** | A tracked computational product (model output, embedding, extraction, tool result) registered in the envelope. |
| **Decision Influence** | Records how context influenced an agent's decision — which categories, weights, and abstraction levels were used. |
| **PROV Graph** | W3C PROV provenance graph linked to the envelope via `provenance_ref`. Maps entities to artifacts, activities to pipeline steps, and agents to DIDs. |
| **Proof** | Cryptographic integrity block: URDNA2015 canonicalization → SHA-256 hash → Ed25519 signature. |

---

## Envelope Structure

All fields match the canonical schema in `jhcontext-core.jsonld`.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `context_id` | string | yes | `ctx-<uuid>` | Unique envelope identifier |
| `schema_version` | string | yes | `"jh:0.3"` | Protocol version |
| `producer` | string (DID) | yes | — | DID of the producing agent |
| `created_at` | ISO 8601 datetime | yes | now | Creation timestamp |
| `ttl` | ISO 8601 duration | yes | `"PT30M"` | Time-to-live |
| `status` | EnvelopeStatus | yes | `"active"` | Lifecycle status |
| `scope` | string | yes | — | Regulatory/business context (e.g. `healthcare_treatment_recommendation`) |
| `semantic_payload` | array of objects | yes | `[]` | Semantic context layers (see UserML below) |
| `artifacts_registry` | array of Artifact | yes | `[]` | Tracked computational artifacts |
| `passed_artifact_pointer` | string \| null | no | `null` | Points to the artifact selected for decision |
| `decision_influence` | array of DecisionInfluence | yes | `[]` | How context influenced decisions |
| `privacy` | PrivacyBlock | yes | defaults | Privacy and data protection metadata |
| `compliance` | ComplianceBlock | yes | defaults | Regulatory compliance metadata |
| `provenance_ref` | ProvenanceRef | yes | defaults | Link to W3C PROV graph |
| `proof` | Proof | yes | defaults | Cryptographic integrity block |

---

## Component Models

### Artifact

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `artifact_id` | string | yes | `art-<hex8>` | Unique artifact identifier |
| `type` | ArtifactType | yes | — | Classification of the artifact |
| `storage_ref` | string \| null | no | `null` | URI to external storage |
| `content_hash` | string \| null | no | `null` | SHA-256 hex digest |
| `model` | string \| null | no | `null` | Model that produced the artifact |
| `timestamp` | ISO 8601 datetime | yes | now | Creation timestamp |
| `deterministic` | boolean | yes | `false` | Whether output is reproducible |
| `confidence` | float \| null | no | `null` | Confidence score [0–1] |
| `dimensions` | integer \| null | no | `null` | Embedding dimensionality |
| `metadata` | object | yes | `{}` | Application-specific metadata |

### DecisionInfluence

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `agent` | string (DID) | yes | — | Agent that made the decision |
| `categories` | array of string | yes | — | Data dimensions used (e.g. `patient_status`, `tumor_response`) |
| `abstraction_level` | AbstractionLevel | yes | `"situation"` | UserML layer of the influence |
| `temporal_scope` | TemporalScope | yes | `"current"` | Whether current or historical data was used |
| `influence_weights` | object (string → float) | yes | `{}` | Per-category weight [0–1] |
| `confidence` | float | yes | `0.0` | Overall decision confidence |

### PrivacyBlock

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `data_category` | DataCategory | yes | `"behavioral"` | GDPR data classification |
| `legal_basis` | string | yes | `"consent"` | Legal basis for processing |
| `retention` | ISO 8601 duration | yes | `"P7D"` | Data retention period |
| `storage_policy` | string | yes | `"centralized-encrypted"` | Storage approach |
| `feature_suppression` | array of string | yes | `[]` | Excluded attributes (e.g. `patient_name`) |

### ComplianceBlock

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `risk_level` | RiskLevel | yes | `"medium"` | Regulatory risk classification |
| `human_oversight_required` | boolean | yes | `false` | Whether human review is mandatory |
| `model_card_ref` | string \| null | no | `null` | URI to model card |
| `test_suite_ref` | string \| null | no | `null` | URI to fairness/safety tests |
| `escalation_path` | string \| null | no | `null` | Contact for escalation |

### ProvenanceRef

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `prov_graph_id` | string \| null | no | `null` | Identifier of the linked PROV graph |
| `prov_digest` | string \| null | no | `null` | SHA-256 digest of serialized PROV graph |

### Proof

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `canonicalization` | string | yes | `"URDNA2015"` | Canonicalization algorithm |
| `content_hash` | string \| null | no | `null` | SHA-256 of canonicalized envelope (excluding proof) |
| `signature` | string \| null | no | `null` | Ed25519 hex signature |
| `signer` | string \| null | no | `null` | DID of the signing agent |

---

## Enumerations

| Enum | Values | Used in |
|------|--------|---------|
| **ArtifactType** | `token_sequence`, `embedding`, `semantic_extraction`, `tool_result` | Artifact.type |
| **RiskLevel** | `low`, `medium`, `high` | ComplianceBlock.risk_level |
| **AbstractionLevel** | `observation`, `interpretation`, `situation` | DecisionInfluence.abstraction_level |
| **TemporalScope** | `current`, `historical` | DecisionInfluence.temporal_scope |
| **EnvelopeStatus** | `active`, `expired`, `deleted` | Envelope.status |
| **DataCategory** | `behavioral`, `biometric`, `sensitive` | PrivacyBlock.data_category |

---

## Semantic Payload (UserML)

The `semantic_payload` carries context as an array of UserML objects with four abstraction layers:

```json
{
  "@model": "UserML",
  "layers": {
    "observation": [
      { "subject": "user:alice", "predicate": "temperature", "object": 22.3 }
    ],
    "interpretation": [
      { "subject": "user:alice", "predicate": "comfort", "object": "high", "confidence": 0.92 }
    ],
    "situation": [
      { "subject": "user:alice", "predicate": "isInSituation", "object": "meeting", "confidence": 0.95 }
    ],
    "application": [
      { "subject": "notification", "predicate": "shouldBeDelivered", "object": false }
    ]
  }
}
```

| Layer | Purpose | Key fields |
|-------|---------|------------|
| **observation** | Raw sensor/data facts | subject, predicate, object |
| **interpretation** | Inferred insights | + `confidence` |
| **situation** | High-level contextual states | + `confidence`, `start` (ISO 8601) |
| **application** | Domain-specific actions | application-defined |

---

## Provenance Model (W3C PROV)

Each envelope links to a W3C PROV graph via `provenance_ref`. The protocol extends PROV with:

- **Entities** → map to artifacts (`jh:artifactType`, `jh:contentHash`)
- **Activities** → pipeline steps (`jh:method`, `prov:startedAtTime`, `prov:endedAtTime`)
- **Agents** → identified by DIDs (`jh:role`)

### PROV Relations

| Relation | Meaning |
|----------|---------|
| `prov:wasGeneratedBy` | Entity created by activity |
| `prov:used` | Activity consumed entity |
| `prov:wasAssociatedWith` | Agent responsible for activity |
| `prov:wasDerivedFrom` | Entity derived from another (causal chain) |
| `prov:wasInformedBy` | Activity depends on another (temporal) |

The envelope references the PROV graph via `provenance_ref.prov_graph_id` and binds it cryptographically via `provenance_ref.prov_digest` (SHA-256 of serialized graph).

**Serialization:** Turtle (primary), JSON-LD, RDF/XML.

**Namespaces:** `jh:` = `https://jhcontext.com/vocab#`, `prov:` = `http://www.w3.org/ns/prov#`

---

## Audit & Compliance Verification

The protocol defines four verification operations that produce structured `AuditResult` records (check_name, passed, evidence, message):

### verify_temporal_oversight — EU AI Act Art. 14

Proves meaningful human oversight by verifying that human review activities occurred **after** AI generation and that total review duration meets a minimum threshold.

**Inputs:** PROV graph, AI activity ID, human activity IDs, min review seconds.
**Evidence:** temporal sequence, review durations.

### verify_negative_proof — EU AI Act Art. 13

Proves that excluded data categories (e.g. biometric, sensitive) are **absent** from a decision's dependency chain via recursive PROV traversal (`wasDerivedFrom` + `used`).

**Inputs:** PROV graph, decision entity ID, excluded artifact types.
**Evidence:** dependency chain, violations (if any).

### verify_workflow_isolation

Proves that two PROV graphs share **zero entities**, preventing cross-workflow data leakage (e.g. grading vs. equity reporting).

**Inputs:** two PROV graphs.
**Evidence:** entity counts per graph, shared entity set.

### verify_integrity

Validates envelope cryptographic integrity: recomputes SHA-256 of URDNA2015-canonicalized JSON-LD (excluding proof block) and verifies Ed25519 signature.

**Inputs:** signed envelope.
**Evidence:** recomputed hash, signer DID, signature validity.

Multiple results combine into an **AuditReport** with `context_id`, `timestamp`, and `overall_passed`.

---

## Changelog

| Version | Changes |
|---------|---------|
| **v0.3** | Added: `scope`, `artifacts_registry`, `passed_artifact_pointer`, `decision_influence`, `privacy` block, `compliance` block. Added 6 enums. Defined 4 audit verification operations. |
| **v0.2** | Initial JSON-LD schema with UserML payload, PROV integration, and cryptographic proof. Included `performative` and `replaces` fields. |
| **v0.1** | Draft specification document. |

**Removed in v0.3:** `performative` (communication semantics — out of scope; protocol focuses on context encapsulation), `replaces` (version chaining — handled externally via PROV `wasDerivedFrom`).

---

## Files

- `jhcontext-core.jsonld` — Normative core envelope schema (JSON-LD, v0.3).
- `prov-example.jsonld` — Example envelope + PROV graph for the smart-office scenario.
- `README.md` — This specification.

## License

This project is licensed under the Apache License, Version 2.0.
