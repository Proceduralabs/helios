# Contributing to Helios

Contributions are welcome — with two hard rules and a few soft ones. The hard rules exist because Helios is dual-licensed (AGPL-3.0 + commercial), and that model is fragile: it survives only if copyright stays consolidated and the codebase stays clean.

## Hard rule 1: sign the CLA

Before your first pull request can merge, you sign the [Contributor License Agreement](CLA.md). It grants Proceduralabs the rights needed to offer your contribution under both the AGPL and commercial licenses. You keep ownership of your work and can reuse it anywhere.

No CLA, no merge. This applies to one-line typo fixes too, because copyright law doesn't care how small a patch is.

## Hard rule 2: no code from incompatible sources

Do **not** copy, port line-by-line, or closely paraphrase code from:

- GPL/LGPL projects — Cosmonium, Stellarium, Celestia, `rantonels/starless`, and most academic black-hole renderers. These are *study-only* references.
- Repositories with **no LICENSE file** (default = all rights reserved).
- Anything you can't name the license of.

Safe sources: MIT, BSD, Apache-2.0, Zlib, public domain — with attribution kept in `THIRD_PARTY_NOTICES` when required. If you're unsure, ask in the PR before writing code. One contaminated snippet forces a rewrite of everything it touched.

Reading a GPL implementation to understand an *algorithm* and then implementing it independently from the paper is fine. Copying its structure is not.

## Before you write code

- Read [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and [docs/DECISIONS.md](docs/DECISIONS.md). PRs that fight a logged decision get closed with a pointer to the log — reopen the decision first (with evidence) if you think it's wrong.
- For anything bigger than a bugfix, open an issue first. Solo-maintainer project: unsolicited 4,000-line PRs are a burden, not a gift.
- Check the [roadmap](ROADMAP.md). Features below the Milestone 10 gate aren't accepted yet, however cool.

## Code expectations

- C++20. Warnings clean at `-Wall -Wextra` / `/W4`. ASan/UBSan build must pass.
- The double/float boundary is sacred: absolute positions live in `helios_universe` as `double`; everything the GPU sees is camera-relative `float32`. A PR that stores an absolute position anywhere else is wrong by construction.
- Terrain generation must stay deterministic — no fast-math, no uninitialized reads, no atomics-order-dependent results in generation kernels.
- New Vulkan objects get debug names. New passes get Tracy zones.
- Tests for `helios_math` (frames, cube-sphere mapping, geodesy) are mandatory with any change there — this is where silent corruption hides.

Every source file carries an SPDX header:

```cpp
// SPDX-License-Identifier: AGPL-3.0-only
// Copyright (c) 2026 Proceduralabs and Helios contributors
```

## Practical flow

1. Fork, branch from `main`.
2. Make the change; keep commits scoped and messages in imperative mood ("Add horizon culling test", not "added stuff").
3. Open the PR; the CLA bot (or, until it exists, a manual checklist) will ask for your CLA signature on first contribution.
4. CI builds + math tests must pass. GPU-dependent behavior gets a `--headless-smoke` check where feasible.

## Not code?

Bug reports with a reproducible camera path, dataset licensing research, DEM pipeline verification against reference imagery, and documentation fixes are all first-class contributions — and none of them can contaminate the license.
