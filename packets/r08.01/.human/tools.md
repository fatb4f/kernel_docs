For `r08.01`, the fastest setup is:

* **VS Code = schema cockpit**
* **terminal = assertion engine**
* **`just` = single front door**

This packet is **JSON-first and declaration-only**. The work breaks into a tight sequence: expand the manifest schema family, add the 3 manifest instances, update the surface index, add valid examples, add invalid examples, update example indexes, then align the docs.

## VS Code setup

### Extension set

Use the built-in JSON tooling as the primary editor surface. VS Code already supports JSON validation and IntelliSense, and you can bind workspace-local schemas with `json.schemas` in `settings.json`. VS Code tasks can also surface command output in the Problems panel through problem matchers. ([Visual Studio Code][1])

Install these:

* **JSON:** built-in only
* **CUE:** `cuelangorg.vscode-cue`
  The old `asdine.cue` extension is deprecated; the official CUE extension is the current one. ([Visual Studio Marketplace][2])
* **Jsonnet:** `cverge.jsonnet-lsp`
  Good fit if this repo will keep growing; it is explicitly aimed at large repos with nested imports. If you only want lightweight Jsonnet support, `Sebbia.jsonnetng` is the simpler fallback. ([Visual Studio Marketplace][3])
* **Markdown:** `DavidAnson.vscode-markdownlint`, `yzhang.markdown-all-in-one`
  Useful here because `README.md`, `kernel.spec.md`, and packet notes are part of the deliverable. ([Visual Studio Marketplace][4])
* **Editor hygiene:** official EditorConfig extension, plus `usernamehw.errorlens`
  EditorConfig keeps repo formatting consistent; Error Lens makes validation failures harder to miss. ([Visual Studio Marketplace][5])
* **Optional:** `redhat.vscode-yaml`
  Only if you keep CI/workflow YAML nearby. ([Visual Studio Marketplace][6])

### `.vscode/extensions.json`

```json
{
  "recommendations": [
    "cuelangorg.vscode-cue",
    "cverge.jsonnet-lsp",
    "DavidAnson.vscode-markdownlint",
    "yzhang.markdown-all-in-one",
    "EditorConfigTeam.EditorConfig",
    "usernamehw.errorlens",
    "redhat.vscode-yaml"
  ]
}
```

### `.vscode/settings.json`

Bind the schema once and make every manifest/example file light up immediately.

```json
{
  "editor.formatOnSave": true,
  "files.trimTrailingWhitespace": true,
  "search.exclude": {
    "**/.git": true,
    "**/node_modules": true,
    "**/dist": true,
    "**/build": true
  },

  "[json]": {
    "editor.defaultFormatter": "vscode.json-language-features"
  },

  "json.schemas": [
    {
      "fileMatch": [
        "kernel/manifests/**/*.bundle.json",
        "kernel/manifests/**/*.projection.json",
        "kernel/manifests/**/*.generator.json",
        "kernel/examples/valid/*.example.json",
        "kernel/examples/invalid/*.example.json"
      ],
      "url": "./kernel/schemas/exported/manifest-control-family.schema.json"
    }
  ],

  "markdownlint.config": {
    "MD013": false
  }
}
```

### Editor layout

Use a **2-column + bottom panel** layout:

* **Left:** `r08.01.final.packet.json` or `r08.01.final.packet.md`
* **Right:** the file currently being authored
* **Bottom:** Problems + Terminal

Do not keep more than one target file open at once. For this packet, context switching is where mistakes happen.

---

## Terminal toolchain

Use this stack:

* **`just`** — front door for all repeatable checks
* **`jq`** — inspect packet structure and assert JSON invariants
* **`rg`** — boundary checks like runtime/controller leakage
* **`fd`** — enumerate target files cleanly
* **`check-jsonschema`** — schema validation CLI
* **`python`** — tiny repo-specific assertions
* **`git diff --word-diff`** — doc alignment pass

`check-jsonschema` is a dedicated JSON Schema CLI and pre-commit hook; its normal usage is `check-jsonschema --schemafile schema.json instance.json`. ([check-jsonschema.readthedocs.io][7])

### Terminal layout

Use 3 terminal tabs:

1. **inspect**
2. **validate**
3. **git**

That gives you a stable loop:

* inspect packet / repo state
* validate current file
* review diff

---

## Recommended authoring order

Use this exact order:

1. `manifest-control-family.schema.json`
2. `kernel-core.bundle.json`
3. `registry.projection.json`
4. `schema-export.generator.json`
5. `surface.index.json`
6. valid examples
7. invalid examples
8. valid/invalid example indexes
9. `README.md` and `kernel.spec.md`

That order matches the packet’s dependency shape. The schema must exist before the instance files are comfortable to author.

---

## Commands to use immediately

### Inspect the packet

```bash
jq '.file_matrix[] | {path, status, manifest_class, initial_lifecycle}' r08.01.final.packet.json
```

```bash
jq '.member_matrix.class_specific_requirements' r08.01.final.packet.json
```

```bash
jq -r '.implementation_order[]' r08.01.final.packet.json
```

### Enumerate the concrete targets

```bash
fd -a '(.+\.bundle\.json|.+\.projection\.json|.+\.generator\.json|.+\.example\.json|surface\.index\.json)$' kernel
```

### Search for boundary leaks

```bash
rg -n 'runtime|controller|executable|spawn|subprocess|server|handler' kernel/manifests kernel/examples
```

### Validate the manifest files against the schema

```bash
check-jsonschema \
  --schemafile kernel/schemas/exported/manifest-control-family.schema.json \
  kernel/manifests/bundles/kernel-core.bundle.json \
  kernel/manifests/projections/registry.projection.json \
  kernel/manifests/generators/schema-export.generator.json \
  kernel/examples/valid/kernel-core.bundle.example.json \
  kernel/examples/valid/registry.projection.example.json \
  kernel/examples/valid/schema-export.generator.example.json
```

### Assert the surface index shape quickly

```bash
jq -e '
  .packet_id == "pkt-r08-01" and
  .active_manifest_root == "kernel/manifests/" and
  (.manifests | length == 3) and
  ([.manifests[].manifest_class] | sort == ["bundle","generator","projection"])
' kernel/manifests/surface.index.json >/dev/null
```

### Review only the docs at the end

```bash
git diff --word-diff -- kernel/README.md kernel/kernel.spec.md
```

---

## Make `just` the control plane

### `justfile`

```make
set shell := ["bash", "-euo", "pipefail", "-c"]

schema := "kernel/schemas/exported/manifest-control-family.schema.json"

validate-manifests:
    check-jsonschema \
      --schemafile {{schema}} \
      kernel/manifests/bundles/kernel-core.bundle.json \
      kernel/manifests/projections/registry.projection.json \
      kernel/manifests/generators/schema-export.generator.json \
      kernel/examples/valid/kernel-core.bundle.example.json \
      kernel/examples/valid/registry.projection.example.json \
      kernel/examples/valid/schema-export.generator.example.json

check-surface-index:
    jq -e '\
      .packet_id == "pkt-r08-01" and \
      .active_manifest_root == "kernel/manifests/" and \
      (.manifests | length == 3) and \
      ([.manifests[].manifest_class] | sort == ["bundle","generator","projection"]) \
    ' kernel/manifests/surface.index.json >/dev/null

check-boundaries:
    ! rg -n 'runtime|controller|executable|spawn|subprocess|server|handler' \
      kernel/manifests kernel/examples

inspect-packet:
    jq '.file_matrix[] | {path, status, manifest_class, initial_lifecycle}' r08.01.final.packet.json

check-r08: validate-manifests check-surface-index check-boundaries
```

---

## Wire it into VS Code tasks

### `.vscode/tasks.json`

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "r08: inspect packet",
      "type": "shell",
      "command": "just",
      "args": ["inspect-packet"],
      "presentation": {
        "reveal": "always",
        "panel": "shared"
      }
    },
    {
      "label": "r08: validate manifests",
      "type": "shell",
      "command": "just",
      "args": ["validate-manifests"],
      "group": "build",
      "presentation": {
        "reveal": "always",
        "panel": "shared"
      }
    },
    {
      "label": "r08: full check",
      "type": "shell",
      "command": "just",
      "args": ["check-r08"],
      "group": "test",
      "presentation": {
        "reveal": "always",
        "panel": "shared"
      }
    }
  ]
}
```

---

## Practical workflow

### Pass 1 — schema first

Only touch `manifest-control-family.schema.json`.

Goal:

* discriminated family
* common required members
* class-specific required members
* structured `inputs` / `outputs`

Run:

```bash
just validate-manifests
```

It will fail early until the instance files exist, which is fine.

### Pass 2 — one manifest per class

Create only these 3:

* `kernel-core.bundle.json`
* `registry.projection.json`
* `schema-export.generator.json`

After each file:

```bash
just validate-manifests
```

### Pass 3 — index and examples

Update:

* `manifests/surface.index.json`
* valid examples
* invalid examples
* example indexes

Then run:

```bash
just check-r08
```

### Pass 4 — doc alignment

Only after the JSON is green, update:

* `kernel/README.md`
* `kernel/kernel.spec.md`

Then:

```bash
git diff --word-diff -- kernel/README.md kernel/kernel.spec.md
just check-r08
```

---

## Bottom line

The best fit here is:

* **built-in JSON validation + local schema binding in VS Code**
* **official CUE extension now, Jsonnet LSP parked for later packets**
* **markdownlint + Markdown All in One for the spec/docs layer**
* **`just` + `jq` + `rg` + `fd` + `check-jsonschema` in terminal**
* **schema-first, then one manifest class at a time, then indexes/examples, then docs**

That gives you a tight loop with minimal mode-switching and makes the packet checklist executable instead of prose.

[1]: https://code.visualstudio.com/docs/languages/json?utm_source=chatgpt.com "Editing JSON with Visual Studio Code"
[2]: https://marketplace.visualstudio.com/items?itemName=cuelangorg.vscode-cue&utm_source=chatgpt.com "CUE"
[3]: https://marketplace.visualstudio.com/items?itemName=cverge.jsonnet-lsp&utm_source=chatgpt.com "Jsonnet LSP"
[4]: https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint&utm_source=chatgpt.com "Markdownlint"
[5]: https://marketplace.visualstudio.com/items?itemName=EditorConfigTeam.EditorConfig&utm_source=chatgpt.com "EditorConfig"
[6]: https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml&utm_source=chatgpt.com "YAML Language Support by Red Hat"
[7]: https://check-jsonschema.readthedocs.io/?utm_source=chatgpt.com "check-jsonschema 0.37.0 documentation"
