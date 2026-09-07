# Appendix B: Findings Register

25 findings from the Open Spatial Lab implementation of the Web of Worlds specification. Of these, 9 carry talk priority 1 (must cover in the presentation). By category: 5 blind spots (not discussed and not specified), 11 known gaps (discussed but not specified), 9 recommendations (adopt by reference from a neighbouring standard). Every finding is treated in at least one of the ten numbered documents in the completion report set.

**Spec commit:** WoWAPI d39a1a0 (2026-05-21). **Row set:** 2026-09-07.

---

## How to read this table

Each row records one finding from the implementation effort. "Finding" states what was found in one sentence. "Recommendation" states what the working group could do about it in one sentence. "Evidence level" names the strength of the evidence:

- **verified-in-code** -- the finding was confirmed by running code in Open Spatial Lab.
- **documented** -- the finding is recorded in project documentation or specification text.
- **inferred-from-map** -- the finding was derived from the architecture map or socket coverage analysis.

"Treated in" names the numbered document and section where the finding is discussed in full.

---

## Find a world

1 finding.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-014 | The README lists URL fragment verbs (#join, #follow, #preview) as Core Requirements but defines no syntax; case sensitivity, compound fragments, and unknown-fragment handling are all ambiguous. | WoW MUST define the URL fragment grammar: case-insensitive verbs, one verb per fragment, defined aspect.id syntax, and unrecognized fragments MUST NOT be hijacked. | verified-in-code | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "Proposed normative text" |

## Place it

4 findings.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-001 | The specification is silent on units, extent, and up-axis; an implementer composing two worlds at different scales has no vocabulary to declare them. | WoW MUST define units (metres as default, with a scale factor), upAxis (enum: y, z), and extent on the World resource. | verified-in-code | [01-coordinate-precision-units-and-extents.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md), section "World resource spatial properties" |
| R-002 | Shared spatial anchoring (two AR devices agreeing on where 'here' is) has zero coverage in the 102-socket architecture map: no MSF subject and no listed standard covers it. | WoW SHOULD define a spatial-anchor vocabulary or adopt the emerging WebXR Anchors Module by reference. | documented | [10-role-and-blind-spots.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md), section "Shared spatial anchors" |
| R-013 | A World resource that declares no units, up-axis, or extent leaves every implementer guessing; a metre read as a centimetre places the asset at the wrong scale on arrival. | World MUST declare units (default: metres), upAxis (enum: y, z), and extent; Node.localTransform MUST be defined as a 16-element column-major 4x4 matrix. | verified-in-code | [01-coordinate-precision-units-and-extents.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md), section "World resource spatial properties" |
| R-022 | A GeoPose without an explicit datum is ambiguous; two implementations could assume different reference frames and place the same world in different locations on Earth. | WoW SHOULD adopt the OGC GeoPose Basic YPR form with an explicit datum field (default: WGS84) and correct lan to lon in the same pass. | documented | [01-coordinate-precision-units-and-extents.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md), section "GeoPose (SHOULD, adopt by reference)" |

## Compose it

1 finding.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-018 | Without cross-world composition, every world is a flat island; the Open Metaverse Browser model (any node may reference a child fabric, independently verified) is the composition answer. | WoW SHOULD define a cross-world reference construct on Node, with composition rules adopted from the Open Metaverse Browser model. | verified-in-code | [08-composition-graph-schema-fixes.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md), section "5. Cross-world reference construct on Node (optional)" |

## Draw it

4 findings.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-007 | The specification says nothing about physics, audio, or input; implementers cannot tell whether these are out of scope or not yet written. | WoW SHOULD explicitly state that physics, audio spatialization, and input device mapping are out of scope and adopted by reference from engine-level standards. | documented | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "Engine-internal scope statement" |
| R-008 | The group discusses rendering at every meeting (121 transcript hits) but the specification defines only format listing, not how a world should present an asset. | WoW SHOULD adopt glTF as the baseline required asset format (a conformant server MUST serve at least model/gltf-binary). | verified-in-code | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "Baseline asset format" |
| R-017 | WoW defines the graph; RP1 Spatial Fabric defines the signed document; the seam between them is the SpatialFabricSubtree extension on Node.spatialAssetURI. | WoW SHOULD adopt the RP1 Spatial Fabric signed document (file suffix .msf) as a recognized subtree asset type alongside glTF leaves, with the SpatialFabricSubtree extension contract as the canonical seam. | verified-in-code | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "Transclusion contract for signed spatial documents" |
| R-023 | Node.spatialAssetURI is an unconstrained string with no format or media type; without a baseline format, two servers could exchange a URI that neither can render. | WoW SHOULD declare glTF (model/gltf-binary) as the required baseline asset format; USD and X3D are optional. | verified-in-code | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "Baseline asset format" |

## Cross a portal

2 findings.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-011 | The canonical Portal schema has id and geoPose but no destination field; no conformant portal graph can describe travel between worlds. | Portal MUST include a destination object with at minimum target_world_id (string, URI format). | verified-in-code | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Portal destination (MUST)" |
| R-019 | WoW defines where a portal goes (once destination is added); OMA3 IWPS defines the handshake (may I come in); the two are complementary and neither references the other today. | WoW SHOULD adopt the IWPS Query then Teleport two-call protocol by reference for portal crossings. | verified-in-code | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "What Open Spatial Lab built and learned" |

## Carry yourself across

2 findings.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-020 | A user's avatar must be portable and verifiable; the WoW User resource defines the slot; Universal Manifest defines the signed envelope. | WoW SHOULD adopt Universal Manifest as the signing envelope for user identity on the User resource. | verified-in-code | [03-portable-user-state-and-identity.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md), section "Proposed normative text" |
| R-021 | User identity that is only a server-local integer cannot survive a portal crossing; a DID bound to a key makes identity portable and verifiable without a central authority. | WoW SHOULD adopt W3C DID as the user identifier format and Universal Manifest's Ed25519/JCS-RFC8785 as the signature profile. | verified-in-code | [03-portable-user-state-and-identity.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md), section "Proposed normative text" |

## Be there together

2 findings.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-012 | The specification defines a presence object (three strings: avatar, navigation, gravity) and user counters on the World resource but no wire protocol for real-time events. | WoW SHOULD define a WebSocket or SSE endpoint for real-time events: user_join, user_depart, node_update, portal_activate at minimum. | verified-in-code | [05-presence-live-sync-and-persistence.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md), section "Real-time event channel" |
| R-024 | The README names the entry verbs join, follow, and preview, but the wire contract for these verbs does not exist. | WoW SHOULD define session lifecycle semantics (join, follow, preview, depart) with explicit state transitions and reference TeleportXR's session vocabulary. | verified-in-code | [05-presence-live-sync-and-persistence.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md), section "Session lifecycle semantics" |

## Trust what you see

1 finding.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-006 | The WoW graph is live unsigned JSON with no signature, integrity, or provenance vocabulary; trust is discussed at meetings but codified nowhere. | WoW SHOULD define a proof-boundary declaration on every /wow response and an optional signed-envelope mechanism for world-to-world trust. | verified-in-code | [04-provenance-and-signed-subtrees.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/04-provenance-and-signed-subtrees.md), section "Proof-boundary declaration" |

## Know what conforms

4 findings.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-009 | The specification contains zero RFC 2119 keywords, no required properties on any schema, and an empty JSON object validates against every schema. | WoW MUST adopt RFC 2119 keywords, declare required properties on every schema, and publish a minimal conformance test corpus. | verified-in-code | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "1. RFC 2119 keyword adoption" |
| R-015 | The README says /wow/scene; the API.yaml defines /wow/spatial; two implementations that chose different paths cannot interoperate. | Errata: adjudicate and close; the README MUST be corrected to /wow/spatial or the API.yaml MUST be corrected to /wow/scene. | verified-in-code | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "5. Graph path adjudication" |
| R-016 | GeoPose.position.lan is a misspelling of longitude baked into every GeoPose occurrence; correcting it later will break interoperability with any implementation that conformed to the letter. | Errata: correct lan to lon in the next schema revision with a deprecation window; implementations SHOULD accept both during the transition. | verified-in-code | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "6. GeoPose.position.lan correction" |
| R-025 | 'Compliant with Web of Worlds' currently has no testable meaning; RFC 2119 keywords and OpenAPI required properties are settled practice. | WoW MUST adopt RFC 2119 keywords, declare required properties in every schema, and publish a conformance test corpus. | verified-in-code | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "1. RFC 2119 keyword adoption" |

## Among its neighbours

4 findings.

| Id | Finding | Recommendation | Evidence level | Treated in |
|---|---|---|---|---|
| R-003 | No portable inventory contract exists; a user who acquires an item in one world cannot carry it through a portal to another. | WoW SHOULD define a portable-inventory schema (item reference, licence, provenance) or adopt by reference from Universal Manifest. | inferred-from-map | [05-presence-live-sync-and-persistence.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md), section "Portable inventory (optional extension)" |
| R-004 | The whitepaper names portable preferences and settings but the OpenUserManifest schema carries only name, age, and avatarAssetURI. | WoW SHOULD define a preference vocabulary on the user manifest covering at minimum accessibility settings, input preferences, and rendering quality. | documented | [05-presence-live-sync-and-persistence.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md), section "Portable preferences (optional extension)" |
| R-005 | The whitepaper names AI integration but the API defines no agent identity, capability declaration, or agent protocol. | WoW SHOULD define an agent identity type on the User resource or explicitly defer to a future extension track with a timeline. | documented | [10-role-and-blind-spots.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md), section "AI agent identity" |
| R-010 | Governance is the most discussed topic at every meeting (599 transcript hits across five meetings) and none of it is written down; the licence and CLA policy was stated at the 2026-08-24 meeting but no document records it. | WoW SHOULD write a governance document covering the initiative hub structure, business council charter, licence and CLA policy, and SDO liaison protocol. | documented | [10-role-and-blind-spots.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md), section "Governance document" |

---

## Counts

**Total findings:** 25

**By category:**

- blind-spot: 5
- known-gap: 11
- recommendation: 9

**Talk priority 1 (must cover):** 9

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

- WoWAPI repository at commit d39a1a0 (2026-05-21)
- Open Spatial Lab findings register (25 rows, 2026-09-07)
- The socket map of the MSF infrastructure architecture (102 sockets); each socket names an interop surface, the Web of Worlds claim, and the covering standards (summarized in document 10)
- The discussion-evidence table (12 topics across five recorded meetings); transcript hit counts, API, README, and whitepaper hits, and classification
- The published-positions register (document 11); each declaration classified as specified, named only, or aspiration
- The completion map (73 rows); see Appendix A
- The ten numbered documents in the completion report set

---

## Change log

- 2026-09-07: first public draft, verified twice.
