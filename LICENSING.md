# Licensing

Helios is **dual-licensed**. Same code, two ways to use it.

## 1. Community license — AGPL-3.0

The default. See [LICENSE](LICENSE) for the full text. In plain terms:

- Use, study, modify, and redistribute Helios freely — including in commercial contexts.
- If you distribute a modified version, or let users interact with it **over a network**, you must make your complete corresponding source available under the AGPL too.
- That network clause is the teeth: you cannot take Helios, improve it, and ship it as a proprietary product or closed service.

This works fine for: individuals, students, researchers, open-source projects, studios willing to publish their derived source, and anyone experimenting.

## 2. Commercial license

If the AGPL doesn't fit — you want to ship a closed-source product, embed Helios in proprietary software, or keep your modifications private — you can buy a commercial license from Proceduralabs instead. Terms are negotiated per case (scope, seats, revenue share) and priced so that small studios can afford them.

Contact: **licensing@proceduralabs.eu** <!-- TODO: confirm this address exists before publishing -->

Commercial licensing is the funding model. It never removes or delays the open version — see the community promise in [CLA.md](CLA.md) §6.

## Which one do I need?

| You want to… | License |
|---|---|
| Learn from the code, experiment, build mods | AGPL (free) |
| Use Helios in research, publish results | AGPL (free) |
| Ship an open-source game/tool on Helios | AGPL (free — your project must be AGPL too) |
| Ship a closed-source game/product on Helios | Commercial |
| Run a modified Helios as a service without publishing your changes | Commercial |
| Contribute code back | AGPL + signed [CLA](CLA.md) |

## Why AGPL and not MIT? Why not "source-available"?

MIT would let any company take five years of work and ship it closed, contributing nothing. Source-available licenses (BSL, PolyForm) would protect against that but aren't open source by the OSI definition — many contributors and all major distros would stay away. AGPL is the strictest license that is still genuinely open source: maximum copyleft pressure on companies, zero restrictions on the community. Full reasoning: [docs/DECISIONS.md](docs/DECISIONS.md) D-011.

## Data and asset licenses

Astronomical and elevation datasets shipped or referenced by Helios carry their own licenses (HYG: public domain; SRTM/MOLA/LOLA: NASA/USGS public domain; Gaia: ESA, attribution required). Each baked dataset ships with a provenance file. Dataset licenses are independent of the code license.
