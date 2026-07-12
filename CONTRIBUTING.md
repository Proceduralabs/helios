# Contributing to Helios

> **Current stage: design review only.** Issues that identify factual errors, contradictions, data sources, or design evidence are welcome. Code and pull-request contributions are not being merged yet because there is no source skeleton/CI and the contributor agreement, legal identity, privacy records, and signing workflow are unfinished.

This policy will change when [Roadmap Milestone 0](ROADMAP.md) lands and the legal contribution process is ready.

## Useful contributions today

- Open an issue with a reproducible correction to a technical or scientific claim.
- Propose an exact Earth DEM product/region with datum, resolution, DOI, terms, and known issues.
- Propose/test a reproducible Gaia DR3 ADQL query and quality policy.
- Add primary-source evidence to reopen a decision.
- Review product wording for measured versus model-derived versus procedural claims.
- Identify a Vulkan/MoltenVK capability risk on named hardware with logs/spec references.

Because PR rights cannot be accepted safely yet, current issues/discussions should contain facts, analysis, and links to authoritative sources only—not patches, original assets, source archives, or attached datasets. The maintainer can independently implement uncontroversial corrections until the workflow opens.

## What must happen before code contributions open

- M0 source/build/test/CI skeleton exists.
- The real maintainer/legal entity and contact details replace every placeholder.
- Austrian/EU-qualified counsel reviews the CLA and licensing guidance.
- Individual and company/employer contribution cases are covered.
- A signature, identity, audit, privacy, and retention workflow exists.
- The project consistently chooses `AGPL-3.0-only` and file/document/asset markings.
- Dependency and dataset manifest policies are operational.

## Future hard rule 1: reviewed contributor agreement

Before a contribution can merge, its author will sign the finalized [Contributor License Agreement](CLA.md) or an approved equivalent/company agreement. The intended agreement grants the rights needed to distribute the contribution under AGPL and separate commercial terms. Contributors retain ownership.

This is **relicensing-rights coverage**, not copyright consolidation. It will apply to original code, shaders, documentation, and other copyrightable contributions according to the finalized agreement.

## Future hard rule 2: rights-clean contributions

Do not copy, port line-by-line, translate, or closely paraphrase source into Helios-owned files from:

- GPL/LGPL projects when Helios lacks the rights required for its planned commercial license;
- repositories or snippets with no license;
- papers, tutorials, answers, generated code, or assets whose reuse terms are unknown;
- employer/client work you are not authorized to contribute.

Permissive dependency candidates (MIT, BSD, Apache-2.0, Zlib, public domain) still require license, version, source, modifications, and notices. Ask before implementing if provenance is uncertain.

Studying an algorithm from a paper/specification and independently implementing it is different from copying another project's expressive code/structure, but provenance should still be documented. The project's commercial-rights policy is stricter than a generic AGPL-compatibility statement.

## Before future implementation work

- Read [docs/INDEX.md](docs/INDEX.md), [docs/PRODUCT.md](docs/PRODUCT.md), [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md), and [docs/DECISIONS.md](docs/DECISIONS.md).
- Check [ROADMAP.md](ROADMAP.md). Features below the Milestone 10 gate are not accepted v1 work.
- Map the work to a product requirement and validation test.
- For anything larger than a narrow fix, open an issue/design note first.
- Reopen a logged decision with evidence and a migration/validation proposal rather than bypassing it in code.

## Future code expectations

- C++20; warnings clean at the selected Clang/GCC/MSVC levels; ASan/UBSan clean where supported.
- Frame-tagged coordinate types enforce the boundary: `helios_universe` owns frame relationships/astronomical transforms; `helios_planet` may own tagged body-local positions; only scene assembly creates camera-relative GPU floats.
- No untagged “absolute” position or GPU subtraction of huge world values.
- Terrain changes state their determinism tier and update generator/cache versions when output meaning changes.
- New Vulkan objects receive debug names; passes and material work receive useful Tracy zones.
- Math, asset parsing, cube topology, time/frame, and cache changes include unit/property/malformed-input tests.
- Resource lifetimes include frames-in-flight and timeline retirement tests.
- Data/catalog changes update manifests, citations, counts, hashes, and provenance UI.
- New performance claims include a device/scenario manifest and percentile data.

Every project-owned C++/Slang source file is intended to carry:

```cpp
// SPDX-License-Identifier: AGPL-3.0-only
// Copyright (c) 2026 the Helios authors and contributors
```

This marking policy is provisional until legal review. External contributors should not submit source changes before contributions open; maintainer-owned M0 bootstrap work may land after the current file-license policy is reviewed.

## Future pull-request flow

1. Fork and branch from `main`.
2. Link an accepted issue/requirement and state the evidence produced.
3. Keep commits scoped and imperative.
4. Complete the finalized contributor-rights workflow.
5. Let CPU, shader, asset, platform, and GPU tests appropriate to the change pass.
6. Disclose every third-party source/data input and generated-code tool.
7. Update user-facing limitations when a fallback or support tier changes.

GPU-dependent behavior uses the smallest reproducible headless fixture plus named-device route evidence where required. A screenshot does not replace validation.

## Community files still needed

Before active contributor recruitment, Helios should also publish a code of conduct, security-reporting policy, governance/maintainer policy, issue/PR templates, and legal/privacy details for contributor records.
