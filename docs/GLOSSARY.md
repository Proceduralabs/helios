# Glossary

This glossary defines how Helios uses its specialist terms. It is intentionally plain-language; implementation documents may add mathematical detail without changing these meanings.

## Product and evidence

**Benchmark route**  
A versioned camera path, scene manifest, quality preset, and date used to compare precision, visual transitions, performance, memory, and output between builds.

**Evidence bundle**  
The machine-readable report, logs, screenshots/capture, hardware/software manifest, and test results attached to a milestone. A marketing video alone is not an evidence bundle.

**Hero body / hero region**  
The one planetary body and one measured terrain region that receive v1's highest detail and validation. Earth is the accepted v1 hero body; the region and exact data product are still to be selected.

**Reference explorer**  
The interactive desktop application shipped by v1. It demonstrates internal engine libraries but is not itself a promise of a stable engine SDK.

**Seamless**  
No explicit scene switch or blocking loading screen interrupts the validated route. Background streaming, generation, fallback tiles, and representation changes still occur.

**Vertical slice**  
One narrow, polished journey that exercises every required layer end to end. For Helios, Milestone 10 and v1 mean the same vertical slice.

## Scale, frames, and time

**Absolute position**  
An imprecise phrase avoided in APIs. Helios instead names the frame and type: barycentric, body-fixed, tile-local, or camera-relative.

**Barycentre / barycentric frame**  
The centre of mass and associated reference frame of a system. Helios's v1 root is the Solar System Barycentre, aligned to a documented inertial convention.

**Body-fixed frame**  
A coordinate frame attached to and rotating with one body. Terrain coordinates remain stable in this frame while the body moves through the Solar System.

**Camera-relative rendering**  
Converting object positions to small offsets from the camera on the CPU before sending `float32` values to the GPU. It avoids subtracting two huge, imprecise floats in a shader.

**Epoch**  
The instant at which an astronomical position or catalog value is defined. Gaia DR3's main astrometric solutions use reference epoch J2016.0; model evaluation can target other instants according to its contract.

**Frame reparenting**  
Changing the coordinate frame used to express a camera pose/state while preserving the same physical position, orientation, and, where applicable, velocities.

**ICRF / ICRS**  
International Celestial Reference Frame/System. Closely related inertial astronomical conventions used to express directions and positions. A data pipeline must state which one its source and output use rather than treating the names as decorative.

**Julian date**  
A continuous astronomical day count. High-precision implementations commonly use two parts instead of one `double` so a large day number does not erase sub-second resolution.

**Local tangent frame**  
An orientation basis at a surface point: typically east, north, and up (or a documented variant). In Helios it is a camera/view basis inside a body-fixed domain, not a fourth global storage tier.

**TDB**  
Barycentric Dynamical Time, a time scale used for Solar-System ephemeris work. It is not interchangeable with UTC.

**Tile-local frame**  
A small coordinate system centred on one terrain tile. Vertices can use `float32` here because the values are metres or kilometres rather than astronomical distances.

**UTC / TAI / TT / UT1**  
Different civil, atomic, terrestrial, and Earth-rotation time scales. Converting between them can require leap-second or Earth-orientation data. Helios must state which conversions it supports.

**ULP**  
Unit in the last place: the gap between adjacent floating-point values at a given magnitude. A `double` has tiny ULP near a body and about 1.953125 mm ULP at 10¹³ m.

## Terrain and data

**Cube-sphere**  
A planet representation created by mapping the six faces of a cube onto a sphere. It avoids polar singularities but requires explicit face-edge and corner rules.

**Datum**  
The reference surface, coordinate convention, and vertical/horizontal basis used by a geospatial dataset. Combining DEMs without handling their datums can move or distort terrain.

**DEM**  
Digital Elevation Model: a grid or raster of measured/derived surface elevation. Its grid spacing, vertical error, coverage, and no-data areas are separate properties.

**Geometric error**  
A bound or estimate of how far a lower-detail representation can deviate from a more detailed one. Projected into pixels, it drives screen-space-error LOD selection.

**Geomorphing**  
Moving vertices gradually between parent and child representations to reduce visible LOD pops.

**HEALPix**  
Hierarchical Equal Area isoLatitude Pixelization of a sphere. Gaia source IDs encode approximate sky regions using nested HEALPix; Helios can use it to partition a star catalog.

**LOD**  
Level of detail: selecting more samples/geometry where they matter on screen and fewer where they do not.

**Measured resolution**  
The spatial support/grid/accuracy of the source dataset. It is never improved merely by interpolating or adding procedural noise.

**No-data value**  
A marker that says a dataset has no valid measurement at a sample. It must not silently become zero elevation.

**Procedural detail**  
Synthetic information generated by an algorithm. It may be visually plausible and deterministic, but it is not measurement.

**Quadtree**  
A hierarchy where each square tile can split into four children. Helios uses one per cube face to choose planet detail.

**Sample spacing**  
The distance between generated/rendered samples near a location. “About 1 m sample spacing” describes geometry density, not source-data accuracy.

**Screen-space error (SSE)**  
Estimated geometric error after projection to pixels. A tile refines when the expected on-screen error exceeds a threshold.

**Skirt**  
Extra downward geometry at a tile edge used to hide temporary gaps. It is a safety net, not the primary seam-correctness method.

## Astronomy and stars

**ADQL**  
Astronomical Data Query Language, used to query archives such as Gaia. A Helios catalog release records its exact query and filters.

**Apparent magnitude**  
A logarithmic measure of how bright an astronomical source appears from an observer. It is not a linear light value and needs a documented conversion for rendering.

**Ephemeris**  
A table or model that gives positions (and often velocities) of astronomical bodies as a function of time.

**Gaia DR3**  
The European Space Agency Gaia mission's third data release. It contains about 1.81 billion sources with varying sets of astrometric, photometric, spectroscopic, and classification fields. Helios uses a versioned derived subset/tile set, not an assumption that all rows are equally complete stars.

**Parallax**  
The apparent positional shift used to estimate stellar distance. Inverting a noisy or negative parallax naively is not a generally valid distance estimator.

**Proper motion**  
The angular motion of a source across the sky over time. Propagating a star from its catalog epoch requires the appropriate fields and model.

**PSF**  
Point-spread function: how an ideal point source spreads over pixels. For stars it controls apparent size/energy distribution and must not create brightness from nothing.

**SPICE**  
NASA/JPL's observation-geometry system and kernel formats. SPICE includes more than a planetary-position file: time, frames, constants, orientations, and kernel management are part of correct use.

## Rendering and GPU

**Async compute**  
Compute work submitted so it may overlap graphics. Vulkan exposes queues, but independent hardware execution and a speedup are not guaranteed; Helios needs a serialized fallback.

**Bindless / descriptor indexing**  
A resource-access pattern in which shaders index large descriptor arrays. Actual limits and feature bits vary by implementation and must be queried.

**HDR**  
High dynamic range. Internally it means representing a broad range of scene light values; it does not by itself mean an HDR monitor output mode.

**MoltenVK**  
A Vulkan portability implementation over Apple's Metal API. It translates Vulkan behavior and SPIR-V shaders to Metal equivalents and therefore needs a dedicated capability/validation tier.

**Pre-exposure**  
Scaling scene light values before storing them in a finite render target, using exposure information, to preserve useful precision and avoid overflow. Camera cuts and sudden bright sources need reset rules.

**Reversed-Z**  
A depth convention that maps far geometry toward zero and near geometry toward one, usually with a floating-point depth buffer and `GREATER` comparison. It improves precision distribution for large view ranges.

**Slang**  
The proposed shader language/compiler. Helios intends to compile it offline to SPIR-V and pin the compiler/toolchain version.

**SPIR-V**  
The binary intermediate language consumed by Vulkan implementations. Identical SPIR-V does not guarantee bitwise-identical floating results across all GPUs/drivers.

**Timeline semaphore**  
A Vulkan synchronization primitive with an increasing counter value. It can represent queue progress without allocating one fence per tile.

**Vulkan validation layers**  
Development layers that report many incorrect API usages. Zero reported validation errors is required evidence, but it does not prove the renderer is otherwise correct.

## Reproducibility and simulation

**Bitwise-identical**  
Every output bit matches. Helios only uses this phrase for products whose algorithms and supported backends make that guarantee testable.

**Conformance**  
Meeting a defined tolerance or invariant even when output bits differ. Portable floating-point terrain may be conformant without being bitwise-identical.

**Deterministic**  
Given a complete versioned input contract, repeated execution produces the promised exact or tolerance-bounded result. “Seed + tile ID” alone is not complete once algorithms and data change.

**Fluid simulation**  
Numerically evolving fluid/material fields, for example with Eulerian grids, SPH, or MPM. Helios does not plan water/ocean simulation. A future celestial-impact feature only becomes fluid/material simulation if it deliberately adopts one of these physical models.

**Celestial-body impact mechanics**  
A possible post-v1 program with deliberately separate fidelity tiers: (1) scripted/kinematic visualization and momentum/energy diagnostics; (2) approximate procedural deformation, crater, heating, ejecta, and debris effects with terrain revision/invalidation; and (3) research-grade physical simulation requiring self-gravity, material models, equations of state, fragmentation, and conservation validation.

**Headless**  
An overloaded term split by Helios into: (1) swapchain-free automated smoke/benchmark execution, (2) a local batch generation tool, and (3) multi-node/cluster execution. Only the first is foundational v1 scope.

**N-body simulation**  
Numerically integrating gravitational interactions among bodies. v1 displays model-derived ephemeris positions and does not evolve the Solar System dynamically.
