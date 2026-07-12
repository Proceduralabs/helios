# Feasibility assessment

Assessment date: 12 July 2026. **Historical baseline inspected:** upstream commit `99c16345a8080bb23920e282b549965e8711df8e`, the original four-commit design-only state. This file and the companion clarification documents are currently a working-tree revision; replace this sentence with their resulting commit/tag when they are committed so the final assessment is reproducible.

This is an engineering assessment, not a promise, quote, legal opinion, or scientific certification. It should be revised whenever a milestone produces evidence that changes an assumption.

## Executive verdict

Helios is **realistic as a multi-year planetary-rendering research project** whose v1 result is one polished Solar-System-to-surface vertical slice on one reference platform.

It is **not realistic as a short solo build or as a general SpaceEngine competitor under the original literal claims**. At the historical baseline above, the upstream repository contained four commits and documentation only: no source, shaders, CMake project, dependency lock, tests, CI, executable, captures, or benchmarks. The expanded documents remain design intent until implementation evidence exists; every performance or portability claim is unvalidated.

The highest-risk but credible work is continuous terrain LOD/residency, coordinate-frame transitions, physical-range HDR, and packaging the result across drivers. The claims that needed correction were interstellar scope, sub-millimetre precision at every distance, bitwise-identical floating-point GPU terrain on all hardware forever, literal “any date,” a release-year hardware target, and equal three-platform expectations.

## What is already well judged

- Camera-relative GPU coordinates, CPU doubles, and reversed-Z are suitable foundations for Solar-System scale.
- A cube-sphere quadtree with fixed grid patches, screen-space error, CPU traversal first, and bounded residency is a sensible solo-project LOD strategy.
- Deferring ECS, a render graph, a dedicated render thread, mesh shaders, n-body physics, walking, and black holes avoids several classic engine-project traps.
- Each roadmap milestone has a visible or measurable artifact and a “fail cheaply” intent.
- One detailed body, one atmosphere preset, one real terrain region, and one benchmark route are defensible v1 cuts.
- Gaia DR3 is a strong authoritative upstream catalog if Helios treats it as an offline, queryable data source rather than a monolithic runtime asset.

## Claim-by-claim assessment

Ratings mean:

- **Feasible:** the design direction is conventional enough to plan, though still unimplemented.
- **Feasible with a bounded contract:** realistic only after the stated range, fallback, or acceptance condition is adopted.
- **Not credible as written:** the literal wording contradicts the proposed architecture or platform guarantees.
- **Post-v1 program:** possible work, but large enough to deserve a new plan after Milestone 10.

| Expectation | Rating | Assessment and required correction |
|---|---|---|
| Interactive interplanetary-to-surface route | Feasible, high effort | This is the coherent core. Say “free-flight camera hovering about 2 m above a rendered surface,” not “standing.” Define the benchmark path and background-streaming behavior. |
| Interstellar-to-surface travel | Not credible as written | The only root is Solar-System barycentric and the intended numerical range is about 10¹³ m. Real interstellar navigation needs a galactic/sector or star-system frame above it. Keep it post-v1. |
| Sub-millimetre precision at every altitude | Not credible as written | A root `double` at 10¹³ m has 1.953125 mm spacing. Promise sub-mm local resolution near a body, and separately specify root placement error and screen-space jitter. |
| Reversed-Z over the v1 range | Feasible with a bounded contract | Camera-relative rendering plus an infinite far projection is sound. Query a supported 32-bit float depth format, define the near-plane policy, and test impostor depth. |
| Continuous LOD to about 1 m | Feasible, high effort | One-metre near-camera sample spacing is possible. It is not one-metre measured terrain accuracy. Residency, cross-face seams, normals, morphing, and memory bounds are the real risks. |
| One real Earth region | Feasible | Select one exact DEM product and footprint. Datum conversion, no-data handling, resampling, license, hash, and source resolution must be part of the deliverable. |
| Global “real data first” terrain | Post-v1 program | v1 has one measured region plus explicit synthetic coverage/detail. Global and multi-body products bring large storage, licensing, and preprocessing programs. |
| Gaia DR3 star source | Feasible with a bounded contract | Gaia DR3 contains 1,811,709,771 sources. A 24-byte minimal record would already be about 43.5 GB before indices; it cannot be one mid-range-GPU SSBO. Pin an ADQL/selection policy, use magnitude/quality bands and HEALPix tiles, and stream or bundle a v1 subset. |
| Gaia solves the HYG licensing problem | Largely yes, with attribution | ESA states Gaia data are open and free to use provided `ESA/Gaia/DPAC` is credited. Helios needs that credit in its manifest/UI/about material and should follow ESA's requested mission/DR3 scholarly citation practice where applicable. Gaia's bright-end and catalog-quality limitations still need handling. |
| Recognizable constellations using Gaia alone | Feasible, validation required | Gaia is not optimized for the very brightest stars and has a bright limit around G≈3 for the full astrometric solution. Validate the named constellation set; if rows are missing, use a separately reviewed bright-star supplement rather than silently inventing them. |
| One Bruneton-style atmosphere and physical-range HDR | Feasible, major subsystem | Reference code and papers exist, but Vulkan integration, units, pre-exposure, eclipse transitions, tone mapping, NaN control, and reference comparisons remain substantial work. |
| Planets in “real positions for any date” | Not credible as written | Analytic series, rotation models, time conversions, and later SPICE kernels all have finite inputs and accuracy regimes. Name the series variant, frame, time scale, supported interval, and tolerance per body. |
| Bitwise-identical GPU terrain on every machine forever | Not credible as written | Vulkan is not cross-implementation pixel exact, floating controls are limited, drivers lower SPIR-V differently, and MoltenVK adds SPIR-V→MSL translation. Exactness requires deliberately integer/fixed-point products; portable floating output needs a tolerance contract. |
| Deterministic shared edges and caches | Feasible | Use canonical integer sample coordinates, explicit integer algorithms for exact products, and cache keys containing generator/data/compiler/backend versions. |
| Mid-range “2019 or newer” GPU | Not a support specification | API version alone does not name feature bits, limits, queue topology, VRAM, resolution, driver, or performance. Replace the date with a tested capability/profile matrix. |
| Linux, Windows, and macOS/MoltenVK release parity | High-risk and under-scoped | Start with one reference platform. Compile/smoke-test the others where possible. MoltenVK is a portability implementation with capability-dependent descriptor/buffer behavior and another shader translation step. |
| Async compute always meeting 1–2 ms | Not portable as a guarantee | Vulkan does not guarantee independent hardware queues or beneficial overlap. The baseline must serialize compute on the graphics queue; measured overlap is an optimization. Bound dispatch sizes so one tile cannot exceed the budget catastrophically. |
| Headless support | Feasible after splitting the term | v1 needs swapchain-free smoke/benchmark execution. A local batch tile tool is a separate deliverable. Multi-node/cluster generation stays post-v1. |
| Replaceable subsystems | Feasible as internal modularity | Static libraries alone do not produce a replacement contract. Define typed interfaces, allowed dependencies, ownership, schemas, and tests. Do not advertise a stable SDK until one exists. |
| Water or ocean simulation | Deliberately out of scope | It adds an unrelated real-time fluid/content program and is not required by the product definition. Keep it excluded. |
| Celestial-body impact mechanics | Post-v1 program | A scripted collision plus conservation checks and procedural crater/ejecta is plausible. Self-gravitating hypervelocity impact physics with equations of state, fragmentation, SPH/MPM/Eulerian hydro, and long-term debris is a research simulator, not a renderer subfeature. Name the fidelity tier before estimating it. |
| AGPL plus commercial licensing | Viable in principle; not operational | The current legal identity/contact fields and signing workflow are incomplete, and the CLA is explicitly unreviewed. Contributor licenses consolidate relicensing rights, not contributor copyright ownership. Counsel and a real process are release prerequisites. |

## Precision reality

The root-frame contradiction is arithmetic, not opinion. For a binary64 value near each distance, the spacing between adjacent representable values is:

| Magnitude | `double` spacing | What it means |
|---:|---:|---|
| 10⁷ m | about 1.86 nm | Body-fixed near-surface math has enormous local headroom |
| 10⁹ m | about 0.119 µm | The original Milestone 1 route is numerically comfortable |
| 10¹² m | about 0.122 mm | Still below one millimetre, before other errors |
| 10¹³ m | exactly 1.953125 mm | A literal sub-mm root-position promise is impossible with one `double` |
| 4×10¹⁶ m | 8 m | Nearest-star scale needs another coordinate tier |

Hierarchical frames solve the rendering problem by keeping precision-sensitive work near a local origin. They do not make a root binary64 value more precise. The validated contract should therefore report at least three different budgets:

1. root-frame body-centre/model placement error;
2. body-fixed/local camera and terrain numeric error; and
3. projected screen-space jitter along the benchmark route.

## Gaia DR3 implications

Gaia DR3 is more detailed than HYG in source count, astrometry, photometry, and available astrophysical products, but “more detailed” does not mean every row belongs in a renderer or every field is complete.

The official DR3 documentation reports:

- 1,811,709,771 total sources;
- about 1.46 billion with five- or six-parameter astrometry;
- G-band mean magnitude for about 1.806 billion;
- BP/RP magnitudes for about 1.54/1.55 billion;
- a reference epoch of J2016.0 for the main astrometric solutions.

A credible v1 pipeline is:

1. pin `gaiadr3.gaia_source` and the documentation release;
2. store the exact ADQL query, selected columns, quality filters, and execution date;
3. preserve `source_id`, reference epoch, astrometric/photometric quality flags, and missing-value semantics;
4. partition by HEALPix and magnitude/quality band;
5. produce a bounded bundled catalog or streamed tile set;
6. convert Gaia photometry to display flux/color through a documented approximation rather than treating BP−RP as a blackbody truth;
7. validate bright named stars and dense fields; and
8. ship the required ESA/Gaia/DPAC credit and follow the official requested paper-citation practice where applicable.

The Gaia Archive remains the authoritative queryable upstream. v1 does **not** need to download or retain the complete table; preserving the exact archived query result or bulk-input hashes is enough to make its bounded derived catalog reproducible.

Official references:

- [Gaia DR3 documentation and source counts](https://gea.esac.esa.int/archive/documentation/GDR3/)
- [Gaia DR3 credit and citation instructions](https://gea.esac.esa.int/archive/documentation/GDR3/Miscellaneous/sec_credit_and_citation_instructions/)
- [Gaia DR3 contents and bright limit](https://www.cosmos.esa.int/web/gaia/dr3)
- [ESA explanation of bright-star limitations](https://www.cosmos.esa.int/web/gaia/dr3-how-bright-are-the-stars)

## Rough effort model

Assumptions: one experienced C++/Vulkan graphics engineer; one reference desktop platform; reuse of the named permissive libraries; no production editor, networking, walking, water, impact simulation, public SDK, or global measured dataset; a bounded Gaia DR3-derived v1 catalog rather than full-catalog runtime streaming.

| Milestone | Full-time engineer-weeks | Dominant uncertainty |
|---|---:|---|
| M0 — skeleton and capability spike | 6–12 | Reproducible toolchain, Slang/MoltenVK, CI, platform choice |
| M1 — precision laboratory | 8–14 | Frame types, reparenting, depth, screen-space validation |
| M2 — cube-sphere | 6–10 | Cross-face topology, balancing, property tests |
| M3 — procedural planet | 8–14 | Exact seams, generator versioning, GPU/CPU determinism tiers |
| M4 — bounded continuous descent | 14–24 | Coupled cache, IO, GPU lifetime, morphing, culling, performance |
| M5 — surface-scale view | 6–10 | Near-field material stability and tangent basis |
| M6 — one measured region | 8–16 | Datum/projection, preprocessing, rights, provenance, seams |
| M7 — Solar-System stage | 10–18 | Time scales, frames, analytic series, rotations, validation |
| M8 — atmosphere and HDR | 12–20 | Physical units, LUT validation, exposure transitions, artifacts |
| M9 — Gaia DR3 sky | 8–16 | Query policy, bake/index, photometry, bright-star validation |
| M10 — vertical slice release | 8–16 | Optimization, UX, packaging, drivers, evidence bundle |

Total planning range: **94–170 full-time engineer-weeks**, roughly **two to three-and-a-half engineer-years** before ongoing support. A part-time maintainer at about half of a full-time week should expect approximately **four to seven calendar years**, because the raw arithmetic also needs rework, documentation, releases, upstream changes, and periods of lower availability.

Adding full Gaia runtime streaming, equal release support on three OSes, or a stable SDK can each add **many months to years**, depending on the support contract. A credible planetary-impact simulator would add a separate multi-year range depending on fidelity.

The estimate assumes specialist review is available when needed. Geodesy/datum conversion, astrometry/time standards, radiometry, numerical GPU behavior, and data/legal review are separate competencies; learning or sourcing them can lengthen calendar time even when the coding estimate is unchanged.

These estimates are deliberately ranges. The project should update them from actual throughput after M0, M2, M4, and M6 rather than pretending initial numbers are a schedule.

## Risk register and retirement gates

| Risk | Probability / impact | Earliest honest retirement test |
|---|---|---|
| Coordinate design jitters or loses state during frame changes | Medium / critical | M1 route to 10¹³ m, repeated in/out reparenting, numeric and pixel reports |
| Chosen Vulkan profile fails on intended hardware/MoltenVK | Medium / high | M0 capability probe and triangle/compute/headless smoke on named devices |
| Cross-face LOD produces cracks or illegal neighbors | Medium / high | M2 exhaustive/property tests plus debug edge captures |
| Generator contract cannot be portable and exact | High / high | M3 CPU/GPU hash and tolerance matrix before procedural content grows |
| Tile lifetime/synchronization causes spikes or corruption | High / critical | M4 adversarial route, validation layers, bounded dispatch and retire tests |
| DEM licensing/datum pipeline is wrong | Medium / high | M6 manifest review, reference points, independent bake round-trip |
| Astronomical claims exceed model range/accuracy | Medium / high | M7 comparison grid over the published supported interval |
| HDR produces unstable exposure or non-finite values | Medium / high | M8 scripted transitions and finite-buffer reductions |
| Gaia asset is too large, visually incomplete, or badly filtered | Medium / high | M9 size budget, named-star test, density tests, provenance audit |
| Cross-platform packaging consumes the roadmap | High / medium | One reference platform selected in M0; others remain experimental until passing |
| Legal/commercial model cannot accept contributions | Medium / high | Counsel-reviewed documents, entity/contact, CLA workflow before code PRs |
| Solo-project fatigue/maintenance | High / high | Milestones split into releasable artifacts; scope remains behind M10 gate |

## Recommended minimum viable contract

To keep the project credible, v1 should promise:

- Solar-System scale only;
- one Earth hero body and one measured DEM region;
- one reference OS/GPU profile, with all others explicitly experimental;
- a named resolution, frame-time percentile, RAM/VRAM cap, and route;
- sub-mm local numeric resolution plus a separate screen-space jitter limit;
- about 1 m procedural/geometry sample spacing, never mislabelled as data accuracy;
- versioned deterministic exact products and tolerance-based portable float products;
- a Gaia DR3-derived catalog whose query, size, quality, and attribution are recorded;
- an interactive explorer and evidence bundle, not a stable SDK;
- no water, walking, general surface collision/response, dynamics, or celestial-impact simulation; a benchmark-only altitude clamp is permitted.

## Positioning reality

Large-scale precision rendering and open space viewers already exist. [Cesium Native](https://cesium.com/learn/cesium-native/ref-doc/) provides high-precision C++ geospatial libraries and streaming; [Celestia](https://github.com/CelestiaProject/Celestia) is an established open-source space visualizer. Helios therefore should not treat “large scale” or “open source” alone as a durable differentiator.

The defensible identity is the combination of an inspectable modern Vulkan implementation, a narrowly validated orbit-to-surface route, and visible separation of measured, model-derived, procedural, and display-only content. If those qualities do not survive implementation, the product case weakens even if the visuals are attractive.

## What would change this verdict?

- A successful M1 may reduce coordinate/precision risk substantially.
- A smooth, memory-bounded M4 route on the minimum GPU would be the first evidence that v1 is more than a research sketch.
- A clean M6 data/provenance pipeline and M9 Gaia pipeline would support the research/education positioning.
- Failure to obtain acceptable cross-platform behavior should reduce the support matrix, not force a renderer rewrite solely to preserve a marketing sentence.
- Evidence of contributor/user demand can justify SDK or additional-body work after M10; it should not be assumed now.

## External technical references

- [Vulkan invariance: Vulkan is not pixel exact across implementations](https://docs.vulkan.org/spec/latest/appendices/invariance.html)
- [Vulkan floating-point control limitations](https://docs.vulkan.org/features/latest/features/proposals/VK_KHR_shader_float_controls2.html)
- [Vulkan queue mapping and concurrency are implementation-defined](https://docs.vulkan.org/guide/latest/queues.html)
- [Vulkan depth-format support and query requirements](https://docs.vulkan.org/guide/latest/depth.html)
- [MoltenVK runtime guide and capability-dependent extensions](https://github.com/KhronosGroup/MoltenVK/blob/main/Docs/MoltenVK_Runtime_UserGuide.md)
- [Bruneton's documented atmosphere implementation and tests](https://ebruneton.github.io/precomputed_atmospheric_scattering/)
- [JPL DE440/DE441 finite ephemeris ranges](https://ssd.jpl.nasa.gov/doc/de440_de441.html)
