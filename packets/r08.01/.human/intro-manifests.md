# Manifests: what they are in this kernel

A **manifest** here is a **typed control object** that declares a bundle, projection, or generator surface. It is not the policy itself, not the rendered output, and not the runtime implementation. It exists to make the kernel’s declared surfaces **machine-readable, indexable, and reviewable** before execution logic is introduced. In `pkt-r08-01`, the manifest contract is expressed through a discriminated family with `manifest_id`, `spec_version`, `manifest_class`, `status`, `owner_plane`, structured `inputs` and `outputs`, plus explicit `admission` and `regen` members.

That means manifests occupy a very specific role in the authority chain. The kernel’s three-plane model says **policy** owns contracts, legality, admissibility, defaults, overlays, and export readiness; **data shaping** owns deterministic projection, read-model shaping, codegen, scaffolding, and event/evidence shapes; **tool composition** owns bounded ingress, runtime adapters, protocol seams, and evidence emission hooks. Manifest control objects sit at the boundary between Policy and Data Shaping: they declare what those surfaces are allowed to be, without becoming the renderer, generator, or runtime themselves.

## The shortest accurate definition

A manifest is:

* a **declaration**
* with a **class** (`bundle`, `projection`, `generator`)
* that carries a stable **manifest_id**
* that declares a fixed **spec_version**
* that names **structured inputs**
* names **structured outputs**
* declares **admission** and **regen** policy
* and records the **owner plane**

That is the minimal member model currently proposed for `r08`.

---

# Why manifests exist at all

Without manifests, the repository may contain schemas, renderers, examples, or generators, but there is no formal answer to:

* which bundle is authoritative,
* which projection classes are declared,
* which generator surfaces are legitimate,
* what inputs they may consume,
* what outputs they are allowed to produce,
* and whether a downstream surface is part of authority or only derived.

The pkt-plan explicitly says the current baseline is “structurally complete but not yet enforceable,” and lists typed control objects for bundle/projection/generator execution surfaces as one of the missing pieces. Manifests are how that missing piece gets closed. 

---

# Relation to JSON Structure, JSON Schema, CUE, and Jsonnet

## 1. JSON Structure

In the amendment material, **JSON Structure** is treated as part of the **core authoring and control stack**. It is the **canonical authoring syntax for the structural model**. That means source-side structure lives here first. A manifest, as a JSON control object, is authored in that source/control universe. It is one of the things that expresses structure and intent at the source boundary. 

Educationally: JSON Structure answers **“what is being authored?”**
For manifests, that means:

* this is a bundle declaration,
* this is a projection declaration,
* this is a generator declaration,
* these are its declared edges and constraints.

JSON Structure is about **human-authored structural intent**.

## 2. JSON Schema

The same amendment says **JSON Schema** is the **derived exported contract layer**, and the minimal expansion rules say **JSON Schema = structural authority**. For manifests, that means the source-authored control object must have a schema-governed structural contract. The file `schemas/exported/manifest-control-family.schema.json` is that contract family. Its job is to distinguish bundle vs projection vs generator without relying on prose.

Educationally: JSON Schema answers **“what shape is valid?”**
For manifests, that means:

* required top-level members,
* allowed classes,
* required enums,
* valid object layout,
* example validation boundaries.

So the manifest instance is the **declaration**, while `manifest-control-family.schema.json` is the **shape authority** for that declaration.

## 3. CUE

The minimal expansion plan defines **CUE = admissibility/composition authority**. It is not the structural source of the manifest and not the renderer. CUE comes later, after manifests and normalization/admission contracts exist, to evaluate legality and admissibility over normalized input. In `pkt-r08-04`, the first runnable CUE bundle is supposed to check things like required kernel family presence, allowed manifest classes, generated/build boundary invariants, compatibility-family separation, and the rule that projection inputs must reference admitted state rather than raw sources.

Educationally: CUE answers **“is this allowed to participate?”**
For manifests, that means CUE can enforce rules such as:

* only declared manifest classes are allowed,
* no projection may consume raw source directly,
* no compatibility material gets promoted into kernel-core authority,
* no generated/build boundary is violated.

So manifests are **declared surfaces**, while CUE decides whether those surfaces are **admissible in a concrete run or bundle**. 

## 4. Jsonnet

The minimal expansion rules define **Jsonnet = deterministic projection authority**. Jsonnet belongs to Plane 2, Data Shaping. It is where declared projection classes become real projected/read-model outputs. The pkt-plan says `pkt-r09-01` activates minimal Jsonnet projections, and requires that every Jsonnet projection has a matching projection control object and that no projection reads raw `structures/**`; it must consume admitted-state inputs.

Educationally: Jsonnet answers **“how do I deterministically shape the admitted state into derived outputs?”**
For manifests, that means a projection manifest declares:

* the projection exists,
* what class it is,
* what it consumes,
* what output class it emits,
* and what renderer identity is associated with it.

But the manifest is **not** the Jsonnet. It is the control declaration that lets Jsonnet exist in a governed way. 

---

# Core idea: manifests are declarations, not implementations

This distinction is the most important educational point.

A manifest is **not**:

* the runtime adapter,
* the Python CLI,
* the Jsonnet program,
* the CUE bundle,
* the generated binding,
* the emitted evidence stream.

It is the control declaration that says those surfaces are declared and governed. The r08 matrix is explicitly scoped to **JSON artifacts only** and excludes `.cue`, `.jsonnet`, Python/runtime code, and Markdown guidance from its formal JSON component matrix. 

---

# Anatomy of a manifest control object

The packet contract proposes these minimum top-level members for `*.bundle.json`, `*.projection.json`, and `*.generator.json`:

* `kind`
* `manifest_id`
* `spec_version`
* `manifest_class`
* `status`
* `owner_plane`
* `inputs`
* `outputs`
* `admission`
* `regen`
* `notes` when needed

A conceptual skeleton looks like this:

```json
{
  "kind": "kernel-manifest-control",
  "manifest_id": "projection.registry.v1",
  "spec_version": "v1",
  "manifest_class": "projection",
  "status": "declared",
  "owner_plane": "data_shaping",
  "inputs": [
    {
      "ref": "schemas/exported/exported-surface.index.json",
      "input_kind": "contract_index",
      "required": true
    }
  ],
  "outputs": [
    {
      "output_class": "projection_index",
      "path": "generated/state/projections/registry.index.json",
      "committed": true
    }
  ],
  "admission": {
    "required": true,
    "scope_ref": "policy/admission/scope.index.json"
  },
  "regen": {
    "policy": "declared_only",
    "authoritative_inputs_only": true
  }
}
```

That is the *minimum* control-object idea: a manifest declares the class, structured inputs, structured outputs, status, ownership plane, and its admission/regeneration policy. The schema family is then responsible for validating that shape and distinguishing the classes.

---

# The three manifest classes

## Bundle manifests

A **bundle manifest** groups and fences a set of authority-adjacent materials. Its purpose is packaging and boundary declaration, not rendering. In the file matrix, `kernel-core.bundle.json` exists to activate the core bundle as a typed manifest surface, while `compatibility.bundle.json` exists to keep compatibility material separated from `kernel-core` authority. In `r08.01`, the bundle class carries `bundle_id`, `bundle_scope`, `includes`, and `excludes`, along with the common manifest contract members.

Educationally: bundle manifests answer **“what belongs together, and what authority boundary does that grouping represent?”**

## Projection manifests

A **projection manifest** declares that a deterministic read-model or view exists, such as a registry view, a review view, or an admitted-state view. It declares projection classes without embedding runtime semantics. The packet contract gives the projection class `projection_id`, `input_state_class`, `renderer_identity`, and `output_class`, but keeps the manifest declared rather than executable.

Educationally: projection manifests answer **“what derived view is legal to render from admitted state?”**

## Generator manifests

A **generator manifest** declares a codegen or derived-output surface. Examples include schema export, Python bindings, and evidence bundle generation. The packet contract gives the generator class `generator_id`, `generator_identity`, `contract_input_refs`, and `derived_output_classes`, but keeps the manifest declared rather than executable.

Educationally: generator manifests answer **“what reproducible derived artifacts may be emitted, from which governed inputs?”**

---

# File-by-file deep dive

## Core manifest instances

### `manifests/bundles/kernel-core.bundle.json`

This is the primary bundle control object for the kernel-core authority surface. It declares that the kernel core exists as a typed bundle, rather than only as directories and prose. It is the canonical “bundle manifest” example for the first activation packet.

### `manifests/projections/registry.projection.json`

This is the first projection declaration. It says a registry/read-model projection class exists, but at `r08` it is still just a declared control object. The rendering logic comes later in the Jsonnet tranche.

### `manifests/generators/schema-export.generator.json`

This declares schema export as a controlled derived surface. It is important because export surfaces need governance too; otherwise exported contracts become an informal byproduct instead of a declared artifact class.

## Valid and invalid examples

### `examples/valid/kernel-core.bundle.example.json`

A positive proof that a bundle manifest can satisfy the schema and declared conventions. It is not just documentation; it is a validation fixture.

### `examples/valid/registry.projection.example.json`

A positive proof for a projection control object. It demonstrates the valid control-object boundary for a projection class.

### `examples/valid/schema-export.generator.example.json`

Conceptually this is the generator-side positive proof: a generator control object that validates and demonstrates the intended declaration pattern. The matrix explicitly requires generator examples to prove valid generator control objects. 

### `examples/invalid/manifest-control.bad-class.example.json`

This exists to prove rejection of an invalid `manifest_class`. It tests the discriminated-family boundary. 

### `examples/invalid/manifest-control.missing-id.example.json`

This proves the schema rejects a manifest missing the required identity field. It tests the minimum identity boundary. 

### `examples/invalid/manifest-control.bad-output.example.json`

This proves output declarations are structurally constrained and cannot be arbitrary garbage. It tests output declaration hygiene. 

### `examples/invalid/manifest-control.runtime-leak.example.json`

This is especially important conceptually: it proves runtime execution semantics are not allowed to leak into manifest declarations. That preserves the plane boundary between declaration/control and tool/runtime implementation.

## Schema, indexes, and prose

### `schemas/exported/manifest-control-family.schema.json`

This is the structural contract for all manifest control objects. Its job is to validate the common fields and distinguish `bundle`, `projection`, and `generator` classes in a machine-readable way. Without this schema, manifests are only conventions.

### `manifests/surface.index.json`

This is the catalog of manifest surfaces. The pkt-plan says it must enumerate active manifest instances, not just class folders. The matrix proposes fields such as `active_manifest_root`, `active_classes`, `active_instances`, `deferred_manifest_families`, and `ambiguity_closed`. In other words, the index answers **“what manifest surfaces currently exist in the repo, and in what state?”**

### `examples/valid/example.index.json` and `examples/invalid/example.index.json`

These are example registries. The matrix says example indexes should minimally include `generated_by_packet`, `class`, and `files`. Their purpose is not to define policy but to make the example suite queryable and packet-scoped. 

### `README.md` and `kernel.spec.md`

These are explanatory and normative prose surfaces, but the packet contract is the authority for machine validation. That means they are useful for human interpretation and conventions, but they are not sufficient as machine authority. The whole direction of `r08` is to stop relying on prose alone where typed contracts are required.

---

# The “next wave” manifest files

These are important because they show manifests are not just for the first bundle.

### `manifests/bundles/compatibility.bundle.json`

This declares the compatibility bundle and keeps it from being merged into `kernel-core` authority. It is a boundary-preserving manifest.

### `manifests/projections/review-view.projection.json`

This declares a review-view projection contract. In the later rendering tranche, review views become deterministic outputs, but here the manifest declares the class first.

### `manifests/projections/admitted-state.projection.json`

This declares the admitted-state projection contract. It matters because the plan is strict that projections should consume admitted state, not raw source material.

### `manifests/generators/python-bindings.generator.json`

This declares typed bindings as a derived codegen surface. The minimal expansion plan explicitly treats codegen and scaffolding as first-class, not optional extras, while also saying generated bindings are never authoritative.

### `manifests/generators/evidence-bundle.generator.json`

This declares evidence bundle generation as a controlled derived output. That aligns with the explicit emphasis on events/logs/evidence as first-class kernel outputs.

---

# How manifests fit the broader JSON-family model

The amendment material expands the kernel into a **JSON-family model**:

* core authoring/control: JSON Structure, JSON Schema, CUE, Jsonnet
* operational/interface envelopes: NDJSON/JSONL, JSON-RPC, JSON:API, optional JSON-LD
* codegen/interop bridges: Structurize, Avrotize 

Manifests are the place where those future surfaces become governable. They do not make NDJSON authoritative, do not make JSON-RPC authoritative, and do not make generated bindings authoritative. Instead, manifests can declare those surfaces as **derived, bounded, and classed**. That is how you prevent “tool soup”: each surface is declared, classed, and lineage-bounded before implementation is allowed.

---

# What manifests are supposed to prevent

A well-designed manifest layer prevents several common architecture failures:

## 1. Hidden coupling

Without manifests, renderers or generators may silently depend on undeclared inputs.

## 2. Authority drift

Generated outputs or runtime config may accidentally become de facto authority.

## 3. Plane collapse

Projection logic may start inventing policy, or runtime code may start re-owning admission rules.

## 4. Unreviewable derivation

You may have Jsonnet, codegen, or runtime outputs without any typed declaration of what classes are supposed to exist.

## 5. Boundary leakage

Runtime/controller semantics may leak into source control objects, which is exactly what invalid examples like `manifest-control.runtime-leak.example.json` are meant to catch.

---

# Educational summary

The clean mental model is:

## JSON Structure

Author the manifest as source-side structure. 

## JSON Schema

Define the structural validity of the manifest family.

## CUE

Decide whether the declared manifest participation is admissible in a real policy bundle.

## Jsonnet

Consume admitted state through declared projection manifests to produce deterministic derived views.

## Manifest

The declaration that links those layers without collapsing them.

If you want, the next useful step is a **class-by-class manifest schema design** for `bundle`, `projection`, and `generator`, with concrete required fields and invariants for each.
