# Architecture

This document describes the intended structure before the code exists, so early contributors and future-me can't drift from it silently. Changes to anything here go through [DECISIONS.md](DECISIONS.md).

## Module layout

Seven static libraries plus the app. Dependencies point strictly downward.

```
helios_app        main loop, camera controller, debug UI wiring
   │
   ├── helios_planet     cube-sphere quadtree, tile cache, GPU generation, atmosphere
   ├── helios_universe   reference frames, time, ephemerides, star catalog, body registry
   ├── helios_sim        (stub — future n-body)
   │        │
   │        ▼
   ├── helios_render     thin scene→Vulkan bridge: passes, per-frame upload, exposure
   │        │
   │        ▼
   ├── helios_vk         device, swapchain, VMA, bindless table, timeline sync
   │
   ├── helios_math       double-precision vectors/quats, frames, geodesy, cube-sphere mapping
   └── helios_core       platform, logging, config, job-system wrapper, profiling macros

tools/   (link helios_core/math only)
   dem_bake      GeoTIFF DEM → quantized cube-face tile pyramid
   star_bake     HYG/Gaia CSV → binary catalog
```

The boundary that matters most: **`helios_universe` owns all double-precision positional truth.** Nothing above it stores absolute positions. `helios_planet` receives positions already expressed in a body-fixed frame; `helios_render` receives positions already camera-relative in `float32`. Precision bugs come from smearing the double→float conversion across a codebase — here it happens in exactly one place.

`helios_vk` knows nothing about planets or frames. It exposes device/queues, a bindless descriptor arena, VMA-backed resource creation, timeline semaphores, and a minimal pass helper. It is not a render graph; with fewer than ~15 passes, hand-written barriers are simpler and easier to debug.

No ECS. A `Body` struct in a vector with index handles covers ~10 planets and a terrain system. Revisit only if a future phase produces thousands of heterogeneous dynamic entities.

## Coordinates and precision

Three frame tiers, and only three:

1. **Barycentric (root).** Solar System Barycenter, ICRF-aligned, meters, `double`. Ephemerides evaluate directly into this frame. ULP of a double at 10¹³ m ≈ 2.2 mm — five orders of magnitude below ephemeris uncertainty for placing planet *centers*, which is all this frame does.
2. **Body-fixed.** One per body, rotating with it (IAU rotation models). All terrain, atmosphere, and any future surface physics live here, where distances are ≤ ~10⁷ m and double ULP is nanometers.
3. **Tile-local.** Each terrain tile stores vertices as `float32` relative to its own origin (a `dvec3` in the body-fixed frame). At the deepest LOD a tile is meters across, so float32 relative to tile center is effectively exact.

Per frame, on the CPU, in double: `rel = to_camera_frame(object) − camera.position`, then cast to `float32` and upload. The view matrix is rotation-only — translation is zero by construction. Never subtract large positions in float; never subtract on the GPU (consumer FP64 rates make it pointless, and the object count needing double math is small).

Camera reparenting: entering a body's sphere of influence (~100× body radius, with hysteresis) reparents the camera to the body-fixed frame; leaving reverses it. Reparenting is one double-precision transform.

Depth: reversed-Z, `D32_SFLOAT`, infinite far plane, `GREATER_OR_EQUAL` compare, altitude-dependent near plane (0.05 m at the surface, meters in space). Distant bodies render as impostors/low-detail spheres with analytic depth — a 3-pixel disc cannot expose meter-scale depth error.

## Planet rendering

Cube-sphere quadtree per body. One canonical integer-lattice mapping: `face + integer coords → exact cube vector → sphere direction → radial displacement`. Edge vertices on adjacent faces derive from the same canonical integer edge representation — never from independently evaluated float UVs.

- 33×33 vertex grid patches, one shared static index buffer.
- Screen-space geometric-error LOD selection (split ≈ 1.5–2 px, merge ≈ 1 px for hysteresis), max one level difference between neighbors, balanced after selection.
- Parent-consistent geomorphing over the last 25–30% of a node's interval; short skirts as a safety net for residency gaps, not as the crack solution.
- CPU quadtree traversal and culling first (frustum in double, conservative spherical horizon test). GPU-driven culling with `vkCmdDrawIndexedIndirectCount` is a later upgrade that consumes the same visible-patch list. Mesh shaders/CBT are a research backend, not the foundation.

## Terrain generation

GPU compute, deterministic, per tile:

```
LOD update (CPU)          → request queue (priority = screen-space error)
streaming thread          → baked DEM pages from disk → staging → transfer queue
async compute queue       → pass chain into an atlas slot:
                              1. base field (band-limited noise stack, or DEM + detail synthesis)
                              2. optional passes (erosion residual, craters, …)
                              3. finalize: normals, min/max reduction
residency (CPU)           → tile drawable when compute_timeline ≥ tile.gen_value
                            parent stays resident until all children ready
```

Rules that keep it honest: every pass reads/writes only atlas slots + a per-tile param block (bindless), so passes compose without descriptor churn; a per-frame GPU generation budget (~1–2 ms, timestamp-measured) so streaming never starves rendering; noise is evaluated in 3D planet coordinates with only the frequency bands representable at the tile's LOD, and parent/child low-frequency terms are identical — a child adds a residual band, it never re-rolls the parent's terrain.

Erosion is non-local (drainage crosses tile borders): it runs offline in `dem_bake`, or later as a cached near-field residual with generous halos. It never silently changes terrain that something else already depends on.

## Sync model

Two frames in flight. One timeline semaphore per queue: graphics (frame counter), async compute (tile generation counter), transfer (upload counter). A tile is drawable when the compute timeline passes its generation value — no per-tile fences, no callbacks. Swapchain binary semaphores are quarantined inside `helios_vk`.

Threading: main thread does sim → LOD → cull → record → submit, with parallel-for on a small job system (enkiTS behind a ~50-line facade). One or two pinned IO threads. No dedicated render thread until profiling demands one.

## HDR pipeline

Scene-referred linear RGB with physical intent (irradiance W·m⁻², solar disk radiance ~2×10⁷ W·m⁻²·sr⁻¹ — overflows FP16 raw). Therefore: pre-exposure (store `radiance × previousFrameExposure` in RGBA16F), FP32 for exposure reductions and sun-adjacent intermediates, log-luminance histogram metering with percentile rejection, split adaptation rates, manual ISO/aperture/shutter available from day one. Stars carry catalog-derived flux through an energy-conserving PSF — never fixed bright pixels. Tone mapper is swappable; false-color luminance debug view is mandatory.

## Determinism and headless mode

Same seed + same tile ID ⇒ bitwise-identical terrain, on every machine, forever. This is what makes bug reports reproducible and research use viable. A `--headless` mode initializes Vulkan without a swapchain and can generate tiles / run the pipeline for CI smoke tests and offline experiments. Multi-node/cluster rendering is out of scope; the renderer sits behind a clean enough interface that someone else can attempt it without forking the planet system.
