## Expanding the triple stack

Yes. The triple stack can be expanded, but the best expansions are usually **adjacent JSON-centric tools with a narrow role**, not more overlapping “policy languages.”

Base stack:

* **JSON Schema** → shape / boundary validity
* **CUE** → admissibility / composition / constraints
* **Jsonnet** → assembly / projection / generation

The next tools usually fall into a few families:

---

# 1. Query, transform, inspect

## **jq**

Most commonly associated tool.

### Role

* inspect JSON
* slice resolved state
* build small checks
* normalize outputs
* glue shell pipelines

### Common use

```bash
jq '.artifacts | keys' resolved/kernel.resolved.json
jq '.profiles.active' manifests/profiles.json
```

### Why it fits

It gives cheap, precise read-paths into every artifact in the stack.

### Best use

* debugging
* CI assertions
* lightweight reporting
* post-render checks

---

## **yq**

Common when YAML exists at the boundary.

### Role

* YAML↔JSON bridging
* querying YAML manifests
* converting inputs before validation

### Why it fits

Many real systems still author in YAML even if the kernel normalizes to JSON.

---

## **gron**

Less universal, but useful.

### Role

* flatten JSON into assignment-like lines
* grep/diff inspection

### Why it fits

Helpful for debugging large resolved artifacts.

---

# 2. Validation and schema tooling

## **Ajv**

Probably the most common JSON Schema validator in JS/Node ecosystems.

### Role

* fast schema validation
* CI validation
* editor/runtime integration
* standalone validator generation

### Why it fits

It operationalizes the JSON Schema plane.

### Typical use

* validate manifests and objects
* compile validators for speed
* use in pre-commit or CI

---

## **python-jsonschema**

Very common in Python ecosystems.

### Role

* JSON Schema validation from Python tools/scripts

### Why it fits

If the repo uses Python-based orchestration, this is the usual validation layer.

---

## **Spectral**

Often associated when schema and structured policy linting need style rules.

### Role

* lint JSON/YAML documents
* apply style/policy conventions
* enforce authoring hygiene

### Why it fits

JSON Schema validates shape; Spectral can validate authoring conventions and design rules.

### Good for

* naming conventions
* required metadata
* anti-pattern detection

---

## **OpenAPI / AsyncAPI toolchains**

Not always part of the kernel, but often adjacent.

### Role

* API contracts expressed as JSON/YAML
* schema extraction and validation
* docs/client generation

### Why it fits

These are JSON-centric contract ecosystems that often feed into the same repo patterns.

---

# 3. Patch, diff, merge, evolution

## **JSON Patch / JSON Merge Patch**

Very commonly associated once overlays and mutations exist.

### Role

* represent changes as data
* apply bounded modifications
* model deltas explicitly

### Why it fits

Useful when you want overlays or promotions to be explicit and auditable.

### Common RFCs

* JSON Patch: RFC 6902
* JSON Merge Patch: RFC 7386

### Good for

* review bundles
* promotion deltas
* bounded repair operations

---

## **jd / dyff / structural diff tools**

Common for semantic JSON diffs.

### Role

* structural diffs rather than line diffs

### Why it fits

Generated JSON artifacts are often noisy under plain text diff.

---

## **json-diff / similar tools**

Useful for regression/golden testing.

### Role

* compare rendered or resolved artifacts semantically

---

# 4. Templating, generation, and shaping

## **Jinja2 / Mustache / Handlebars**

Often nearby, though less ideal than Jsonnet for structure-first generation.

### Role

* text/template emission
* docs/scripts/config fragments

### Why it fits

Sometimes you need final text outputs after JSON-level normalization.

### Best practice

Use them only at the final text edge, not for core structured composition.

---

## **Dhall**

Not JSON-native, but often mentioned in the same design space.

### Role

* typed config language with imports and normalization

### Why it is adjacent

Similar motivations to CUE/Jsonnet, though usually you would not add it on top of this stack unless there is a strong reason.

---

## **Starlark**

Another adjacent structured-configuration/programmatic option.

### Role

* deterministic configuration logic
* build-ish data composition

### Why it appears nearby

Some teams prefer it where they want constrained programmability.

---

# 5. Data modeling and typing around JSON

## **TypeScript type generation tools**

Very common companion layer.

### Examples

* `json-schema-to-typescript`
* `quicktype`
* `datamodel-code-generator` for Python models

### Role

* generate typed client/server/domain models from schema
* keep implementation types aligned with boundary contracts

### Why it fits

JSON Schema becomes the contract source; typed code becomes the consumer layer.

---

## **Pydantic / Zod / io-ts**

These often appear as runtime type systems around schema-driven workflows.

### Role

* typed validation in application code
* bridge between schema contracts and runtime logic

### Why it fits

They operationalize JSON-based contracts inside applications.

---

## **JMESPath**

Less common than jq in CLI workflows, but widely used in cloud tooling.

### Role

* query JSON documents with a standard expression language

### Why it fits

Useful if the broader toolchain already uses it.

---

# 6. Packaging, bundling, normalization

## **JSON Lines (JSONL)**

Very common adjacent format.

### Role

* event logs
* evidence streams
* incremental output records

### Why it fits

A kernel repo often grows into pipelines, audit logs, or event trails.

---

## **NDJSON**

Same general family.

### Good for

* append-only operational artifacts
* logs
* batch transforms

---

## **Schema bundlers / dereferencers**

Very common if schemas get modular.

### Examples

* tools that dereference `$ref`
* schema bundlers for distribution

### Role

* produce a portable bundled schema artifact

### Why it fits

Useful for releases, external consumers, editors, CI portability.

---

# 7. Metadata, graph, registry, and catalog tools

## **cargo metadata**

You already mentioned this earlier, and it fits well when Rust workspaces are involved.

### Role

* machine-readable workspace/package graph
* source of structured metadata
* feed kernel manifests or resolved views

### Why it fits

It is JSON-producing and slots naturally into the same policy/render flow.

---

## **package-lock/package manifests / ecosystem metadata**

General pattern:

* consume machine-readable JSON metadata from external tools
* normalize through CUE
* project with Jsonnet

This is common with:

* npm
* Cargo
* Python packaging metadata
* CI metadata

---

## **Graph export tools**

Often the resolved state gets exported into graph-friendly JSON formats.

### Role

* dependency graph projection
* registry projection
* impact analysis views

---

# 8. Linting, formatting, authoring ergonomics

## **prettier**

Common for JSON and JSON Schema formatting.

### Role

* consistent formatting
* cleaner diffs

---

## **jsonlint**

Simple but still common.

### Role

* syntax sanity checks

---

## **VS Code JSON Schema support**

Very common part of the working stack.

### Role

* completions
* hover docs
* validation in editor

### Why it fits

It makes the schema plane usable for daily authoring.

---

## **Schema Store integration**

Often associated with JSON Schema-driven repos.

### Role

* schema discovery
* better editor integration

---

# 9. Observability, testing, and assurance

## **Golden-file testing tools**

Very commonly associated with Jsonnet/render pipelines.

### Role

* compare expected rendered JSON with actual output

### Why it fits

Great for artifact-plane regression control.

---

## **OPA / Rego**

Not JSON-only, but often added when policy decisions need operational enforcement.

### Role

* evaluate decisions over JSON input
* admission/gating policy
* CI/runtime allow/deny checks

### Why it fits

If CUE is repo legality, OPA is often added later for operational decision points.

### Good split

* **CUE** for model legality/composition
* **OPA** for explicit allow/deny policy evaluation over structured input

---

## **CEL**

Another adjacent rule language.

### Role

* lightweight policy/condition evaluation over structured data

### Why it fits

Useful where full OPA is too heavy.

---

# 10. Common expanded-stack patterns

Here are the most common real-world expansions.

## Pattern A — Minimal operational expansion

* JSON Schema
* CUE
* Jsonnet
* jq
* Ajv
* CI runner

This is the smallest serious stack.

---

## Pattern B — Kernel/policy-plane expansion

* JSON Schema
* CUE
* Jsonnet
* jq
* Ajv
* JSON Patch
* registry/index manifests
* golden tests

This is common for authoritative config/policy repos.

---

## Pattern C — Developer-tooling expansion

* JSON Schema
* CUE
* Jsonnet
* jq/yq
* generated TS/Python models
* VS Code schema integration
* prettier/jsonlint
* CI drift checks

This is common for teams that want strong editor ergonomics.

---

## Pattern D — Governance / enforcement expansion

* JSON Schema
* CUE
* Jsonnet
* jq
* Ajv
* OPA/Rego or CEL
* JSON Patch
* provenance/evidence JSONL
* bundle/export tooling

This is common when the repo becomes a control plane.

---

# Best JSON-centric additions by role

## If you want **inspection/debugging**

Add:

* `jq`
* `gron`

## If you want **stronger validation**

Add:

* `Ajv`
* `Spectral`

## If you want **typed code from contracts**

Add:

* `json-schema-to-typescript`
* `datamodel-code-generator`
* `quicktype`

## If you want **operational deltas**

Add:

* JSON Patch / Merge Patch
* structural diff tooling

## If you want **governed decisions**

Add:

* OPA/Rego or CEL

## If you want **better authoring**

Add:

* VS Code JSON Schema support
* prettier
* schema bundling tools

---

# Recommended expanded stack for a kernel repo

A very practical expanded stack is:

## Core

* **JSON Schema**
* **CUE**
* **Jsonnet**

## Common adjacent tools

* **jq** for inspection
* **Ajv** for schema validation
* **JSON Patch** for explicit deltas
* **Spectral** for linting conventions
* **golden tests** for rendered outputs
* **VS Code schema integration** for editing
* **JSONL** for evidence/events
* **optional OPA/Rego** for allow/deny gates

That is a very normal, high-leverage expansion.

---

# Good rule for expansion

Add tools by missing responsibility, not by novelty.

Use this test:

* **Need to validate shape?** → JSON Schema/Ajv
* **Need to validate composition?** → CUE
* **Need to render artifacts?** → Jsonnet
* **Need to inspect/query JSON?** → jq
* **Need explicit deltas?** → JSON Patch
* **Need style/lint rules?** → Spectral
* **Need typed code?** → code generators
* **Need operational policy decisions?** → OPA/CEL
* **Need audit/event streams?** → JSONL

## Concise answer

The most commonly associated JSON-centric tools around this stack are:

* **jq**
* **Ajv**
* **yq**
* **Spectral**
* **JSON Patch / Merge Patch**
* **typed model generators**
* **schema bundlers/dereferencers**
* **JSONL/NDJSON**
* **golden diff/testing tools**
* **optional OPA/CEL for decision gates**

They extend the triple stack into a full authoring, validation, policy, rendering, and operational toolchain.

I can turn this into a **stack map** next: a layered table showing `tool -> role -> kernel repo fit -> when to add`.
