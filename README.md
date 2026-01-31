# jhcontext Protocol — Draft Specification (v0.1)

## Abstract
The jhcontext protocol is a research protocol for managing semantic context in multi-agent AI systems with explicit support for provenance, lifecycle management, and auditability. The protocol is designed to encapsulate existing context models without constraining their internal semantics, enabling traceable and verifiable context usage in distributed AI systems.

## Scope
This document specifies the core concepts, structure, and intended usage of the jhcontext protocol. It is a draft research specification and is subject to change.

## Non-Goals
- Defining new context ontologies
- Mandating a specific agent architecture
- Standardizing inference mechanisms

## Status of This Document
This is a working draft released for academic discussion and early experimentation.

The publication currently under work for submission.

## Status
- Version: v0.1-draft
- Stability: Research draft (subject to change)
- Intended use: Academic research and prototyping


Files:
- `jhcontext-core.jsonld` — Minimal, normative core envelope (JSON-LD).
- `prov-example.jsonld` — Sample envelope + PROV graph for the smart-office scenario.
- `README.md` — This file.

Usage:
- Use `jhcontext-core.jsonld` as the basis for schema validation in implementations.
- Use `prov-example.jsonld` as a test vector for canonicalization, signing and provenance mapping.


## License
This project is licensed under the Apache License, Version 2.0.