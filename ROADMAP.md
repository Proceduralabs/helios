# Roadmap

Every milestone ends in something you can see — a screenshot, a captured camera path, a measurable test. No six-month dark periods. Milestones are ordered; dates are deliberately absent (this is a part-time solo project until it isn't).

Progress key: ⬜ not started · 🟨 in progress · ✅ done

## Phase 0 — Foundation

Goal: prove the coordinate strategy before building anything on top of it.

### ⬜ Milestone 1 — "Impossible depth range"
Sunlit sphere, star background, 1 m diagnostic grid. Reversed-Z, camera-relative rendering, camera position held as `dvec3` in a hierarchical frame.
- **Proof:** scripted camera descent from 10⁹ m to 1 m altitude with zero visible jitter, Tracy connected, ImGui overlay running.
- This milestone *is* the precision architecture. If it fails, everything above it is redesigned — cheaply, because nothing exists yet.

### ⬜ Milestone 2 — Cube-sphere revealed
Six-face cube-sphere, quadtree subdivision, wireframe patch boundaries, LOD debug coloring.
- **Proof:** no face gaps, no illegal neighbor configurations (max 1 level difference), verified by a debug edge view.

## Phase 1 — The planet

### ⬜ Milestone 3 — First procedural planet
Deterministic GPU height generation (band-limited noise stack), normals, slope/altitude coloring.
- **Proof:** bitwise-identical heights on shared patch edges and cube-face seams.

### ⬜ Milestone 4 — Continuous descent
Bounded tile cache, generation budget (~1–2 ms/frame on async compute), geomorphing with skirts as safety net, frustum + horizon culling.
- **Proof:** orbit-to-valley scripted descent with a hard cap on resident tiles and no frame spikes; patch count scales with screen-space demand, not tree depth.

### ⬜ Milestone 5 — Surface-scale view
Local tangent-frame camera, multi-band procedural material detail near the ground.
- **Proof:** eye-height shot toward a curved horizon; centimeter camera movement stays stable; no texture swimming. (Free flight — still not walking. Walking is not on this roadmap.)

### ⬜ Milestone 6 — One real place
Offline GDAL pipeline: one real DEM region (single dataset, single body) baked to cube-face tiles with mip pyramid, min/max and error metadata.
- **Proof:** a recognizable real mountain or crater next to a reference photo, with documented datum handling and honest resolution limits — no fake "1 m accuracy" from 30 m data.

## Phase 2 — The solar system

### ⬜ Milestone 7 — Atmosphere and sunset
Bruneton precomputed scattering (one Earth preset), physical light units, pre-exposure, histogram auto-exposure, tone mapping.
- **Proof:** stable exposure and finite HDR values through a scripted day-to-night-to-eclipse sequence.

### ⬜ Milestone 8 — Solar-system stage
VSOP87 + ELP2000 ephemerides, TDB/Julian-date time system, body rotation models. Sun, Earth, Moon + planets placed for any epoch.
- **Proof:** positions match an external ephemeris service to documented tolerance at three test epochs.

### ⬜ Milestone 9 — Real sky
HYG star catalog (~120k stars): parallax → position, B-V → blackbody color, magnitude → physical flux through an energy-conserving PSF.
- **Proof:** recognizable constellations from Earth's surface; bright-star directions match catalog at a chosen epoch.

### ⬜ Milestone 10 — The vertical slice
Polish exactly one journey: interplanetary view → approach → descent → surface-scale shot. Package a downloadable build.
- **Proof:** 30–60 s captured sequence; runs on one stated minimum GPU with no validation errors and bounded memory.

**Milestone 10 is the decision gate.** Only after it ships does anything below get promoted to the active roadmap.

## Beyond the gate (unordered, unscheduled, deliberately vague)

- SPICE/DE440 ephemerides upgrade (moons, small bodies, spacecraft trajectories)
- GPU n-body (symplectic integrators; direct O(N²) before anyone says Barnes-Hut)
- More atmospheres, more bodies, ring systems
- Gaia-subset star streaming
- Comet tails, planetary destruction, relativistic rendering (black hole lensing via geodesic ray marching, neutron stars)
- Collision + surface locomotion (the actual "walking" project)
- Headless/cluster generation for research use

These are listed so nobody files "when black holes?" issues. The answer is: after Milestone 10.
