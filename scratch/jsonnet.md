## High-signal Jsonnet

### Tier 1 — high-signal by default

**1. Kapitan**
Best fit if you want Jsonnet to become a **general configuration control plane**, not just a Kubernetes manifest generator. Kapitan’s core model is an **Inventory** as SSOT, with hierarchical variables feeding templating/compilation; its docs explicitly position it for Kubernetes, Terraform, docs, scripts, and business logic, and its target model gives you a strong host/role/environment abstraction. That makes it the strongest option in your list for stretching the “pod” idea beyond a cluster and onto localhost. ([Kapitan][1])

**2. Grafana Tanka**
Very high-signal if the center of gravity is still **Kubernetes**. Tanka is explicitly framed as a Jsonnet-based alternative to YAML **for Kubernetes clusters**, with first-class `tk diff`, Helm integration, Kustomize support, and an Environment model driven by `spec.json` that points at a Kubernetes API server or kube context. It is excellent for cluster config, but it is fundamentally **cluster-first**, not host-first. ([GitHub][2])

**3. jrsonnet**
High-signal when you want Jsonnet as an **embeddable runtime**, especially in Rust-heavy tooling, local agents, supervisors, or config compilers. jrsonnet is a Rust implementation of Jsonnet that ships both as a library and as an alternative executable, which makes it the strongest item here if your plan is to build your own local-host “pod controller” rather than just write manifests. ([GitHub][3])

---

### Tier 2 — high-signal, but conditional

**4. argo-workflows-libsonnet**
High-signal only if your abstraction is specifically **workflow CRDs / Argo-style orchestration**. It is a generated Jsonnet library for Argo Workflows with versioned docs across multiple API versions, so it is useful as a schema-safe building block, but it is a **domain library**, not a control-plane foundation. ([jsonnet-libs.github.io][4])

**5. kapitan-reference**
Useful because it is a **bootstrap and repo-shape reference**, not because it adds new primitives. Its README explicitly says it is meant to bootstrap a Kapitan setup and surface best practices, with an `init` branch for a minimal starting point. Good to steal layout from; not the main thing to adopt conceptually. ([GitHub][5])

---

### Adjacent, not core Jsonnet signal

**6. jsonargparse**
Useful, but not really a Jsonnet platform choice. It is primarily a Python CLI/config parser that can **consume Jsonnet files** via `parser_mode="jsonnet"` or `ActionJsonnet`, including external variables. That makes it a good **edge adapter** for Python tools, but not a high-signal Jsonnet architecture decision by itself. ([GitHub][6])

---

## For **extending the k8s pod concept to localhost**

### Best stack from your list

**Kapitan + jrsonnet** is the highest-signal combination.

Why:

* **Kapitan** gives you the right control model: inventory, targets, inheritance, secrets/references, and multi-output compilation beyond Kubernetes. That is exactly what you need if a “local pod” means a host-scoped bundle of processes, env, volumes, secrets, and policies. ([Kapitan][1])
* **jrsonnet** gives you a fast, embeddable evaluator for building your own local runtime/controller in Rust. ([GitHub][3])

### Tanka’s role

Use **Tanka** only when localhost is still acting as a **real Kubernetes target**. Its environment model and config are tied to kube API/context selection, diff/apply workflows, and cluster semantics. That makes it excellent for k3s/kind-style local clusters, but weaker for plain host service orchestration. ([GitHub][2])

---

## Practical recommendation

### Strongest adoption order

1. **Kapitan**
2. **jrsonnet**
3. **Tanka**
4. **argo-workflows-libsonnet**
5. **kapitan-reference**
6. **jsonargparse**

### If your goal is specifically “localhost pods”

Model it like this:

* **target** = host / role / machine class
* **component** = service bundle
* **sidecars** = companion daemons/helpers
* **volumes** = filesystem mounts/dirs
* **configmaps/secrets** = compiled files + Kapitan refs
* **scheduler/controller** = systemd, Podman, Nomad, or your own Rust runner

Under that framing, **Kapitan is the architectural core**, **jrsonnet is the runtime enabler**, and **Tanka is optional only when a real kube API remains in the loop**. ([Kapitan][1])

## Bottom line

If you want **Jsonnet as a serious host+cluster control language**, the highest-signal items here are:

* **Kapitan** — best architectural leverage
* **jrsonnet** — best runtime leverage
* **Tanka** — best Kubernetes-native leverage

The others are supporting pieces, not the center.

I can do the same pass next for **libsonnet ecosystems around systemd / Nomad / Podman / ArgoCD / Crossplane**.

[1]: https://kapitan.dev/0.34/pages/kapitan_overview/ "Core Concepts - Kapitan: Keep your ship together"
[2]: https://github.com/grafana/tanka "GitHub - grafana/tanka: Flexible, reusable and concise configuration for Kubernetes · GitHub"
[3]: https://github.com/CertainLach/jrsonnet "GitHub - CertainLach/jrsonnet: Rust implementation of Jsonnet language · GitHub"
[4]: https://jsonnet-libs.github.io/argo-workflows-libsonnet/ "argo-workflows jsonnet library"
[5]: https://github.com/kapicorp/kapitan-reference/blob/master/README.md "kapitan-reference/README.md at master · kapicorp/kapitan-reference · GitHub"
[6]: https://github.com/omni-us/jsonargparse/blob/main/DOCUMENTATION.rst "jsonargparse/DOCUMENTATION.rst at main · omni-us/jsonargparse · GitHub"
