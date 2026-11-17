# File Format Encapsulation Layers

**Author:** Jay Reck
**Updated:** 2026-11-17

---

## Concept

File formats follow the same encapsulation logic as OSI layers. Each level wraps the previous one's data with more metadata, more structure, and more overhead. Reading from bottom to top — raw bits at Layer 1, full envelope at Layer 7.

---

## The Stack

```
Layer 7 — Envelope     → SOAP / MD
Layer 6 — Verbose      → XML
Layer 5 — Annotated    → YAML
Layer 4 — Structured   → JSON
Layer 3 — Streaming    → NDJSON
Layer 2 — Flat         → CSV
Layer 1 — Raw          → Binary / Protobuf / Bits
```

---

## Layer Descriptions

Layer 1 — Binary / Protobuf / Bits. No human-readable structure. Pure packed data defined by a schema outside the file itself. Most compact, fastest to parse, zero overhead. Protocol Buffers and MessagePack live here.

Layer 2 — CSV. Flat tabular data. Rows and columns. A header row labels the fields but the file carries no other metadata. No nesting, no types, no relationships. What you see is exactly the data and nothing else.

Layer 3 — NDJSON (Newline-Delimited JSON). Each line is a self-contained JSON object. Hybrid between CSV's row-based structure and JSON's key-value labelling. Every field is semantically named but the file remains streamable line by line. Used by streaming APIs and event logs.

Layer 4 — JSON. Full nested key-value structure. Arrays and objects. Every field is labelled, values can be nested to any depth, types are expressed (string, number, boolean, null). The standard for structured data exchange.

Layer 5 — YAML. A superset of JSON with human-readable formatting. Supports comments, anchors, and references between nodes — none of which JSON can express. The standard for configuration files and pipelines where humans need to read and edit the content directly.

Layer 6 — XML. Tags, attributes, namespaces, and schema validation through XSD. Every value is explicitly wrapped in named opening and closing tags with optional metadata attributes. Verbose but formally validatable against a defined schema.

Layer 7 — SOAP / MD. The envelope layer. SOAP wraps XML in a Header block carrying processing metadata and a Body block carrying the payload — the closest file format equivalent to a fully encapsulated network packet. Markdown sits here too as the richest human-readable layer, carrying prose, structure, code blocks, and links — the full document form.

---

## Repo Folder Mapping

```
ccna/ and cloudquest/
  CAN/    → JSON   — structured knowledge trees, course plans
  COULD/  → CSV    — tabular comparisons, payload examples
  MAY/    → LOG    — event logs, time-series records (NDJSON-adjacent)
  MIGHT/  → MD     — rich asset notes, full prose documents
  PIPE/   → YAML   — pipelines, workflow definitions
```

---

## What Sits Between the Current Formats

Between CSV and JSON — NDJSON fills the gap. Each row becomes a JSON object. Useful when the data has consistent structure per row but each row needs typed, labelled fields rather than positional columns.

Between JSON and YAML — BSON (Binary JSON) or TOML sit here. BSON trades human-readability for speed and binary compactness while keeping JSON's structure. TOML offers a cleaner, more explicit config syntax than YAML with less ambiguity.

Between YAML and XML — no widely adopted format occupies this space. YAML already handles most of what XML does for config purposes. XML's main advantage over YAML is formal schema validation, which has no YAML equivalent.

Between XML and SOAP — WSDL (Web Services Description Language) describes the structure of a SOAP service in XML. It is the schema layer that sits just beneath the full SOAP envelope.

---

## Analogy to OSI

Just as an OSI packet nests the application data inside successive headers going down the stack — TLS record wrapping HTTP, inside a TCP segment, inside an IP packet, inside an Ethernet frame — a SOAP message nests the actual business data inside a Body block, inside an Envelope, expressed in XML, which could itself be transported as a JSON string, inside a CSV cell, stored as binary on disk.

Each format is a valid stopping point depending on how much structure, verbosity, and human-readability the use case requires.
