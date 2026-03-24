# Triple-Stack Additional Workflows

## Purpose

This document consolidates the additional workflow surfaces discussed in the thread around a contract-first triple stack:

- **JSON Schema** for structural contract definition
- **CUE** for legality, defaults, composition, and policy resolution
- **Jsonnet** for projection and artifact generation
- **Specialized generators** for typed Python where appropriate
- **Python** for runtime behavior, orchestration, validation, and side effects

The emphasis here is on **additional workflows beyond typed object generation**, especially development-surface generation, Codex-oriented assets, registries, and orchestration artifacts.

---

## Core Model

### Layer roles

| Layer | Primary responsibility |
|---|---|
| JSON Schema | Structural contract, type vocabulary, boundary validity |
| CUE | Admission, defaults, composition, cross-object legality |
| Specialized generator | Typed Python generation where appropriate |
| Jsonnet | Projection of files, scaffolds, registries, workflows, and developer artifacts |
| Python | Runtime implementation, operational validation, orchestration, adapters, side effects |

### Canonical pipeline

```text
JSON Schema/CUE define authority
CUE resolves/exports
specialized generator generates typed Python where appropriate
Jsonnet projects scaffolds and surrounding artifacts
Python implements runtime behavior
```

### Expanded pipeline

```text
authoritative object + workflow definitions
  -> JSON Schema defines structural contracts
  -> CUE resolves legality/defaults/composition
  -> specialized generator generates typed Python where appropriate
  -> Jsonnet generates project and workflow artifacts
  -> Python executes runtime logic and operational checks
```

---

## Additional Workflow Families

## 1. Typed Python Model Generation

This is the most direct schema-driven workflow.

### Best fit
- JSON Schema
- CUE exported schema/value surfaces
- specialized code generators such as model generators for Pydantic, dataclasses, TypedDicts, and enum/type bindings

### Typical outputs
- Pydantic models
- dataclasses
- TypedDicts
- enums
- nested object bindings
- request/response models
- contract objects

### Role split
- **JSON Schema** defines structural object shape
- **CUE** resolves defaults and legality before generation when needed
- **specialized generators** create typed Python bindings
- **Python** uses the generated models at runtime

---

## 2. Python Skeleton and Stub Generation

Jsonnet can generate Python skeletons aligned with JSON Schema and CUE.

### Good generation targets
- model stubs
- validator scaffolds
- emitter skeletons
- package/module layout files
- registry/index modules
- base classes
- test scaffolds

### Recommended split

**Generated**
- base models
- field/type declarations
- enum declarations
- docstrings from contract metadata
- validator hook placeholders
- emitter interfaces
- test skeletons

**Handwritten**
- operational validation
- git/filesystem probing
- external integrations
- orchestration logic
- side-effectful emitters
- complex runtime checks

### Safe extension pattern
Generate base files and extend them in handwritten code.

```text
generated/python/models/_packet_contract_base.py
src/python/models/packet_contract.py
```

This avoids regeneration clobbering runtime behavior.

---

## 3. Project Scaffold Generation

The generation surface extends naturally beyond models into whole-project scaffolds.

### Common targets
- `uv` project scaffold
- `pyproject.toml`
- package layout under `src/`
- test directory structure
- tool configuration sections
- dependency groups
- script entry points
- example files
- bootstrap files

### Why this fits
Project scaffolds are structured text/file artifacts derived from policy and project metadata. Jsonnet is well suited to emitting them reproducibly.

### Typical split
- **Schema/CUE** define what project/profile/tooling combinations are legal
- **Jsonnet** emits the scaffold files
- **Python** may provide bootstrap validation, environment checks, or follow-up initialization logic

---

## 4. Justfiles and Recipe Modules

These are strong projection targets when workflow surfaces should be derived from policy.

### Common targets
- root `justfile`
- imported `just` modules
- environment-aware recipe sets
- task registries
- standardized build/test/lint/release surfaces
- controller entrypoints

### Why this fits
If workflows are policy-defined, the task surface should not drift from the underlying contract. Generating `justfile` surfaces preserves alignment.

### Typical split
- **Schema/CUE** define legal phases, allowed commands, and profile-specific enablement
- **Jsonnet** emits the `justfile` and imported modules
- **Python** executes the underlying controllers or scripts invoked by recipes

---

## 5. Git Hooks, Commit Messages, and CI Workflows

These also belong in the projection plane.

### Common targets
- `.githooks/*`
- commit message templates
- pre-commit / pre-push wrappers
- `.github/workflows/*.yml`
- PR templates
- issue templates
- repository policy companion docs
- CI helper configs

### Why this fits
These are policy-carrying text artifacts. If a repo has defined gates, budgets, or promotion rules, the hook and CI surfaces should be derived from the same authority.

### Typical split
- **Schema/CUE** define gate conditions, budgets, required evidence, allowed actions
- **Jsonnet** renders hook scripts, CI YAML, and message templates
- **Python** performs the actual checks, validation, or evidence collection during execution

---

## 6. Shell Script Generation

Shell scripts are common operational adapters and are straightforward generation targets.

### Common targets
- bootstrap scripts
- validation wrappers
- CI helper scripts
- local setup scripts
- release scripts
- repo hygiene scripts
- thin command wrappers

### Why this fits
These scripts often reflect stable workflow shapes that should be reproducible and profile-aware.

### Typical split
- **Schema/CUE** define which adapters are legal and under which profiles
- **Jsonnet** emits the scripts
- **Python** remains the deeper logic layer where needed

---

## 7. Codex Config Generation

Codex configs are first-class artifact outputs.

### Common targets
- repo-local Codex configuration
- environment/profile overlays
- tool allowlists and defaults
- execution policies
- workspace bootstrap config
- app-server or client-side config surfaces

### Why this fits
These are declarative control surfaces. They should be derived from the same contract/policy layer that governs tools, skills, and workflow phases.

### Typical split
- **Schema/CUE** define which config objects are legal
- **Jsonnet** emits concrete Codex config files
- **Python** may consume, validate, or reconcile them operationally

---

## 8. Skill Bundle Generation

Skills are especially good generation targets.

### Common targets
- `SKILL.md`
- skill metadata files
- references indexes
- helper script wrappers
- fixtures/examples
- skill registry entries
- package-local catalogs

### Skill contract dimensions
- skill identity
- trigger metadata
- tool access policy
- required inputs/outputs
- evidence expectations
- path/scope limits
- references and helper assets

### Typical split
- **Schema/CUE** define the skill contract and constraints
- **Jsonnet** emits `SKILL.md`, wrappers, references, and registry entries
- **Python** or other runtime layers execute the actual behavior the skill calls into

---

## 9. Tool, Prompt, and RAG Registry Generation

Registries are mixed artifacts: part contract, part projection, part runtime lookup surface.

### Contract-side concerns
- registry item IDs
- aliases
- tags
- profiles
- source pointers
- retrieval constraints
- authority references
- consumer permissions
- freshness/invalidation metadata

### Artifact-side outputs
- `index.json`
- `manifest.json`
- alias maps
- bundle views
- consumer-specific lookup tables
- prompt assemblies
- retrieval manifests
- inventories/docs

### Typical split
- **JSON Schema** defines registry object validity
- **CUE** defines composition and legality of registry state
- **Jsonnet** emits concrete registry files and views
- **Python** handles runtime lookup, retrieval, caching, and assembly

---

## 10. Agentic Orchestration Pipeline Assets

This workflow spans all layers.

### Contract objects
- task
- packet
- work order
- plan
- evidence capsule
- gate decision
- transition
- controller config
- tool contract
- registry binding

### Policy concerns
- admissible transitions
- phase order
- required evidence
- tool allowlists per phase
- budget constraints
- profile/environment defaults
- controller invariants

### Artifact outputs
- skill bundles
- tool registries
- prompt registries
- workflow specs
- CI gates
- justfiles
- wrapper scripts
- app-server/client config
- pipeline manifests
- task catalogs

### Runtime concerns
- orchestration logic
- tool invocation
- event handling
- retries and backpressure
- evidence collection
- gating
- promotion / iteration
- state reconciliation

### Layer split
- **JSON Schema** defines typed objects for agents, tools, registries, and pipeline nodes
- **CUE** defines legality of composition and transitions
- **specialized generators** generate typed Python bindings where appropriate
- **Jsonnet** emits configs, skills, registries, workflow artifacts, and scaffolds
- **Python** or another runtime executes the live orchestration pipeline

---

## Common Generation Surfaces

The discussion implies a broad generation surface. In practice, the stack can generate:

- typed Python models
- Python source skeletons
- validator stubs
- emitter scaffolds
- `uv` project scaffolds
- `pyproject.toml`
- `justfile` and imported just modules
- git hooks
- commit message templates
- CI workflows
- shell scripts
- Codex configs
- skills and supporting wrappers
- tool registries
- prompt registries
- RAG registries
- pipeline manifests
- package/module registries
- docs and inventories
- test skeletons and fixtures

---

## Recommended Ownership Rules

## Put in JSON Schema when
- the concern is structural shape
- the concern is stable object vocabulary
- the concern is boundary validity
- the concern should be language-neutral

## Put in CUE when
- the concern is defaults
- the concern is composition
- the concern is admissibility
- the concern is cross-object legality
- the concern depends on profile/environment resolution

## Use a specialized generator when
- you need typed Python bindings from formal contracts
- you want model fidelity from schema-defined objects
- you are generating Pydantic/dataclass/TypedDict-style outputs

## Put in Jsonnet when
- the output is file-shaped or text-shaped
- you need scaffolds, registries, workflow surfaces, or repo glue
- you need reproducible projections for environments or profiles
- you want generated Python source files, configs, registries, test skeletons, or other artifacts from templates/projections

## Put in Python when
- the concern is runtime behavior
- the concern is operational validation
- the concern depends on git/filesystem/process/API state
- the concern is orchestration, retries, or event handling
- the concern has side effects

---

## Practical Consolidated Model

```text
JSON Schema = structural contract
CUE         = legality, defaults, composition
specialized generator = typed Python where appropriate
Jsonnet     = project/dev/agent artifact generation
Python      = runtime behavior, validation, orchestration
```

## Extended formulation

```text
authoritative contract/policy
  -> schema + cue
  -> cue resolution/export
  -> specialized generator generates typed Python where appropriate
  -> Jsonnet generates Python source files, configs, registries, test skeletons,
     uv project scaffolds, justfiles and modules, git hooks, commit message
     templates, CI workflows, shell scripts, Codex configs, skills, and other
     project/dev-workflow artifacts from templates/projections
  -> Python completes runtime behavior, operational validation, and orchestration
```

---

## Final Summary

The thread expands the triple-stack model from a narrow “schema to Python models” workflow into a broader **contract-defined development and agent workflow surface**.

The main conclusion is:

- **JSON Schema** and **CUE** define the legal object and workflow space
- a **specialized generator** creates typed Python bindings where that is the right tool
- **Jsonnet** serves as the projection engine for repo, workflow, registry, skill, and agent artifacts
- **Python** implements the live operational layer

This supports not just typed contracts, but also:

- development scaffolds
- repo control surfaces
- generated workflow tooling
- Codex skills/configuration
- prompt/tool/RAG registries
- agentic orchestration assets

all from a single contract/policy core.
