# GitOps Execution Cell
## Structured Proposal Draft

**Status:** Draft v0.1  
**Purpose:** Reconcile the uploaded notes into a single proposal for a declarative, review-gated local execution architecture.

---

## 1. Proposal Summary

This proposal defines a **GitOps Execution Cell**: a Git-declared, iteration-scoped execution unit that reconciles a **desired artifact transition**, not just a desired runtime state.

The cell is intended for local or repo-scoped agentic workflows where each iteration is derived from versioned authority, executed within bounded constraints, observed for evidence, and concluded with an explicit **promotion or denial decision**.

The core idea is:

> Treat each workflow iteration as a transactional reconciliation loop over a constrained state space, where the target is a valid artifact/evidence end state.

This combines four concerns into a single coherent model:

1. **Authority and desired state** from Git-tracked specifications.
2. **Constraint and admissibility** over workflows, inputs, tools, and outputs.
3. **Execution and reconciliation** within a bounded local runtime.
4. **Evidence and promotion gating** before any promoted outcome.

---

## 2. Problem Statement

The current stack already implies a structured control system, but its pieces are distributed across different concepts:

- `kernel-spec` as authority
- `pipeline-spec` as workflow/control shape
- `codex` and local tools as executors
- `rag-registry` as mounted knowledge/config surface
- MCP as a read-oriented lookup plane

The missing piece is a single proposal that explains how these fit together as one local control architecture.

The problem is not simply “how to run tools locally.” It is:

- how to declare a workflow iteration as a bounded object,
- how to derive it from authority deterministically,
- how to admit or deny it before execution,
- how to reconcile runtime toward that declared iteration,
- how to verify outputs and evidence,
- and how to decide whether the result is promotable.

---

## 3. Core Thesis

The system should be modeled as a **GitOps execution cell**.

This is stronger than a simple “local pod” metaphor.

A pod metaphor is still useful for co-located context and tools, but the real unit here is:

- **pod-shaped** in local composition,
- **job-like** in bounded execution,
- **controller-like** in reconciliation,
- and **review-gated** in promotion.

The cell does not reconcile toward “containers are running.”
It reconciles toward:

- a rendered desired iteration,
- a bounded execution envelope,
- schema-conformant outputs,
- a complete review bundle,
- and a final promotion decision.

In that sense, the cell reconciles **desired artifact state** and **transition validity**.

---

## 4. Design Principles

### 4.1 Git remains mutation authority
All authoritative state is versioned and declared through Git-tracked specifications.

### 4.2 Runtime is not authority
The local runtime materializes declared state but does not become the source of truth.

### 4.3 MCP is a lookup and bounded interface plane
MCP may expose resources, prompts, and carefully bounded tools, but should not bypass Git authority.

### 4.4 Desired state is transactional
The target is a valid iteration end state, not merely steady-state runtime convergence.

### 4.5 Illegal states are denied early
Constraints should narrow the admissible state space before execution begins.

### 4.6 Promotion is explicit
Promotion must follow verification and evidence collection, never implicit runtime success.

---

## 5. Conceptual Model

The cleanest model is:

- **Constraint-defined state space**
- **Local reconcile loop**
- **Evidence-conformant promotion gate**

This can be decomposed into two major layers.

### 5.1 State-space and admissibility layer
This layer defines:

- what a valid workflow object is,
- what bindings and capabilities are allowed,
- what inputs are admissible,
- what outputs are required,
- and what policy combinations are legal.

This is the most CUE-like part of the architecture.

### 5.2 Control and action layer
This layer:

- observes current local state,
- compares observed vs desired state,
- creates or updates the execution envelope,
- runs the workflow,
- verifies results,
- and emits a decision.

This is the reconciler/controller part of the architecture.

---

## 6. Proposed Object Model

The proposal is best expressed through three primary runtime objects and a small set of supporting decision objects.

### 6.1 Primary objects

#### `ExecutionCellTemplate`
A stable, Git-tracked definition of a cell.

Contains:

- references to `kernel-spec`
- references to `pipeline-spec`
- registry lookup policy
- allowed tools and actions
- schema definitions
- review bundle requirements
- promotion policy

#### `ExecutionCellRun`
One concrete iteration of the cell.

Derived from:

- template
- current base ref
- selected inputs
- registry resolutions
- runtime parameters

Produces:

- rendered manifest
- realized schema bindings
- outputs
- evidence
- review bundle
- decision

#### `PromotionRecord`
The final record of the iteration’s result.

Contains:

- `ALLOW`, `DENY`, `NEEDS_REPAIR`, or `REQUEUE`
- reasons
- promoted artifacts or denied artifacts
- base/head refs
- review bundle hash
- output hash
- next action

### 6.2 Workflow-facing model
For workflow portability and API shape, a second, more general object family is useful.

#### `WorkflowClass`
A canonical workflow type.

Examples:

- `registry_lookup`
- `context_assembly`
- `plan_generate`
- `review_request`
- `packet_execute`
- `reconcile_runtime`
- `publish_projection`

#### `WorkflowInstance`
A declared or concrete workflow run.

Suggested fields:

- `workflowClassRef`
- `inputs`
- `capabilityBindings`
- `resourceRefs`
- `policies`
- `outputs`
- `status`

#### `CapabilityBinding`
A declared permission or surface binding.

Examples:

- `read.registry`
- `read.git`
- `read.runtime_state`
- `write.generated_artifacts`
- `run.codex`
- `emit.evidence`
- `open.pr`

#### `Projection`
A derived rendering from admitted state.

Examples:

- pipeline manifest
- review bundle
- status snapshot
- MCP resource index
- generated config surface
- catalog/index view

---

## 7. Desired State and Reconciliation Model

The most important design decision is to treat each iteration as reconciliation of **desired artifact state**.

That means the controller is responsible for asking:

- Was the manifest correctly derived from authority?
- Were all inputs admissible and frozen?
- Did execution remain within declared bounds?
- Do outputs conform to schema?
- Is the review bundle complete?
- Does policy allow promotion?

### Minimal reconcile loop

#### 1. Resolve
Load and derive:

- `kernel-spec`
- `pipeline-spec`
- registry references
- concrete manifest
- concrete schemas
- frozen input set

#### 2. Validate
Before execution, assert:

- authority exists
- derivation is deterministic
- lookups are bounded
- inputs are admissible
- actions are allowed
- output contract is known

#### 3. Materialize
Create the execution envelope:

- workspace
- runtime directories
- manifests
- bound inputs
- tool availability map

#### 4. Execute
Run the agentic pipeline within declared limits.

#### 5. Observe
Collect:

- outputs
- logs
- traces
- diffs
- touched files
- decision trace

#### 6. Verify
Check:

- output schema validity
- manifest/output consistency
- required artifacts present
- review bundle completeness
- promotion preconditions

#### 7. Decide
Emit one of:

- `ALLOW`
- `DENY`
- `NEEDS_REPAIR`
- `REQUEUE`

---

## 8. Relationship to Existing Stack

The proposal maps cleanly to the current components.

| Current term | Proposed role |
|---|---|
| `kernel-spec` | authority and derivation source |
| `pipeline-spec` | workflow/control definition |
| manifest | rendered desired iteration |
| schema | admissibility and output contract |
| inputs | bounded reconcile input set |
| outputs | desired produced state |
| review bundle | evidence package for gate |
| iteration | one reconcile transaction |
| `codex` | executor inside the cell |
| `rag-registry` | read-only knowledge/config surface |
| MCP | lookup and bounded interaction plane |
| promotion | explicit post-verification decision |

---

## 9. MCP Reach Model

MCP should be treated as a **layered interface over the cell**, not as the mutation authority.

### Level 0 — Read-only resources
Safe default surface:

- registry entries
- manifest indexes
- generated docs
- evidence artifacts
- status snapshots
- topology or graph views

### Level 1 — Prompt surfaces
User- or workflow-driven prompting surfaces:

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

### MCP boundary rule
Use this boundary consistently:

- **Git remains mutation authority**
- **MCP remains discovery and bounded execution interface**
- **controller remains reconciler**
- **runtime never becomes authority**

---

## 10. Internal Plane Decomposition

A clean implementation should separate the cell into explicit planes.

### 10.1 Structure plane
Canonical workflow object model.

### 10.2 Contract plane
Exported schemas for:

- workflow validity
- capability bindings
- projection contracts
- evidence objects
- status objects

### 10.3 Policy plane
Admission and legality checks:

- allowed workflow classes
- required bindings
- forbidden combinations
- profile legality
- promotion policy

### 10.4 Artifact plane
Derived projections such as:

- generated configs
- manifests
- runtime scaffolds
- review bundles
- catalog/index surfaces

### 10.5 Runtime plane
Controllers and executors that:

- observe state
- reconcile local runtime
- execute workflows
- collect evidence
- emit decisions

---

## 11. Suggested Intermediate Representation

A minimal intermediate representation can anchor validation, admission, rendering, execution, and reconciliation.

```yaml
kind: WorkflowInstance
spec:
  class: review_request
  inputs:
    source_ref: git://repo/path
    registry_query: "kernel spec admission"
  bindings:
    - read.registry
    - read.git
    - run.codex
    - write.generated_artifacts
  policies:
    profile: readonly-review
    admission_bundle: review-default
  outputs:
    - review.bundle.json
    - evidence.json
```

This gives one declared object that can be:

- validated,
- admitted,
- rendered,
- executed,
- observed,
- and decided.

---

## 12. Non-Goals

This proposal does **not** treat the following as primary goals:

- direct runtime mutation outside the reconcile path,
- ad hoc tool invocation without declared bindings,
- implicit promotion based only on execution success,
- or replacing Git authority with runtime state or transient tool state.

---

## 13. Proposed Implementation Sequence

### Phase 1 — Formalize the cell API
Define the minimum canonical objects:

- `CellSpec`
- `WorkflowClass`
- `WorkflowInstance`

### Phase 2 — Formalize status and evidence
Define:

- `AdmissionDecision`
- `ReconcilePlan`
- `RuntimeStatus`
- `EvidenceCapsule`
- `PromotionRecord`

### Phase 3 — Formalize admission and rendering
Implement:

- deterministic manifest derivation
- legality checks
- output contract checks
- projection generation

### Phase 4 — Expose MCP resources
Expose:

- registry URIs
- workflow definitions
- admitted-state snapshots
- status/evidence URIs
- graph/index views

### Phase 5 — Add bounded MCP tools
Limit the default operational surface to:

- validate
- render
- diff
- reconcile-plan
- emit-evidence
- open-pr

### Phase 6 — Optional external projections
Generate derived views for adjacent ecosystems if useful, but keep them non-authoritative.

Examples:

- catalog views
- workflow views
- config bundle projections
- status dashboards

---

## 14. Key Architectural Decisions

### Decision 1
The primary abstraction is **GitOps execution cell**, not just local pod.

### Decision 2
Each iteration is a **reconcile transaction** over desired artifact state.

### Decision 3
The system separates **state-space/admissibility** from **control/action**.

### Decision 4
Git is the only mutation authority for canonical state.

### Decision 5
MCP is layered and bounded; it does not replace the controller or authority path.

### Decision 6
Evidence and promotion are first-class outputs of every run.

---

## 15. Open Questions

The following questions remain for the next draft:

1. What is the exact canonical schema set for `CellSpec`, `WorkflowClass`, and `WorkflowInstance`?
2. How should bindings be modeled: capability strings, typed objects, or both?
3. What is the minimal required evidence bundle for every run?
4. Should `NEEDS_REPAIR` be a decision or a subordinate transition plan?
5. Which parts of the runtime plane are purely local, and which should remain projection-only?
6. How much of the admission/policy plane should be static vs runtime-evaluated?

---

## 16. Draft Conclusion

The reconciled proposal is:

A **GitOps Execution Cell** is a declarative, iteration-scoped local control unit that derives desired workflow state from versioned authority, admits only legal configurations, materializes a bounded execution envelope, reconciles execution toward a schema-conformant artifact/evidence end state, and concludes with an explicit promotion decision.

This framing unifies the uploaded documents into one architecture:

- **desired state is real and explicit,**
- **constraint narrowing happens before execution,**
- **controller logic owns observation, action, and verification,**
- **MCP provides layered access without becoming authority,**
- and **promotion is gated by evidence rather than assumed from runtime success.**

That gives a clean base for the next step: turning this draft into a canonical schema set and a minimal controller contract.
