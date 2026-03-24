## Scoped model: capabilities and control requirements

This is the right reduced surface.

It stays aligned with the current kernel baseline:

* canonical structural model is authority
* JSON Structure is authoring syntax
* JSON Schema is the exported boundary contract
* CUE admits normalized state
* Jsonnet renders from admitted state only
* Structurize/Avrotize stay derived interop/codegen bridges 

It also matches the execution draft’s current split:

* `codex` is the executor
* `rag-registry` is a read-only knowledge/config surface
* MCP is a lookup and bounded interaction plane
* JSON-RPC / NDJSON / JSON:API / optional JSON-LD are the operational surfaces around execution and evidence

---

## 1. Scope

### In scope

* **Codex** as primary executor
* **MCP** as bounded lookup / tool surface
* **App-Server** as sessioned control surface
* **RAG registry** as read-only knowledge/config surface
* **JSON-family extensions actually required by those surfaces**
* **lineage, admissibility, and evidence requirements** needed to operate them safely

### Out of scope

* generalized `CellStateSpec`
* generalized `CellExecSpec`
* broader execution-cell schema families beyond what Codex/MCP/App-Server/RAG force right now

---

## 2. Primary surfaces and their role

### A. RAG registry

**Role:** read-only authority-adjacent resource surface.

Use it for:

* registry lookup
* context assembly
* config discovery
* versioned knowledge selection

The draft already treats `rag-registry` as a mounted read-only knowledge/config surface and a lookup input to the reconcile loop.

### B. MCP

**Role:** bounded interaction plane over admitted and derived state.

Use it for:

* read-only resources
* prompt surfaces
* bounded operational tools like validate/render/diff/emit-evidence/open-draft-pr

The draft already defines MCP as layered and bounded, not mutation authority.

### C. App-Server

**Role:** sessioned control plane for Codex-facing execution.

Use it for:

* session/bootstrap/init
* capability negotiation
* turn lifecycle
* event streaming
* controlled execution orchestration

This maps directly onto the proposal’s operational control lane and the JSON-RPC control-plane role. 

### D. Codex

**Role:** executor inside the bounded control system.

Use it for:

* run declared workflows
* consume admitted inputs
* produce outputs and evidence
* stay inside capability/profile limits

The draft already makes `codex` the executor inside the cell.

---

## 3. Capability model

Model capabilities as **stable IDs plus typed params**.

That keeps audit/policy simple while preserving runtime precision, which is consistent with the earlier review direction on capability bindings and with the draft’s `CapabilityBinding` model. 

### Capability families

#### Registry capabilities

* `read.registry`
* `search.registry`
* `resolve.registry_ref`

#### Authority / source capabilities

* `read.git`
* `read.kernel_contract`
* `read.admitted_state`

#### Projection / contract capabilities

* `validate.object`
* `render.projection`
* `diff.desired_vs_observed`
* `resolve.schema_contract`

#### Execution capabilities

* `run.codex`
* `start.turn`
* `resume.turn`
* `interrupt.turn`

#### Evidence capabilities

* `emit.evidence`
* `stream.events`
* `write.review_view`

#### Promotion-adjacent bounded capabilities

* `open.draft_pr`
* `publish.resource_view`

### Recommended representation

```yaml
capabilityBindings:
  - id: read.registry
    params:
      uri_patterns:
        - "registry://skills/*"
        - "registry://schemas/*"

  - id: run.codex
    params:
      profile: readonly-review
      budget:
        max_steps: 8
        max_tool_calls: 12

  - id: emit.evidence
    params:
      sinks:
        - "generated/state/evidence"
        - "generated/state/events"
```

---

## 4. Surface-specific control requirements

## A. RAG registry controls

### Required capabilities

* read/search/resolve only

### Required controls

* **read-only**
* all resolved entries must be version-pinned or digest-pinned
* registry resolution must emit provenance
* registry content may inform execution but may not become mutation authority
* registry queries must be bounded and recorded

### Required evidence

* `registry-resolution.json`
* `registry-input-manifest.json`
* digest list for selected entries

### Deny conditions

* unresolved or ambiguous registry refs
* nondeterministic resolution
* unpinned registry artifacts in production mode

---

## B. MCP controls

### Allowed capability bands

From the draft’s layered model:

* **Level 0:** read-only resources
* **Level 1:** prompt surfaces
* **Level 2:** bounded tools
* **Level 3:** direct mutation is excluded by default

### Required controls

* MCP tools must be declared by capability ID
* MCP may not mutate canonical authority
* MCP may not bypass admission
* MCP outputs must be typed and attributable
* MCP resources must distinguish:

  * authority-derived
  * admitted-state-derived
  * runtime-derived
  * evidence-derived

### Required evidence

* `mcp-invocation-log.ndjson`
* `mcp-resource-resolution.json`
* `mcp-tool-result.json`

### Deny conditions

* undeclared tool use
* tool result without typed contract
* any MCP operation that mutates authority directly
* any MCP operation reading non-admitted inputs for render/derive paths

---

## C. App-Server controls

### Required capabilities

* initialize / negotiate
* start/resume/interrupt turn
* stream events
* invoke bounded execution actions

### Required controls

* session contract must be explicit
* capability negotiation must be logged
* every turn must declare:

  * admitted inputs
  * allowed capabilities
  * output contract
  * evidence sinks
* event stream must be append-only
* backpressure / resume semantics must be deterministic enough to replay or explain

### Required evidence

* `session-contract.json`
* `session-events.ndjson`
* `turn-manifest.json`
* `turn-result.json`
* `decision-trace.ndjson`

### Deny conditions

* turn starts without admitted inputs
* session capability exceeds admitted profile
* event stream missing for executed turn
* outputs lack declared contract or evidence link

---

## D. Codex controls

### Required controls

* Codex runs only inside an admitted profile
* tool use must be capability-gated
* writable paths must be bounded
* produced artifacts must map to declared output classes
* execution success is insufficient without evidence and verification

That matches the kernel’s existing render/admit/integrity discipline and the proposal’s “authority / admission / actuation / evidence” model.

### Required evidence

* `execution-report.json`
* `output-manifest.json`
* `touched-files.json`
* `review-bundle.json`
* `promotion-input.json`

---

## 5. JSON-family extensions required now

This is the minimum practical set.

### Core

* **JSON Structure** — source authoring syntax
* **JSON Schema** — exported contract/boundary validation
* **CUE** — admissibility over normalized state
* **Jsonnet** — render/projection from admitted state only 

### Required operational extensions

* **JSON-RPC** — App-Server control envelope
* **NDJSON** — event/evidence streaming
* **JSON:API** — durable run/result/review resources
* **JSON-LD** — optional for semantic/provenance projection, not baseline-required on day one 

### Required derived bridges

* **Structurize / Avrotize** — recognized as derived codegen/interop bridges, but only exercised where consumer contracts require them 

---

## 6. Lineage requirements

This is the main kernel amendment payload.

## A. JSON Schema lineage

All exported schemas need reconstructable ancestry.

### Required fields

* `schema_id`
* `schema_version`
* `source_family`
* `source_refs`
* `export_mode`
* `ref_graph_digest`
* `generated_by`

### Required export artifacts

* `export-report.json`
* `lineage.index.json`
* `ref-graph.json`
* `bundle-report.json`
* `dereference-report.json`

---

## B. Registry lineage

Every resolved registry input needs:

* source URI
* version/digest
* resolution timestamp
* selected variant/profile
* consumer workflow/run id

---

## C. Runtime lineage

Every run result needs linkage to:

* admitted-state digest
* schema contract ids
* registry resolutions
* session id / turn id
* produced artifact digests

---

## 7. Admissibility requirements

CUE remains the legality layer over normalized state, which is already fixed by the kernel. 

For this narrower scope, admission must answer:

### Registry admissibility

* are requested registry refs allowed?
* are selected variants/profiles legal?
* is the resolution deterministic enough?

### MCP admissibility

* are requested MCP tools/resources allowed by profile?
* are any forbidden capability combinations present?
* does the tool set remain non-authoritative?

### App-Server admissibility

* is the requested session/turn shape legal?
* are event sinks declared?
* are allowed output contracts present?

### Codex admissibility

* is `run.codex` permitted here?
* are budgets and writable surfaces declared?
* are outputs and evidence obligations declared?

### Admission evidence

Keep the current kernel artifacts, then extend:

* `decision.json`
* `violations.json`
* `admitted-state.json`
* `admission-input-manifest.json`
* `policy-bundle-lock.json`
* `surface-admissibility-report.json` 

---

## 8. Evidence model

The current kernel already requires explicit gate evidence and explicit admission artifacts. 

For this scoped surface set, make evidence mandatory in four buckets:

### A. Contract/export evidence

* schema export reports
* lineage reports
* compatibility export reports

### B. Admission evidence

* allow/deny decision
* violations
* admitted-state
* policy/input locks

### C. Runtime evidence

* App-Server event stream
* MCP invocation stream
* Codex execution report
* touched files / outputs / diffs

### D. Review/promotion evidence

* review bundle
* promotion decision input
* result resource
* final decision trace

### Preferred stream format

Use **NDJSON** for append-only operational evidence. That fits the proposal and the stack expansion notes directly.

---

## 9. Kernel amendment deltas implied by this scope

This is the minimum amendment set.

### Add artifact classes

* `event-stream`
* `evidence-stream`
* `resource-surface`
* `rpc-envelope`
* `lineage-report`
* `compatibility-export`

### Extend gate evidence

#### G2 Contract export

Add:

* `lineage.index.json`
* `ref-graph.json`
* `bundle-report.json`

#### G4 Admission

Add:

* `admission-input-manifest.json`
* `policy-bundle-lock.json`
* `surface-admissibility-report.json`

#### G5/G6 Runtime + integrity

Add:

* `session-events.ndjson`
* `mcp-invocation-log.ndjson`
* `execution-report.json`
* `review-bundle.json`

### Add extension registry entries

Under `structures/extensions/`, register:

* `json_rpc`
* `ndjson`
* `json_api`
* `json_ld` with status `optional`
* `structurize`
* `avrotize`

---

## 10. Minimal control object set for this phase

Do not add full cell specs yet.

Use a smaller control set:

### `WorkflowClass`

Canonical workflow type.

### `WorkflowInstance`

Declared run object with:

* `class`
* `inputs`
* `capabilityBindings`
* `resourceRefs`
* `policies`
* `outputs`
* `status`

### `AdmissionDecision`

Allow/deny plus reasons.

### `RuntimeStatus`

Session/turn/result state.

### `EvidenceCapsule`

Bounded run evidence.

### `PromotionRecord`

Terminal decision record.

This stays close to the current proposal without prematurely hardening `CellStateSpec` / `CellExecSpec`.

---

## 11. Decision rule

Use this as the narrow operating rule:

> A production Codex workflow is legal only if its registry inputs are lineage-resolved, its capability set is admitted, its control surface is bounded through MCP/App-Server contracts, and its execution emits complete gate evidence.

That is the correct reduced model.

If useful, I can turn this into a **kernel amendment outline** or a **capability matrix** next.
