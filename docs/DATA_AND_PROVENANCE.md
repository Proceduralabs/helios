# Data and provenance contract

Status: **accepted policy; dataset selections mostly proposed**. No runtime datasets are currently vendored or released.

This document owns the distinction between measured data, model outputs, procedural synthesis, and display transforms. It also defines the metadata that lets a user reproduce a Helios asset. It is not a substitute for reviewing each source's actual terms.

## Core rule

Helios must be able to answer, for any important scene component:

1. Where did it come from?
2. Which exact release, query, or algorithm produced it?
3. In what coordinates, units, datum, reference epoch, and time scale?
4. What was measured, what was inferred by a model, and what was synthesized?
5. Which transformations were applied?
6. What resolution, uncertainty, coverage, and known issues remain?
7. What license/terms, acknowledgement, citation, and redistribution conditions apply?
8. Which hashes and tool versions reproduce the runtime artifact?

If those questions cannot be answered, the asset is not release-ready.

## Four source classes

| Class | Definition | Allowed user-facing language |
|---|---|---|
| **Measured / observed** | Comes from an instrument or survey product with provenance and uncertainty | “Derived from [exact product] at [native spacing/accuracy]” |
| **Model-derived** | Evaluated from a named mathematical model, series, kernel, fit, or convention | “Computed using [model/version] over [supported interval/tolerance]” |
| **Procedural / synthetic** | Generated to provide coverage, plausible structure, or detail | “Procedural,” “synthetic,” or “artistically generated”; never “measured” |
| **Display transform** | Alters presentation after source/model values exist | “Tone mapped,” “false colour,” “exposure adjusted”; never additional source accuracy |

An asset may combine classes, but the boundary must remain inspectable. Example: “30 m measured elevation resampled into runtime tiles, plus procedural bands below 30 m, then slope-coloured for display.”

## Dataset register

This register describes intended sources. A row marked **candidate** is not permission to ship it.

| Domain | Intended source | Status | Rights/provenance action |
|---|---|---|---|
| Stars | ESA Gaia Data Release 3, primarily `gaiadr3.gaia_source` | **Accepted upstream; v1 query TBD** | Gaia data are open/free with `ESA/Gaia/DPAC` credit according to the official DR3 instructions. Record the exact acknowledgement, papers, query, selected fields, filters, and archive date. |
| Bright-star completeness | Gaia DR3 first; a small supplement only if named-star validation fails | **Open decision** | Gaia is not optimized for the brightest stars. Any supplement gets its own license/provenance review and must not silently be called Gaia data. |
| Earth terrain | One exact SRTM/NASADEM or other suitable product and one named region | **Candidate** | Select the product, DOI, tile list, horizontal/vertical datum, resolution, void policy, and product-specific terms. NASA's general policy is not a substitute for the selected product page. |
| Moon/Mars terrain | LOLA/MOLA family | **Post-v1 candidate** | Pick exact PDS products/releases and citations only when promoted. Do not list an agency name as a license. |
| Planet positions | Exact VSOP87 variant and coefficient source | **Proposed** | Record source/distribution rights, axes, origin, units, truncation, supported dates, and validation reference. |
| Moon position | Exact ELP implementation/series variant | **Proposed** | Record coefficient provenance, license, axes/origin, units, terms retained, and validation reference. |
| Body dimensions/mass | Exact source for radii, reference ellipsoids/shapes, mass parameters, and eclipse geometry | **Proposed** | Pin report/kernel/product edition, units, uncertainty, body convention, and which value each subsystem uses. |
| Body rotations | Named IAU report/model and parameter source | **Proposed** | Record model edition and per-body convention; distinguish Earth orientation simplifications. |
| Time conversion | ERFA plus versioned leap-second/Earth-orientation inputs as required | **Proposed** | Pin software/data versions and licenses; state update and offline behavior. |
| Solar radiometry/geometry | Exact solar spectrum or irradiance convention, solar radius/angular-size model, and reference distance | **Proposed** | Record source/version, units, integration/conversion, uncertainty, and whether a display approximation replaces spectral data. |
| Global Earth appearance | Explicitly synthetic material by default; optionally one exact low-resolution land/ocean/albedo product | **Open decision** | Never imply procedural continents are measured Earth. Any real global layer gets its own resolution, rights, transform, and known-issues manifest. |
| Gaia photometric calibration | Exact Gaia DR3 passband/zero-point/calibration source used for G/BP/RP conversion | **Proposed** | Pin equations/tables/version, units, extinction/color assumptions, and citations; separate calibrated source values from display approximation. |
| Atmosphere parameters | Documented Earth preset and chosen Bruneton-style reference | **Proposed** | Record parameter values/units, code/paper version, license if code is reused, LUT-generation version, and approximation notes. |

The former HYG candidate is intentionally removed: the archived [official repository](https://github.com/astronexus/HYG-Database) and [license snapshot](https://raw.githubusercontent.com/astronexus/HYG-Database/main/LICENSE), checked 12 July 2026, publish the database under CC BY-SA 4.0 rather than public domain. Gaia DR3 avoids that share-alike dependency, but still carries credit/citation conditions.

## Gaia DR3 policy

### Why Gaia

The Gaia Archive is the canonical queryable upstream star source because Gaia DR3 provides vastly more sources and richer astrometric/photometric metadata than HYG. v1 need not download or retain the complete table. The official documentation reports 1,811,709,771 total sources, with different fields available for different subsets.

That scale is an offline data-engineering advantage, not permission to ignore runtime budgets. Even 24 bytes per source would exceed 43 GB before indices, quality data, or compression.

### v1 output

Milestone 9 produces a **Helios Gaia DR3 catalog release**, not a copy of the entire archive in one file. Its release manifest must contain:

- archive/schema: `gaiadr3` and exact source table(s);
- documentation/version date;
- exact ADQL query or reproducible bulk-source recipe, plus immutable bulk-input or archived-query-result hashes;
- selected columns and reason for each;
- null/missing-value behavior;
- astrometric and photometric quality policy;
- treatment of non-stellar/classification candidates;
- reference epoch and propagation rules;
- magnitude bands and HEALPix partition level;
- source count before and after every filter;
- canonical row order (`source_id`), units, null/NaN, duplicate, rounding, and byte-serialization rules;
- conversion equations for position, flux, colour, and PSF energy;
- named-bright-star and density validation results;
- archive retrieval date, tool versions, and input/output hashes;
- required ESA/Gaia/DPAC credit text and the official requested paper citations.

Likely minimal fields include `source_id`, `ra`, `dec`, `ref_epoch`, `phot_g_mean_mag`, and available BP/RP photometry. Parallax, proper motion, and radial velocity are optional per row and cannot be assumed complete. The bake stores epoch-tagged astrometric inputs; `helios_universe`, not the baker, owns time propagation. Distance inference must not blindly invert low-quality or negative parallax.

### Runtime shape

The format should support:

- a bounded bundled bright/visible catalog for v1;
- HEALPix and magnitude-tier lookup;
- incremental streaming if later views justify faint sources;
- a compact GPU record separate from richer CPU/provenance metadata;
- exact source-ID traceability back to the manifest/query;
- conservative page-neighbor/motion bounds so high-proper-motion sources cannot disappear after crossing a static HEALPix boundary;
- versioned binary headers, checksums, size limits, and endian rules.

Pages store catalog-epoch astrometric records. `helios_universe` propagates them according to the scenario `Instant` and emits runtime direction/flux inputs; `helios_render` consumes those prepared values and owns only residency/PSF drawing.

The renderer should consume runtime render-direction/flux records prepared by the catalog layer. “Apparent place” is not claimed until observer-dependent parallax, aberration, light-time, and any refraction contract are explicitly selected. The renderer should not know ADQL or Gaia archive schemas.

### Known caveats

- Gaia's source table contains astronomical sources, not a guarantee that every row is a normal star.
- Field coverage varies; full astrometry, colour, radial velocity, and astrophysical parameters are not available for every source.
- Gaia is not designed for the very brightest naked-eye stars; v1 must test recognizable named stars/constellations explicitly.
- BP/RP colour is affected by stellar spectrum, extinction, and instrument passbands. A blackbody approximation is a display model, not measured spectral radiance.
- Catalog values have uncertainties and known issues that quality filters cannot erase.

Official sources:

- [Gaia DR3 documentation](https://gea.esac.esa.int/archive/documentation/GDR3/)
- [Gaia DR3 credit and citation instructions](https://gea.esac.esa.int/archive/documentation/GDR3/Miscellaneous/sec_credit_and_citation_instructions/)
- [Gaia DR3 contents](https://www.cosmos.esa.int/web/gaia/dr3)
- [Gaia bright-star explanation](https://www.cosmos.esa.int/web/gaia/dr3-how-bright-are-the-stars)

## Terrain policy

### Selection record

Before a DEM is downloaded for a release, record:

- product title, version, DOI/landing page, distributor, and retrieval date;
- exact files/tiles and hashes;
- geographic footprint and coordinate reference system;
- horizontal sample spacing and documented horizontal accuracy;
- vertical units, datum, scale/offset, and documented vertical accuracy;
- no-data sentinel, voids, water treatment, and quality masks;
- license/terms, citation, attribution, and redistribution permission;
- whether the source contains non-NASA or otherwise restricted components.

“NASA data” or “30 m” alone is not enough.

### Bake record

The DEM bake must record:

- GDAL and Helios tool versions;
- source-to-body/cube-face transforms;
- datum/geoid/ellipsoid conversion;
- resampling kernel and nodata behavior;
- face orientation, border/halo, and corner rules;
- quantization scale/offset and error bound;
- mip/min/max/geometric-error derivation;
- procedural residual inputs and the exact frequency range added;
- output schema version, file hashes, and validation report.

Measured and procedural heights should remain separable in metadata even if the GPU consumes a combined tile.

## Asset manifest

Every release dataset or generated artifact has a machine-readable manifest. The final serialization is chosen during Milestone 0; conceptually it contains:

```yaml
schema: helios.asset-manifest/v1
asset_id: earth-region/example
asset_version: 1
classification: measured_plus_procedural
components:
  - component_id: measured_dem
    classification: measured
    source:
      product: "TBD exact DEM product"
      release: "TBD"
      uri: "TBD authoritative landing page"
      retrieved_at: "YYYY-MM-DD"
      sha256: "..."
    rights:
      terms: "TBD exact identifier or URL"
      redistribution_review: "pending|approved|restricted"
      required_credit: ["..."]
      requested_citations: ["DOI or full citation"]
    coverage:
      spatial: "TBD footprint/body"
      temporal_or_epoch: "not-applicable"
    quality:
      native_spacing_m: null
      horizontal_accuracy_m: null
      vertical_accuracy_m: null
      uncertainty_model: "TBD"
      no_data_policy: "TBD"
      known_issues: ["..."]
    coordinates:
      source_crs: "TBD"
      horizontal_datum: "TBD"
      vertical_datum: "TBD"
      reference_frame: "TBD or not-applicable"
  - component_id: procedural_detail
    classification: procedural
    generator:
      id: helios.terrain.example
      version: "..."
      algorithm_sha256: "..."
      executable_sha256: "..."
      seed: "..."
      parameters_sha256: "..."
      backend_profile: "..."
    rights:
      terms: "TBD project-output terms"
      required_credit: []
transforms:
  - tool: helios_dem_bake
    version: "..."
    executable_sha256: "..."
    toolchain_manifest_sha256: "..."
    parameters_sha256: "..."
outputs:
  - path: "..."
    format_version: 1
    sha256: "..."
aggregate_output:
  terms: "TBD combined-output terms and retained source conditions"
  required_notices: ["..."]
  claims:
    procedural_min_wavelength_m: null
validation:
  report_sha256: "..."
```

Actual manifests must not contain `TBD`. Domain extensions add mandatory fields rather than omitting irrelevant-looking facts: terrain adds tile lists/datum/resampling/quantization; Gaia adds query/filter/count/photometric rules; model inputs add validity interval/frame/units/tolerance; reference evidence adds snapshot/terms/citation.

## Runtime provenance UI

The normal UI may stay minimal, but the diagnostic/provenance overlay must show or link to:

- active body/region and data classification;
- dataset/catalog release and manifest hash;
- native measured resolution versus current rendered sample spacing;
- procedural generator and version;
- epoch, coordinate frame, and model backend;
- exposure/tone-map mode;
- known limitations and a path to complete notices.

Screenshots/captures in an evidence bundle embed the manifest IDs so an attractive image cannot be detached from its provenance accidentally.

## Licensing separation

- Helios's AGPL/commercial options apply only to code and material the project has rights to license that way.
- Dependencies, compiler runtimes, datasets, catalogs, papers, logos, and assets keep their own rights and notices.
- A commercial Helios license cannot waive ESA/NASA/dataset attribution or a dependency license.
- `THIRD_PARTY_NOTICES.md` tracks software; data manifests track datasets/catalogs; release packaging includes both.
- Dataset outputs may need to be distributed separately from code even when both are free to use.

## File-format and input safety

Offline tools and the runtime parse untrusted or corrupted large files. Every format must have:

- magic bytes and explicit schema version;
- endian/width/unit declarations;
- checked dimensions, offsets, counts, and multiplication;
- upper size/allocation limits;
- per-file or per-chunk checksum;
- rejection of unsupported versions and non-finite values;
- fuzzable parsers and small malformed fixtures;
- atomic cache writes and validation before rename/use.

Provenance without robust parsing is not reproducibility.

## Release gate

A dataset is releasable only when its manifest has no placeholders, rights have been reviewed, required notices are packaged, the bake is reproducible from recorded inputs, validation thresholds pass, and the UI can identify measured versus synthetic content.
