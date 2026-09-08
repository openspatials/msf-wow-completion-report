# Web of Worlds: Implementation Findings and Proposed Bindings

Web of Worlds already defines a useful architecture and an application programming interface (API) for worlds, users, views, portals, composition nodes, assets and user manifests. A local implementation exposed places where independently built clients and servers need more explicit agreements, especially for world transitions. This report sets those questions beside the published architecture and offers concrete proposals.

The specification is [WoWAPI 0.0.1 at commit d39a1a009aa4ef8fb6d14aa66d588cffb74c33de](https://github.com/WebOfWorlds/WoWAPI/tree/d39a1a009aa4ef8fb6d14aa66d588cffb74c33de), which matched upstream HEAD in the September 7, 2026 source check. It has individual-node read/write operations, typed fields, asset negotiation and resource-access responses. Its permissive schemas do not by themselves establish useful behavior. The proposals address missing or ambiguous bindings while preserving those existing capabilities.

This package contains **eleven numbered chapters, two appendices, this entry point and the method record: fifteen Markdown documents**. Document 11 covers the full named publication set with **76 active public-source comparison entries, 80 preserved public identifiers including four aliases, and five separate meeting notes**. These are documentation units, not completeness percentages or evidence of working interoperability.

## How to read the report

Each subject chapter compares current text, local experience, proposed changes and open questions. A source provision, an implementation result and a proposal carry different evidence. A reference to another chapter explains where the deeper argument lives; it does not imply acceptance by the group.

| Chapter | What it contributes |
|---|---|
| [01. Coordinate Precision, Units, and World Extents](01-coordinate-precision-units-and-extents.md) | Frames, units, transforms and error budgets; renderer normalization, root choice and hysteresis remain implementation choices. |
| [02. Portal Destination and the Traversal Protocol](02-portal-destination-and-traversal.md) | Optional destination, resolution and arrival mapping; visual continuity is distinct from presence and authority transfer. |
| [03. Portable User State and Identity](03-portable-user-state-and-identity.md) | Portable data and user control, separating signed bytes, trusted assertions, holder control and admission. |
| [04. Provenance and Signed Subtrees](04-provenance-and-signed-subtrees.md) | Optional signed content, verification/refusal and permissions; a capability declaration is not a test receipt. |
| [05. Presence, Live Sync, and Persistence](05-presence-live-sync-and-persistence.md) | Snapshots, events, session semantics, reconnect, replay, expiry and persistent references. |
| [06. Discovery and Addressing](06-discovery-and-addressing.md) | Entry URLs, service bases, fragments and resolver behavior with defined path/query/fragment semantics. |
| [07. Assets and the Render Seam](07-assets-and-the-render-seam.md) | Negotiated formats, candidate content profiles and external composition, keeping renderer choices informative. |
| [08. Composition Graph Schema Fixes](08-composition-graph-schema-fixes.md) | Existing node operations, embedded and proposed reference forms, external links and compatibility. |
| [09. Conformance Vocabulary and Errata](09-conformance-vocabulary-and-errata.md) | Useful data, behavioral profiles, observable failures and path/field corrections. |
| [10. Web of Worlds among Its Neighbours](10-role-and-blind-spots.md) | Current scope, possible responsibilities and questions exposed by the reviewed ecosystem sources. |
| [11. Published Positions and Current State](11-published-positions-and-current-state.md) | Complete named-source comparison, declaration mapping, current bindings, evidence classes and treating chapters. |

[Appendix A](A-completion-map.md) indexes specification surfaces. [Appendix B](B-findings-register.md) indexes implementation findings. [Method and Sources](METHOD-AND-SOURCES.md) records source versions, executed checks, historical receipt identities and limits.

## Five decisions for the working group

1. **Portal destination.** Define an optional canonical `Portal.destination` for a named version/profile, preserving numeric Portal IDs. Agree target resolution and unsupported or unresolved destination behavior; a stricter traversal profile is a separate decision.
2. **Behavioral conformance.** Agree required useful data, observable success/failure outcomes and a seed test corpus. Keep optional feature profiles explicit. Keep `geoPose` optional for non-georeferenced worlds. Requirement wording supports the contract; it does not replace behavioral tests.
3. **`scene` and `spatial`.** Choose the authoritative path mapping and document aliases or migration. The World README, paper example and pinned simpleWorlds API use `scene`; the canonical World API alone uses `spatial` and adds a graph identifier. Resolve both the path and graph-identity difference.
4. **`lan` and `lon`.** Align longitude with the chosen GeoPose profile and define the compatibility window, emitted form and conflicting-field behavior. Basic YPR has fixed WGS-84/ENU and ellipsoidal-height semantics; local-coordinate mapping must be explicit.
5. **Signed-subtree evaluation.** Open an optional extension track with sample payloads and refusal tests. Define publisher trust, signed-byte scope, mutable parent transforms, resource permissions and limits. The local implementation is one candidate.

These decisions preserve the broader architecture, including selective disclosure, live graph references, human/AI symmetry and planned content profiles. They do not require a particular rendering pipeline or imply adoption of every report proposal.

## Publication reference set

- [Full whitepaper](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf), March 31, 2026, 33 pages, and its separate [2026-Q1 executive summary](https://webofworlds.github.io/initial_MSF_Whitepaper/).
- [Announcement post](https://metaverse-standards.org/news/blog/announcing-the-web-of-worlds-whitepaper-a-concrete-path-to-the-open-metaverse/), visible June 2, 2026; metadata June 3 UTC.
- [Linked Spatial Experiences post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/), April 2, 2025; metadata modified September 4, 2025.
- [WebOfWorlds home](https://webofworlds.github.io/), captured September 7, 2026, including both implementation tables and architecture diagrams.
- [WoWAPI](https://github.com/WebOfWorlds/WoWAPI/tree/d39a1a009aa4ef8fb6d14aa66d588cffb74c33de), API version 0.0.1, including all three API files, five READMEs and architecture diagrams.
- [simpleWorlds published API](https://github.com/WebOfWorlds/simpleWorlds/blob/13d2cbea991e17df0b14857f11f693712e6171cb/packages/wow-spec/src/schema.yaml#L124-L228) and [README API table](https://github.com/WebOfWorlds/simpleWorlds/blob/13d2cbea991e17df0b14857f11f693712e6171cb/README.md#L77-L92), used only for declared path comparison at that pin. Implementation internals and conformance are not assessed.

## Evidence and status

Draft for working-group review, September 7, 2026. Historical local visual and contract receipts support specific implementation paths. Retained September 7 schema, numerical, controller-isolation and signing checks support their stated cases. Source-server exit-intent removal is distinguished from the isolated presence probe in the method record. These checks were not rerun for this publication comparison. Neither these results nor document-review counts establish independent cross-engine, network-scale or production acceptance. Proposed text remains unadopted.

## How to respond

Open an issue for the relevant chapter and reference its proposal or open question. For the publication comparison, name the declaration identifier and source page/section. Useful responses include an existing binding, an implementation receipt with its limits, a compatibility concern or a concrete acceptance case.

## Author and licence

Grig Bilham, Open Spatial Lab. Co-chair, Metaverse Standards Forum Infrastructure Working Group.

Licence to be decided by the author. Do not redistribute without permission until a licence is declared.
