# Licensing

> **Draft policy — not legal advice and not yet operational.** Helios currently has no released code, no verified commercial-licensing contact/process, and an unreviewed CLA draft. Austrian/EU-qualified counsel must review the model and plain-language guidance before external code contributions or commercial licenses are accepted.

Helios intends to make project-owned code available in two ways:

1. a community edition under **GNU AGPL-3.0-only**; and
2. a separately negotiated commercial license for Helios-owned and contributor-licensed code.

Dependencies, datasets, catalogs, documentation, the banner/logo, and other assets may have separate terms. A commercial Helios license cannot override them.

## Community code license — AGPL-3.0-only

The repository includes the full [AGPL-3.0 license text](LICENSE) and selects the `-only` policy. Broadly, the license permits use, study, modification, and redistribution subject to its conditions.

Important nuance:

- Conveying/distributing covered code or modified versions brings source and license obligations described by the license.
- AGPL section 13 adds a source-offer obligation when users interact remotely over a computer network with a **modified version** that supports that interaction.
- Merely making private/internal changes is not accurately summarized as “you must buy a commercial license to keep them private.” Whether distribution, network use, linking, combination, or a particular product creates obligations is fact-specific.
- “Open source project” or “commercial use” alone does not answer whether a larger work must use AGPL. Architecture and derivative/aggregate boundaries matter.

Read the [official AGPL text](https://www.gnu.org/licenses/agpl-3.0.html) and obtain legal advice for an actual product. This file intentionally does not invent a universal linking matrix.

## Planned commercial license

A commercial license is intended for organizations that want rights Helios can grant for a proprietary product, service, or fork without relying on the AGPL path. Scope, term, redistribution, support, seats, revenue, warranty, and price would be negotiated per case.

It is **not available yet**. Before claiming otherwise, the project needs:

- a real legal person/entity and complete notices;
- a verified contact and privacy/record process;
- counsel-reviewed community/commercial terms;
- rights records for every accepted contribution;
- a reviewed individual and, where needed, corporate contributor agreement;
- a functioning signature/audit workflow;
- a dependency/data/asset bill of materials stating what Helios cannot relicense.

Commercial terms would cover only material for which the licensor has the necessary rights. Users still comply with licenses and acknowledgements for SDL, Vulkan libraries, Gaia data, DEMs, and every other third-party component.

Commercial contact: **not yet published**.

## Which path might apply?

This table is orientation only, not a legal conclusion.

| Intended activity | Starting point |
|---|---|
| Read/learn from the code or run the unmodified explorer | Review AGPL-3.0-only; no commercial license is inherently required |
| Make private/internal experiments | Review AGPL-3.0-only; private modification is not automatically a commercial trigger |
| Redistribute Helios or a modified version | Follow AGPL conveyance/source obligations or negotiate commercial rights |
| Run a modified network-interactive Helios service | Review AGPL section 13 carefully or negotiate commercial rights |
| Combine/embed Helios in a closed-source product | Obtain product-specific legal advice; the planned commercial path is intended for this case |
| Publish research using Helios/data | Code and each dataset have separate conditions; cite/credit the data and release papers as required |
| Contribute code or documentation | Wait until contributions open, then use the reviewed CLA/workflow plus the repository contribution policy |

## Why this model?

The intended hypothesis is that a strong copyleft community edition preserves an open implementation, while organizations that require proprietary rights can fund development through commercial terms. That business hypothesis is unvalidated. AGPL adoption can also reduce integration and contribution interest, especially for engine/library code, so it should be evaluated from actual users after a runnable vertical slice exists.

Before external code rights are accepted, the project can still reconsider how **future** material is licensed. Existing AGPL-3.0-only grants cannot be silently withdrawn, and the intended CLA community promise keeps accepted contributions available under that path. Later evidence could justify an additional permissive/source-available license for rights-controlled material, or abandoning the commercial offering while retaining AGPL; it cannot retroactively erase community rights. [docs/DECISIONS.md](docs/DECISIONS.md) D-011 records the current direction and revisit trigger.

## Contributor rights

The draft [CLA](CLA.md) is a copyright **license**, not copyright assignment. Contributors would retain ownership while granting broad sublicensing/relicensing rights needed for community and commercial editions.

That distinction corrects the earlier phrase “copyright stays consolidated.” What must be consistently recorded is the project's right to distribute and commercially license accepted contributions.

The current draft cannot be signed or relied upon. It needs counsel, complete identity/contact fields, a company/employer contribution route, signature records, and a privacy/retention policy.

## Software dependencies

Helios may use permissively licensed third-party libraries. Those components remain under their own licenses and notices in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

The project's rule against copying GPL/LGPL/unlicensed source into Helios-owned files exists because Helios must be able to grant the planned commercial rights to those files. It is not a claim that every GPL/AGPL combination is forbidden under the licenses themselves. External-library use and aggregation need a separate dependency review.

## Data and asset terms

Data provenance and rights are tracked per exact product/release in [docs/DATA_AND_PROVENANCE.md](docs/DATA_AND_PROVENANCE.md). Current direction:

- **Gaia DR3:** authoritative v1 star source. ESA's official instructions say Gaia data are open and free to use provided `ESA/Gaia/DPAC` is credited. The derived Helios catalog must ship that credit and should follow ESA's requested mission/DR3 paper-citation practice where applicable.
- **HYG:** no longer planned. The archived [official repository](https://github.com/astronexus/HYG-Database) and its [license snapshot](https://raw.githubusercontent.com/astronexus/HYG-Database/main/LICENSE), checked 12 July 2026, publish CC BY-SA 4.0 rather than public-domain terms.
- **Earth DEM:** exact product still TBD. NASA/agency branding alone is not a license; verify the selected landing page, components, citation, and redistribution terms.
- **MOLA/LOLA and other bodies:** post-v1 candidates requiring exact PDS product manifests.
- **Gaia/DEM derived files:** a runtime bake does not erase the source terms, acknowledgement, or provenance.

Official Gaia instructions: [Gaia DR3 credit and citation](https://gea.esac.esa.int/archive/documentation/GDR3/Miscellaneous/sec_credit_and_citation_instructions/).

## Current file-license matrix

The repository is already public, so the current position must be explicit even before code exists. The commit authors/rightsholders have published the following licensing intent, but their complete public legal identity and chain-of-title notice are not finalized; that must be resolved before anyone relies on a commercial license or the project accepts outside rights.

| Material | Current copyright-license position |
|---|---|
| Future project-owned C++/Slang code | Intended `AGPL-3.0-only`, with per-file SPDX markings |
| Repository-authored Markdown, including this policy and the draft CLA | Covered by the root AGPL-3.0-only policy unless a file later adds an additional explicit license |
| `assets/banner.png` | Covered by the root AGPL-3.0-only policy as repository project material; no separate trademark policy or trademark grant is established |
| GNU license text in `LICENSE` | Redistributed as the license's official text |
| Third-party code/data/assets | Their own terms; none are vendored today |
| Future generated/release assets | Must declare terms in their manifest/package before release |

Before a public code release, counsel should confirm the documentation/image treatment, source-offer mechanism, trademark position, and whether any additional documentation/asset license is desirable. Future community code remains `AGPL-3.0-only` unless an additional license is deliberately added without removing AGPL-3.0-only availability.
