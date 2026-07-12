# Decision log

This log records accepted directions, their tradeoffs, and what evidence can reopen them. It does not mean the choices are implemented or validated. Architecture detail lives in [ARCHITECTURE.md](ARCHITECTURE.md).

Decision status:

- **Accepted:** current direction.
- **Provisional:** enough to run a spike, but the named milestone must confirm it.
- **Superseded:** retained for history with a link to the replacement.

To reopen a decision, add evidence, state which requirement changes, describe migration cost, and propose a validation plan. Enthusiasm alone is not evidence.

## D-001 — Solar-System coordinates use frame-tagged doubles and local GPU floats

**Status:** Accepted · **Date:** 12 July 2026 · **Validate:** M1

**Decision:** v1 uses a Solar-System barycentric root, body-fixed frames, tile-local coordinates, and a camera-relative GPU representation. CPU world/domain positions use frame-tagged `double`; the renderer performs one explicit same-frame subtraction and casts the small relative result to `float32`.

**Why:** millimetre fixed-point barely spans 10¹³ m and does not solve interstellar scope. CPU doubles are adequate for body centres at Solar-System range, while body-fixed/local frames retain far more precision for surface work. Typed frames prevent semantically invalid arithmetic better than a comment saying “absolute.”

**Important limit:** binary64 spacing at 10¹³ m is 1.953125 mm, so this does not support a literal sub-mm root-position promise. Genuine interstellar navigation requires a future sector/star-system tier; it is not a constant change.

**Rejected for v1:** one global `float`, GPU FP64 world coordinates, one millimetre `int64` universe, or undocumented floating-origin shifts scattered through views.

**Revisit if:** M1 cannot meet the local transform and screen-space jitter contract, or interstellar navigation becomes an accepted product requirement.

References: Cozzi & Ring, *3D Engine Design for Virtual Globes* (relative-to-eye material); [VALIDATION.md](VALIDATION.md).

## D-002 — Reversed-Z with a queried 32-bit floating depth attachment

**Status:** Accepted · **Date:** 12 July 2026 · **Validate:** M0/M1

**Decision:** use reversed-Z, an infinite far projection, `GREATER`/`GREATER_OR_EQUAL`, and a near plane chosen from camera context. Query `D32_SFLOAT` or an accepted 32-bit-float depth/stencil format rather than assuming one attachment enum.

**Why:** it preserves early/hierarchical depth behavior and provides a strong precision distribution for the camera-relative scene. A logarithmic fragment-depth path complicates depth-aware effects and writes without first proving reversed-Z inadequate.

**Consequence:** distant impostor/low-detail depth and near-plane changes need explicit transition tests.

**Revisit if:** a supported profile cannot provide the required float depth format or measured artifacts fail the v1 route.

References: [Vulkan depth format guidance](https://docs.vulkan.org/guide/latest/depth.html), Nathan Reed, *Depth Precision Visualized*.

## D-003 — Cube-sphere quadtree grid patches before research LOD backends

**Status:** Accepted · **Date:** 12 July 2026 · **Validate:** M2–M4

**Decision:** use a cube-sphere quadtree, initially 33×33 grid patches, screen-space geometric error, split/merge hysteresis, one-level neighbor balance, parent-consistent morphing, and skirts only as residency safety. Start with CPU traversal/culling.

**Why:** the algorithm is understandable, property-testable, compatible with tiled DEM assets, and achievable for a solo project. CBT/meshlet/mesh-shader schemes add implementation/debugging risk before the core data/residency model is known.

**Consequence:** face/corner topology, neighbor transformations, and error metadata must be completely specified; a cube-sphere is not seam-free automatically.

**Revisit if:** M4 profiling shows CPU selection/draw submission is a material bottleneck after simpler indirect-draw improvements.

Reference: Strugar, *Continuous Distance-Dependent Level of Detail for Rendering Heightmaps* (2009).

## D-004 — Vulkan 1.3 is a capability profile, not a hardware-year promise

**Status:** Provisional · **Date:** 12 July 2026 · **Validate:** M0

**Decision:** start with Vulkan 1.3, dynamic rendering, synchronization2, timeline semaphores, buffer device address, a declared descriptor-indexing minimum, timestamp queries, and a writable float depth format. Query every feature/limit and publish a device report.

**Why:** this keeps the native backend modern without adopting Vulkan 1.4 solely for version novelty. The required features align with the renderer direction.

**Consequence:** “2019 or newer” is removed as a support claim. M0 may reduce a requirement if a bounded fallback preserves the product and materially improves hardware coverage.

**Revisit if:** capability spikes exclude intended reference devices or the renderer does not actually need a proposed hard requirement.

## D-005 — Slang shaders compiled offline to SPIR-V

**Status:** Provisional · **Date:** 12 July 2026 · **Validate:** M0/M3

**Decision:** use pinned Slang binaries/source revisions, offline compilation, checked-in build integration, reflection/ABI tests, and explicit optimization/float profiles. Do not check generated SPIR-V into source unless release/offline-build needs justify it.

**Why:** modules, generics, and multi-target tooling suit a compute-heavy renderer better than a growing preprocessor convention.

**Consequence:** Helios owns another toolchain dependency. macOS adds SPIR-V→MSL translation through MoltenVK, so M0 must exercise actual shaders, not just compile a trivial file.

**Fallback:** documented GLSL/glslang path only if Slang blocks supported platforms or reproducible builds; a theoretical fallback is not maintained in parallel for free.

**Revisit if:** pinned releases cannot produce stable ABI/reflection behavior or MoltenVK translation prevents required shaders.

Reference: [Slang project](https://github.com/shader-slang/slang).

## D-006 — Analytic ephemerides first; SPICE later behind explicit contracts

**Status:** Provisional · **Date:** 12 July 2026 · **Validate:** M7

**Decision:** begin with an exact selected VSOP87 variant for major planets and an exact selected lunar series/backend, behind an interface that names centre, axes, time scale, supported interval, and uncertainty/tolerance. Add SPICE/DE only after v1 unless validation proves the analytic backend inadequate.

**Why:** a bounded analytic backend is easier to distribute and exercise for the v1 visual stage. SPICE adds kernel discovery/versioning, global-state boundaries, time/frame inputs, and more operational surface than “read planet coordinates.”

**Consequence:** “any date” is rejected. The chosen series variants, transforms (including ecliptic/ICRF differences), two-part time representation, leap-second inputs, body rotations, and per-body tolerances are M7 deliverables.

**Revisit if:** the accepted date/tolerance contract cannot be met, or spacecraft/small-body trajectories become the next post-v1 program.

References: [NAIF SPICE time subsystem](https://naif.jpl.nasa.gov/pub/naif/toolkit_docs/C/req/time.html), [JPL DE440/DE441 ranges](https://ssd.jpl.nasa.gov/doc/de440_de441.html).

## D-007 — Gaia DR3 is the authoritative v1 star source

**Status:** Accepted · **Date:** 12 July 2026 · **Validate:** M0/M9

**Decision:** replace HYG with ESA Gaia Data Release 3 as the upstream catalog. v1 ships a reproducibly selected, versioned Gaia-derived catalog organized by magnitude and HEALPix; it does not place all ~1.81 billion sources in one resident buffer.

**Why:** Gaia DR3 provides far more extensive astrometry and photometry, rich quality/provenance fields, and open/free use with required ESA/Gaia/DPAC credit. The archived [official HYG repository](https://github.com/astronexus/HYG-Database) and [license snapshot](https://raw.githubusercontent.com/astronexus/HYG-Database/main/LICENSE), checked 12 July 2026, publish CC BY-SA 4.0 rather than public-domain terms.

**Consequence:** M9 becomes a real data-pipeline milestone: exact ADQL/bulk recipe, quality filters, missing fields, reference epoch, compact format, counts/hashes, photometric approximation, bright-star validation, and attribution/citations. Gaia is not optimized for the very brightest stars, so a separately reviewed supplement remains an explicit decision if validation needs it.

**Rejected:** an unversioned download, a hand-edited star list, all-source resident SSBO, or silently substituting missing astrometry/colour with measured zeros.

**Revisit if:** Gaia archive/redistribution terms or bright-source completeness cannot support the v1 sky, or a later full-galaxy streaming product needs another source model.

References: [Gaia DR3 documentation](https://gea.esac.esa.int/archive/documentation/GDR3/), [credit instructions](https://gea.esac.esa.int/archive/documentation/GDR3/Miscellaneous/sec_credit_and_citation_instructions/).

## D-008 — Adopt focused libraries; do not outsource the renderer architecture

**Status:** Provisional · **Date:** 12 July 2026 · **Validate:** M0 and per dependency

**Decision:** candidates are volk, vk-bootstrap, VulkanMemoryAllocator, SDL3, Dear ImGui/ImPlot, Tracy, enkiTS behind a thin facade, doctest, KTX-Software, meshoptimizer, ERFA, and GDAL in offline tools only. Pin immutable revisions and record licenses/notices.

**Why:** rebuilding platform, allocation, UI, profiling, image, time-standard, and geospatial foundations adds risk without differentiating Helios. A general renderer/framework would obscure the Vulkan/planet architecture the project exists to explore.

**Consequence:** no dependency is approved merely because it appears in this paragraph. M0 verifies maintenance, license, platform, binary/runtime, offline build, and transitive dependency implications.

**Rejected initially:** a custom allocator/job system/test framework, runtime GDAL, an ECS framework, or a package-manager policy that fetches mutable branches.

**Revisit if:** a candidate's actual license/platform/maintenance cost fails the dependency review.

## D-009 — No ECS, render thread, general render graph, or custom job system until evidence

**Status:** Accepted · **Date:** 12 July 2026 · **Validate:** M4 profiling

**Decision:** use explicit body/tile structures, handwritten pass/synchronization code, main-thread orchestration/recording, and a small job-system facade.

**Why:** these abstractions have real payoff at sufficient scale and equally real upfront/debug cost. v1 has a small body set and a bounded pass plan.

**Consequence:** handwritten synchronization still needs a precise resource-state/lifetime model and tests. “No render graph” is not permission for scattered barriers.

**Revisit if:** measured pass/resource dependency complexity or CPU recording time becomes a dominant problem.

## D-010 — v1 is a surface-scale rendering explorer, not walking or simulation

**Status:** Accepted · **Date:** 12 July 2026 · **Validate:** M10

**Decision:** v1 ends with a free-flight camera roughly 2 m above one Earth terrain region. It has no avatar, vehicle, general collision detection/response, walking, gameplay, fuel/orbital dynamics, n-body evolution, water, or celestial-impact simulation. A benchmark-only altitude clamp is allowed.

**Why:** each excluded item adds another product and content/physics/interaction validation surface. The rendering transition is already a multi-year project.

**Consequence:** user-facing language says “hover” or “camera,” never “standing.” The camera may need a simple safety-floor policy for the benchmark, but that is not a general collision engine.

**Revisit if:** v1 ships and evidence/users justify a separately scoped program.

## D-011 — Plan AGPL-3.0-only plus commercial licensing, but do not claim operations before legal readiness

**Status:** Provisional legal/business policy · **Date:** 12 July 2026 · **Validate:** before accepting code contributions

**Decision:** repository-owned code is intended to remain available under AGPL-3.0-only, while a rights-complete commercial license may be offered for proprietary uses. External code is accepted only under a counsel-reviewed contributor agreement granting the necessary relicensing rights.

**Why:** this aims to preserve an open edition while allowing proprietary customers to fund development.

**Important correction:** contributors retain copyright under the draft CLA; the needed thing is consistent relicensing/sublicensing rights, not “100% copyright consolidation.” Third-party libraries and datasets are not magically dual-licensed by Helios.

**Consequences:**

- The legal identity, contact, agreement, privacy/record process, company-contribution path, and signature workflow must be complete first.
- Plain-language AGPL trigger/derivative-work guidance is fact-specific and must be reviewed by counsel.
- Helios-owned files cannot incorporate code whose rights do not permit the intended commercial license. This is a commercial-relicensing policy, not a claim that GPL and AGPL can never interact under their own terms.
- `AGPL-3.0-only` is the selected community identifier. Any later open license would be additional rather than silently changing existing files to “or later.”

**Revisit if:** counsel, contributor adoption, distribution channels, or customer evidence shows the model is unworkable.

## D-012 — Determinism has exact and tolerance-based tiers

**Status:** Accepted · **Date:** 12 July 2026 · **Validate:** M3

**Decision:** exact cross-platform bit identity is promised only for canonical products implemented with specified exact operations. Floating GPU output is backend-repeatable or cross-device tolerance-conformant according to a declared profile. Generator/data/tool/backend versions are part of every cache key.

**Why:** Vulkan does not guarantee pixel-exact output across implementations, floating controls do not cover every operation, drivers may contract/approximate differently, and MoltenVK translates SPIR-V to MSL. “No fast-math” is necessary but insufficient for “every machine forever.”

**Consequence:** seams/topology can remain exact while nonessential heights/normals use validated error bounds. If product requirements demand exact heights everywhere, use an integer/fixed-point canonical generator and pay its performance/algorithm cost knowingly.

**Revisit if:** M3 demonstrates a broader exact profile on all supported backends with a specified algorithm, or tolerances visibly break caching/research requirements.

References: [Vulkan invariance](https://docs.vulkan.org/spec/latest/appendices/invariance.html), [float-controls limitations](https://docs.vulkan.org/features/latest/features/proposals/VK_KHR_shader_float_controls2.html).

## D-013 — “Honest scale” is the product identity

**Status:** Accepted · **Date:** 12 July 2026 · **Validate:** M6/M9/M10

**Decision:** the explorer and asset formats classify important content as measured/observed, model-derived, procedural/synthetic, or display transform. The active source/model/generator manifests remain inspectable.

**Why:** large-scale rendering and open space viewers already exist. Visible provenance unifies Helios's real-data, reproducibility, extensibility, and education/research intentions into a concrete differentiator.

**Consequence:** “1 m geometry” cannot be marketed as “1 m terrain accuracy”; model positions need date/frame/tolerance; exposure and false colour cannot be confused with source data.

**Revisit if:** never for honesty itself; implementation/UI details may evolve with usability evidence.

## D-014 — One reference platform earns support first

**Status:** Provisional · **Date:** 12 July 2026 · **Validate:** M0/M10

**Decision:** choose one reference OS/toolchain/device profile in M0. Linux, Windows, and macOS can have build/experimental coverage, but only profiles passing the full validation route are called supported.

**Why:** driver, packaging, shader, queue, and performance differences are significant. Equal three-platform expectations before one renderer exists multiply uncertainty.

**Consequence:** macOS/MoltenVK receives an explicit portability spike. A successful compile or triangle does not imply v1 support.

**Revisit if:** automation/hardware/contributor capacity makes another profile cheap to validate continuously.

## D-015 — Tools and runtime share versioned asset schemas

**Status:** Accepted · **Date:** 12 July 2026 · **Validate:** M0/M6/M9

**Decision:** add `helios_assets` for bounded binary schemas, manifests, checksums, and readers/writers used by tools and runtime. Source-specific formats stay offline.

**Why:** the DEM and Gaia pipelines are part of the product's correctness. Independent ad-hoc readers/writers would drift and make old caches unsafe.

**Consequence:** every format has versioning, explicit units/coordinates/endian, size limits, hashes, malformed-input tests, and migration/rebuild policy.

**Revisit if:** schemas naturally split into more focused libraries after real formats exist; never return to duplicated structs serialized by `sizeof`.

## D-016 — Celestial impacts are a post-v1 fidelity ladder; water simulation is not planned

**Status:** Accepted scope clarification · **Date:** 12 July 2026 · **Validate:** post-M10 proposal

**Decision:** do not plan water/ocean/weather fluid simulation. If celestial-body impacts are promoted after v1, scope them in explicit levels:

1. scripted/kinematic impact visualization with momentum/energy diagnostics;
2. approximate procedural crater, deformation, heat, ejecta, and debris effects;
3. only if justified, offline or research-grade coupled self-gravity/material/hydrodynamics using named methods, equations of state, and conservation validation.

**Why:** “impact mechanics” can mean a visual event or a planetary-material simulator; their cost and claims differ by orders of magnitude. Calling all collision “fluid simulation” obscures the choice.

**Consequence:** celestial impacts do not imply camera/surface collision, walking, general rigid-body gameplay, or water. Even tier 2 needs a terrain event/revision layer, cache-key changes, save/replay provenance, and invalidation/regeneration of affected children, parents, normals, bounds, and geometric-error metadata; it is not a render-pass toggle. The highest-fidelity tier is a separate multi-year research program.

**Revisit if:** M10 has shipped and a concrete fidelity/use-case proposal includes data, compute, validation, and product scope.
