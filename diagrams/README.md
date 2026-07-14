# Diagrams

*Status: Draft*

Diagram **source** lives in [`src/`](src/) as text (Mermaid `.mmd`, PlantUML `.puml`, or D2 `.d2`) so diagrams are diffable and reviewable. Rendered images are generated in CI; do not hand-edit rendered output.

## Conventions
- One diagram per file; file name matches the concept (`layered-architecture.mmd`).
- Reference a diagram from the spec by relative link; never paste rendered images inline into normative text.
- Sequence diagrams for protocols/flows; component diagrams for structure; state diagrams for lifecycles.

## Catalog (as authored)
| Source | Depicts | Referenced by |
|--------|---------|---------------|
| [`src/layered-architecture.mmd`](src/layered-architecture.mmd) | The Mission→Knowledge transformation chain | Book 01 §03 |
| `src/kernel-managers.mmd` | Kernel core + Managers + plugin boundary | Book 03 (*to author*) |
| `src/compile-pipeline.mmd` | High IR → … → RuntimeGraph | Books 04–05 (*to author*) |
| `src/reconciliation-loop.mmd` | observe→diff→act→emit | Book 07 (*to author*) |
