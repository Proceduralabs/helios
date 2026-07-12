# Third-party software notices

No third-party code is vendored or linked yet. Names in architecture/decision documents are candidates, not evidence that a dependency is present or approved.

When Milestone 0 adds a dependency, this file records at least:

| Field | Required value |
|---|---|
| Name | Upstream project/library name |
| Version | Exact tag plus commit/hash where applicable |
| Source | Authoritative upstream URL |
| SPDX/license | Exact license identifier/text |
| Helios use | Linked, compiled tool, vendored source, runtime component, etc. |
| Modifications | Patch list or “none” |
| Required notice | Verbatim attribution/notice where required |
| Review | Date and reviewer/check result |

Dependency fetches use immutable revisions. Transitive dependencies and compiler/runtime redistribution requirements are included rather than hidden behind the top-level library name.

Datasets and catalogs are not software notices. Gaia DR3, DEMs, atmosphere parameters, and generated/baked assets use per-release manifests under [docs/DATA_AND_PROVENANCE.md](docs/DATA_AND_PROVENANCE.md), and their required notices are packaged alongside this file in a release.
