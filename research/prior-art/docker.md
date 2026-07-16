# Prior-Art Study: Docker

*Status: Accepted · Informs: Books 06 (runtimes), 12 (packages, distribution), 11 §Ch10 (isolation).*

## 1. System in one paragraph
Docker's technical ingredients — namespaces, cgroups, union filesystems — all predated it. What Docker contributed was **packaging and distribution ergonomics**: a content-addressed image format, a layered build, a registry protocol, and a one-command path from "it works on my machine" to "it runs anywhere." The lesson is not about containers. It is that **a format plus a registry plus a trivial install command is what creates an ecosystem**, and that isolation, if it is not the default, will not happen.

## 2. Core ideas
1. **The image as an immutable, content-addressed artifact** — layers identified by digest, so identical layers are shared and an image is verifiable by hash.
2. **Layered, cached builds.** A build is a sequence of steps, each cached by content; changing the last step rebuilds only the last step.
3. **A registry protocol, not just a tool.** Push/pull as an open protocol is what turned images into an ecosystem; the OCI standardization ratified it.
4. **Isolation as the default execution shape** — a process gets its own filesystem, network, and PID namespaces without asking.
5. **Declared runtime contract** — entrypoint, env, ports, volumes: the interface between an image and its host, declared in the artifact.
6. **Separation of build-time and run-time.** The image is built once, run many times, unchanged.

## 3. Design decisions & trade-offs
- **Shared-kernel isolation** buys near-zero startup cost and density, at the cost of a large shared attack surface — container escape is a kernel bug away, which is why gVisor, Firecracker, and Workers-style isolates exist ([study](cloudflare-workers.md)).
- **Layer caching** buys fast builds, at the cost of the layer-ordering discipline every Dockerfile author learns painfully, and of secrets baked into layers by accident.
- **Mutable tags over immutable digests** buys convenience (`:latest`), at the cost of reproducibility — the single most-regretted default in the format.
- **Root by default** bought adoption, and cost the ecosystem a decade of privilege bugs. Defaults are policy.
- **The registry as a distribution point** buys reach, at the cost of supply-chain exposure: pull-by-tag from a public registry is trust-by-vibes, which is why signing arrived later, bolted on.

## 4. Relevance to Sankalpa
Book 12 is Docker's ecosystem lesson applied to a different artifact: a Package manifest (§Ch02), deterministic resolution (§Ch03), signing and provenance (§Ch04), an install lifecycle (§Ch05), and a marketplace (§Ch06). Book 11 §Ch10's plugin isolation shares Docker's problem — running third-party code you did not write and cannot audit — and its threat model (Book 11 §Ch01) explicitly assumes the malicious-package case Docker's defaults invited. Book 06's runtime abstraction may *use* containers, but is not defined by them.

## 5. What we adopt
- **Immutable, content-addressed, digest-identified artifacts** for Packages (Book 12 §Ch04) — the same reasoning as Git ([study](git.md)) and IR (Book 04 §Ch07).
- **Build once, run many; the artifact never mutates in place.** Installation is reconciled toward a declared version (Book 12 §Ch03 §5), never patched live.
- **A declared contract in the artifact itself.** Docker's entrypoint/env/ports become our manifest's `provides`, `requires.capabilities`, and `compatibility` (Book 12 §Ch02) — the difference being that ours declares *authority*, not just ports.
- **Distribution as an open protocol, not a product.** The marketplace (Book 12 §Ch06) must be a specified format and protocol so a private or air-gapped registry is a first-class case, exactly as OCI made registries plural.
- **Isolation as the default, not an option** (Book 11 §Ch10) — Docker's clearest lesson is that opt-in isolation is no isolation.
- **Caching by content digest** across build/install steps.

## 6. What we reject / change
- **The container as the unit of work.** Sankalpa's workload is an AOS IR execution graph, not a pod or an image (Book 01 §Ch06 §2, Book 15 §Ch04). A runtime *may* be container-backed; the model must not assume it (IR-P7).
- **Mutable tags.** Resolution is deterministic and digest-pinned (Book 12 §Ch03 §2); there is no `:latest` semantics anywhere in Package resolution. Docker's own ecosystem spent years unlearning this.
- **Install implies privilege.** Docker's `docker run` grants whatever the image asks for, and the ecosystem normalized `--privileged`. Book 12's rule is the inverse: **install ≠ privilege** (Book 15 §Ch04); capabilities are granted explicitly, per capability, and re-authorized on upgrade (RFC-0008) — because the upgrade path is exactly how Docker's trust model gets defeated in practice.
- **Signing as an afterthought.** Notary/Sigstore arrived after the ecosystem's shape was set, so most pulls are still unverified. Signing and provenance are mandatory from the start (Book 12 §Ch04), not a later opt-in.
- **Secrets in the artifact.** Build-arg and layer-baked secrets are a genre of incident. P7 makes it structurally impossible: values never enter an artifact, only `SecretRef`s (Book 11 §Ch04, Book 06 §Ch06).
- **Shared-kernel isolation as sufficient.** For untrusted third-party plugins under our threat model, namespace isolation is a floor, not a ceiling; Book 11 §Ch10 specifies isolation *properties*, leaving stronger mechanisms (microVMs, isolates, Wasm) available.
- **Dockerfile as an imperative build.** Non-deterministic builds (`apt-get update` at build time) are the norm there; our artifacts are content-addressed and reproducibility matters (P6).

## 7. Open questions
- Book 11 §Ch10 specifies isolation properties rather than a mechanism. Is there a stated *minimum* — and does a shared-kernel container clear it for a plugin from an unknown publisher, or only for a first-party one?
- Docker's ecosystem grew because the artifact was trivial to produce. What is our equivalent of `docker build` — how cheap is it to author and publish a Capability Package, and does Book 12 §Ch05 keep that path short without weakening §Ch04?
- Is a Package's payload format specified (OCI-compatible?), or deliberately left open? Reusing OCI registries would be a large free ride on existing infrastructure and a large constraint on our manifest.

## 8. References
- The OCI Image and Distribution Specifications; Docker documentation on layers, caching, and BuildKit secrets; the Linux namespaces/cgroups literature that predates Docker; Sigstore/Notary supply-chain work; container-escape CVE history.
