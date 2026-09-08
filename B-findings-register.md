# Appendix B: Findings Register


This selected register preserves 25 historical finding identifiers across ten journey stages. The corrected findings distinguish API omissions from published architecture, local evidence from independent interoperability, and optional proposals from adopted requirements. Every row points to its treatment in the report.

The earlier blind-spot/known-gap categories were derived partly from keyword thresholds. They are not retained as evidence that a topic was never discussed or never specified. Counts below describe these authored rows only, not completeness, priority or confidence.

**Spec commit:** WoWAPI d39a1a0 (2026-05-21). **Row set:** 2026-09-07.

---

## How to read this table

Each finding states a scoped result or open question. Each recommendation is an unadopted proposal for the group. Evidence basis means:

- **source-text**: the cited specification or publication supports the scoped statement.
- **local-checks**: retained local code, numerical or contract checks support a bounded implementation result; not a fresh browser or cross-engine test.
- **source-and-local**: both source text and retained local evidence are relevant, with their limits in the treating chapter.
- **review-question**: the selected corpus exposed a question; it does not establish worldwide absence.

The table does not use a code-read label to imply that behavior was executed. Historical identifiers remain stable even where the original conclusion has been corrected.

---

## Find a world

1 finding.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-014 | The README names entry fragments and the whitepaper shows view.5845; case, encoding, kinds and target-failure behavior still need a common profile. | Agree a fragment profile and aspect-resolution/failure rules; case-insensitive matching is a local candidate, not established behavior. | source-and-local | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "Proposed normative text" |

## Place it

4 findings.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-001 | The whitepaper discusses units/origins; the pinned World API does not bind local units, axes or extent. | Add optional coordinate metadata and define frame/unit conversion in a named profile; extent is optional and no renderer algorithm is mandated. | source-and-local | [01-coordinate-precision-units-and-extents.md](01-coordinate-precision-units-and-extents.md), section "Optional World coordinate metadata" |
| R-002 | No shared-anchor discovery/localization binding was located in the reviewed WoW sources; a selected map cannot establish global absence. | Name the shared localization, permissions, lifetime and error contract; WebXR tracked anchors alone do not provide cross-device sharing. | review-question | [10-role-and-blind-spots.md](10-role-and-blind-spots.md), section "Shared spatial anchors" |
| R-013 | The local transform array lacks a length, storage-order and frame-direction binding; existing numeric constraints still apply. | Evaluate a versioned 16-element column-major profile with explicit frames and units; required data and tighter shapes need compatibility rules. | source-and-local | [01-coordinate-precision-units-and-extents.md](01-coordinate-precision-units-and-extents.md), section "Candidate localTransform profile" |
| R-022 | OGC Basic YPR fixes WGS-84/ENU and ellipsoidal height; the WoW lan spelling and local-frame mapping remain unbound. | Adopt Basic YPR semantics by reference, migrate lan to lon with conflict handling, and define geodetic-to-local conversion. | source-text | [01-coordinate-precision-units-and-extents.md](01-coordinate-precision-units-and-extents.md), section "GeoPose adoption by reference" |

## Compose it

1 finding.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-018 | The whitepaper names Data Inline (p.21) and external Node references (p.22), with Anchor/Inline lineage (p.10); the YAML lacks their shared binding. | Evaluate optional external-node profiles with resolution, frames, cycles and limits; a signed fabric is one candidate content class. | source-and-local | [08-composition-graph-schema-fixes.md](08-composition-graph-schema-fixes.md), section "5. Cross-world reference construct on Node (optional)" |

## Draw it

4 findings.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-007 | The reviewed API does not bind physics, audio or input behavior; keyword absence alone does not decide architectural scope. | Agree which observable behaviors belong in the profile and which engine mechanisms or referenced standards remain outside it. | source-text | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "Engine-internal scope statement" |
| R-008 | Asset negotiation exists, but the listed formats do not guarantee a common representation for every pair of participants. | Evaluate GLB as a baseline for a defined static-mesh profile, with unsupported-feature and negotiation-failure behavior. | source-text | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "Baseline asset format" |
| R-017 | Open Spatial Lab composes signed .msf subtrees through a local extension with a test-anchor trust policy. | Open an optional signed-subtree track with sample payloads, byte scope, trust, execution limits and refusal cases; do not adopt one backend implicitly. | source-and-local | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "Transclusion contract for signed spatial documents" |
| R-023 | Node.spatialAssetURI is a string, while OpenSpatialAsset already lists negotiated representations. | Define URI-reference resolution and an agreed asset profile; do not require every spatial document to have a faithful GLB form. | source-text | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "Baseline asset format" |

## Cross a portal

2 findings.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-011 | The canonical Portal defines numeric id and geoPose without a destination member; extra fields are schema-legal. | Add optional destination with target identity/resolution rules, preserve numeric ids, and separate any stricter traversal profile. | source-and-local | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Optional Portal destination" |
| R-019 | IWPS offers a possible crossing reference; the local two-call study uses demo-native fields and declares no IWPS conformance. | Evaluate the exact IWPS version, mandatory fields and security profile before adopting it; test independent endpoints and failure cases. | source-and-local | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "What Open Spatial Lab built and learned" |

## Carry yourself across

2 findings.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-020 | The User and manifest resources carry avatar information; a signed assertion protects bytes but does not establish admission or asset rights. | Evaluate an optional portable assertion profile with explicit issuer trust, holder control, disclosure and destination policy. | source-and-local | [03-portable-user-state-and-identity.md](03-portable-user-state-and-identity.md), section "Proposed normative text" |
| R-021 | A local numeric id needs cross-world identity mapping; a key-based identifier alone does not prove a person, age or current holder control. | Preserve canonical numeric ids and evaluate optional identity references plus a defined signature/assurance profile. | source-and-local | [03-portable-user-state-and-identity.md](03-portable-user-state-and-identity.md), section "Proposed normative text" |

## Be there together

2 findings.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-012 | World has presence metadata and user counters, but the reviewed API lacks a shared live-event and recovery binding. | Use the proposed vocabulary user_joined, user_left, node_created, node_updated and node_deleted; also agree transport, payloads, revisions, deduplication and reconnect/resync fixtures. | source-and-local | [05-presence-live-sync-and-persistence.md](05-presence-live-sync-and-persistence.md), section "Real-time event channel" |
| R-024 | Join/follow/preview are named, but participation, expiry and failure behavior need a shared binding. | Bind the published no-additional-user preview intent and authorization; define source exit removal, finite tombstones, lease recovery and session authority separately from visual continuity. | source-and-local | [05-presence-live-sync-and-persistence.md](05-presence-live-sync-and-persistence.md), section "Session lifecycle semantics" |

## Trust what you see

1 finding.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-006 | The world API lacks a signed-object profile; asset/manifest HEAD authorization, ETags, HTTP authentication and single sign-on already have distinct published roles. | Evaluate optional signed-content policy and labeled capability declarations; a signed flag is not a receipt, identity assurance or execution permission. | source-and-local | [04-provenance-and-signed-subtrees.md](04-provenance-and-signed-subtrees.md), section "Proof-boundary declaration" |

## Know what conforms

4 findings.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-009 | Six resource schemas accept empty objects and reject wrong types; structural validity alone does not establish useful behavior. | Choose a behavioral profile and seed corpus, required data for its scenario, and BCP 14 wording without erasing existing constraints. | source-and-local | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "1. RFC 2119 keyword adoption" |
| R-015 | README, whitepaper p.29 and the reference schema use scene; OpenSpatialWorld/API.yaml uses spatial with a graph id. Their route and field mappings need agreement. | Choose the mapping and document aliases/version migration; adapters can bridge routes, so differing strings alone do not prove incompatibility. | source-text | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "5. Graph path adjudication" |
| R-016 | The pinned GeoPose-shaped fields use lan; an uncoordinated spelling change can leave literal consumers without longitude. | Adopt lon with an explicit legacy window and reject conflicting lan/lon values; preserve the selected OGC frame semantics. | source-text | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "6. GeoPose.position.lan correction" |
| R-025 | Existing OpenAPI constraints have testable meaning, but no sufficiently defined behavioral profile and corpus was located in the reviewed sources. | Publish versioned inputs and observable success/failure cases; shape counts and signing vectors are seed evidence, not behavioral conformance. | source-and-local | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "1. RFC 2119 keyword adoption" |

## Among its neighbours

4 findings.

| Id | Finding | Recommendation | Evidence basis | Treated in |
|---|---|---|---|---|
| R-003 | No portable-item exchange binding was located in the pinned WoW APIs; that is not a worldwide absence finding. | Evaluate optional item references, rights/provenance, consent and destination acceptance independently of portal adoption. | review-question | [05-presence-live-sync-and-persistence.md](05-presence-live-sync-and-persistence.md), section "Portable inventory (optional extension)" |
| R-004 | The whitepaper describes preferences/settings; the pinned manifest schema has name, age and avatarAssetURI without that vocabulary. | Evaluate optional preferences with disclosure, supported-setting and observable-behavior rules; data transfer alone is not accessibility conformance. | source-text | [05-presence-live-sync-and-persistence.md](05-presence-live-sync-and-persistence.md), section "Portable preferences (optional extension)" |
| R-005 | The whitepaper describes common human/AI access; the pinned API lacks a dedicated actor/delegation/permission binding. | Evaluate optional actor declarations separately from trusted identity and granted capabilities, or reference a chosen authorization profile. | source-text | [10-role-and-blind-spots.md](10-role-and-blind-spots.md), section "Software-agent declarations" |
| R-010 | Keyword counts do not prove discussion frequency or absence of governance documents; the whitepaper already describes an adoption path. | Locate authoritative governance/contribution/license/liaison documents first, then ask the responsible group to fill confirmed gaps. | review-question | [10-role-and-blind-spots.md](10-role-and-blind-spots.md), section "Governance document references" |

---

## Inventory counts

**Total findings:** 25

**By stage:**

- Find a world: 1
- Place it: 4
- Compose it: 1
- Draw it: 4
- Cross a portal: 2
- Carry yourself across: 2
- Be there together: 2
- Trust what you see: 1
- Know what conforms: 4
- Among its neighbours: 4

**Findings not treated in a numbered document:** 0

---

## Sources

- [Linked Spatial Experiences: The Web of Worlds, April 2, 2025](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/): published units, preview, authorization and aspect intent.

- [WoWAPI at d39a1a0](https://github.com/WebOfWorlds/WoWAPI/tree/d39a1a0/specification), checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf), with printed-page provenance in the linked chapters.
- [OGC GeoPose 1.0](https://docs.ogc.org/is/21-056r11/21-056r11.html), [OpenAPI 3.0.4](https://spec.openapis.org/oas/v3.0.4.html) and [W3C Verifiable Credentials 2.0 trust model](https://www.w3.org/TR/vc-data-model-2.0/#trust-model).
- Retained Open Spatial Lab findings, implementation sources and local receipts, September 7, 2026. Public reproduction of the exact local bytes is not established. The treating chapters state each proof class and its limits.
- The selected infrastructure map and topic-keyword inventory are discovery aids; they do not establish worldwide absence, discussion frequency or completeness. [Chapter 11](11-published-positions-and-current-state.md) handles the separate publication comparison; [Appendix A](A-completion-map.md) preserves 73 selected surface records.

## Change log

- 2026-09-07: corrected all 25 historical findings and their evidence basis; removed keyword-derived absence categories and updated treating-section links without renumbering records.
