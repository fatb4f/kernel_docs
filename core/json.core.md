The current kernel already names:

* `json_structure`
* `json_schema`
* `cue`
* `jsonnet`
* `structurize_avrotize`

What is missing is a **formal extension taxonomy** and the rules for how each JSON-family artifact participates in:

* authority
* lineage
* admissibility
* rendering
* interface transport
* operational evidence
* codegen

---

# What should be added to the amendment

## 1. Add a `json_family_model` section

This should explicitly classify the JSON-family layers.

### Recommended taxonomy

#### A. Core authoring and control

* **JSON Structure** = canonical authoring syntax for the structural model
* **JSON Schema** = derived exported contract layer
* **CUE** = admissibility / legality over normalized state
* **Jsonnet** = rendering / projection from admitted state

#### B. Operational and interface envelopes

* **NDJSON / JSONL** = append-only evidence, event, and stream artifacts
* **JSON-RPC** = request/response/session control envelope for App-Server-class interaction surfaces
* **JSON:API** = resource-oriented API envelope for external boundary surfaces
* **JSON-LD** = semantic graph / linked-data extension when graph semantics are required

#### C. Derived interop and codegen

* **Structurize** = structure-oriented derived interop/codegen bridge
* **Avrotize** = schema/record/message-oriented derived interop/codegen bridge

This avoids flattening everything into “JSON,” which is now too coarse for the kernel.

---

## 2. Update the authority model

The authority model should stop at the right boundary.

### Recommended interpretation

* **Canonical structural model** is authority
* **JSON Structure** is authoritative authoring syntax for that model
* **JSON Schema** is derived contract/export surface
* **CUE** is admissibility over normalized state
* **Jsonnet** is rendering over admitted state
* **NDJSON** is derived operational evidence/event serialization
* **JSON-RPC** is a bounded interaction envelope, not authority
* **JSON:API** is a bounded resource/API envelope, not authority
* **JSON-LD** is a derived semantic projection, not authority
* **Structurize/Avrotize** are derived interop/codegen bridges, not authority

### Important correction

`json-structure` is not just a side syntax note anymore.
It belongs in the **core stack**, not in a footnote.

---

## 3. Expand the state flow

Current:

```text
raw sources -> normalized state -> admitted state -> rendered artifacts
```

Required expanded view:

```text
raw sources
-> normalized state
-> admitted state
-> derived exec state
-> rendered artifacts
-> interface envelopes / evidence streams / codegen artifacts
```

---

## 4. Add a formal extension registry concept

The amendment should define a **kernel extension registry** for JSON-family formats.

### Suggested object

* `json-family-extension.index.json`

### Each entry should include

* `extension_id`
* `kind`
* `authority_role`
* `artifact_class`
* `admission_scope`
* `lineage_requirement`
* `allowed_planes`
* `producer_layer`
* `consumer_layer`
* `stability`
* `notes`

### Example extension kinds

* `json_structure`
* `json_schema`
* `cue`
* `jsonnet`
* `ndjson`
* `json_rpc`
* `json_api`
* `json_ld`
* `structurize`
* `avrotize`

---

# How each addition should be treated

## JSON Structure

This belongs in the amendment as **core**, not optional.

### Kernel role

* canonical human authoring syntax
* input to normalization
* source-side structure material

### New invariant

* JSON Structure is source-authority syntax, but legality is not decided until CUE admission.

---

## NDJSON / JSONL

This should be made **required** for operational evidence/event classes where append-only streaming matters.

### Kernel role

* evidence streams
* event logs
* per-step run traces
* batch operational records

### New artifact classes

* `event-stream`
* `evidence-stream`

### New evidence locations

* `generated/state/events/.../*.ndjson`
* `generated/state/evidence/.../*.ndjson`

### New invariant

* NDJSON artifacts are derived evidence/event streams and may not serve as authority inputs.

---

## JSON-RPC

This should be formalized as the **session/control transport envelope** for App-Server-class surfaces.

### Kernel role

* bidirectional request/response/event transport
* turn/session control
* bounded execution interaction plane

### Important boundary

* JSON-RPC does not create authority
* it carries admitted operations only

### New invariant

* JSON-RPC envelopes may expose control interactions but may not bypass admission, provenance, or promotion rules.

---

## JSON:API

This should be formalized as the **resource-oriented external boundary surface**.

### Kernel role

* consumer-facing API resources
* resource indexing
* external contract surfaces

### New invariant

* JSON:API exports are derived contract surfaces with lineage back to canonical families.

---

## JSON-LD

This should be added as **conditionally admitted**, not necessarily baseline-required for every kernel run.

### Recommended status

* `optional-but-recognized`
* admitted only when semantic graph export is declared by manifest or policy

### Kernel role

* linked-data graph projection
* semantic registry/export
* ontology-aligned interoperability when needed

### New invariant

* JSON-LD is a derived semantic projection over admitted state or exported contract state; it is not a source authority format.

That is the safest way to include it now without overcommitting the baseline.

---

## Structurize / Avrotize

These should move from “mentioned in authority model” to **first-class codegen/interoperability bridge responsibilities**.

### Kernel role

* downstream codegen inputs
* schema/record/message bridge generation
* interop projection for typed consumers

### New artifact classes

* `interop-bridge`
* `codegen-bridge`
* `message-contract`
* `record-contract`

### New invariant

* Structurize/Avrotize outputs are derived from admitted/exported kernel state and may not become alternative authority sources.

---

# Required changes to `artifact_classes`

The current list is too small. Add at least:

* `exec-blueprint`
* `lineage-report`
* `schema-bundle`
* `schema-dereference`
* `compatibility-export`
* `event-stream`
* `evidence-stream`
* `rpc-envelope`
* `api-surface`
* `semantic-projection`
* `interop-bridge`
* `codegen-bridge`

---

# Required changes to `planes`

## Structure plane

Keep:

* canonical model
* JSON Structure authoring inputs

## Contract plane

Expand to include:

* exported JSON Schema
* schema bundles
* dereferenced schema exports
* compatibility schemas
* JSON:API contract surfaces
* JSON-RPC message contracts
* optional JSON-LD contract projections
* Structurize/Avrotize codegen bridge inputs

## Policy plane

Expand to include:

* CUE admissibility for extension legality
* interface envelope legality
* semantic projection legality
* codegen bridge admissibility

## Artifact plane

Expand to include:

* NDJSON evidence streams
* JSON-RPC traces or session event views
* JSON:API rendered resource outputs
* JSON-LD semantic projections
* Structurize/Avrotize generated interop artifacts

---

# Required changes to the gate model

## G2 — Contract export

Must now validate:

* JSON Schema `$ref` lineage
* export mode identity
* JSON:API surface export validity
* JSON-RPC contract export validity
* optional JSON-LD projection export validity
* Structurize/Avrotize bridge generation success where declared

### New required evidence

* `lineage.index.json`
* `ref-graph.json`
* `bundle-report.json`
* `dereference-report.json`
* `extension-export-report.json`

---

## G4 — Admission

Must now validate:

* extension legality
* interface envelope legality
* whether JSON-RPC surfaces are allowed
* whether JSON:API surfaces are allowed
* whether JSON-LD projection is permitted
* whether Structurize/Avrotize bridge emission is permitted

### New required evidence

* `admission-input-manifest.json`
* `policy-bundle-lock.json`
* `extension-admissibility-report.json`

---

## G5 — Rendering / derivation

Should explicitly include:

* JSON:API surface generation
* JSON-LD projection generation
* NDJSON evidence sink declaration
* Structurize/Avrotize output generation where declared

---

# Required repo additions

## Under `structures/`

Add extension declarations:

```text
structures/extensions/
  json-family-extension.index.json
  interface-envelope.index.json
  codegen-bridge.index.json
```

## Under `schemas/exported/`

Add lineage/export support:

```text
schemas/exported/
  lineage.index.json
  bundles/
  dereferenced/
  api/
  rpc/
  semantic/
```

## Under `policy/`

Add:

```text
policy/kernel/
  extension-kinds.cue
  lineage.cue

policy/admission/
  extension-admission.cue
  interface-envelope-admission.cue
  codegen-bridge-admission.cue
```

## Under `generated/`

Add:

```text
generated/schemas/lineage/
generated/state/exec/
generated/state/events/
generated/state/evidence/
generated/state/api/
generated/state/rpc/
generated/state/semantic/
generated/state/codegen/
```

---

# The shortest correct amendment shape

The amendment should introduce a section like this:

## JSON Family Stack and Extension Model

And define:

### Core stack

* JSON Structure
* JSON Schema
* CUE
* Jsonnet

### Recognized derived extensions

* NDJSON
* JSON-RPC
* JSON:API
* JSON-LD

### Recognized derived codegen bridges

* Structurize
* Avrotize

### Rule

All recognized JSON-family extensions must declare:

* authority role
* plane membership
* lineage requirements
* admissibility requirements
* output class
* generator/producer identity

---

# Recommended stance on JSON-LD

Do **not** make JSON-LD universally required in the first amendment.

Make it:

* recognized
* policy-controlled
* optional until a semantic export surface is declared

That keeps the amendment tight while still reserving the capability.

---

# Bottom line

Yes — the amendment has to expand beyond `$ref` lineage and CUE admissibility alone.

It now needs to formalize a **JSON-family kernel model** with:

* **core:** JSON Structure, JSON Schema, CUE, Jsonnet
* **operational/interface:** NDJSON, JSON-RPC, JSON:API
* **conditional semantic extension:** JSON-LD
* **codegen/interop bridges:** Structurize, Avrotize

