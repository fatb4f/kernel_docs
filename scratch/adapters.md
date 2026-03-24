## Tooling matrix

| Tool family                    | Best placement in your kernel                            | Best at                                                               | Not for                                                    | Signal                              |
| ------------------------------ | -------------------------------------------------------- | --------------------------------------------------------------------- | ---------------------------------------------------------- | ----------------------------------- |
| **jsonargparse**               | **G1 parser/SDK ingress** for Python controllers         | CLI + env + config-file intake, typed/class-bound Python entrypoints  | Admission, normalization authority, or rendering authority | **High**, but only at the edge      |
| **pydantic-settings**          | **Python runtime config** inside the live app/controller | App-local settings from env, `.env`, secrets, source-priority control | Repo-wide workflow ingress or multi-surface projection     | **Medium-high**, but narrower       |
| **generator-based bindings**   | **Typed binding lane** off exported contracts            | Pydantic/dataclass/TypedDict contract objects                         | Policy, legality, or scaffold/render ownership             | **High** for model fidelity         |
| **CUE CLI (`vet` / `export`)** | **Admission + validated export**                         | Legality checks, concreteness checks, validated concrete outputs      | Operator CLI UX or final file projection                   | **Highest** for policy/admission    |
| **Jsonnet runners**            | **G5 rendering / projection**                            | File/text/config/scaffold/registry generation from admitted state     | Raw-source reads, legality, or runtime orchestration       | **Highest** for artifact projection |

## Per-tool read

### 1) `jsonargparse`

Use it as the **Python-facing ingress adapter**. Its docs explicitly center CLI construction and Python configurability, with support for configuration files, environment variables, automatic/type-hint-driven parser generation, class/function argument binding, class instantiation, and documented sections for **JSON Schemas** and **Jsonnet files**. The same docs do **not** present a CUE integration surface, which is why it fits your stack as a **G1 parser/SDK layer**, not as policy or admission. That matches your kernel spec, where **G1 = parser/SDK**, **G4 = CUE**, and **G5 = Jsonnet**. ([jsonargparse.readthedocs.io][1]) 

### 2) `pydantic-settings`

Use it **inside the Python runtime**, not as the repo/kernel front door. The official docs position it around settings loaded from **environment variables**, **dotenv files**, and **secrets**, with explicit **field value priority** and **customizable settings sources**. In your stack, that makes it a good fit for a generated or handwritten Python controller that needs runtime config, but not for the broader contract/policy/render pipeline. ([Pydantic][2]) 

### 3) Generator-based bindings

These are the right answer for **typed Python contract objects**. Your workflow document already assigns this lane to **specialized generators** for Pydantic/dataclasses/TypedDicts/enums, while keeping Python as the runtime/orchestration layer. In the kernel, that means bindings should be derived from the **exported contract surface**, not elevated into authority or policy.  

### 4) CUE CLI

This is the strongest tool family for **admission and validated export**. Official CUE docs state that `cue vet` validates **CUE and other data files** and can validate JSON, YAML, TOML, and text inputs against CUE schemas, while `cue export` evaluates a configuration and emits **concrete data**, with output encodings including JSON, YAML, CUE, and TOML. That aligns directly with your kernel, where CUE owns **legality over normalized state** and emits explicit admission artifacts. ([CUE][3]) 

### 5) Jsonnet runners

These are the right tool for **projection/rendering**. Official Jsonnet docs show imports, `importstr`/`importbin`, and parameterization via **top-level arguments** or **ExtVars**; the language docs explicitly prefer **Top-level Arguments** as the canonical parameterization mechanism and warn that ExtVars create global composability problems. In your kernel, Jsonnet must still obey the harder rule: it renders **from admitted state only** and must not read raw authoring sources directly. That makes Jsonnet the best projection engine, but only after CUE admission. ([Jsonnet][4])  

## Recommended cut for your pipeline

```text
raw operator input
  -> jsonargparse               # Python CLI/env/config ingress
  -> normalizer                 # your kernel-owned normalization step
  -> CUE vet/export             # legality + concrete admitted/exported state
  -> Jsonnet runner             # files, configs, registries, scaffolds
  -> Python runtime
       |- pydantic-settings     # runtime env/.env/secrets
       `- generated bindings    # typed contract objects
```

This is the cleanest ownership split under your spec because it preserves:

* **JSON Schema** as boundary/export contract
* **CUE** as legality/admission
* **Jsonnet** as projection
* **specialized generators** as typed-binding emitters
* **Python** as runtime/orchestration
* **jsonargparse** as ingress, and **pydantic-settings** as app-local runtime config  

## Bottom line

If the question is **“high-signal edge adapters over this kernel?”**, the ranking is:

1. **jsonargparse** for **Python ingress**
2. **pydantic-settings** for **runtime settings**
3. **generator-based bindings** for **typed contract objects**
4. **CUE CLI** for **admission/export**
5. **Jsonnet runners** for **projection/rendering**

So the real pairings are:

* **jsonargparse + pydantic-settings** on the **Python side**
* **CUE + Jsonnet** on the **kernel/policy/projection side**
* **specialized generators** between contract and runtime model layers ([jsonargparse.readthedocs.io][1])  

Next useful pass is a **failure-mode matrix**: drift risk, authority leakage, regeneration safety, and where each tool can incorrectly absorb responsibilities.

[1]: https://jsonargparse.readthedocs.io/en/latest/ "jsonargparse — jsonargparse  documentation"
[2]: https://docs.pydantic.dev/2.0/usage/pydantic_settings/ "Settings Management - Pydantic"
[3]: https://cuelang.org/docs/reference/command/cue-help-vet/ "cue help vet | CUE"
[4]: https://jsonnet.org/learning/tutorial.html "Jsonnet - Tutorial"
