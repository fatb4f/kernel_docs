# GitOps Execution Cell
## Normative Proposal Draft

**Status:** Draft v0.2  
**Purpose:** Define a declarative, review-gated local execution architecture centered on admitted desired artifact state rather than runtime state alone.

---

## 1. Executive Summary

This proposal defines a **GitOps Execution Cell**: a Git-declared, iteration-scoped reconcile unit that targets a **valid artifact/evidence end state**.

The cell is not primarily a local pod, task runner, or tool wrapper. It is a **transactional reconcile unit over desired artifact state**.

Each iteration:

1. derives desired state from Git-tracked authority,
2. admits or denies that state before execution,
3. materializes a bounded execution envelope,
4. executes within declared capabilities and policies,
5. emits evidence and status,
6. ends in an explicit promotion decision.

The proposal asks the reader to accept these core decisions:

- **Git is mutation authority** for canonical state.
- **Runtime is not authority**.
- **MCP is a bounded interface plane**, not an authority path.
- **Nothing renders, executes, or emits evidence unless first admitted as normalized IR**.
- **Every run ends in evidence-backed ALLOW, DENY, or REQUEUE**.

The system therefore centers on four first-class properties:

- **authority**
- **admission**
- **actuation**
- **evidence**

---

## 2. Scope and Non-Goals

### 2.1 In scope

This proposal covers:

- Git-tracked authority objects for local or repo-scoped workflows,
- admission and legality checks before execution,
- deterministic derivation into normalized IR,
- bounded runtime materialization and execution,
- evidence emission and review bundle completion,
- explicit promotion or denial decisions.

### 2.2 Non-goals

This proposal does **not** treat the following as primary goals:

- direct runtime mutation outside the reconcile path,
- ad hoc tool invocation without declared bindings,
- implicit promotion based only on execution success,
- replacing Git authority with runtime state or transient tool state,
- making external ecosystem projections authoritative.

---

## 3. System Model and Invariants

The GitOps Execution Cell is a local control architecture organized around four invariants.

### 3.1 Core invariants

| Invariant | Meaning |
|---|---|
| **Authority** | Canonical desired state is declared in Git-tracked objects. |
| **Admission** | Only legal, schema-valid, policy-admissible state may proceed to render or execution. |
| **Actuation** | Runtime materializes and executes admitted state within bounded capabilities. |
| **Evidence** | Every run emits durable evidence sufficient for verification and decision. |

### 3.2 Hard rules

1. **Git remains mutation authority.**
2. **Runtime never becomes source of truth.**
3. **MCP may not bypass admission or promotion.**
4. **All renderable artifacts derive only from admitted normalized IR.**
5. **Every run must emit evidence, including failure paths.**
6. **Promotion requires contract-valid outputs and complete review evidence.**
7. **Promoted artifacts must link to refs and evidence hashes.**

### 3.3 Formal boundary rule

> Nothing renders, executes, or emits evidence without first existing as admitted normalized IR.

This is the central contract boundary of the architecture.

---

## 4. Canonical Object Model

This proposal normalizes the model into **authority**, **run**, and **decision/evidence** families.

### 4.1 Authority objects

#### `CellSpec`
Stable, Git-tracked definition of an execution cell.

Contains:

- references to `kernel-spec`
- references to `pipeline-spec`
- workflow class allow-list
- registry lookup policy
- capability policy
- contract and schema references
- evidence and promotion requirements
- MCP exposure policy

#### `WorkflowClass`
Canonical description of an allowed workflow type.

Examples:

- `registry_lookup`
- `context_assembly`
- `plan_generate`
- `review_request`
- `packet_execute`
- `reconcile_runtime`
- `publish_projection`

Defines:

- required inputs
- allowed capability IDs
- output contracts
- status model
- legality constraints

### 4.2 Run object

#### `WorkflowInstance`
Concrete admitted iteration of a workflow.

Represents one reconcile transaction.

Contains:

- `cellSpecRef`
- `workflowClassRef`
- frozen inputs
- normalized bindings
- policy selections
- output contract refs
- base/head refs
- admitted normalized IR digest
- runtime envelope metadata
- current status ref

### 4.3 Decision and evidence objects

#### `AdmissionDecision`
Result of legality and admissibility checks before runtime execution.

Terminals:

- `ALLOW`
- `DENY`
- `REQUEUE`

May include:

- reasons
- violated constraints
- required repairs or missing prerequisites
- admitted IR digest when allowed

#### `RuntimeStatus`
Observed execution status for a `WorkflowInstance`.

Contains:

- phase
- timestamps
- touched paths
- logs/traces pointers
- decision trace
- runtime envelope state
- verification checkpoint status

#### `EvidenceCapsule`
Durable record of what happened during the run.

Contains:

- manifest hash
- admitted IR hash
- output hashes
- review bundle hash
- diffstat
- touched files
- command/action trace
- validation results
- policy results
- provenance links

#### `PromotionRecord`
Final decision record for the run.

Terminals:

- `ALLOW`
- `DENY`
- `REQUEUE`

Contains:

- final decision
- reasons
- refs
- promoted or denied artifact hashes
- linked evidence capsule
- optional next action
- optional repair plan reference

#### `RepairPlan`
Structured follow-on plan attached to a `DENY` result when the next legal action is known.

This is **not** a terminal decision.

---

## 5. Admitted IR Boundary

The architecture requires a mandatory normalized intermediate representation.

### 5.1 Purpose

The normalized IR is the admitted boundary between:

- authority and admission,
- render and projection,
- runtime and reconciliation,
- evidence and decision.

### 5.2 Rule

Every `WorkflowInstance` must be backed by an admitted normalized IR object before any runtime materialization occurs.

### 5.3 Compile chain

The supporting chain is:

```text
Git authority
  -> JSON Schema validate
  -> CUE legalize
  -> normalized IR
  -> Jsonnet render
  -> execution manifest / schemas / bindings
  -> runtime execution
  -> evidence emission
  -> verify
  -> promote or deny
```

The runtime does not author manifests directly; it consumes artifacts rendered from admitted state.

---

## 6. Reconcile Lifecycle

Each iteration is a reconcile transaction over desired artifact state.

### 6.1 Canonical lifecycle

1. **Resolve**  
   Load `CellSpec`, `WorkflowClass`, authority refs, registry refs, and frozen inputs.

2. **Validate**  
   Check schema validity, legality, capability policy, required contracts, and deterministic derivation preconditions.

3. **Admit**  
   Produce `AdmissionDecision`. If allowed, seal admitted normalized IR.

4. **Materialize**  
   Create the bounded execution envelope from admitted IR only.

5. **Execute**  
   Run the workflow within declared capabilities, policy bounds, and runtime budget.

6. **Observe**  
   Collect outputs, logs, traces, touched paths, diffs, and decision trace.

7. **Verify**  
   Validate output contracts, manifest/output consistency, required artifacts, and review bundle completeness.

8. **Decide**  
   Emit `PromotionRecord` with `ALLOW`, `DENY`, or `REQUEUE`.

### 6.2 Decision lattice

```text
AdmissionDecision / PromotionRecord terminals:
  ALLOW | DENY | REQUEUE

RepairPlan:
  optional structured follow-on attached to DENY
```

This keeps controller logic minimal while preserving actionable denial output.

---

## 7. Capability Binding Model

Capability bindings must support both policy matching and runtime precision.

### 7.1 Required shape

Each binding has:

- a stable **string ID** for policy, audit, and evidence,
- a typed **params payload** for execution behavior.

### 7.2 Example

```yaml
bindings:
  - id: read.registry
    params:
      uri_patterns:
        - "registry://skills/*"
  - id: read.git
    params:
      allow_paths:
        - "control/**"
        - "spec/**"
  - id: run.codex
    params:
      profile: readonly-review
      budget:
        max_steps: 8
  - id: write.generated_artifacts
    params:
      allow_paths:
        - "out/**"
```

This preserves easy policy evaluation while supporting replay, evidence precision, and bounded runtime behavior.

---

## 8. Projection vs Runtime Materialization

The model must distinguish derived views from runtime-authoritative artifacts.

| Category | Meaning | Examples |
|---|---|---|
| **Projection-only** | Derived, non-authoritative views over admitted or observed state | graph/index views, dashboards, catalog views, external ecosystem views |
| **Runtime-materialized** | Artifacts created for execution, verification, and decision | manifests, bindings, workspace state, evidence stream, review bundle, status snapshot |

Projection-only outputs may aid discovery or interoperability, but they do not change authority and are never used to bypass admission.

---

## 9. Interface Planes

A clean implementation separates the cell into explicit planes.

### 9.1 Structure plane

Canonical object model:

- `CellSpec`
- `WorkflowClass`
- `WorkflowInstance`

### 9.2 Contract plane

Exported schemas for:

- workflow validity
- capability bindings
- admission decisions
- evidence objects
- status objects
- promotion records

### 9.3 Policy plane

Admission and legality checks:

- allowed workflow classes
- required bindings
- forbidden combinations
- profile legality
- promotion policy

### 9.4 Artifact plane

Derived runtime artifacts:

- rendered manifests
- runtime scaffolds
- review bundles
- evidence capsules
- status resources

### 9.5 Runtime plane

Controllers and executors that:

- observe state,
- reconcile local runtime,
- execute workflows,
- collect evidence,
- emit decisions.

---

## 10. MCP Reach Model

MCP is a layered interface over the cell, not the mutation authority.

### Level 0 — Read-only resources

Safe default surface:

- registry entries
- workflow definitions
- admitted-state snapshots
- status/evidence resources
- graph/index views

### Level 1 — Prompt surfaces

Examples:

- review this packet
- assemble workflow context
- summarize registry results
- explain admission denial

### Level 2 — Bounded operational tools

Allowed only when they do not bypass Git authority:

- validate object
- render projection
- diff desired vs observed
- compute reconcile plan
- emit evidence capsule
- open draft PR

### Level 3 — Direct mutation

Not recommended as a default:

- editing canonical authority through MCP
- mutating runtime outside reconcile flow
- bypassing admission or promotion

### Boundary rule

- **Git remains mutation authority**
- **MCP remains discovery and bounded execution interface**
- **controller remains reconciler**
- **runtime never becomes authority**

---

## 11. Relationship to Existing Stack

The proposal maps cleanly to current terms.

| Current term | Proposed role |
|---|---|
| `kernel-spec` | authority and derivation source |
| `pipeline-spec` | workflow/control definition |
| manifest | rendered runtime materialization from admitted IR |
| schema | validity and contract surface |
| CUE legality | admission plane |
| registry | read-only knowledge/config surface |
| `codex` | executor inside runtime plane |
| review bundle | evidence package for gate |
| iteration | one reconcile transaction |
| MCP | lookup and bounded interaction plane |
| promotion | explicit post-verification decision |

---

## 12. Minimal Canonical IR Example

This example is illustrative of the required shape.

```yaml
kind: WorkflowInstance
spec:
  cellSpecRef: git://repo/control/cells/review-default
  workflowClassRef: review_request
  inputs:
    source_ref: git://repo/path
    registry_query: "kernel spec admission"
  bindings:
    - id: read.registry
      params:
        uri_patterns:
          - "registry://skills/*"
    - id: read.git
      params:
        allow_paths:
          - "control/**"
    - id: run.codex
      params:
        profile: readonly-review
        budget:
          max_steps: 8
    - id: write.generated_artifacts
      params:
        allow_paths:
          - "out/**"
  policies:
    profile: readonly-review
    admission_bundle: review-default
  outputs:
    contracts:
      - review.bundle.json
      - evidence.json
```

This object is useful because one admitted shape can be:

- validated,
- legalized,
- rendered,
- executed,
- observed,
- verified,
- and decided.

---

## 13. Implementation Sequence

### Phase 1 — Canonical schema pack

Define:

- `CellSpec`
- `WorkflowClass`
- `WorkflowInstance`
- `AdmissionDecision`
- `RuntimeStatus`
- `EvidenceCapsule`
- `PromotionRecord`

### Phase 2 — Admission and IR sealing

Implement:

- deterministic derivation
- legality checks
- normalized IR sealing
- admitted IR digests

### Phase 3 — Render and runtime materialization

Implement:

- manifest derivation from admitted IR
- runtime scaffolds
- bounded execution envelope
- output contract loading

### Phase 4 — Evidence and decision

Implement:

- streamed evidence capture
- verification checkpoints
- promotion record emission
- repair plan attachment on denial

### Phase 5 — MCP resources and bounded tools

Expose:

- admitted-state resources
- workflow and status resources
- evidence URIs
- validate/render/diff/reconcile-plan/emit-evidence/open-pr tools

### Phase 6 — Optional projection surfaces

Generate non-authoritative derived views such as:

- dashboards
- catalog/index surfaces
- ecosystem projections

---

## 14. Open Design Decisions

The following items still require explicit choices:

1. Exact field set and versioning strategy for the schema pack.
2. Minimum required evidence bundle for every run.
3. Static vs runtime policy evaluation split.
4. Extent of local-only runtime plane vs exported status resources.
5. Whether additional workflow graph semantics are needed beyond `WorkflowClass` + `WorkflowInstance`.

---

## 15. Conclusion

A **GitOps Execution Cell** is a declarative, iteration-scoped local control unit that derives desired workflow state from Git-tracked authority, admits only legal configurations, materializes a bounded execution envelope from admitted normalized IR, reconciles execution toward a contract-valid artifact/evidence end state, and concludes with an explicit promotion decision.

This framing unifies the stack cleanly:

- desired state is explicit and versioned,
- illegality is denied before runtime actuation,
- runtime is bounded and non-authoritative,
- evidence is first-class,
- promotion is explicit and review-gated.

The cell is therefore best understood as a **transactional reconcile unit over desired artifact state**.

