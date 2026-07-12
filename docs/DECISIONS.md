# Decision log

Contested choices, the ruling, and why the runner-up lost. Reopening a decision requires new evidence, not new enthusiasm. Format is deliberately terse.

---

### D-001 · Global coordinates: `double` + hierarchical frames — not int64 fixed-point
Int64 at mm units barely covers 10¹³ m and not interstellar; at 1 m units it can't represent smooth camera motion. Double gives 2.2 mm ULP at 10¹³ m, and with body-fixed frames nothing precision-sensitive ever operates that far from a frame origin. Fixed-point's determinism benefit targets multiplayer physics we don't have. References: Thorne, *Floating-Origin* (2005); Cozzi & Ring, *3D Engine Design for Virtual Globes* (RTE chapter).

### D-002 · Depth: reversed-Z `D32_SFLOAT` infinite far — not logarithmic depth
Reversed float depth gives near-logarithmic distribution while preserving early-Z/hierarchical-Z; log depth writes fragment depth and complicates every depth-aware effect. Altitude-dependent near plane. References: Nathan Reed, *Depth Precision Visualized*; Outerra blog.

### D-003 · LOD: CDLOD-style quadtree grid patches — not CBT/meshlets, not geometry clipmaps
33×33 patches, SSE selection, geomorph + skirts, CPU traversal first. CBT (Dupuy) is finer-grained but dramatically higher implementation and debugging risk for a part-time solo project, and integrates poorly with tiled DEM caches. Mesh shaders remain a possible later backend consuming the same visible-patch list. Reference: Strugar, *Continuous Distance-Dependent LOD for Rendering Heightmaps* (2009).

### D-004 · Vulkan 1.3 baseline — not 1.4
1.4 adds nothing decisive for this renderer and cuts hardware coverage. Required features: dynamic rendering, sync2, timeline semaphores, buffer device address, descriptor indexing, scalar block layout, multi-draw indirect + count, D32 depth, timestamp queries.

### D-005 · Shaders: Slang — not GLSL
Large compute-heavy shader codebase ahead; Slang's modules and generics pay for themselves. Mitigations that made this acceptable: pinned compiler version in-repo, offline compilation to SPIR-V in the build, shader ABI tests (bindings, push-constant sizes, struct offsets, matrix convention). GLSL + glslang remains the documented fallback if Slang breaks tooling.

### D-006 · Ephemerides: VSOP87 + ELP2000 first — SPICE/DE440 later, behind the same interface
VSOP87 is self-contained, integrable in an evening, covers the planets; ELP2000 covers the Moon. CSPICE's global-state C API and kernel management are real friction with zero visual payoff at this stage. `helios_universe` API is designed so SPICE replaces the backend without touching callers. Internal standard: TDB expressed as Julian date, J2000 epoch, ICRF axes. Time-scale conversions via `liberfa/erfa` (BSD-3) — not hand-rolled, and not the sloppy "UTC = TDB − ΔT" shortcut.

### D-007 · Stars: HYG (~120k) — not Gaia
Gaia DR3 is 1.8 billion stars and a data-engineering project of its own. HYG covers everything naked-eye-plus and fits in one SSBO. Gaia subsets return post-Milestone 10.

### D-008 · Libraries: adopt, don't rebuild
volk, vk-bootstrap, VulkanMemoryAllocator, SDL3, Dear ImGui + ImPlot, Tracy, enkiTS (behind a thin facade), doctest, KTX-Software, meshoptimizer. FetchContent, no vcpkg/Conan. Explicitly rejected: bgfx/The-Forge (own the Vulkan backend — that's the point of the project), OpenSceneGraph/Ogre (wrong architecture era), Eigen (MPL + compile times, unneeded features), ECS frameworks (no entity problem yet), GDAL at runtime (offline tools only).

### D-009 · No render graph, no render thread, no custom job system — until profiling says otherwise
Hand-written barriers are fine below ~15 passes. Main-thread recording with a worker pool is easier to debug and won't be the bottleneck (the GPU will). Every one of these is a known solo-project time sink with deferred payoff.

### D-010 · v1 scope: "surface-scale viewing" — not "walking realism"
Walking implies collision, locomotion, near-field content, centimeter materials, contact shadows — a second project. The v1 contract: seamless free flight to 2 m above a heightfield surface, stable precision, continuous LOD, procedural near-field detail, no collision, no avatar. Written down so scope creep has to argue with a document.

### D-011 · License: AGPL-3.0 + CLA + commercial dual licensing — not permissive, not source-available
Goals: genuinely open source, free for individuals/researchers/small studios, companies shipping proprietary products pay. Permissive (MIT/Apache) gives companies everything for free. Source-available (BSL/PolyForm) loses the open-source label and repels contributors. AGPL's network copyleft makes proprietary use commercially unattractive, which is exactly the pressure that sells commercial licenses (Qt/MongoDB model). Requires 100% copyright consolidation ⇒ CLA from the first external commit, and zero GPL-contaminated code ever.

### D-012 · Determinism is a feature, not an optimization target
Same seed + tile ID ⇒ bitwise-identical output on all hardware. Constrains noise implementations (no fast-math in generation kernels, careful transcendental use) but buys reproducible bug reports, cacheable tiles, and research credibility. Decided now because retrofitting determinism is nearly impossible.
