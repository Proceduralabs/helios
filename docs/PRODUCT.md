# Product definition

Status: **accepted direction; unimplemented**. This document owns the meaning of Helios v1. Milestone ordering belongs in [ROADMAP.md](../ROADMAP.md), implementation design in [ARCHITECTURE.md](ARCHITECTURE.md), and proof thresholds in [VALIDATION.md](VALIDATION.md).

## One-sentence definition

**Helios v1 is an interactive, free-flight Solar-System rendering vertical slice that carries a camera from an interplanetary view to an eye-height hover above one documented Earth terrain region, while exposing which parts of the view are measured, model-derived, procedural, or display-only.**

## Vision versus v1

The long-term vision is an inspectable foundation for large-scale space and planetary visualization: more bodies, richer real datasets, extensible generation, scripted scientific views, and eventually navigation between star systems.

v1 is deliberately smaller. It is the release at [Milestone 10](../ROADMAP.md), and it proves one end-to-end journey. It does not claim to be a SpaceEngine replacement, flight simulator, game engine, GIS, mission-analysis package, or stable embeddable SDK.

The distinction matters because each of those products brings a second scope:

- a game needs collision, interaction, content, save state, and accessibility;
- a flight or orbital simulator needs dynamics, vehicles, controls, and validated physics;
- a GIS needs projections, analysis tools, layers, queries, and data-management workflows;
- mission analysis needs traceable standards, kernels, uncertainty, and much tighter validation;
- a public SDK needs stable APIs, compatibility promises, examples, and support.

Helios may grow toward some of them only after the v1 rendering foundation works.

## Product surfaces

### 1. Reference explorer — the v1 product

A desktop application that bundles one reproducible scenario, provides a free camera and a preset approach, exposes diagnostic/provenance overlays, and can replay the benchmark route. This is what a first-time user downloads.

### 2. Internal engine libraries — implementation structure

The renderer, coordinate system, universe model, planet LOD, asset schemas, and platform layer live behind explicit internal boundaries. They are structured so they can later become reusable, but v1 does not promise binary compatibility, plugins, or a supported embedding API.

### 3. Offline toolchain — required support surface

Small command-line tools convert a selected DEM and star catalog into versioned runtime formats. A swapchain-free smoke path supports automation. General-purpose batch generation and multi-node/cluster work are post-v1 unless promoted explicitly.

## Who v1 is for

Primary audiences:

- graphics and engine programmers who want an inspectable reference implementation;
- technically curious users who want to explore a scientifically grounded scale transition;
- educators and visualization practitioners who value clear data provenance;
- contributors interested in coordinates, planet LOD, rendering, numerical tests, and data pipelines.

Potential later audiences, not yet served by v1:

- researchers needing stable batch APIs or validated analysis outputs;
- studios needing a supported, documented engine integration;
- creators needing authoring tools, mod packaging, or a content ecosystem;
- players expecting missions, progression, vehicles, walking, or multiplayer.

## The v1 user journey

The first downloadable build should make the project understandable without reading its source:

1. **Launch.** The app opens a bundled scenario with a documented generator version, data manifest, epoch, clock policy, quality preset, and expected storage footprint.
2. **Orient.** The user sees the Sun/Earth/Moon context and a minimal explanation that this is a model Solar System, not a live observation or orbital simulation.
3. **Choose control.** The user can start the benchmark route or use a six-degree-of-freedom free camera with explicit speed controls. The default 30–60 s benchmark freezes astronomical time; a route may advance it only through a versioned scripted time track recorded in the route manifest.
4. **Approach.** The camera accelerates toward Earth. Distant representations, planet patches, atmosphere, and terrain transition without an explicit scene reload.
5. **Descend.** Streaming and LOD refine the hero region. Parent content remains visible while children load; diagnostics make residency and error visible.
6. **Hover.** The route ends about two metres above the rendered heightfield. This is a camera pose, not a character standing on collision geometry.
7. **Inspect truth.** An overlay identifies the source DEM, its native resolution and datum, the procedural enhancement version, current coordinate frame, model epoch, and display transform.
8. **Replay or capture.** The same route can be replayed to compare output, frame times, memory, and numerical checks across builds.

Early development builds may expose only the scripted route and debug camera. Milestone 10 is responsible for making the interaction coherent enough for someone outside the project.

## v1 content model

### Detailed content

- **Hero body:** Earth, with Earth radius/shape conventions stated in the data manifest.
- **Measured terrain:** one named region selected before Milestone 6; exact product, footprint, resolution, datum, license, and no-data behavior must be recorded.
- **Procedural terrain:** global fallback and/or added high-frequency detail, always labelled synthetic.
- **Atmosphere:** one Earth preset derived from the chosen Bruneton-style implementation and documented parameters.

Outside the measured hero region, v1 does not invent real Earth continents/topography and present them as data. The default is an explicitly synthetic global height/material field on an Earth-shaped body. A recognizable global land/ocean or albedo layer may be added only from a separately selected, licensed, manifested low-resolution product. The hero-region transition uses a recorded mask/feather rule and the provenance overlay changes classification at the boundary.

### Context content

- Sun, Earth, Moon, and major planets placed by the selected analytic ephemeris backend.
- Low-detail or impostor representations where angular size does not justify terrain.
- One reproducibly selected and versioned Gaia DR3-derived catalog rendered as a sky, not a complete navigable galaxy. The Gaia Archive remains a queryable upstream; v1 does not require a local copy of the complete source table.

Detailed terrain on every planet, complete global Earth imagery, weather, clouds, oceans, rings, minor bodies, spacecraft, and navigable stellar systems are not implied.

## Data truth model: “honest scale”

Every visible or queryable value belongs to one of four classes:

| Class | Meaning | Example |
|---|---|---|
| **Measured / observed** | Derived from an instrument dataset with recorded provenance and uncertainty | Elevation sample from a named DEM product |
| **Model-derived** | Computed from a published mathematical/astronomical model | Planet centre from a pinned VSOP series at a TDB epoch |
| **Procedural / synthetic** | Generated for plausible detail or coverage | Noise-based terrain below the DEM's native resolution |
| **Display transform** | Changes presentation, not underlying physical or source values | Exposure, tone mapping, false-colour debug view |

The UI and asset metadata must preserve these distinctions. “Realistic,” “physical,” and “high resolution” are not substitutes for provenance.

## v1 requirements

These requirement IDs are stable references for the roadmap and validation plan. Thresholds that still need a spike are marked **TBD**, not guessed into existence.

| ID | Requirement |
|---|---|
| P-001 | Ship an interactive desktop reference explorer and one reproducible benchmark scenario. |
| P-002 | Complete the benchmark route from the documented interplanetary start to a camera approximately 2 m above the hero terrain without an explicit scene-loading transition. |
| P-003 | Preserve sub-millimetre numeric resolution for body-local camera motion near the surface, while meeting a separate screen-space jitter threshold across the full route. |
| P-004 | Refine terrain continuously to approximately 1 m near-camera sample spacing on the hero body, with no visible face cracks and bounded resident memory. |
| P-005 | Render one Earth atmosphere preset and a finite HDR/exposure pipeline through day, night, sunset, and an eclipse/occlusion test. |
| P-006 | Place major Solar-System bodies over a documented date interval using named models, frames, time scales, and per-body tolerances. |
| P-007 | Render a recognizable Gaia DR3-derived star sky from a pinned query, quality policy, release manifest, and legally reviewed attribution package. |
| P-008 | Expose provenance for the active DEM, procedural generator, astronomical model, epoch, and display transform. |
| P-009 | Make benchmark inputs and outputs versioned and reproducible: exact hashes for canonical integer products and tolerance-based conformance for portable floating-point products. |
| P-010 | Expose validation-layer status, coordinate/LOD diagnostics, frame-time statistics, memory/residency counts, and capture metadata. |
| P-011 | Package a reproducible release on one reference platform and publish explicit experimental/unsupported status for all others. |
| P-012 | Finish the route with zero Vulkan validation errors and no NaN/Inf values in validated HDR buffers. |
| P-013 | Ship a complete legal/release package: AGPL license and corresponding-source/build access, About/legal UI, third-party notices, dataset/catalog credits and requested citations, exact manifests, and no unresolved public placeholders. |

## What key phrases mean

### “Seamless”

No explicit scene switch or blocking loading screen interrupts the validated route. It does not mean that every resource is resident from launch, that no fallback representation is ever visible, or that a network connection is required. Background generation and streaming are core parts of the proof.

### “About 1 m geometry”

Approximately one-metre terrain sample or vertex spacing near the validated camera, depending on the final representation. It says nothing about whether the source data measured one-metre features. A 30 m DEM enhanced with procedural detail remains 30 m measured data plus synthetic detail.

### “Physical lighting”

The pipeline uses documented radiometric/photometric conventions and exposure math. It does not make Helios a calibrated scientific instrument. Validation must state where approximations enter.

### “Real positions”

Model-derived positions evaluated in named coordinate frames and time scales, compared to an external reference over a finite date interval. They are not exact observations, and they are not valid for literally any date.

### “Deterministic”

The generator version, data hashes, parameters, shader/compiler profile, and backend are part of the input. Cross-machine exactness is promised only for operations deliberately designed for it; other outputs use recorded tolerances.

## Explicit v1 non-goals

- interstellar travel or generated exoplanet systems;
- general camera/surface collision detection or response, walking, vehicles, fuel, lift, or orbital dynamics; a benchmark-only altitude clamp is allowed and is not a surface-physics system;
- gameplay, missions, progression, multiplayer, or save games;
- vegetation, buildings, roads, clouds/weather, volumetric oceans, or water/fluid simulation;
- global high-resolution measured terrain or imagery;
- n-body gravity, relativity, black holes, or planetary destruction;
- stable public plugin, scripting, or embedding APIs;
- mission-grade navigation, GIS analysis, or scientific inference from rendered pixels;
- cluster rendering or a supported research batch platform;
- equal release support for every desktop OS at the first milestone.

## Product success

v1 succeeds when a new user can download one supported build, understand what the view represents, complete or replay the route, and inspect evidence showing that precision, seams, memory, performance, astronomy, HDR finiteness, and data provenance meet the documented contract.

A beautiful 30–60 second video is necessary communication, but it is not sufficient evidence. Conversely, a pile of engine libraries without a coherent downloadable journey—or a binary without its required source/notices/data credits—is not the v1 product.

## Post-v1 directions

The Milestone 10 gate decides which direction has earned investment:

- more measured bodies and atmospheres;
- SPICE kernels, spacecraft trajectories, and wider verified epoch ranges;
- a stable C++ or C API and documented embedding examples;
- local headless batch generation and, separately, cluster execution;
- scripted tours, annotations, and educational packages;
- celestial-body impacts as a distinct product program if promoted: start with scripted visualization and conservation diagnostics, separate approximate deformation/ejecta effects, and treat coupled hydrodynamics/material failure as a research simulator rather than a renderer feature;
- star-system/sector coordinates for genuine interstellar navigation;
- rings, comets, relativistic views, or simulation research.

None is automatically next. Evidence, users, maintainership, data rights, and funding decide.

## Open product decisions

Decisions close when the milestone that can produce evidence needs them; they are not all front-loaded into terrain work.

| Decision | Deadline |
|---|---|
| Reference platform, candidate minimum GPU, initial display scenario, and capability profile | M0 |
| Earth hero region, exact DEM product, global synthetic treatment, and any real global albedo/land mask | Candidate in M0; final before M6 implementation |
| Whether the route uses a benchmark-only altitude clamp while general collision remains absent | M5 |
| Supported astronomical date interval and user-facing date controls | M7 |
| Exact Gaia DR3 selection/quality recipe, magnitude/page layout, and bright-star supplement policy | M9 |
| Whether the committed user replay feature exposes a stable/public route file format or keeps it internal/versioned | M10 productization |
| Normal-UI versus diagnostic provenance detail | M10 productization |
| Release asset/install/cache size and bundled versus separate data | Preliminary budgets in M4/M9; final in M10 |
