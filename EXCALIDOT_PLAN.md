# exacaliDOT — Build Plan (v0.2)

> **Status.** Planning artifact. **No implementation in this document.** The plan describes architecture, data shapes, work units, acceptance criteria, and test specifications. It does not include source code in any language.
> 
> **Changelog v0.1 → v0.2 (this revision).** Authored on personal hardware in a separate Claude Project, after the plan was moved per Section 0.3 of v0.1. The architectural substance (Parts 0–10) is unchanged. What changed:
> 
> - **Part 11** is reframed as the *contract input* to the autonomy harness rather than as a hand-execution checklist. Each work unit will become a `task` need in `docs/needs/tasks/` and is dispatched by the orchestrator, not by a human.
> - **Part 12** is augmented with mutation testing, CRAP complexity gates, output verification by a cheap second model, and the deterministic / local-LLM / frontier audit tiering. The Part 12 numeric targets become floor invariants enforced at merge time, not aspirations.
> - **Part 13** is no longer a stub. It now specifies the full operational topology: the three-layer pipeline (Strategic → Contract → Execution), the seven-specialist subagent fleet, the git-worktree parallelism model, the four routines (three of which run on the maintainer's hardware against a local LLM), the strict TDD + property-based-testing invariants, the audit gates, and the deterministic replay log. This is the section the build actually runs from.
> - **Part 14** is trimmed: open decisions resolved by harness adoption (test runner choice, branching model, CI provider topology, agent runtime, lockfile and package manager) are removed. The remaining items are genuinely strategic.
> - **Appendix D** (new) — Git worktree topology and the parallel session model.
> - **Appendix E** (new) — Mapping each Part 11 work unit to a sphinx-needs task identifier.
> 
> The IP fences in Section 0.2 and the handoff protocol in Section 0.3 are preserved unchanged. This revision was authored on personal hardware; no employer-machine artifacts other than this file are referenced.
> 
> **IP context.** This file is authored on the maintainer's employer-issued workstation. To preserve a clean separation between the maintainer's professional work and this independent side project, this document deliberately contains:
> 
> - No TypeScript, JavaScript, Python, or other source code
> - No package.json, lockfiles, or build configuration
> - No vendored or copied source from the Drawesome or Excalidraw codebases
> - No proprietary employer assets, code references, or schemas
> 
> Everything below is descriptive specification. The plan is intended to be moved to a separate Claude Project on a non-employer device, where it will be expanded into a fully autonomous agent-driven build system with significant test coverage. The expansion phase — and only the expansion phase — produces source code.
> 
> **Lineage.** This plan derives from publicly-describable architectural ideas captured in the maintainer's personal Drawesome whitepaper (Part XIV — excaliDOT Fork Plan). It deviates from that plan in three deliberate ways:
> 
> 1. The codebase walker (`fromCode` equivalent) is **kept**, not dropped — exacaliDOT must produce DOT from codebase inference as a first-class workflow, not just from Visio.
> 2. A new **Specification & Requirements Metadata Layer (SRML)** sits on top of the `ds_*` semantic namespace, allowing each node and edge to carry rich, structured spec/requirement payloads round-tripped through DOT.
> 3. The **codebase-styleguide cascade** is retained and treated as a primary input to both Visio import and codebase inference, not just to image trace.
> 
> **Target audience for this plan.** A future Claude (or Claude Code) instance, running in a separate project on a separate machine, that will expand this plan into autonomous agent task packs and build it.
> 
> **Out of scope for this document.** Pricing, hosting, distribution, domain selection, marketing, support model. Those are strategic decisions deferred to the build-plan author.

---

## Table of Contents

- [Part 0 — Naming, identity, IP fences](#part-0--naming-identity-ip-fences)
- [Part 1 — Goals and non-goals](#part-1--goals-and-non-goals)
- [Part 2 — Capability matrix vs Drawesome whitepaper Part XIV](#part-2--capability-matrix-vs-drawesome-whitepaper-part-xiv)
- [Part 3 — Architecture overview](#part-3--architecture-overview)
- [Part 4 — Data model: typed DOT, taxonomy, SRML](#part-4--data-model-typed-dot-taxonomy-srml)
- [Part 5 — The four ingest pipelines](#part-5--the-four-ingest-pipelines)
- [Part 6 — Style-guide cascade for codebases](#part-6--style-guide-cascade-for-codebases)
- [Part 7 — Diagram-standards subset](#part-7--diagram-standards-subset)
- [Part 8 — Export pipeline (DOT, SVG)](#part-8--export-pipeline-dot-svg)
- [Part 9 — CLI surface](#part-9--cli-surface)
- [Part 10 — Web app surface](#part-10--web-app-surface)
- [Part 11 — Phased build sequence (work-unit packs)](#part-11--phased-build-sequence-work-unit-packs)
- [Part 12 — Test strategy](#part-12--test-strategy)
- [Part 13 — Autonomous-agent expansion brief](#part-13--autonomous-agent-expansion-brief)
- [Part 14 — Open decisions deferred to the build-plan author](#part-14--open-decisions-deferred-to-the-build-plan-author)
- [Part 15 — Risk register](#part-15--risk-register)
- [Appendix A — Permissive-only OSS dependency policy](#appendix-a--permissive-only-oss-dependency-policy)
- [Appendix B — Glossary](#appendix-b--glossary)
- [Appendix C — File and directory layout (target shape)](#appendix-c--file-and-directory-layout-target-shape)
- [Appendix D — Git worktree topology and the parallel session model](#appendix-d--git-worktree-topology-and-the-parallel-session-model)
- [Appendix E — Mapping Part 11 work units to sphinx-needs task IDs](#appendix-e--mapping-part-11-work-units-to-sphinx-needs-task-ids)

---

## Part 0 — Naming, identity, IP fences

### 0.1 Project identity

- **Working name.** `exacaliDOT`. Deliberately distinct in spelling from the whitepaper's `excaliDOT` to mark this as a separate planning lineage owned by the maintainer's personal work. Final brand and domain are deferred (see Part 14).
- **Relationship to upstream Excalidraw.** A clean re-fork of the Excalidraw open-source codebase under its MIT license. Attribution preserved in `LICENSE` and `NOTICE` files. No code copied from any Drawesome fork.
- **Relationship to Drawesome.** None at the source-code level. Conceptual influence only — architectural ideas the maintainer has independently developed and described in personal planning artifacts. No proprietary employer code is referenced, copied, or paraphrased.
- **Repository.** A fresh git repository. No history rewrites, no submodules pointing at private repos.

### 0.2 IP fences this plan respects

Throughout the plan, the following are explicitly out of bounds:

| Out of bounds | Why |
| --- | --- |
| Source code copied from any private repo | Independence requirement |
| Schemas, names, or comments lifted verbatim from private code | Independence requirement |
| Implementation that begins on the employer-issued workstation | Provenance hygiene — actual implementation begins on the maintainer's personal hardware in a separate Claude project |
| Vendored binary assets or stencil libraries with non-permissive licenses | License hygiene (see Appendix A) |
| Patent-encumbered diagram-translation algorithms | None known in scope, but flagged in Part 15 |

### 0.3 Document handoff protocol

When this document moves to the personal-machine Claude project for the autonomous-agent expansion phase, the following must accompany it:

1. This file (`EXACALIDOT_PLAN.md`).
2. Nothing else from the employer machine.

The expansion-phase project will start from a clean Excalidraw fork and produce all code itself. This plan is a specification, not a starting codebase.

---

## Part 1 — Goals and non-goals

### 1.1 Primary goals

1. **Visio liberation.** Open `.vsdx` and `.vdx` files, extract their structural intent, and emit clean DOT plus a rendered SVG. Defense, regulated-industry, and internal-tooling audiences own large libraries of trapped Visio assets.
2. **Codebase inference.** Walk a target source tree (JavaScript/TypeScript first; Python/Go/Rust at the v1.5 boundary), infer a system-view diagram, and emit it as DOT with full taxonomy and SRML metadata.
3. **Specification-grade metadata round-trip.** Each node and edge in the emitted DOT carries a structured, validated, round-trippable payload describing its requirements, specifications, quality attributes, traceability links, and provenance. The DOT is the source of truth; SVG is a build artifact.
4. **Codebase styleguide enforcement.** A cascade of style rules (universal baseline → diagram standard → project-specific styleguide → scene overrides) governs both rendering and validation. The styleguide is declarative and lives in the target repo as a single YAML file.
5. **Standards-aware emission.** Four pilot diagram standards (C4 Container, UML Class, SysML IBD, UAF SV-1) are wired in. The active standard shapes both the inference pipeline and the validation pass.
6. **Two-channel surface.** A minimal web canvas for human review and edit; a CLI for automation and pipelines. Both link the same internal packages.

### 1.2 Explicit non-goals (v0)

- Whiteboard collaboration (no presence, no live cursors, no shared rooms).
- AI Copilot chat, AI Diagram-from-Text dialogs, AI Lasso, AI Apply-Style-Guide.
- Drawing primitives beyond what DOT can render: rectangles, ellipses, arrows, frames, text. No freedraw, no boolean ops, no warps, no shapes catalog.
- Mermaid, PlantUML, D2 import or export. DOT only.
- PNG export. SVG only.
- BYOK for non-vision providers. The Image Trace path uses one vision-capable provider; everything else is offline.
- A presentation mode, planning poker, breakout frames, emoji reactions, or any other social-canvas surface.
- A Layers panel. Subgraph clusters in DOT are the only nesting concept.
- An 8-tab style-guide editor. A read-only "active styleguide" summary panel only.
- A right-sidebar Standards / Style Guides directory. A toolbar dropdown only.
- A bridge / dev-server / canvas-API split. Web app and CLI link the same packages directly.

### 1.3 Goals that remain after the autonomous-agent expansion

This v0.2 revision commits to the following execution-side goals on top of the v0.1 product goals. These are realized by the autonomy harness specified in Part 13:

- **Hands-free agent execution under a written contract.** A system in which the orchestrator (Opus 4.7) dispatches tasks to a fleet of Sonnet 4.6 worker subagents running in isolated git worktrees, with output verification by Haiku 4.5 and audit gates by deterministic tooling and a local LLM. Human attention is reserved for five named control points (approach selection, PRD/diagram sign-off, phase-start approval, critical-audit triage, prod deploy). The target is under two hours of human involvement per week while the fleet executes full-speed.
- **Strict TDD as an execution invariant, not a habit.** The `test-writer` subagent runs before the `implementer` subagent on every task. The pre-merge check fails any PR whose production code change lacks a corresponding test change in the same PR. The only exception is a `spike/*` branch, which cannot merge to main.
- **Property-based testing by default for the categories the harness names.** Round-trip transformations (DOT, SRML, surface mirror, cascade, SVG-with-embedded-DOT), state machines, parsers, and complex business rules each require a property-based test suite alongside example-based tests. This is not optional; the contract-guardian subagent rejects PRs that omit PBT for these categories.
- **Significant test coverage at merge.** The Part 12 numeric targets — line ≥ 85%, branch ≥ 75%, mutation ≥ 60% on `srml`, `style-guide`, and `graph` — are floor invariants enforced by the auditor subagent before any release tag is cut. CRAP score above 30 on any module tagged `critical` is a CRITICAL audit finding.
- **Spec-driven development with a machine-readable contract.** The PRD lives in `prd/` as ~15 files plus `MANIFEST.yaml`; every requirement is a sphinx-needs node; every diagram in the canon links to the requirements it covers; tasks are sphinx-needs `task` nodes generated by `/ultraplan` per phase. Subagents load only the PRD sections their retrieval policy names. There is no monolithic spec document and no tribal knowledge.
- **Deterministic replay.** Every subagent invocation, every dispatch decision, and every verification result is appended to `.claude/audit-log/YYYY-MM-DD/`. When something goes wrong two weeks later, `git log` plus the audit log lets the maintainer reconstruct exactly what happened and why.

These goals are pursued from day one of the build. They are operational realities, not aspirations layered on at the end.

---

## Part 2 — Capability matrix vs Drawesome whitepaper Part XIV

This plan is a deliberate evolution of the Part XIV `excaliDOT` brief in the maintainer's personal Drawesome whitepaper. The deltas are tabulated explicitly so the build-plan author can see exactly where this plan diverges.

| Capability | Whitepaper Part XIV | exacaliDOT v0 (this plan) | Rationale for delta |
| --- | --- | --- | --- |
| Excalidraw re-fork | Re-fork upstream | Same | Identical |
| Drawing primitives | Keep 5 (rect, ellipse, arrow, frame, text) | Same | Identical |
| DOT round-trip | Keep | Same | Identical |
| Mermaid / PlantUML / D2 | Drop | Same — drop | Identical |
| Visio XML import | New (vsdx + vdx) | Same | Identical |
| Visio raster import (jpg, png) | Via Image Trace | Same | Identical |
| Visio vector import (svg) | Via Image Trace | **Plus a direct SVG-to-DOT path** when SVG carries Visio shape metadata | SVG export from Visio sometimes carries hint metadata; bypass image trace when extractable |
| Image Trace (vision LLM) | Keep, primary workflow | Keep, but secondary to Visio XML | Visio-XML path is preferred when available because it's lossless; image trace is the fallback |
| Codebase walker (`fromCode`) | **DROP** | **KEEP — primary workflow alongside Visio import** | The user's actual workflow needs DOT-from-codebase; this is the second of the two ingest sources promised by the project name implication ("exacaliDOT") |
| AI Diagram-from-Text | Drop | Same — drop | Identical |
| AI Lasso / Apply / Validate dialogs | Drop | Same — drop | Identical |
| Style-guide cascade | Keep (full port) | Keep (full port) — codebase-styleguide-flavored | Identical scope, opinionated codebase-domain rule packs included |
| Diagram standards | Keep subset (4) | Keep same subset (C4 Container, UML Class, SysML IBD, UAF SV-1) | Identical |
| Sidebar Standards directory | Replace with toolbar dropdown | Same | Identical |
| StyleguidePanel (8-tab editor) | Read-only summary only | Same | Identical |
| LayersPanel | Drop | Same — drop | Identical |
| Frame chrome + CAD title block | Keep | Keep | Useful for SVG export legends carrying the SRML summary |
| Layout pipeline | Keep | Keep, with one new stage (`srml` enrichment, see Part 4) | New stage hosts SRML metadata enrichment after layout, before style |
| Validator | Keep | Keep, **plus SRML validators (SRML-COMPLETENESS, SRML-TRACE, SRML-SCHEMA)** | New requirement: validate the spec/requirements payload, not just routing/style |
| BYOK providers | One vision provider only | Same — one vision provider only | Identical |
| CLI surface | 6 commands | Expanded — see Part 9 | New commands for codebase inference, SRML validation, traceability |
| Project memory / Codebase Diagram Jam / Terminal Lasso | Drop | Same — drop | Identical |
| Collaboration | Drop | Same — drop | Identical |

**Three deltas worth restating:**

1. **Codebase walker is back in v0.** The whitepaper Part XIV deferred this; this plan elevates it. Without it, exacaliDOT only liberates legacy Visio assets; with it, exacaliDOT also produces fresh diagrams from a current codebase, closing the loop the maintainer actually wants.
2. **SRML — Specification & Requirements Metadata Layer.** New for this plan. A structured payload attached to each node and edge that survives DOT round-trip. Designed to carry the kind of information regulated-industry teams already keep in DOORS, IBM ELM, Polarion, or in Word/Excel sidecars; the goal is to liberate that metadata into the diagram source itself.
3. **Codebase-styleguide rule packs.** The whitepaper's DUS is universal; this plan carries the universal baseline plus a default codebase-domain rule pack opinionated about source-code-derived diagrams (cluster-by-package, edge-by-import, descriptor-by-test-coverage, etc.).

---

## Part 3 — Architecture overview

### 3.1 The five layers

```
┌───────────────────────────────────────────────────────────────────┐
│  Surface          Web canvas (minimal)        CLI (single binary) │
├───────────────────────────────────────────────────────────────────┤
│  Pipeline         ingest → enrich → layout → style → validate →   │
│                   render → persist                                │
├───────────────────────────────────────────────────────────────────┤
│  Domain packages  graph    taxonomy    srml    style-guide        │
│                            standards   layout                     │
├───────────────────────────────────────────────────────────────────┤
│  Foundation       common   math    element                        │
├───────────────────────────────────────────────────────────────────┤
│  Adapters         visio-import   codebase-walker   image-trace    │
│                   dot-export     svg-export                       │
└───────────────────────────────────────────────────────────────────┘
```

**Property the architecture must hold.** Every transformation is a pure function from one typed value to another typed value, with a side report describing what changed. No hidden global state, no canvas-side singletons reading from disk. The CLI and the web surface are interchangeable consumers of the same pipeline.

### 3.2 Why this shape

- **Pure-pipeline-with-reports** is what makes property-based and round-trip tests cheap. Every stage is independently testable.
- **No bridge/SSE relay** because there is no second client to coordinate. The CLI builds the scene in-memory and emits artifacts.
- **No graph database** because all graph queries (cluster ancestry, edge classification, traceability lookup) run on the in-memory typed scene.
- **No long-lived dev server** because the canvas and CLI link the same packages and run the same code.

### 3.3 Package catalog

A thumbnail of each package, deliberately not in source-code form. Detailed responsibilities in Parts 4–8.

| Package | Responsibility |
| --- | --- |
| `common` | Shared primitive types, identifiers, error envelope, branded ID kinds |
| `math` | 2D geometry helpers, bbox, polyline, segment intersection — pure |
| `element` | Element type definitions and manipulation primitives |
| `graph` | Typed-DOT parser, emitter, AST walk, attribute namespace handling |
| `taxonomy` | The 8-type entity taxonomy (System, Component Group, Component, Operation, Sub-operation, Descriptor, Message, Message Type) plus a default StylePack |
| `srml` | Specification & Requirements Metadata Layer — schema, parser, emitter, validator, traceability resolver |
| `style-guide` | Cascade resolver, validator engine, AI-prompt renderer (used by Image Trace only), DUS baseline, codebase-rule-pack |
| `standards` | The 4 pilot diagram standards as typed registry entries |
| `layout` | The pipeline stages: ingest, enrich, layout, style, validate, render, persist |
| `visio-import` | VSDX/VDX parser, Visio AST, shape-mapping rules, fallback emitter |
| `codebase-walker` | Repo walker, language adapters, dependency graph, taxonomy inference |
| `image-trace` | Image-to-typed-DOT bridge through one vision provider |
| `dot-export` | Typed-DOT emitter with dual-attribute mode, byte-stable formatter |
| `svg-export` | SVG emitter with SRML legend block and frame title chrome |
| `cli` | Single-binary CLI exposing the pipeline as commands |
| `core` | Stripped React canvas — rect, ellipse, arrow, frame, text only |
| `web` | The web app shell — toolbar, file ops, read-only styleguide panel, SRML inspector panel |

Workspace dependency graph: `common < math < element < graph < taxonomy < srml < style-guide < standards < layout < {visio-import, codebase-walker, image-trace} < {dot-export, svg-export} < {cli, core, web}`.

### 3.4 Build target

- Single Tauri or Electron desktop binary as one option; static-site SaaS as another; both produced from the same packages. The decision is deferred to the build-plan author (Part 14). The architecture admits both.

---

## Part 4 — Data model: typed DOT, taxonomy, SRML

### 4.1 The taxonomy (unchanged, restated)

Every node and edge in the typed scene is one of eight types:

| Type | Role | Visual default (DUS Layer 1) |
| --- | --- | --- |
| System | Top-level bounded responsibility domain | Cluster, thick stroke, mint fill |
| Component Group | Logical grouping inside a system | Cluster, dashed, lavender fill |
| Component | Executable unit | Box, light-blue fill |
| Operation | Functional action on a component | Circle, amber fill |
| Sub-operation | Internal step | Diamond or point, small |
| Descriptor | Annotation classifying a node | Note shape, dashed, gray |
| Message | Edge — interface between entities | Solid arrow |
| Message Type | Edge metadata — sync/async, protocol, payload | Style modifier on the edge |

The default type assignment for an untyped import is `node → component`, `edge → message`, with an `ds_inferred=true` flag and a banner inviting confirmation.

### 4.2 The `ds_*` namespace (unchanged baseline)

Semantic attributes carried on DOT nodes and edges use the prefix `ds_`. External Graphviz renderers ignore unknown attributes; the prefix avoids collision with native rendering attributes (shape, color, penwidth, style, arrowhead). The DOT emitter writes BOTH layers:

- The semantic layer — every `ds_*` attribute the scene knows.
- The visual layer — Graphviz-native attributes computed by the style resolver.

External renderers consume the visual layer. Re-importing the DOT into exacaliDOT consumes the semantic layer; the visual layer is treated as explicit per-element overrides at the third precedence layer of the cascade.

Round-trip property: emit-then-parse-then-emit is byte-identical for both layers.

### 4.3 The SRML layer (new)

The Specification & Requirements Metadata Layer (SRML) is a structured payload attached to each node and edge of the typed scene. It is round-tripped through DOT using a small set of `srml_*` prefixed attributes, with the bulk of the payload stored in a per-element JSON-encoded blob attribute.

#### 4.3.1 SRML payload shape (conceptual schema, no code)

A node's SRML payload contains, at the top level:

- **Identity block** — stable internal ID, optional external IDs (DOORS, JIRA, GitHub Issue, RTM row), display name, kind (mirror of taxonomy type, kept for SRML-internal validation independence).
- **Specification block** — an ordered list of specification statements, each with: statement text, statement kind (functional, non-functional, constraint, assumption, derived), source citation (document name, section, line range or anchor URL), confidence tier (asserted, derived, inferred, unverified), authoring metadata (who, when, tool).
- **Requirements block** — an ordered list of requirements with: shall-statement text, requirement ID, requirement kind (capability, performance, interface, safety, security, regulatory, operational), priority, verification method (analysis, demonstration, inspection, test), verification status, parent requirement ID, child requirement IDs.
- **Quality-attribute block** — a key-value table of quality attributes with: attribute name, value, unit, target/threshold/range, measurement source. Common keys: latency, throughput, availability, MTBF, recovery time objective, recovery point objective, encryption mode, auth mode, residency.
- **Traceability block** — an ordered list of trace links with: link kind (satisfies, refines, derives-from, allocates-to, verifies, conflicts-with), target identifier, target system (DOORS, JIRA, etc.), bidirectionality flag.
- **Provenance block** — an audit trail of how the payload reached its current state: source pipeline (visio-import, codebase-walker, image-trace, manual), timestamp, content hash, prior content hash.

Edge SRML payloads share the same structure but are biased toward interface-specification content: protocol family, message schema reference, SLA, retry semantics, ordering guarantee, idempotency assertion.

#### 4.3.2 SRML wire encoding in DOT

Two complementary encodings, both must be supported on every emit:

- **Surface attributes.** The most commonly inspected fields are emitted as individual `srml_*` attributes (for example, `srml_id`, `srml_kind`, `srml_priority`). Surface attributes are human-readable when reading raw DOT and survive partial-parser tools.
- **Blob attribute.** The full payload is also emitted as a single `srml_blob` attribute holding a JSON string. The blob is the authoritative source on re-parse; surface attributes are derived. On re-emit, blob and surface are reconciled via the surface mirror rules in Section 4.3.3.

Both encodings together ensure: human readability, third-party-tool tolerance, and lossless round-trip even when third-party tools strip unknown attributes.

#### 4.3.3 Surface-mirror rules

A small fixed set of payload paths is mirrored to surface attributes. The mirror map is part of the SRML schema and versioned with it. Example mappings (illustrative, finalized in the build plan):

- `identity.id` → `srml_id`
- `identity.kind` → `srml_kind`
- `requirements[0].priority` → `srml_top_priority` (highest-priority requirement summary)
- `qualityAttributes.latency.value+unit` → `srml_latency`
- `provenance.source` → `srml_source`

The mirror rules are deterministic and pure. Round-trip property: emitting a payload, then parsing it back, then re-emitting produces identical surface attributes and identical blob content.

#### 4.3.4 SRML schema versioning

The SRML schema carries a version field. Older payloads are upgraded on parse via a chain of pure, idempotent migrations. The pipeline refuses to emit a payload tagged with an unknown future version unless `--allow-unknown-srml-version` is passed.

#### 4.3.5 SRML validators

Three validator families:

- **SRML-COMPLETENESS** — flags missing required fields per the active diagram standard. C4 Container requires `identity.kind` and at least one specification statement; UAF SV-1 requires `qualityAttributes` for messages of kind `data-flow`.
- **SRML-TRACE** — flags broken trace links: a `verifies` link whose target ID is not present in the loaded trace registry (if any), or a `derives-from` link whose target predates the source per provenance timestamps.
- **SRML-SCHEMA** — flags shape violations against the SRML zod-equivalent schema (no zod here — that is the build-plan author's library choice).

### 4.4 Element shape on canvas

Each canvas element (taxonomy node or edge) carries:

- A reference to its taxonomy type
- Its `ds_*` attribute set as a typed object
- Its SRML payload as a typed object
- Its computed visual attributes (the Graphviz-native layer)
- Its style-guide provenance (which layer of the cascade contributed each visual attribute)

Edits flow through pure setters that produce a new element value; the canvas re-renders from the new element.

---

## Part 5 — The four ingest pipelines

Four ways to populate a typed scene. Each is a pure function from input to typed scene plus a structured report. Each must produce SRML payloads on every node and edge — empty scaffolds are acceptable but not absent payloads.

### 5.1 Visio XML ingest (vsdx, vdx)

#### 5.1.1 Inputs

- `.vsdx` — a ZIP container with `visio/` directory. Pages live as XML under `visio/pages/`, masters under `visio/masters/`. Schema documented in the public MS-VSDX specification.
- `.vdx` — single XML file (Visio 2003–2010), with `<Shapes>`, `<Connects>`, `<Masters>` root elements.

#### 5.1.2 Pipeline

```
vsdx | vdx
   ↓ (unzip if needed; XML parse)
Visio AST { pages, shapes, connectors, masters, styles }
   ↓ (per-page) shape mapping using the active diagram standard's shape taxonomy
Typed-graph fragment { nodes, edges, clusters, ds_* attributes }
   ↓ (SRML scaffolding pass — empty payloads with provenance.source = "visio-import")
Typed scene
   ↓ (active-standard normalizer — see 7.3)
Validated typed scene
```

#### 5.1.3 Shape-mapping rules (initial cut)

A configurable rule table maps Visio shape master IDs to taxonomy types and `ds_*` attributes:

- Visio Container shape → DOT subgraph cluster (cluster taxonomy type follows nesting depth: outermost = System, mid = Component Group, leaf = Component)
- Visio Process shape (rounded rect) → `ds_type=component`
- Visio Cylinder → `ds_type=component`, `ds_classification=data-store`
- Visio Person / Actor → `ds_type=descriptor`, `ds_classification=person`
- Visio plain rectangle → `ds_type=component`
- Visio ellipse → `ds_type=component` (with a warning if the active standard forbids ellipses, e.g. UAF SV-1)
- Connector with arrowhead → `ds_type=message`, `ds_protocol=uses` (the active standard refines)

#### 5.1.4 Fallback for unknown shapes

When a Visio shape doesn't match any rule, emit it as `ds_type=descriptor`, `ds_classification=visio-imported`, `shape=rectangle`, and the original Visio master name as the label. The scene tags itself with a counter `customData.import.unmappedShapeCount`. The user can then re-tag manually or trigger the Image Trace pipeline on the original page raster.

#### 5.1.5 SRML population at this stage

A scaffold payload is created for each node and edge with:

- `identity.id` from a stable hash of the Visio master ID + position
- `identity.kind` mirroring the taxonomy type
- `provenance.source = "visio-import"`
- `provenance.timestamp` of the ingest
- All other blocks empty arrays / empty objects

### 5.2 Visio raster/vector ingest (jpg, png, svg)

#### 5.2.1 Two paths

- **Raster path (jpg, png) and SVG-without-metadata.** Routed to the Image Trace pipeline (5.4).
- **SVG-with-metadata.** Some Visio export paths emit SVG with `data-visio-master` or similar attributes. When the SVG carries any reliable structural hint, a direct SVG-to-typed-graph adapter is used and Image Trace is skipped.

#### 5.2.2 SVG metadata sniffer

A small adapter inspects an incoming SVG for known metadata signals (Visio export hints, Lucidchart hints, draw.io hints — all without copying their tooling). If the score exceeds a threshold, the direct path is used. Otherwise the SVG is rasterized and routed to Image Trace.

### 5.3 Codebase ingest (the new piece kept in v0)

#### 5.3.1 Inputs

A file-system path to a source-code repository plus an optional `.exacalidot/style-guide.yaml` and `.exacalidot/srml-policy.yaml` at the repo root.

#### 5.3.2 Language scope

- v0: TypeScript and JavaScript (npm workspaces, pnpm workspaces, single-package projects, monorepos).
- v0.5: Python (pyproject, requirements, src layout).
- v1: Go and Rust.

The walker dispatches by detecting language manifests.

#### 5.3.3 Pipeline

```
repo path
   ↓ (manifest detection; language adapter selection)
Repo digest { packages, modules, files, public symbols }
   ↓ (dependency graph extraction — imports, dynamic require, type imports, test → src links)
Code graph { nodes = modules/packages, edges = imports }
   ↓ (taxonomy inference per active style-guide cluster rules)
Typed-graph fragment { System = repo, Component Groups = packages, Components = modules, Operations = exported functions, Messages = imports }
   ↓ (SRML enrichment from source signals — see 5.3.5)
Typed scene
```

#### 5.3.4 Taxonomy inference rules

Default rules (overridable by the codebase-rule-pack in the active style guide):

- Top-level repo → System.
- Each workspace package → Component Group.
- Each non-test module → Component.
- Each exported public symbol (function, class) → Operation.
- Each cross-package import → Message edge from importing component to imported component.
- Each test file → Descriptor with `ds_classification=test-coverage`, attached to the module under test.
- Each TODO/FIXME comment with an owner tag → Descriptor with `ds_classification=open-issue`.

#### 5.3.5 SRML enrichment from code

The walker harvests metadata from source signals:

- **Specification block.** Module/file header comments with structured tags (`@spec`, `@requirement`, `@derives-from`) are parsed into specification statements. Free-text headers are captured verbatim under `confidence=unverified`.
- **Requirements block.** Files matching `**/*.requirement.md` or comments with explicit `@requirement` tags become requirement entries with verification method derived from the comment.
- **Quality-attribute block.** JSDoc/TSDoc fields like `@latency`, `@throughput`, `@idempotent` populate quality attributes.
- **Traceability block.** Conventional-commit footers with `Closes #N`, `Fixes JIRA-123`, `Refs RTM-456` are parsed into trace links pointing at GitHub Issues, JIRA, or a generic requirements tracker. The link target system is inferred from the URL pattern in the repo's `.exacalidot/srml-policy.yaml`.
- **Provenance block.** Source path, file SHA at walk time, walker version.

#### 5.3.6 Configurable depth

A `depth` parameter limits walking past N package levels. Useful for large monorepos.

#### 5.3.7 Output

A typed scene with SRML payloads on every element, ready to flow into the layout pipeline. Same shape as the Visio import output.

### 5.4 Image Trace ingest (the existing pattern, ported)

#### 5.4.1 Inputs

A raster image (jpg, png) or SVG-without-metadata.

#### 5.4.2 Pipeline

```
image
   ↓ (rasterization if SVG)
single-pass vision-LLM call with the active standard's extraction prompt and the
   active style-guide's AI contract injected as system context
   ↓ (LLM returns structured typed-DOT response)
Typed-graph fragment
   ↓ (SRML scaffolding — provenance.source = "image-trace", confidence=inferred)
Typed scene
```

#### 5.4.3 SRML population at this stage

Every node and edge receives a scaffold SRML payload with:

- `provenance.source = "image-trace"`
- `provenance.timestamp` of the trace
- All extracted text labels mapped to the specification block as a single `confidence=inferred` statement with citation pointing at the source image filename
- All other blocks empty

A subsequent author-intent pass (optional) prompts the LLM to attempt deeper SRML population given the full image. Off by default.

#### 5.4.4 Provider abstraction

The image-trace adapter speaks to one vision-capable provider. Provider selection is a build-plan-author decision (Part 14). The adapter exposes a single typed function from `(image, activeStandard, activeStyleGuide) → TypedScene`. Swapping providers is a one-file change.

---

## Part 6 — Style-guide cascade for codebases

### 6.1 Cascade shape (unchanged)

```
Universal Style Guide (USG, exacaliDOT-flavored DUS)
       ↓ overrides
Standard.<id>
       ↓ overrides
Project StylePack from .exacalidot/style-guide.yaml
       ↓ overrides
Scene-Level Overrides
       ↓
ActiveRules
```

### 6.2 USG content (the v0 baseline)

The Universal Style Guide ships as a single declarative YAML document. Sections:

- `routing` — manhattan-only, perpendicular entry/exit, sharp-corner allowance, port distribution, edge-through-node prohibition.
- `style` — typography, palette ramps, stroke conventions, shape catalog per taxonomy type.
- `layout` — paper size, margins, gutter spacing, cluster aspect-ratio targets.
- `frame` — title-block content fields (TITLE, REV, DRAWN BY, DATE, PROJECT, DRAWING NO, SCALE, SHEET, FORMAT), placement, font.
- `validators` — the rule catalog with severity levels (DUS-ROUTING-01 through DUS-ROUTING-05, DUS-STYLE-01 through DUS-STYLE-02, plus the new SRML validators from 4.3.5).
- `ai-contract` — the prompt-side rendering rules used by Image Trace.

### 6.3 Codebase-domain rule pack (new flavor)

Added on top of USG, opinionated for source-code-derived diagrams:

- **Cluster-by-package.** Workspace packages always become Component Group clusters; their styling is inherited from the System color ramp shifted one tier.
- **Edge-by-import-strength.** Import edges are stylized by frequency: hot edges (imported by N modules) get thicker stroke; rarely-used edges get dashed.
- **Descriptor-by-test-coverage.** Components with no associated test descriptor are flagged with a `low-coverage` warning halo.
- **Edge-density routing budget.** Components with more than K incoming or outgoing edges trigger a warning; the visualization may be unreadable and the user is invited to introduce a Component Group split.
- **Public-API surface emphasis.** Operations corresponding to exported symbols get a heavier shape weight than internal-only operations.

### 6.4 Project StylePack

A single file at `.exacalidot/style-guide.yaml` in the target project root. Same schema as USG. Any field not specified inherits from USG and the active standard. Includes optional `srml-policy` block governing required SRML fields per taxonomy type.

### 6.5 Cascade resolver behavior

Pure function: `resolveCascade(USG, activeStandard, projectStylePack, sceneOverrides) → ActiveRules`. Provenance per leaf path is preserved so the read-only summary panel can show "this rule comes from the project StylePack overriding the standard's value."

### 6.6 Validator engine

Operates on a typed scene and the resolved active rules. Produces a list of violations: `{ code, severity, elementId, rulePath, message, suggestion? }`. Severity levels: error, warning, info.

CLI surface (Part 9) exposes `validate`, `violations`, `active`, `show <id>`, `list`.

### 6.7 AI contract renderer

A pure function `renderForPrompt(active) → string` produces the Markdown-or-JSON system-prompt block that Image Trace injects. Ensures the LLM's emission matches the cascade.

---

## Part 7 — Diagram-standards subset

### 7.1 Standards in v0

Four pilots, each shipping the same typed shape:

- **C4 Container** — Containers, Components, Persons, External Systems, Containers.
- **UML Class** — Classes, Interfaces, Abstract Classes, generalization/realization/dependency edges.
- **SysML IBD** — Internal Block Diagram, blocks, ports, connectors typed with item-flows.
- **UAF SV-1** — Systems View 1 from the Unified Architecture Framework — Resources, Performers, Activity Flows.

The four are deliberately codebase- and systems-engineering-oriented. BPMN, ERD, and other domain standards are not in v0.

### 7.2 The DiagramStandard shape (no code)

Each standard is a typed registry entry containing:

- `id` (kebab-case slug)
- `displayName`, `description`, `version`
- `taxonomy` — the per-standard mapping from taxonomy types to allowed shapes (e.g. UAF SV-1 forbids ellipses for resources)
- `extractionPrompt` — the LLM prompt fragment used by Image Trace
- `styleOverrides` — partial USG override, applied at cascade layer 2
- `validationChecks` — additional validators specific to the standard
- `srmlRequirements` — fields each taxonomy type must populate to be standard-conformant
- `exemplar` — a reference DOT diagram demonstrating a fully-conformant scene

### 7.3 Active-standard normalizer

A pipeline stage that runs after the ingest stages. Given an active standard and a typed scene, it:

- Renames `ds_*` attributes to standard-canonical names where applicable
- Applies the standard's style overrides via the cascade
- Surfaces SRML-completeness warnings as advisories
- Does not silently add or delete elements

### 7.4 Surfacing the active standard

- Toolbar dropdown: `Standard: [None | C4 Container | UML Class | SysML IBD | UAF SV-1]`
- CLI: `--standard <id>` flag on every ingest command
- Bare scenes (Standard = None) get USG and the codebase-rule-pack only; useful for free-form codebase exploration

---

## Part 8 — Export pipeline (DOT, SVG)

### 8.1 DOT export

#### 8.1.1 Format

Standard Graphviz DOT, with the exacaliDOT dialect:

- Header comment block carrying scene metadata and SRML schema version
- Graph attributes from the active style guide
- Node statements with both layers of attributes (`ds_*` and visual)
- Edge statements similarly dual-attributed
- Subgraph clusters preserving containment from taxonomy
- Footer comment block carrying the exact resolved-active-rules hash so re-parsing can confirm the scene came from the same cascade

#### 8.1.2 SRML emission

For each node and edge, both encodings (4.3.2) are emitted: surface `srml_*` attributes for the mirrored fields and a single `srml_blob` attribute holding the full payload as a JSON string.

#### 8.1.3 Stability rules

- Attributes are emitted in a deterministic order (alphabetical, namespace-grouped).
- Numeric values are formatted with a fixed number of significant digits.
- Whitespace is normalized.
- Identifiers are sorted within their containing scope.

These rules make `emit ∘ parse ∘ emit` byte-identical, which is the round-trip test.

### 8.2 SVG export

#### 8.2.1 Renderer

Two implementations possible:

- **Native renderer.** A small purpose-built SVG renderer that reads the typed scene and emits SVG with a known style sheet. Predictable, no Graphviz dependency at render time.
- **Graphviz-via-WASM renderer.** Pipe the DOT through a WASM build of Graphviz and emit its SVG. Produces classic Graphviz layouts.

V0 ships the native renderer. The Graphviz-via-WASM renderer is a v1 addition behind a flag.

#### 8.2.2 SVG content

- The diagram itself
- A frame chrome (CAD title block) when frames are enabled
- An optional SRML legend block — one column per element, listing top-priority requirements and quality attributes — toggleable via export flag

#### 8.2.3 Embedded source

The SVG carries the originating DOT as an XML comment near the root, so the SVG file alone is enough to reconstruct the diagram in exacaliDOT.

#### 8.2.4 No PNG export in v0

Users who need PNG can rasterize the SVG with their tool of choice; not exacaliDOT's job.

---

## Part 9 — CLI surface

A single binary, presumably named `exacalidot`. The full v0 command set:

| Command | Purpose |
| --- | --- |
| `exacalidot import-visio <file>` | VSDX or VDX → typed DOT to stdout |
| `exacalidot import-visio <file> --standard <id>` | Same, normalized to a standard |
| `exacalidot import-visio <file> --srml-policy <path>` | Same, with custom SRML policy |
| `exacalidot codebase <path>` | Walk a repo → typed DOT |
| `exacalidot codebase <path> --standard <id>` | Same, normalized |
| `exacalidot codebase <path> --depth <n>` | Limit walk depth |
| `exacalidot codebase <path> --languages <list>` | Restrict to listed adapters |
| `exacalidot image-trace <image> --standard <id>` | Vision-LLM → typed DOT |
| `exacalidot export dot` | Current scene → DOT to stdout |
| `exacalidot export dot --output <file>` | Same, to file |
| `exacalidot export svg --output <file>` | Render SVG |
| `exacalidot export svg --legend` | Render SVG with SRML legend block |
| `exacalidot style-guide list` | All validators with severity codes |
| `exacalidot style-guide show <id>` | One rule + its current value at the resolved cascade |
| `exacalidot style-guide active [filter]` | Flat provenance — path → source layer |
| `exacalidot style-guide validate` | Run all validators against the current scene |
| `exacalidot style-guide violations` | Full violation list |
| `exacalidot standards list` | All registry entries |
| `exacalidot standards show <id>` | One standard with full content |
| `exacalidot srml validate` | Run SRML-COMPLETENESS, SRML-TRACE, SRML-SCHEMA |
| `exacalidot srml inspect <element-id>` | Print the SRML payload for one element |
| `exacalidot srml trace <requirement-id>` | Print the trace forest rooted at one requirement |
| `exacalidot srml export-rtm --output <file>` | Emit a Requirements Traceability Matrix as CSV |
| `exacalidot pipeline run <ingest-cmd> --to <export-cmd>` | Full end-to-end without intermediate files |
| `exacalidot --help` | Help |
| `exacalidot --version` | Version |

CLI commands all return structured exit codes and emit machine-readable reports on `--report json`. This is critical for the autonomous-agent expansion phase (Part 13).

---

## Part 10 — Web app surface

A minimal stripped-down Excalidraw shell. Only the surfaces named below ship in v0.

### 10.1 Layout

```
┌─────────────────────────────────────────────────────────────────┐
│ Top toolbar                                                     │
│  [File ▾] [Standard ▾] [Validate] [Export ▾]                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                         Canvas                                  │
│                  (rect, ellipse, arrow, frame, text)            │
│                                                                 │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│ Status bar — active standard, cascade-summary glyph, validation │
│              violation count, hover element ID                  │
└─────────────────────────────────────────────────────────────────┘

Right-side floating panels (toggleable):
  - SRML Inspector (pinned by default; shows selected element's payload)
  - Active Style Guide summary (read-only)
  - Violations list (toggle on validate)
```

### 10.2 File menu

- New scene
- Open `.dot` file
- Open Visio file (`.vsdx`, `.vdx`)
- Open Visio image (`.jpg`, `.png`, `.svg`)
- Open repo folder (codebase walk; opt-in dialog confirming the path)
- Save as `.dot`
- Export as `.svg`
- Recent files

No autosave to cloud. No share link. No collaboration menu.

### 10.3 SRML Inspector panel

The most important new surface vs the whitepaper Part XIV plan. When an element is selected, the panel shows:

- Identity block (read-only at the system fields, editable at display name)
- Specification block as an ordered editable list with kind, text, source, confidence
- Requirements block as an ordered editable list with all SRML requirement fields
- Quality-attribute block as an editable key-value table
- Traceability block as an editable list of trace links with a chip per link
- Provenance block (read-only)

Editing is committed via undo-aware state mutations. The panel's edits invalidate cached visuals and trigger a re-render.

### 10.4 Validate button behavior

Invokes the same pipeline as `exacalidot style-guide validate` and `exacalidot srml validate`. Surfaces violations in the right-side Violations panel, with click-to-jump-to-element navigation.

### 10.5 What does NOT ship in the web app v0

- No login, no account, no billing.
- No collaboration cursors or presence.
- No comments, no reactions.
- No presentation mode.
- No Layers panel.
- No 8-tab styleguide editor (only the read-only summary).
- No right-sidebar Standards directory.
- No AI Copilot chat carve-out.
- No drawing tools beyond the five primitives.

---

## Part 11 — Phased build sequence (work-unit packs)

Each phase is a pack of work units. A work unit has: name, scope, inputs, outputs, acceptance criteria, dependencies, test specifications, rollback plan. **No source code in this section** — the work unit is a contract, not a recipe.

### How Part 11 is consumed by the harness

Under the autonomy harness specified in Part 13, each work unit below maps to exactly one sphinx-needs `task` node in `docs/needs/tasks/phase-N/`. The mapping is enumerated in Appendix E. The orchestrator generates these `task` files at the start of each phase by running `/ultraplan` against the requirements in `prd/03-features/`, the dependency graph in `prd/05-dependencies.md`, and the work-unit table below; the human approves the generated DAG at the `phase_start` control point. Each `task` carries:

- The Part 11 acceptance criteria, restated as machine-checkable assertions
- The `:implements:` link to the FRs/NFRs it satisfies
- The `:tests:` link to the tests the `test-writer` subagent must produce first
- The `:depends_on:` link to upstream tasks (drawn from the dependency column below)
- The bounded workspace (the `packages/` directory the worker is allowed to touch)
- The artifact contract (files the task must produce; files it must not produce)

A worker picks up tasks whose `:depends_on:` chain is fully `status: done`. The `contract-probe` subagent writes a 200-word "what I'm about to do" note before any code is written; the orchestrator approves or kicks back. The `test-writer` then produces failing tests; the `implementer` writes code to pass them; the Haiku 4.5 verifier checks the output against the acceptance criteria; the auditor runs the in-flight audit set; the `contract-guardian` reviews any PR that touches `prd/`, `docs/needs/`, `docs/diagrams/`, or `AUTONOMY-MANIFEST.yaml`; `/ultrareview` runs before merge.

The work-unit tables below stay as written. The columns are the contract; the harness handles the rest.

### Phase 0 — Re-fork and brand

| Work unit | 0.1 Re-fork Excalidraw |
| --- | --- |
| Scope | Fresh fork from upstream Excalidraw at a pinned tag. Rename project, update LICENSE/NOTICE, rebrand visible asset files, strip unused features at the source-tree level. |
| Acceptance | Repo builds, runs, displays the canvas; no Excalidraw branding visible; LICENSE preserves MIT attribution; NOTICE adds the exacaliDOT attribution. |
| Tests | Smoke: `pnpm dev` (or chosen tooling) starts; canvas renders; one rectangle can be drawn. |
| Dependencies | None. |
| Rollback | Discard repo. |

| Work unit | 0.2 Strip non-needed feature modules |
| --- | --- |
| Scope | Remove modules from upstream Excalidraw that exacaliDOT v0 does not need: collaboration, library catalog, image generation, embeds, freedraw beyond the line tool. Keep the canvas core, rectangle/ellipse/arrow/frame/text element types, undo, persistence to local storage. |
| Acceptance | Canvas renders five primitive types only; bundle size reduced by an agreed margin; no dead-code-eliminated import errors. |
| Tests | Smoke; bundle-size check; lint; typecheck. |
| Dependencies | 0.1. |
| Rollback | Restore stripped modules from upstream. |

### Phase 1 — Foundation packages

| Work unit | 1.1 `common` package |
| --- | --- |
| Scope | Shared primitive types, branded ID kinds, error envelope, result type. |
| Acceptance | All other packages can `import { Id, Result, makeId } from "@exacalidot/common"`. |
| Tests | Unit tests for ID minting, result type combinators. |
| Dependencies | 0.  |

| Work unit | 1.2 `math` package |
| --- | --- |
| Scope | 2D geometry helpers — bbox union, polyline intersection, segment-vs-segment, point-in-polygon. Pure. |
| Acceptance | Property tests on bbox associativity and segment-intersection symmetry. |
| Tests | Unit + property. |
| Dependencies | 1.1. |

| Work unit | 1.3 `element` package |
| --- | --- |
| Scope | Element type union for rect, ellipse, arrow, frame, text. Constructors, transformers (move, resize), serializers. Pure. |
| Acceptance | Element types match canvas expectations. Round-trip serialize-deserialize identity. |
| Tests | Unit + round-trip property. |
| Dependencies | 1.1, 1.2. |

### Phase 2 — Domain packages (graph, taxonomy, srml, style-guide, standards)

| Work unit | 2.1 `graph` package — DOT parse/emit |
| --- | --- |
| Scope | DOT parser → typed AST; AST → typed graph; emitter from typed graph back to DOT with deterministic formatting. Handle subgraph clusters. Handle dual-attribute (`ds_*` + visual) emission. |
| Acceptance | Parse a corpus of 30 known DOT documents (built fresh, not copied) and re-emit byte-identical. |
| Tests | Unit (per AST node kind), integration (round-trip on the corpus), property (random valid DOT generators). |
| Dependencies | 1.1, 1.3. |

| Work unit | 2.2 `taxonomy` package |
| --- | --- |
| Scope | The 8-type taxonomy as a typed enum-equivalent, default StylePack mapping types to visual defaults, type-inference helpers. |
| Acceptance | Each taxonomy type has a default visual; defaults are not mutated by callers. |
| Tests | Unit on default lookup; property test that every taxonomy type has a default. |
| Dependencies | 1.1. |

| Work unit | 2.3 `srml` package — schema |
| --- | --- |
| Scope | SRML payload schema for nodes and edges (Section 4.3). Schema validation against arbitrary JSON. Migration chain for schema versions. |
| Acceptance | Schema validates a hand-crafted corpus of 20 valid payloads and rejects 20 invalid payloads with helpful error messages. Migrations chain idempotently. |
| Tests | Unit (per schema rule), integration (corpus), property (invariant: round-trip through serialization preserves payload identity). |
| Dependencies | 1.1. |

| Work unit | 2.4 `srml` package — surface mirror & blob encoding |
| --- | --- |
| Scope | The mirror map (Section 4.3.3). Pure functions `payload → surfaceAttrs + blobAttr` and `surfaceAttrs + blobAttr → payload`. |
| Acceptance | Round-trip property holds: emit-then-parse-then-emit produces identical surface and identical blob. |
| Tests | Property (invariant: round-trip is identity). Unit (per mirror rule). Integration (combined with `graph`). |
| Dependencies | 2.3, 2.1. |

| Work unit | 2.5 `srml` package — validators |
| --- | --- |
| Scope | SRML-COMPLETENESS, SRML-TRACE, SRML-SCHEMA. Each is a pure function from typed scene + active standard + active style guide → violations. |
| Acceptance | Synthetic scenes covering each violation kind produce expected reports. |
| Tests | Unit per validator. Integration on a few hand-built scenes. |
| Dependencies | 2.3, 2.4. |

| Work unit | 2.6 `style-guide` package — schema, USG baseline, cascade resolver |
| --- | --- |
| Scope | YAML schema for style guides; the USG-baseline YAML document; the cascade resolver as a pure function with provenance tracking. |
| Acceptance | Resolve a 4-layer cascade, query any leaf path, get the value plus the source layer. |
| Tests | Unit (resolver semantics), integration (USG + standard + project + scene), property (commutativity-of-no-ops). |
| Dependencies | 1.1. |

| Work unit | 2.7 `style-guide` package — validators (DUS-ROUTING, DUS-STYLE, DUS-LAYERS) |
| --- | --- |
| Scope | The 8 USG validators from Section 6.6. |
| Acceptance | Synthetic scenes covering each violation kind produce expected reports. |
| Tests | Unit per validator. Integration on hand-built scenes. |
| Dependencies | 2.6, 1.3. |

| Work unit | 2.8 `style-guide` package — codebase-domain rule pack |
| --- | --- |
| Scope | The five codebase-flavored rules from Section 6.3. |
| Acceptance | Codebase rule pack engages only when the codebase walker is the ingest source, or when explicitly enabled by `--rule-pack codebase`. |
| Tests | Unit per rule. Integration: codebase walk produces violations consistent with the rule pack. |
| Dependencies | 2.6. |

| Work unit | 2.9 `standards` package — 4 pilots |
| --- | --- |
| Scope | C4 Container, UML Class, SysML IBD, UAF SV-1 as typed registry entries. Each entry includes its `srmlRequirements`. |
| Acceptance | Registry exposes 4 entries; each lookup returns a complete entry. |
| Tests | Unit per standard. |
| Dependencies | 2.6, 2.3. |

### Phase 3 — Layout pipeline

| Work unit | 3.1 `layout` — ingest stage |
| --- | --- |
| Scope | Adapt typed-graph fragments from any of the four ingest pipelines into a normalized typed scene. |
| Acceptance | Inputs from visio-import, codebase-walker, image-trace, and dot-import all produce identically-shaped typed scenes. |
| Tests | Integration: synthetic fragments from each source. |
| Dependencies | 2.1, 2.2. |

| Work unit | 3.2 `layout` — frame stage |
| --- | --- |
| Scope | Apply the active style guide's frame rules: paper size, margins, CAD title block. |
| Acceptance | Scene gets a frame element with title-block fields populated. |
| Tests | Unit (frame placement), integration (multi-page synthesis). |
| Dependencies | 3.1, 2.6. |

| Work unit | 3.3 `layout` — layout stage (Graphviz JSON consumer + native router) |
| --- | --- |
| Scope | Either consume Graphviz-WASM JSON output to position elements, or implement a native hierarchical layout. The build-plan author chooses one. Both must be Manhattan-routing capable. |
| Acceptance | Synthetic graphs with 50 nodes lay out in under a budget time, no overlaps, all edges routed Manhattan with perpendicular entry/exit. |
| Tests | Unit per algorithm helper. Integration on a graph corpus. Performance budgets. |
| Dependencies | 3.1, 2.7. |

| Work unit | 3.4 `layout` — style stage |
| --- | --- |
| Scope | Apply the resolved active rules to compute Graphviz-native visual attributes per element. Three-layer precedence (defaults → cascade → explicit per-element). |
| Acceptance | Element visuals match expected values per layer; provenance reflects the source layer per attribute. |
| Tests | Unit on three-layer precedence. Integration on a styled scene. |
| Dependencies | 3.3, 2.6. |

| Work unit | 3.5 `layout` — srml-enrichment stage |
| --- | --- |
| Scope | Run SRML scaffolding/upgrade if any element lacks a payload or has an old-version payload. |
| Acceptance | After this stage, every element has an up-to-date SRML payload with `provenance.source` set. |
| Tests | Integration on scenes with mixed-state SRML. |
| Dependencies | 3.4, 2.3, 2.4. |

| Work unit | 3.6 `layout` — validate stage |
| --- | --- |
| Scope | Run all USG, codebase-rule-pack, standard-specific, and SRML validators. Collect violations. |
| Acceptance | Violation list matches expectation on a hand-built scene with known violations. |
| Tests | Integration. |
| Dependencies | 3.5, 2.7, 2.5. |

| Work unit | 3.7 `layout` — render stage (typed scene → canvas elements) |
| --- | --- |
| Scope | Project the typed scene into the Excalidraw element types for rendering on the canvas. |
| Acceptance | Synthetic typed scenes render correctly on the canvas. |
| Tests | Snapshot tests on small scenes. |
| Dependencies | 3.4, 1.3. |

| Work unit | 3.8 `layout` — persist stage (typed scene → DOT, → SVG) |
| --- | --- |
| Scope | Two persistence outputs, both pure: emit-as-DOT and emit-as-SVG. |
| Acceptance | DOT round-trip byte-identical; SVG renders the same scene visually. |
| Tests | Round-trip property on DOT; visual diff on SVG. |
| Dependencies | 3.4, 2.1, 2.4. |

### Phase 4 — Adapters

| Work unit | 4.1 `visio-import` — VSDX parser |
| --- | --- |
| Scope | Unzip a VSDX file, parse the contained XMLs, build the Visio AST. |
| Acceptance | Parse a corpus of 5 hand-crafted minimal VSDX files (built fresh in this phase, not copied). |
| Tests | Integration on the corpus. Property: every Visio element kind that appears in the corpus is recognized. |
| Dependencies | 1.1. |

| Work unit | 4.2 `visio-import` — VDX parser |
| --- | --- |
| Scope | Parse legacy single-file XML to the same Visio AST. |
| Acceptance | Parse a corpus of 5 hand-crafted minimal VDX files. |
| Tests | Integration. |
| Dependencies | 4.1 (shared AST). |

| Work unit | 4.3 `visio-import` — shape-mapping rules |
| --- | --- |
| Scope | The Section 5.1.3 rules. Configurable. Includes the unmapped-shape fallback. |
| Acceptance | Each rule fires correctly on a synthetic Visio AST. |
| Tests | Unit per rule. Integration. |
| Dependencies | 4.1, 2.2. |

| Work unit | 4.4 `visio-import` — SRML scaffolding pass |
| --- | --- |
| Scope | Populate empty SRML payloads with provenance (Section 5.1.5). |
| Acceptance | Every emitted node and edge has a payload with `provenance.source = "visio-import"`. |
| Tests | Integration. |
| Dependencies | 4.3, 2.3. |

| Work unit | 4.5 `codebase-walker` — TypeScript adapter |
| --- | --- |
| Scope | Walk a TS/JS workspace, build the repo digest. |
| Acceptance | Walk a synthetic repo with 5 packages and 30 files, get the expected digest. |
| Tests | Integration on a synthetic repo built fresh in this phase. |
| Dependencies | 1.1. |

| Work unit | 4.6 `codebase-walker` — taxonomy inference |
| --- | --- |
| Scope | The Section 5.3.4 rules. Output is a typed graph fragment. |
| Acceptance | Synthetic repo produces expected taxonomy. |
| Tests | Integration. |
| Dependencies | 4.5, 2.2. |

| Work unit | 4.7 `codebase-walker` — SRML enrichment from code |
| --- | --- |
| Scope | The Section 5.3.5 enrichment. |
| Acceptance | A repo with seeded `@spec`, `@requirement`, `@latency`, conventional-commit footers, and TODOs produces a fully-populated SRML payload graph. |
| Tests | Integration on a synthetic repo with all enrichment signals seeded. |
| Dependencies | 4.6, 2.3. |

| Work unit | 4.8 `image-trace` — vision-LLM adapter |
| --- | --- |
| Scope | Single function from `(image, activeStandard, activeStyleGuide) → typed scene fragment`. |
| Acceptance | Adapter speaks to one provider; adapter signature is stable across provider swaps. |
| Tests | Integration with a recorded provider response (no live calls in CI). |
| Dependencies | 2.6, 2.9. |

### Phase 5 — Export and CLI

| Work unit | 5.1 `dot-export` package |
| --- | --- |
| Scope | Stable formatter for typed-DOT with dual-attribute emission and SRML wire encoding. |
| Acceptance | Round-trip property holds across the corpus from 2.1 plus all of Phase 4's outputs. |
| Tests | Round-trip property. |
| Dependencies | 2.1, 2.4, 3.8. |

| Work unit | 5.2 `svg-export` package — native renderer |
| --- | --- |
| Scope | Render a typed scene to SVG with frame chrome and optional SRML legend. |
| Acceptance | Sample scenes render to deterministic SVG (visual diff against snapshots). |
| Tests | Snapshot. |
| Dependencies | 3.4, 3.7. |

| Work unit | 5.3 `cli` — Commander or equivalent skeleton |
| --- | --- |
| Scope | Single-binary CLI with the command set in Part 9. JSON-report mode. |
| Acceptance | All Part 9 commands return correct exit codes and emit valid JSON reports. |
| Tests | Integration per command. End-to-end on the pipeline. |
| Dependencies | 4.1–4.8, 5.1, 5.2. |

### Phase 6 — Web app surfaces

| Work unit | 6.1 Top toolbar |
| --- | --- |
| Scope | File menu, Standard dropdown, Validate button, Export menu. |
| Acceptance | All menu items invoke the right pipeline calls; keyboard accelerators work. |
| Tests | UI integration. |
| Dependencies | Phase 5. |

| Work unit | 6.2 SRML Inspector panel |
| --- | --- |
| Scope | The Section 10.3 panel. Edits commit through undo. |
| Acceptance | Editing every block of every element kind persists through a save/reload. |
| Tests | UI integration. Property: edits compose with undo/redo correctly. |
| Dependencies | 6.1. |

| Work unit | 6.3 Active Style Guide summary panel |
| --- | --- |
| Scope | Read-only panel that lists the resolved cascade with provenance per leaf. |
| Acceptance | Provenance reflects which layer contributed each leaf. |
| Tests | UI integration. |
| Dependencies | 6.1. |

| Work unit | 6.4 Violations panel |
| --- | --- |
| Scope | List violations with click-to-jump-to-element. |
| Acceptance | Click navigates the canvas to the offending element and selects it. |
| Tests | UI integration. |
| Dependencies | 6.1. |

### Phase 7 — End-to-end smoke and packaging

| Work unit | 7.1 E2E corpus |
| --- | --- |
| Scope | A test corpus with: 3 hand-built VSDX files, 3 hand-built VDX files, 1 hand-built TS monorepo, 3 hand-built whiteboard photographs (or AI-generated stand-ins), 5 reference DOT documents. |
| Acceptance | Each artifact runs through the full pipeline and produces the expected output. |
| Tests | E2E. |
| Dependencies | All prior phases. |

| Work unit | 7.2 Packaging — one-of (web static site, Tauri desktop, Electron desktop) |
| --- | --- |
| Scope | Build-plan author picks one; packaging recipe added. |
| Acceptance | A user can install or open the artifact and run a Visio import end-to-end. |
| Tests | Manual smoke + a CI build job. |
| Dependencies | 7.1. |

| Work unit | 7.3 Documentation — README, sample workflow walkthrough |
| --- | --- |
| Scope | Five documents only: README, Visio-import quickstart, codebase-walk quickstart, image-trace quickstart with API-key setup, SRML primer. |
| Acceptance | Each quickstart can be followed by a fresh user without external help. |
| Tests | Manual review. |
| Dependencies | 7.1. |

### Phase budget

The eleven Drawesome whitepaper Part XIV plan estimated three weeks of focused work for an excaliDOT v0 with no codebase walker and no SRML. With the codebase walker added back and SRML added on top, this plan estimates **five to seven weeks of focused single-engineer work** for the v0, doubled to **ten to fourteen weeks for a polished v1**. The autonomous-agent expansion is expected to compress wall-clock time substantially while raising quality through aggressive parallel test execution.

---

## Part 12 — Test strategy

### 12.1 Test taxonomy

Five test kinds, each with a coverage target:

| Kind | Coverage target | Tooling expectation |
| --- | --- | --- |
| Unit | Every exported function in every package | Fast runner, watch mode |
| Property-based | Every round-trip and every pure transformer | Property testing library |
| Integration | Every cross-package boundary and every pipeline stage | Same runner as unit |
| End-to-end (CLI) | Every Part 9 command on a representative input | CLI invocation in CI |
| End-to-end (web) | Every user-visible workflow in Part 10 | Browser automation |

### 12.2 Round-trip properties (the foundational invariant)

The core correctness property of the system: every transformation that has a left-and-right-inverse must round-trip exactly.

- DOT emit ∘ DOT parse = identity, byte-level.
- SRML serialize ∘ SRML parse = identity, payload-level.
- Surface mirror ∘ Surface unmirror = identity, payload-level.
- SVG-with-embedded-DOT → reopen → re-export-SVG produces identical SVG byte content given identical fonts and SVG renderer version.
- Cascade resolve ∘ Cascade flatten = identity, leaf-level.

These are all property tests with at least 200 generated cases each, plus a regression corpus of hand-crafted edge cases.

### 12.3 Validator tests

Every validator from Sections 4.3.5, 6.6, and 7.x:

- A positive corpus (scenes that should pass) and a negative corpus (scenes that should fail with a specific violation code).
- For each violation code, at least 5 distinct positive and 5 distinct negative examples.

### 12.4 Adapter tests

For each of the four ingest pipelines:

- A small canonical corpus (5–10 inputs) with hand-built expected outputs.
- Synthetic-input fuzz: generators that produce the input format and the pipeline produces a valid typed scene without crashing.

### 12.5 Performance budgets

Per-pipeline wall-clock budgets at a fixed input size:

- Visio import (vsdx, 100 shapes, 50 connectors) — under 2 seconds.
- Codebase walk (TS monorepo, 50 packages, 1000 files) — under 10 seconds.
- Image trace (single page, single LLM call) — bounded by provider response time plus 500 ms of pipeline time.
- DOT round-trip (200 nodes, 400 edges) — under 200 ms.
- SVG render (200 nodes, 400 edges) — under 500 ms.

All budgets are checked in CI with a generous slack to avoid flakiness.

### 12.6 Significant test coverage — the bar enforced at merge

These targets are floor invariants. The `auditor` subagent computes them in the in-flight audit set (per-PR) and the `nightly-local-audit` routine recomputes them on `main`. Any breach is a CRITICAL finding that blocks release.

- Line coverage ≥ 85% across all `packages/`.
- Branch coverage ≥ 75%.
- Mutation coverage ≥ 60% on the `srml`, `style-guide`, and `graph` packages (these are the highest-leverage targets — round-trip correctness, cascade resolution, DOT structural integrity).
- Property-test suite executes ≥ 1000 cases per pipeline-round-trip property in CI (the v0.1 floor of 50 is replaced; 1000 is feasible because the property runs in-process and the inputs are cheap).
- One E2E test per CLI command and per web-app workflow.
- Zero `Critical` or `High` findings from the deterministic security audit set (see 12.8).

### 12.7 Mutation testing as a gate

Mutation testing runs in the `nightly-local-audit` routine on the modules changed in the previous 24 hours, plus a weekly full-pass on the three high-leverage packages. The toolchain is Stryker (TypeScript / JavaScript) for the web app and shared packages.

- Below 60% kill rate on a `critical`-tagged module → CRITICAL audit finding.
- Below 70% kill rate on a non-critical module → HIGH finding.
- The auditor opens an issue with the surviving mutants enumerated; the test-writer subagent picks it up and adds tests that kill them.

### 12.8 CRAP complexity audit

CRAP = `comp(m)² × (1 − cov(m)/100)³ + comp(m)`. Run nightly via the routine in the harness §7.4 script, adapted for the TypeScript codebase using `ts-complexity` or the equivalent and the package coverage report.

- CRAP > 30 → CRITICAL finding. Refactor or add tests until the score drops.
- CRAP > 15 → HIGH finding.
- The CRAP audit is parameterized to skip generated code (`packages/graph/__generated__/*`, `packages/dot-export/__generated__/*`) which has artificially-inflated complexity.

### 12.9 Output verification (the Haiku check)

After the `implementer` declares a task done, the diff plus the task's acceptance criteria are passed through Haiku 4.5 for an independent PASS / FAIL / UNCLEAR judgment per acceptance criterion. Any FAIL or UNCLEAR sends the task back to the implementer with the verifier's reasoning attached. The cost is negligible; the value is catching the "technically did something but missed an acceptance criterion" class of bug. The Haiku check log is captured in `.claude/audit-log/`.

### 12.10 Audit tiering

The roughly 80–150 audits selected from the vendored `audits/taxonomy/` (a copy of `turbobeest/audits` brought in at Phase 0) are classified by execution tier. Target distribution: ≈60% deterministic, ≈30% local-LLM, ≈10% frontier. The tiering protects the token budget and keeps the nightly sweep on the maintainer's hardware:

- **Deterministic** — semgrep (security patterns), license-checker, npm audit, gitleaks (secrets), `ts-prune` (dead code), `dependency-cruiser` (cross-layer violations), Stryker (mutation), `ts-complexity` + coverage (CRAP), bundle-size budget. No LLM required. Run by the `nightly-local-audit` routine on the maintainer's hardware.
- **Local-LLM** — error-handling consistency, API contract coherence (CLI command surface vs `cli` package internal surface), naming convention coherence across packages, docs-vs-code drift, SRML-payload-vs-codebase drift. Runs against Ollama / `qwen2.5-coder:32b`. Zero token cost.
- **Frontier** — cross-cutting architecture audits (does the cascade actually compose the way Section 6 says?), threat-model audits requiring novel reasoning (does the image-trace adapter leak provider keys to the typed scene?), audits flagging patterns that span multiple packages and need wide context. Runs in the `auditor` subagent, costs tokens, runs sparingly.

Each selected audit YAML in `audits/selected/` declares its tier in a top-level `tier:` field. The routine reads the field and dispatches accordingly.

### 12.11 Failure semantics

A test failure halts the responsible subagent and emits a structured issue with the failure context appended. The orchestrator's retry policy:

1. First failure → re-dispatch with the failure context as added input.
2. Second failure on the same task → escalate to the human via a `contract-gap` GitHub issue if the failure suggests an under-specified FR; otherwise to a `task-stuck` issue.
3. Three failures on the same task → halt the phase. Human triage required at the next control point.

This matches the v0.1 retry-with-context pattern, made concrete.

---

## Part 13 — Autonomous-agent expansion: the operating manual

> Part 13 in v0.1 was a stub addressed to "the next-project Claude that expands this plan." This v0.2 revision is that expansion. It specifies the full operational topology under which exacaliDOT is built. The maintainer should read this as a binding instruction set for the orchestrator and the subagent fleet.

### 13.1 The autonomy contract — three layers

The build runs as a three-layer pipeline:

1. **Strategic Layer** (claude.ai Project, already in use as you read this) — humans and Claude collaborate to produce the machine-readable contract: PRD as a directory of ~15 files, the diagram canon, the audit selection, the deploy plan. This is where this very document gets refined into `prd/`.
2. **Contract Layer** (versioned in the target repo) — the `prd/` directory, `docs/needs/` sphinx-needs graph, `docs/diagrams/` canon, `audits/selected/`, `deploy/`, plus the four control files: `AUTONOMY-MANIFEST.yaml`, `prd/MANIFEST.yaml`, `CLAUDE.md`, `CONTRIBUTING.md`. This is the handoff bundle that bootstraps the execution layer.
3. **Execution Layer** (Claude Code, running locally and in cloud sessions) — the orchestrator, the seven-specialist subagent fleet, parallel git worktrees, the four routines, and the deterministic replay log in `.claude/audit-log/`. Tasks dispatch in dependency order; merges land on `integration` after verification; `integration` promotes to `main` after `/ultrareview` and any necessary contract-guardian sign-off.

The honest definition of "autonomous" here: the execution layer can run for hours or days without human input, *but* the contract layer is engineered so that any ambiguity surfaces as an explicit `contract-gap` issue rather than a silent agent guess. A weak contract makes autonomy into hallucination at scale. The Part 11 work units, Part 12 test discipline, and the diagram canon are what makes the contract strong enough to run against.

### 13.2 Tool stack

| Tool | Role | Layer |
| --- | --- | --- |
| Claude Project on claude.ai | Strategic intake; PRD authoring; contract assembly | 1   |
| Opus 4.7 | Orchestrator, `/ultraplan`, `/ultrareview`, `contract-guardian` | 1, 3 |
| Sonnet 4.6 | Default worker subagent model | 3   |
| Haiku 4.5 | Output verification (the Haiku check) | 3   |
| Qwen2.5-Coder-32B via Ollama | Local-LLM-tier audits and `naming-check.py` | 3   |
| Claude Code CLI | Local orchestrator surface; subagent dispatch | 3   |
| Claude Code Desktop | Multi-session sidebar; per-worktree session windows | 3   |
| Claude Code on the web | Cloud sessions per package (parallel PRs) | 3   |
| sphinx-needs | Requirements traceability matrix; the project knowledge graph | 2   |
| PlantUML + Graphviz | Diagram canon; `dot` is mandatory for M6 and M8 | 2   |
| `turbobeest/audits` | Vendored at Phase 0 into `audits/taxonomy/`; 80–150 selected per project | 2, 3 |
| Stryker (mutation), semgrep, gitleaks, `ts-prune`, `dependency-cruiser`, `ts-complexity` | Deterministic-tier audit toolchain | 3   |
| Routines API | `pr-auto-address` and `contract-drift-monitor` only; the other two run locally on cron | 3   |
| GitHub + Claude GitHub App | Source control; webhook surface for the cloud routines | 2, 3 |

Deliberately not in the stack:

- No task-master and no OpenSpec — sphinx-needs `task` nodes plus `/ultraplan` per phase replace both. There is one source of truth for tasks, not three.
- No FalkorDB on day one — sphinx-needs *is* the contract knowledge graph, and the Tier 1 retrieval scheme (split PRD + `MANIFEST.yaml`) handles the context-loading problem. Escalate to a vector index only if evidence shows it's needed.
- No Mermaid in the canon. Mermaid is restricted to `README.md` and GitHub issue comments. M1, M2, M3, M4, M5, M7 are PlantUML; M6 and M8 are Graphviz DOT. See `DIAGRAM-CANON.md`.

### 13.3 Strategic Layer (what this very document supports)

The output of the Strategic Layer is the contents of `prd/`. This `EXACALIDOT_PLAN.md` is the seed; in the personal-machine Claude Project it gets decomposed into the 15-file PRD plus the diagram canon. Specifically:

- `prd/00-vision.md` — Parts 0 and 1 of this document, restated as a vision file.
- `prd/01-executive-summary.md` — distilled from Parts 1 and 2.
- `prd/02-architecture.md` — Part 3, augmented with the M1 and M2 diagrams generated as PlantUML in `docs/diagrams/01-context/` and `docs/diagrams/02-containers/`.
- `prd/03-features/FR-NNN-<slug>.md` — one file per FR. Most FRs come from Parts 4 through 10. Each gets RFC-2119 keywords, GIVEN/WHEN/THEN acceptance criteria, a reserved sphinx-needs ID, an owner, a priority, and dependencies. The M3 ERD and the M4 primary-flow sequences cover these.
- `prd/04-nfrs.md` — concrete-metric NFRs. The Part 12 performance budgets become rows here. "Fast" and "scalable" are not acceptable; "Visio import of 100 shapes / 50 connectors completes in under 2 seconds at p95 on commodity hardware" is.
- `prd/05-dependencies.md` — the topological ordering between FRs. Layer 0 has zero dependencies. Cycles are fatal. This file is the input to the M8 requirements DAG, generated as Graphviz DOT.
- `prd/06-phases.md` — the Part 11 phase model (Phase 0 through Phase 7), with explicit exit criteria per phase and named deliverables (FR IDs).
- `prd/07-code-structure.md` — Appendix C of this document, expanded into the full directory specification.
- `prd/08-test-strategy.md` — Part 12, with the strict-TDD invariant and the PBT-by-default rules called out at the top.
- `prd/09-integration-testing.md` — the cross-package test plan. Each pipeline (visio → DOT, codebase → DOT, image → DOT, DOT → SVG) has its own integration corpus.
- `prd/10-documentation.md` — the docs in Part 7.3 of the work-unit table.
- `prd/11-operations.md` — local-only operations for v0 (no cloud SaaS, no telemetry); the M5 deployment topology is "user's machine" and the M6 CI/CD pipeline (mandatorily DOT) covers the GitHub Actions flow.
- `prd/12-risks.md` — Part 15 of this document, restated with sphinx-needs IDs and mitigation owners.
- `prd/13-success-metrics.md` — derived from Part 1.1.
- `prd/14-approval.md` — named approver: the maintainer.
- `prd/MANIFEST.yaml` — the retrieval manifest. See 13.4.

The Strategic Layer also produces the diagram canon. The minimum set required before Phase 2 handoff: M1 Context, M2 Container, M3 ERD, M4 Primary-flow Sequences (one per ingest pipeline plus one per export pipeline), M5 Deployment, M6 CI/CD (DOT), M7 Threat Model, M8 Requirements DAG (DOT). The contextually-required full-set diagrams for this project: F2 (state machines for the `validate` and `srml-enrichment` stages), F4 (error-twin sequences for each of the four ingest pipelines), F8 (authorization matrix is moot — single-user tool — so this one is *not* required), F9 (incident decision tree for the CLI process), F13 (observability is local-log-only; minimal). F7, F10, F11 are not required.

### 13.4 Contract Layer (the bundle that bootstraps execution)

The `AUTONOMY-MANIFEST.yaml` for this project, populated:

```yaml
version: 0.2
project:
  name: exacalidot
  primary_ecosystem: typescript
paths:
  prd_dir: prd/
  prd_manifest: prd/MANIFEST.yaml
  rtm: docs/_build/rtm.html
  specs: docs/needs/
  diagrams: docs/diagrams/
  audits: audits/selected/
  deploy: deploy/
  audit_log: .claude/audit-log/
models:
  orchestrator: claude-opus-4-7
  worker: claude-sonnet-4-6
  reviewer: claude-opus-4-7
  verifier: claude-haiku-4-5
  local: qwen2.5-coder-32b
agents:
  - researcher
  - contract-probe
  - test-writer
  - implementer
  - auditor
  - contract-guardian
  - deployer
invariants:
  tdd: strict
  contract_probe: required
  output_verification: required
  diagram_coverage: required
  spike_branches: spike/*
  worktree_per_task: required
control_points:
  - phase_start
  - critical_audit_fail
  - pre_merge_review
  - pre_prod_release
routines:
  - { name: pr-auto-address,         where: anthropic_cloud, trigger: github_webhook }
  - { name: nightly-local-audit,     where: local,           trigger: cron(0 2 * * *), llm: ollama/qwen2.5-coder-32b }
  - { name: org-warden,              where: local,           trigger: cron(0 3 * * 0) }
  - { name: contract-drift-monitor,  where: anthropic_cloud, trigger: pre_merge }
```

The retrieval policy in `prd/MANIFEST.yaml` for exacaliDOT-specific task types:

```yaml
version: 0.1
retrieval_policies:
  implement_feature:
    always: [00-vision.md, 02-architecture.md, 04-nfrs.md, 07-code-structure.md, 08-test-strategy.md]
    on_task: ["03-features/{FR_ID}.md"]
  write_tests:
    always: [04-nfrs.md, 08-test-strategy.md, 09-integration-testing.md]
    on_task: ["03-features/{FR_ID}.md"]
  audit:
    always: [02-architecture.md, 04-nfrs.md, 12-risks.md]
    on_task: []
  refactor:
    always: [02-architecture.md, 07-code-structure.md, 04-nfrs.md]
    on_task: ["03-features/{FR_ID}.md"]
  contract_change:
    always: [00-vision.md, 02-architecture.md, 05-dependencies.md, 12-risks.md]
    on_task: []
  # exacaliDOT-specific
  add_validator:
    always: [04-nfrs.md, 08-test-strategy.md]
    on_task: ["03-features/{FR_ID}.md", "03-features/FR-SRML-*.md", "03-features/FR-STYLE-GUIDE-*.md"]
  add_ingest_pipeline:
    always: [02-architecture.md, 04-nfrs.md, 07-code-structure.md, 08-test-strategy.md, 09-integration-testing.md]
    on_task: ["03-features/{FR_ID}.md"]
```

`CLAUDE.md` is finalized at Phase 2 with the strict-TDD, contract-probe, output-verification, diagram-coverage, and worktree-per-task invariants explicitly enumerated, plus the subagent routing table from §13.5 and the forbidden actions: inventing requirements, editing tests written by `test-writer`, merging with failing acceptance criteria, skipping the contract probe, working on `main` directly without a worktree.

### 13.5 The subagent fleet

Seven specialists, each with a Markdown definition in `.claude/agents/`. Names match the harness; behaviors are tuned for this project below.

**`researcher`** — reads code and docs; never writes. Used for "what does package X do" and "find all places where the cascade resolver is called." Cheap, frequent. Runs on Sonnet 4.6.

**`contract-probe`** — before any code is written for a task, this subagent loads the relevant FR, the relevant Part 11 work-unit table row, and writes a 200-word note: what I understand the task to require, what my plan is, what could go wrong. The orchestrator approves or kicks back with comments. Cost is trivial; the catch rate for off-path drift is high. This is the single highest-leverage step in the pipeline.

**`test-writer`** — given a task need, produces failing tests covering every GIVEN/WHEN/THEN acceptance criterion. For tasks involving any of {data transformation, state machines, parsers, round-trips, invariants, complex business rules}, also produces property-based tests using the `fast-check` library (TypeScript). The work units that mandatorily get PBT for exacaliDOT are: 1.2 (`math` geometry), 1.3 (`element` round-trip), 2.1 (`graph` DOT round-trip), 2.3 (`srml` schema round-trip), 2.4 (`srml` surface mirror), 2.6 (`style-guide` cascade resolver), 3.6 (`layout` validate stage with synthetic-violation generators), 3.8 (`layout` persist stage round-trip), 5.1 (`dot-export` round-trip). Tests must fail for the right reason — the failure message must match the acceptance criterion. The implementer is not dispatched until the tests are in place and failing as expected.

**`implementer`** — the only subagent that writes production code. Loaded only after `test-writer` has produced the failing tests. Does not edit tests; if a test seems wrong, it opens an issue back to `test-writer`. Declares done only when all relevant tests pass. After "done," the Haiku verifier runs.

**`auditor`** — loads the audit YAMLs in `audits/selected/`, dispatches deterministic-tier audits to local tooling (Stryker, semgrep, gitleaks, etc.), local-LLM-tier audits to Ollama, and frontier-tier audits in its own context. Produces a report to `audits/reports/YYYY-MM-DD-<scope>.md`. Runs in two modes: in-flight (per PR; runs the `pre_merge` audit set) and post-flight (called by the `nightly-local-audit` routine).

**`contract-guardian`** — triggered on any PR that modifies `prd/`, `docs/needs/`, `docs/diagrams/`, or `AUTONOMY-MANIFEST.yaml`. Asks: "does this change propagate correctly to all dependent artifacts?" If a new FR is added but no diagram covers it, the PR is blocked. If a diagram is updated but its `@needs-id` and `@covers` headers are stale, the PR is blocked. If the cascade rules in `prd/02-architecture.md` change but the M2 container diagram doesn't, the PR is blocked. Runs in the `contract-drift-monitor` routine pre-merge as a backstop.

**`deployer`** — for v0 of exacaliDOT, "deployment" is restricted to (a) packaging the desktop binary at Phase 7 and (b) publishing the static-site build of the web app. The `deployer` reads the deploy runbooks in `deploy/runbooks/` and runs the chosen flow. There is no production cloud infrastructure; the `pre_prod_release` control point is a release-tag approval, not a cloud promotion.

Each subagent's full Markdown definition (purpose, model, allowed tools, instructions, escalation, output format) is stored in `.claude/agents/<name>.md` per the harness `PREP-DIRECTORY.md` template. The skeletons ship in the prep directory; the exacaliDOT-specific behavior tuning above is layered on at Phase 2.

### 13.6 Git worktrees as the parallelism primitive

The orchestrator runs all parallel work in git worktrees, not in shared workspaces. Each task gets its own worktree branched from the integration branch.

```
exacalidot/                          ← main checkout (orchestrator's process working dir)
.worktrees/
├── task-2.1-graph-dot/              ← worker session 1
├── task-2.4-srml-surface-mirror/    ← worker session 2
├── task-2.6-style-guide-cascade/    ← worker session 3
├── task-4.5-codebase-walker-ts/     ← worker session 4
└── ...
```

Workflow:

1. Orchestrator selects a ready task (all `:depends_on:` chain `status: done`).
2. Orchestrator runs `git worktree add .worktrees/task-<id>-<slug> -b task/<id>-<slug> integration` on the maintainer's machine, or its cloud equivalent on Claude Code on the web.
3. A new Claude Code session is launched in that directory. Its `.claude/` config is inherited from the parent repo via Claude Code's project-config resolution (it walks up the tree looking for the nearest `CLAUDE.md`).
4. The worker session loads the task need from `docs/needs/tasks/`, the PRD sections per `prd/MANIFEST.yaml`, and the bounded workspace constraint from the task's `artifact_contract`.
5. `contract-probe` → `test-writer` → `implementer` → Haiku verifier → `auditor` (in-flight set), all within the worktree.
6. Worker pushes `task/<id>-<slug>` to GitHub and opens a PR against `integration`.
7. `/ultrareview` runs on the PR. `contract-guardian` runs if any contract path was touched.
8. On approval, PR merges to `integration`. Worktree is removed (`git worktree remove .worktrees/task-<id>-<slug>`). Task status set to `done`.
9. The post-merge nightly routine runs the full audit set on `integration`. Periodically, `integration` is fast-forwarded to `main` after `/ultrareview` on the merge.

Why worktrees (not branches in shared checkouts):

- **Isolation.** Each worker session has its own `node_modules`, its own dist cache, its own Vitest watch state. Workers cannot accidentally observe or corrupt each other's intermediate state. A misbehaving worker is contained.
- **Filesystem natural.** Each session has its own working directory; no cross-talk in the editor surfaces.
- **Cheap parallelism.** A worktree is metadata in `.git/`; spinning up the fifth concurrent task is no more expensive than the first.
- **Easy rollback.** A failed task is `git worktree remove --force` away from gone. No `git stash` triage, no half-applied patches in a shared workspace.
- **Parallel test runs.** `pnpm test --filter <package>` runs in each worktree against its own `node_modules`. No port collisions on the watcher; no cache invalidation thrash on the shared store.

A small wrapper script lives in `bin/spawn-worker.sh` that does the dance: create the worktree, hard-link `.git/hooks/`, run `pnpm install --frozen-lockfile` (using pnpm's content-addressed store, so this is fast), open the Claude Code session pointed at the worktree directory.

The orchestrator caps concurrent worktrees at 5 by default — enough parallelism for the human to keep up at the control points, low enough that the maintainer's machine doesn't melt. The cap is configurable in `AUTONOMY-MANIFEST.yaml`. Cloud sessions on Claude Code on the web are not subject to the local cap; the orchestrator can keep an additional 5–10 cloud sessions in flight on independent packages.

### 13.7 The four routines

| Routine | Where | Cost | Trigger |
| --- | --- | --- | --- |
| `pr-auto-address` | Anthropic cloud | Tokens | GitHub webhook on PR comment matching `@claude fix: …` |
| `nightly-local-audit` | Maintainer's machine | Zero (Ollama + deterministic tools) | `cron(0 2 * * *)` |
| `org-warden` | Maintainer's machine | Zero | `cron(0 3 * * 0)` |
| `contract-drift-monitor` | Anthropic cloud | Small token spend | GitHub webhook on PR open touching `prd/`, `docs/needs/`, `docs/diagrams/`, `AUTONOMY-MANIFEST.yaml` |

`pr-auto-address` is the only routine worth paying significant tokens for; it's the high-leverage "address my code-review nit without a context switch" surface. The others are mostly local and free.

Routines run independently of the orchestrator. They don't wait for, and don't block, the worker fleet.

### 13.8 Strict TDD and PBT-first invariants — enforcement

The `pre-tool` hook in `.claude/hooks/pre-tool.sh` watches `Write` and `Edit` tool calls. If the subagent role is `implementer` and the touched path is under `packages/<pkg>/src/`, the hook checks for a sibling test file under `packages/<pkg>/test/` modified in the same PR. If absent, the tool call is rejected with a message instructing the worker to dispatch `test-writer` first. The same hook accepts `Write` and `Edit` from `test-writer` unconditionally.

For the PBT-mandatory list in 13.5, the `contract-guardian` runs at PR open and rejects the PR if any of the listed packages has a touched function but no `fast-check` property in the PR. The list of PBT-mandatory functions per package is maintained in `packages/<pkg>/test/.pbt-required.txt`; the `test-writer` subagent updates this list as new round-trip transformers are introduced.

Spike branches (`spike/*`) are exempt from both invariants; they cannot merge to `integration` (enforced by GitHub branch protection). They exist for exploration, not for shipping.

### 13.9 Audit gates

Three temporal bands, each with its own audit selection:

- **Pre-flight** (during Phase 2 contract assembly) — architecture audits run by the `auditor` against the PRD and diagram canon. Catches contract mistakes before they get coded. Examples for exacaliDOT: cascade-conflict audit (does the four-layer cascade actually monotonically refine?), SRML-schema-completeness audit (every standard's required fields present in the schema?).
- **In-flight** (per PR, by the `auditor` subagent) — the lightweight subset that runs on every PR. Bounded: must complete in under 3 minutes per PR to keep velocity. Examples: semgrep rules, dependency vulnerability scan on changed `package.json` files, license-compatibility check, Stryker on changed modules, the `pre_merge` SRML round-trip check.
- **Post-flight** (nightly via the `nightly-local-audit` routine) — the full sweep on `main`. Examples: full mutation pass on the three high-leverage packages, full CRAP across all `packages/`, license-scan on all transitive dependencies, the `org-warden` weekly pass on Sundays.

Critical findings open a GitHub issue with the `audit-critical` label. The maintainer triages at the next `critical_audit_fail` control point; the auditor does not unilaterally pause the fleet, but the orchestrator will not promote `integration` to `main` while a critical finding is open against `integration`.

### 13.10 Control points (human-in-the-loop)

Five control points, no others. Total weekly maintainer involvement target: under 2 hours.

| Control point | When | Time | Decision |
| --- | --- | --- | --- |
| Approach selection | Phase 1.2 of the Strategic Layer (already past in v0.2) | n/a — done | The four ingest pipelines, SRML, the codebase-styleguide cascade are locked. |
| PRD + diagram sign-off | Phase 1 → Phase 2 boundary | 60–120 min | Does the contract reflect intent; are the M1–M8 diagrams correct. |
| Phase-start approval | Beginning of each Part 11 phase | 5–15 min per phase | Is the `/ultraplan` output aligned with the PRD; is the task DAG sane. |
| Critical audit triage | Whenever `audit-critical` issues open | 5–30 min per finding | Real positive vs false positive; accept-risk vs fix. |
| Pre-release approval | Each release tag (typically once per phase milestone) | 5–10 min | Is the release notes draft accurate; is the build artifact signed; do the smoke tests pass. |

Kill switches: `git revert` on `AUTONOMY-MANIFEST.yaml` halts fleet work. Routines have a `paused: true` flag. Uninstalling the Claude GitHub App disables the cloud routines.

### 13.11 Deterministic replay log

Every subagent invocation appends to `.claude/audit-log/YYYY-MM-DD/`:

```
.claude/audit-log/2026-05-12/
  001-orchestrator-dispatch-contract-probe-task-2.4.jsonl
  002-contract-probe-output-task-2.4.md
  003-orchestrator-approved-task-2.4.jsonl
  004-test-writer-dispatch-task-2.4.jsonl
  005-test-writer-output-task-2.4.md
  006-implementer-dispatch-task-2.4.jsonl
  007-implementer-output-task-2.4.md
  008-haiku-verifier-task-2.4.jsonl
  009-auditor-in-flight-task-2.4.md
  ...
```

JSONL files capture inputs, tool calls, and decisions. Markdown files capture prose outputs. This log plus `git log` lets the maintainer reconstruct any decision chain weeks later. The log is committed to the repo (under `.claude/audit-log/`); it is not gitignored. Storage cost is negligible.

### 13.12 Hand-off checklist (revised)

The v0.1 hand-off checklist was about getting this document to a personal machine. That's done. The v0.2 checklist is about getting from this document to the running fleet:

- [ ] `EXACALIDOT_PLAN.md` v0.2 (this file) saved in the personal-machine Claude Project's knowledge.
- [ ] `PROJECT-AUTONOMY-HARNESS.md`, `DIAGRAM-CANON.md`, `PREP-DIRECTORY.md`, `CLAUDE-PROJECT-INSTRUCTIONS.md` saved alongside it (already present in the project).
- [ ] The intake Claude Project decomposes this document into `prd/` (the 15 files plus `MANIFEST.yaml`) and the M1–M8 diagram canon plus the contextually-required full-set diagrams. The intake produces sphinx-needs source in `docs/needs/` with reserved IDs.
- [ ] The maintainer signs off the contract-validation checklist (harness §3.5).
- [ ] A fresh repo is created from the prep-directory template; the intake bundle is committed.
- [ ] Phase 0 (the re-fork from upstream Excalidraw at a pinned tag, plus the strip pass) is the first executed phase. It runs on the local machine, not via the fleet, since it's a one-shot setup.
- [ ] The vendored audit taxonomy is dropped into `audits/taxonomy/`; the 80–150 selection is curated into `audits/selected/{deterministic,local-llm,frontier}/` with the `tier:` field set in each YAML.
- [ ] Ollama is running with `qwen2.5-coder:32b` pulled.
- [ ] `bin/init-project.sh` from the prep-directory template has run; cron entries are installed for `nightly-local-audit` and `org-warden`.
- [ ] `.envrc` sets `CLAUDE_CODE_SUBAGENT_MODEL=claude-sonnet-4-6`, `ENABLE_PROMPT_CACHING_1H=1`, and the project-specific paths.
- [ ] A throwaway dry-run feature is executed end-to-end through the fleet to verify all subagents, the Haiku verifier, the auditor, the contract-guardian, and all four routines actually work. Planted defects (a missing acceptance criterion check, an out-of-band PRD edit) must be caught.
- [ ] Only after the dry-run passes does Phase 1 of Part 11 begin under fleet execution.

---

## Part 14 — Open decisions deferred to the build-plan author

Each remaining item is a decision that can be made in either direction without changing the architecture above. They are flagged here so the maintainer is not surprised. Items resolved by adoption of the harness in Part 13 (test-runner choice, branching model, CI provider topology, agent runtime, lockfile and package-manager choice, the per-task workspace model) have been removed from this list.

| Decision | Options | Notes |
| --- | --- | --- |
| Distribution shape | (a) static-site SaaS, (b) Tauri desktop binary, (c) Electron desktop binary, (d) all of the above | Architecture admits any. (a) is fastest to ship; (b) is most defensible for the regulated-industry audience. The `deployer` subagent's runbook is templated against this choice in `deploy/runbooks/`. |
| Domain and brand | A name and TLD will be needed. exacaliDOT is a working name. | Trademark search needed before the brand commitment. |
| Vision provider for Image Trace | One provider, single adapter. | A provider that supports vision plus structured output is preferred. |
| Layout engine | (a) Graphviz-WASM consumer, (b) native hierarchical layout | (a) is more proven; (b) eliminates the EPL-licensed dependency. The Phase 3 work unit 3.3 carries the choice. |
| Project StylePack file format | YAML chosen here; could also be Markdown-with-frontmatter or TOML | YAML is the most ecosystem-friendly. |
| Pricing | Free / freemium with hosted Image Trace / paid binary / open-source | Strategic; not a technical decision. |

---

## Part 15 — Risk register

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| VSDX shape diversity outpaces the rule table | High | Moderate | The unmapped-shape fallback emits descriptors with the original master name; users can re-tag manually or trigger Image Trace as a second pass. |
| SRML schema churn breaks round-trip on legacy documents | Medium | High | Schema versioning with a migration chain, see Section 4.3.4. |
| The image-trace provider changes its API contract | Medium | Low | Provider adapter is one file; stable internal contract. |
| Graphviz-WASM layout produces overlapping clusters on large codebases | Medium | Moderate | Native hierarchical layout as fallback. |
| Performance degrades on a large monorepo (10K+ files) | Medium | Moderate | Walker honors a depth flag; a future v1 task is incremental walking. |
| Patent encumbrance on a layout or import algorithm surfaces late | Low | High | Stick to public-spec algorithms; no novel layout primitives in v0. |
| License contamination from a misjudged dependency | Low | High | Permissive-only policy in Appendix A; license-scan job in CI in the expansion phase. |
| Brand confusion with Excalidraw or Drawesome | Medium | Low | Marketing copy explicitly positions exacaliDOT as a third project with a narrow purpose. |
| The codebase walker mis-clusters a project that does not follow the inferred conventions | High | Low | The Project StylePack overrides; users can author cluster rules in YAML. |
| SRML payloads grow large enough to bloat DOT files | Medium | Moderate | Blob attribute uses compact JSON; an opt-in `--srml-mode external` writes the SRML to a sidecar file and stores a content-addressed reference in the DOT. v1 work, not v0. |

---

## Appendix A — Permissive-only OSS dependency policy

The v0 build uses only dependencies under permissive licenses: MIT, BSD-2-Clause, BSD-3-Clause, ISC, Apache-2.0, MPL-2.0 (with reservations). Specifically:

- **Allowed without further review.** MIT, BSD-2/3, ISC, Apache-2.0.
- **Allowed with explicit notice.** MPL-2.0 (file-level copyleft only; acceptable for clearly-separated module use).
- **Allowed for WASM-as-binary.** EPL-1.0 (Graphviz-WASM falls here if chosen). Subject to legal review at v0.2.
- **Forbidden.** GPL family, AGPL family, source-available-but-non-OSI licenses (RSALv2, BUSL, SSPL, etc.).

A license-scan CI job is part of the autonomous-agent expansion verification suite.

---

## Appendix B — Glossary

- **`ds_*` namespace.** Prefix for semantic attributes on DOT nodes and edges. Survives external Graphviz tooling because Graphviz ignores unknown attributes.
- **DUS / USG.** Drawesome Universal Style Guide / Universal Style Guide — the baseline cascade layer.
- **Cascade.** The four-layer rule resolution: USG → standard → project StylePack → scene overrides.
- **Active rules.** The output of cascade resolution.
- **SRML.** Specification & Requirements Metadata Layer — the structured payload attached to each node and edge.
- **Surface attribute.** A `srml_*`-prefixed DOT attribute that mirrors a top-level SRML payload field.
- **Blob attribute.** The `srml_blob` DOT attribute holding the full SRML payload as a JSON string.
- **Taxonomy.** The 8-type scheme for nodes and edges (System, Component Group, Component, Operation, Sub-operation, Descriptor, Message, Message Type).
- **Standard.** A typed registry entry describing one diagram convention (C4 Container, UML Class, SysML IBD, UAF SV-1).
- **Codebase rule pack.** The codebase-domain-flavored override layer that engages when the ingest source is the codebase walker.
- **Image Trace.** The pipeline from a raster or vector image to a typed scene, mediated by a vision-capable LLM.
- **RTM.** Requirements Traceability Matrix — a tabular summary of trace links rooted at requirement entries.
- **Harness.** Shorthand for the autonomy harness specified in `PROJECT-AUTONOMY-HARNESS.md` and operationalized for this build in Part 13.
- **Subagent fleet.** The seven specialists in `.claude/agents/`: `researcher`, `contract-probe`, `test-writer`, `implementer`, `auditor`, `contract-guardian`, `deployer`.
- **Worktree.** A second working tree of the same git repository, on its own branch, with its own filesystem state. Used here as the unit of parallelism: one worker subagent per worktree.
- **Contract probe.** The 200-word "what I'm about to do and why" note written by the `contract-probe` subagent before any code is written for a task. The orchestrator's lowest-cost defense against off-path drift.
- **Output verification (the Haiku check).** Independent PASS/FAIL/UNCLEAR judgment by Haiku 4.5 on each subagent output, scored against the task's acceptance criteria.
- **Audit log.** The deterministic replay log under `.claude/audit-log/`. Every dispatch, output, and verification result is captured.
- **PRD-as-directory.** The convention of splitting the product requirements document into ~15 files under `prd/` plus a `MANIFEST.yaml` retrieval policy. Replaces the monolithic-PRD anti-pattern.
- **Control point.** A named place in the pipeline where human review is required: `phase_start`, `critical_audit_fail`, `pre_merge_review`, `pre_prod_release`, plus the one-shot `approach_selection` and `prd_signoff`.
- **Diagram canon.** The set of PlantUML and Graphviz DOT diagrams required by the harness. M1–M8 are the minimum set; F1–F14 apply contextually.
- **`ultraplan` / `ultrareview`.** Cloud-based planning and review slash-commands invoked at phase start and pre-merge respectively.
- **CRAP score.** `comp(m)² × (1 − cov(m)/100)³ + comp(m)`. Combined complexity-and-coverage metric used as an audit gate.
- **Routine.** A scheduled or webhook-triggered automation. The four routines in this build: `pr-auto-address`, `nightly-local-audit`, `org-warden`, `contract-drift-monitor`. Three run locally against the maintainer's hardware to preserve token budget.

---

## Appendix C — File and directory layout (target shape)

This is the intended directory layout for the eventual repo. Reproduced as a tree only — no source code, no configuration files.

```
exacalidot/
├── apps/
│   └── web/
│       └── src/
│           ├── features/
│           │   ├── importVisio/
│           │   ├── codebaseWalk/
│           │   ├── imageTrace/
│           │   ├── exportDot/
│           │   ├── exportSvg/
│           │   ├── srmlInspector/
│           │   ├── styleGuideSummary/
│           │   └── violations/
│           ├── components/
│           └── App
├── packages/
│   ├── common/
│   ├── math/
│   ├── element/
│   ├── graph/
│   ├── taxonomy/
│   ├── srml/
│   ├── style-guide/
│   ├── standards/
│   ├── layout/
│   ├── visio-import/
│   ├── codebase-walker/
│   ├── image-trace/
│   ├── dot-export/
│   ├── svg-export/
│   ├── core/
│   └── cli/
├── docs/
│   ├── README
│   ├── quickstart-visio
│   ├── quickstart-codebase
│   ├── quickstart-image-trace
│   ├── srml-primer
│   └── style-guide-cascade/
└── corpus/
    ├── visio-vsdx/
    ├── visio-vdx/
    ├── codebase-fixtures/
    ├── image-trace-fixtures/
    └── dot-fixtures/
```

The `corpus/` directory holds hand-crafted fixtures (no copied content from any private source) that drive the integration test suites. Building these fixtures is part of the relevant Phase 4 and Phase 7 work units.

Two additional directories are required by the harness, not shown in the v0.1 layout:

```
exacalidot/
├── .claude/
│   ├── agents/                  # 7 subagent definitions
│   ├── commands/                # project-specific slash commands
│   ├── hooks/                   # pre-tool, post-subagent hooks (TDD invariant, Haiku verifier kick-off)
│   └── audit-log/               # deterministic replay log; committed
├── .worktrees/                  # gitignored; ephemeral per-task working trees
├── prd/                         # 15 files + MANIFEST.yaml (the contract)
├── docs/
│   ├── adr/                     # ADR set
│   ├── needs/                   # sphinx-needs source (req, nfr, task, test, audit, diagram, impl, decision)
│   └── diagrams/                # canon directory tree per DIAGRAM-CANON.md
├── audits/
│   ├── taxonomy/                # vendored copy of turbobeest/audits
│   ├── selected/                # 80–150 chosen YAMLs, classified by tier
│   └── reports/                 # routine output
├── deploy/
│   ├── pipelines/               # GitHub Actions
│   ├── runbooks/                # rollback, release
│   └── observability/           # local-log only for v0
├── routines/                    # 4 routine scripts
├── prompts/                     # bootstrap, phase kickoff, audit sweep
├── scripts/                     # crap.ts, gen-req-dag.ts, validate-contract.ts
├── bin/                         # init-project.sh, spawn-worker.sh, render-diagrams.sh
├── AUTONOMY-MANIFEST.yaml
├── CLAUDE.md
└── CONTRIBUTING.md
```

These directories ship populated with skeletons in the prep-directory template (`harness-template`); they get filled in during the Strategic Layer and Phase 2 contract assembly.

---

## Appendix D — Git worktree topology and the parallel session model

This appendix expands on the §13.6 mechanics. Every worker subagent in this build runs in its own git worktree. There are no shared workspaces, no `git stash`-based context-switching, no "let me just check out the other branch real quick."

### D.1 Layout

```
exacalidot/                              ← orchestrator's working tree on `integration` (or `main`)
.worktrees/
├── task-1.2-math/                       ← worker session: branch task/1.2-math, off integration
├── task-2.1-graph-dot/                  ← worker session: branch task/2.1-graph-dot, off integration
├── task-2.4-srml-surface-mirror/        ← branch task/2.4-srml-surface-mirror
├── task-2.6-style-guide-cascade/        ← branch task/2.6-style-guide-cascade
└── ...
```

`.worktrees/` is gitignored. The branches `task/<id>-<slug>` are pushed to GitHub for PR creation; the worktrees themselves are local-only.

### D.2 Concurrency budget

The default cap on concurrent local worktrees is 5. This is set in `AUTONOMY-MANIFEST.yaml` under `worktrees.local_max`. The cap exists because:

- Each worktree has its own `node_modules` and Vitest watcher; five is the comfortable limit before a 32GB-RAM development machine starts to swap.
- Five concurrent control flows is the limit at which the maintainer can keep up with control-point reviews without losing track.
- Higher concurrency is available via Claude Code on the web — cloud sessions per package, no local resource pressure. Cloud concurrency cap is 10 by default.

The orchestrator load-balances: tightly-coupled tasks (those touching shared packages) prefer local worktrees so the maintainer can spot-check; independent module work (one whole package per task) prefers cloud sessions.

### D.3 Spawn mechanics

`bin/spawn-worker.sh <task-id> <slug>` does:

```
git fetch origin integration
git worktree add .worktrees/task-${id}-${slug} -b task/${id}-${slug} origin/integration
cd .worktrees/task-${id}-${slug}
pnpm install --frozen-lockfile         # fast: pnpm CAS shared with parent checkout
ln -sf ../../.claude .claude            # share subagent definitions and hooks (read-only at runtime)
claude code .                           # open Claude Code session in this worktree
```

The `.claude` symlink keeps subagent definitions and hooks consistent across all worktrees without the orchestrator having to re-deploy them on every spawn. The `audit-log/` directory under `.claude` is *not* symlinked; each worktree writes its own audit-log subdirectory and the orchestrator collects them on PR merge.

### D.4 Teardown

On PR merge:

```
git worktree remove --force .worktrees/task-${id}-${slug}
git branch -D task/${id}-${slug}        # local cleanup; remote is GC'd by GitHub policy
```

On task abort or three-strikes failure:

```
git worktree remove --force .worktrees/task-${id}-${slug}
# branch is kept on remote for forensic review; tagged forensic/task-${id}-${slug}
```

### D.5 What worktrees buy versus shared workspaces

- Filesystem isolation. A task that mutates `package.json` doesn't perturb the four other concurrent tasks.
- Independent test runners. Vitest watcher-mode in worktree A does not invalidate the watch state in worktree B.
- Independent `node_modules`. A task that needs to add a dev-dependency for testing doesn't propagate that change globally.
- Cheap rollback. Force-removal of the worktree erases all evidence of a failed task.
- Parallel Claude Code sessions. The desktop app's session sidebar shows all five worktrees at once; the maintainer can context-switch between them visually without filesystem confusion.

### D.6 What worktrees do not solve

- Cross-task race conditions on shared resources. If two tasks both want to add a row to `prd/MANIFEST.yaml`, the orchestrator must serialize them. The `contract-guardian` enforces this at PR time; the orchestrator avoids it at dispatch time by not assigning conflicting tasks concurrently.
- Database migrations. Not applicable to v0 of exacaliDOT (no database), but worth noting for completeness: shared mutable state outside the repo is not protected by worktree isolation.
- Lockfile drift. Two tasks that both run `pnpm install` in their worktrees and arrive at slightly different lockfile updates will conflict at merge. Mitigation: the orchestrator dispatches a `lockfile-update` task as its own dedicated worktree, sequentially, when a dependency change is needed; worker tasks use `--frozen-lockfile`.

---

## Appendix E — Mapping Part 11 work units to sphinx-needs task IDs

Each Part 11 work unit maps to one sphinx-needs `task` node. The ID convention: `TASK_P<phase>_<work-unit>` where phase is `0` through `7` and work-unit is the dotted slot.

| Work unit | Task ID | sphinx-needs file | Bounded workspace |
| --- | --- | --- | --- |
| 0.1 Re-fork Excalidraw | `TASK_P0_01` | `docs/needs/tasks/phase-0/refork.rst` | repo root (one-time) |
| 0.2 Strip non-needed feature modules | `TASK_P0_02` | `docs/needs/tasks/phase-0/strip.rst` | `apps/web/`, repo root |
| 1.1 `common` package | `TASK_P1_01` | `docs/needs/tasks/phase-1/common.rst` | `packages/common/` |
| 1.2 `math` package | `TASK_P1_02` | `docs/needs/tasks/phase-1/math.rst` | `packages/math/` |
| 1.3 `element` package | `TASK_P1_03` | `docs/needs/tasks/phase-1/element.rst` | `packages/element/` |
| 2.1 `graph` DOT parse/emit | `TASK_P2_01` | `docs/needs/tasks/phase-2/graph.rst` | `packages/graph/` |
| 2.2 `taxonomy` package | `TASK_P2_02` | `docs/needs/tasks/phase-2/taxonomy.rst` | `packages/taxonomy/` |
| 2.3 `srml` schema | `TASK_P2_03` | `docs/needs/tasks/phase-2/srml-schema.rst` | `packages/srml/` |
| 2.4 `srml` surface mirror & blob | `TASK_P2_04` | `docs/needs/tasks/phase-2/srml-mirror.rst` | `packages/srml/` |
| 2.5 `srml` validators | `TASK_P2_05` | `docs/needs/tasks/phase-2/srml-validators.rst` | `packages/srml/` |
| 2.6 `style-guide` schema + cascade | `TASK_P2_06` | `docs/needs/tasks/phase-2/style-guide-cascade.rst` | `packages/style-guide/` |
| 2.7 `style-guide` USG validators | `TASK_P2_07` | `docs/needs/tasks/phase-2/style-guide-validators.rst` | `packages/style-guide/` |
| 2.8 `style-guide` codebase rule pack | `TASK_P2_08` | `docs/needs/tasks/phase-2/style-guide-codebase.rst` | `packages/style-guide/` |
| 2.9 `standards` 4 pilots | `TASK_P2_09` | `docs/needs/tasks/phase-2/standards.rst` | `packages/standards/` |
| 3.1 `layout` ingest stage | `TASK_P3_01` | `docs/needs/tasks/phase-3/layout-ingest.rst` | `packages/layout/` |
| 3.2 `layout` frame stage | `TASK_P3_02` | `docs/needs/tasks/phase-3/layout-frame.rst` | `packages/layout/` |
| 3.3 `layout` layout stage | `TASK_P3_03` | `docs/needs/tasks/phase-3/layout-layout.rst` | `packages/layout/` |
| 3.4 `layout` style stage | `TASK_P3_04` | `docs/needs/tasks/phase-3/layout-style.rst` | `packages/layout/` |
| 3.5 `layout` srml-enrichment | `TASK_P3_05` | `docs/needs/tasks/phase-3/layout-srml.rst` | `packages/layout/` |
| 3.6 `layout` validate stage | `TASK_P3_06` | `docs/needs/tasks/phase-3/layout-validate.rst` | `packages/layout/` |
| 3.7 `layout` render stage | `TASK_P3_07` | `docs/needs/tasks/phase-3/layout-render.rst` | `packages/layout/` |
| 3.8 `layout` persist stage | `TASK_P3_08` | `docs/needs/tasks/phase-3/layout-persist.rst` | `packages/layout/` |
| 4.1 `visio-import` VSDX parser | `TASK_P4_01` | `docs/needs/tasks/phase-4/visio-vsdx.rst` | `packages/visio-import/` |
| 4.2 `visio-import` VDX parser | `TASK_P4_02` | `docs/needs/tasks/phase-4/visio-vdx.rst` | `packages/visio-import/` |
| 4.3 `visio-import` shape-mapping | `TASK_P4_03` | `docs/needs/tasks/phase-4/visio-mapping.rst` | `packages/visio-import/` |
| 4.4 `visio-import` SRML scaffolding | `TASK_P4_04` | `docs/needs/tasks/phase-4/visio-srml.rst` | `packages/visio-import/` |
| 4.5 `codebase-walker` TS adapter | `TASK_P4_05` | `docs/needs/tasks/phase-4/walker-ts.rst` | `packages/codebase-walker/` |
| 4.6 `codebase-walker` taxonomy inference | `TASK_P4_06` | `docs/needs/tasks/phase-4/walker-taxonomy.rst` | `packages/codebase-walker/` |
| 4.7 `codebase-walker` SRML enrichment | `TASK_P4_07` | `docs/needs/tasks/phase-4/walker-srml.rst` | `packages/codebase-walker/` |
| 4.8 `image-trace` vision adapter | `TASK_P4_08` | `docs/needs/tasks/phase-4/image-trace.rst` | `packages/image-trace/` |
| 5.1 `dot-export` package | `TASK_P5_01` | `docs/needs/tasks/phase-5/dot-export.rst` | `packages/dot-export/` |
| 5.2 `svg-export` native renderer | `TASK_P5_02` | `docs/needs/tasks/phase-5/svg-export.rst` | `packages/svg-export/` |
| 5.3 `cli` skeleton | `TASK_P5_03` | `docs/needs/tasks/phase-5/cli.rst` | `packages/cli/` |
| 6.1 Top toolbar | `TASK_P6_01` | `docs/needs/tasks/phase-6/toolbar.rst` | `apps/web/src/components/`, `apps/web/src/features/` |
| 6.2 SRML Inspector panel | `TASK_P6_02` | `docs/needs/tasks/phase-6/srml-inspector.rst` | `apps/web/src/features/srmlInspector/` |
| 6.3 Active Style Guide summary | `TASK_P6_03` | `docs/needs/tasks/phase-6/style-summary.rst` | `apps/web/src/features/styleGuideSummary/` |
| 6.4 Violations panel | `TASK_P6_04` | `docs/needs/tasks/phase-6/violations.rst` | `apps/web/src/features/violations/` |
| 7.1 E2E corpus | `TASK_P7_01` | `docs/needs/tasks/phase-7/corpus.rst` | `corpus/` |
| 7.2 Packaging | `TASK_P7_02` | `docs/needs/tasks/phase-7/packaging.rst` | `apps/web/`, `deploy/` |
| 7.3 Documentation | `TASK_P7_03` | `docs/needs/tasks/phase-7/docs.rst` | `docs/` |

The `:depends_on:` chains follow the dependencies column from the Part 11 tables. Phase boundaries are dependency cliffs: no Phase N+1 task starts until every Phase N task is `status: done`. Within a phase, the orchestrator parallelizes maximally, subject to the per-package bounded-workspace exclusion (no two concurrent tasks may have the same bounded workspace).

---

**End of plan v0.2.**

The next document version (v0.3) will be produced after Phase 0 completes (re-fork plus strip). v0.3's purpose is to record concrete answers to the remaining Part 14 open decisions and to capture any contract-gap issues raised during Phase 0 that warrant PRD revision before Phase 1 begins.