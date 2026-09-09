# Diagrams

Every diagram in this repo is authored as Mermaid, inline in the markdown file where it's
discussed — GitHub renders those natively, so that's the source of truth for context. This
folder exists for the two things inline Mermaid doesn't give you: a standalone source file per
diagram (`.mmd`) and a rendered export (`exports/*.svg`) for anyone reading the submission
offline, in a PDF export, or in a viewer that doesn't render Mermaid.

If you edit a diagram, edit it in both places: the inline block in its source `.md` file (the
one judges actually read in context) and the matching `.mmd` file here, then re-render:

```
npx -y @mermaid-js/mermaid-cli@latest -i diagrams/<name>.mmd -o diagrams/exports/<name>.svg -b transparent
```

## Index

| Diagram | Lives inline in | Source | Export |
|---|---|---|---|
| Business-lever map | [README.md](../README.md) | [business-lever-map.mmd](business-lever-map.mmd) | [exports/business-lever-map.svg](exports/business-lever-map.svg) |
| System context | [02-architecture.md](../02-architecture.md) | [system-context.mmd](system-context.mmd) | [exports/system-context.svg](exports/system-context.svg) |
| Container view | [02-architecture.md](../02-architecture.md) | [container-view.mmd](container-view.mmd) | [exports/container-view.svg](exports/container-view.svg) |
| Edge sequence (store-and-forward) | [03-edge-and-connectivity.md](../03-edge-and-connectivity.md) | [edge-sequence.mmd](edge-sequence.mmd) | [exports/edge-sequence.svg](exports/edge-sequence.svg) |
| AI capability pipeline | [05-ai-platform.md](../05-ai-platform.md) | [ai-capability-pipeline.mmd](ai-capability-pipeline.mmd) | [exports/ai-capability-pipeline.svg](exports/ai-capability-pipeline.svg) |
| Verification loop | [06-verification.md](../06-verification.md) | [verification-loop.mmd](verification-loop.mmd) | [exports/verification-loop.svg](exports/verification-loop.svg) |
| Roadmap phasing | [09-roadmap.md](../09-roadmap.md) | [roadmap-phasing.mmd](roadmap-phasing.mmd) | [exports/roadmap-phasing.svg](exports/roadmap-phasing.svg) |

Legend is the one defined in the main [README.md](../README.md#diagram-legend) — used
consistently across every diagram above, not redefined per-diagram.
