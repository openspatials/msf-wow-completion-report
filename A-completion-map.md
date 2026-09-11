# Appendix A: Implementation and Proposal Map

The September 11 architecture revision preserves this historical register's identifiers and counting unit. It expands the contracts in [chapter 03](03-portable-user-state-and-identity.md), [chapter 05](05-presence-live-sync-and-persistence.md), [06](06-discovery-and-addressing.md), [07](07-assets-and-the-render-seam.md), [08](08-composition-graph-schema-fixes.md), [09](09-conformance-vocabulary-and-errata.md), and [the whole-system map in chapter 10](10-role-and-blind-spots.md#the-whole-architecture-and-its-connections). Those additions cover stationary and distributed entities, client behavior, ongoing interaction, scoped authority and persistence. This register is a selected index, not an exhaustive specification for the whole architecture; proposed new acceptance cases are not historical test results.


This selected inventory retains 73 historical surface identifiers across ten stages of the Web of Worlds journey. It includes canonical provisions, local interpretations, implementation extensions and open proposals. It is not a count of normative requirements, completed interoperability tests or all declarations in the publications.

The status distribution below describes the row labels, not a completion score. Local implementation evidence ranges from source inspection and contract checks to particular historical browser receipts; the relevant chapter states that boundary. No live or independent cross-engine acceptance is inferred from an implemented label.

**Spec commit:** WoWAPI d39a1a0 (2026-05-21). **Row set:** 2026-09-07.

---

## How to read this table

Each row records one selected source provision, local extension or proposed topic. Inclusion does not decide which layer should standardize it. "Surface" is the schema field, endpoint, or concept. "Status" is what Open Spatial Lab did with it:

- **implemented** -- recorded as implemented locally; this label alone is not a fresh runtime check, proof of every failure case or standards conformance.
- **interpretation** -- Open Spatial Lab recorded a local reading or implementation choice; the group has not adopted it.
- **open-question** -- the proposed common binding needs a group decision; local experiments can still exist.
- **drafted** -- the design is documented but not yet live in code.
- **deferred** -- identified as needed but not yet built.
- **refused** -- examined and deliberately not attempted, with the reason documented.

"Spec citation" names the API.yaml or README.md line at commit d39a1a0, or records that the specific binding was not located in the pinned API/README. Architecture in the full whitepaper is a separate evidence layer. "Treated in" names the numbered document and section where this surface is discussed in full.

---

## Find a world

10 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-056 | Optional pose/place/breadcrumb address design | drafted | README.md:11–15 provides basic entry and sharing; richer grammar is a local proposal | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-057 | spatialID path parameter type | implemented | API.yaml:141-146 at d39a1a0 | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-058 | Aspect kinds: published View and user cases; local user/node resolver | interpretation | README.md:21–31; whitepaper p.28; April 2, 2025 post | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "What Open Spatial Lab built and learned" |
| CM-059 | Entry URL versus service-base interpretation | interpretation | README.md:22–31; whitepaper page 28 query/fragment examples | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "4. Entry URL and API-base resolution." |
| CM-060 | History semantics | drafted | No binding located in pinned API/README | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-061 | Fabric services pointer (manifest services[]) | implemented | No binding located in pinned API/README | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "What Open Spatial Lab built and learned" |
| CM-062 | GET / | refused | API.yaml:17-29 at d39a1a0 | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "Open questions for the working group" |
| CM-063 | Core Requirement: fragment grammar syntax | interpretation | README.md:11-13 at d39a1a0 | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-064 | Core Requirement: unrecognized URL fragments | interpretation | No binding located in pinned API/README | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-065 | .well-known/spatial-fabric discovery | implemented | No binding located in pinned API/README | [06-discovery-and-addressing.md](06-discovery-and-addressing.md), section "What Open Spatial Lab built and learned" |

## Place it

6 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-044 | Local host-units-per-fabric-metre transclusion field | implemented | World API lacks this binding; whitepaper page 7 discusses units/origins | [01-coordinate-precision-units-and-extents.md](01-coordinate-precision-units-and-extents.md), section "What Open Spatial Lab built and learned" |
| CM-045 | Local up-axis mapping for known transclusion frames | implemented | World API lacks this binding; full frame semantics remain a profile question | [01-coordinate-precision-units-and-extents.md](01-coordinate-precision-units-and-extents.md), section "What Open Spatial Lab built and learned" |
| CM-046 | Optional World units/up-axis/handedness/extent proposal | open-question | API.yaml:268-349 at d39a1a0 | [01-coordinate-precision-units-and-extents.md](01-coordinate-precision-units-and-extents.md), section "Optional World coordinate metadata" |
| CM-047 | Informative renderer flattening / dRenderScale | interpretation | No renderer algorithm prescribed in pinned API/README | [01-coordinate-precision-units-and-extents.md](01-coordinate-precision-units-and-extents.md), section "Informative precision strategies" |
| CM-048 | Node.localTransform | interpretation | API.yaml:488-491 at d39a1a0 | [01-coordinate-precision-units-and-extents.md](01-coordinate-precision-units-and-extents.md), section "Candidate localTransform profile" |
| CM-049 | Informative precision-root and 4× hysteresis design | drafted | No universal threshold or algorithm in pinned API/README | [01-coordinate-precision-units-and-extents.md](01-coordinate-precision-units-and-extents.md), section "Informative precision strategies" |

## Compose it

8 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-001 | Optional child-reference representation; direct node GET/PUT already exist | implemented | API.yaml:189–241, 482–487; optional embedded children, undefined omitted-child update semantics | [08-composition-graph-schema-fixes.md](08-composition-graph-schema-fixes.md), section "1. Node.children: embedded and reference forms" |
| CM-002 | GET /wow/spatial/{spatialID} default response | implemented | API.yaml:133-153 at d39a1a0 | [08-composition-graph-schema-fixes.md](08-composition-graph-schema-fixes.md), section "2. Content negotiation on GET /wow/spatial/{spatialID}" |
| CM-003 | POST /wow/spatial/{spatialID}/node/{nodeId} canonical body handling | implemented | API.yaml:158-188 at d39a1a0 | [08-composition-graph-schema-fixes.md](08-composition-graph-schema-fixes.md), section "3. POST body-shape handling and error codes" |
| CM-004 | Node extension mechanism (WebOfWorldsExtension bag) | implemented | API.yaml:471-495 at d39a1a0 | [08-composition-graph-schema-fixes.md](08-composition-graph-schema-fixes.md), section "4. Named extension point on Node (optional)" |
| CM-005 | GET /wow/spatial/{spatialID} form query parameter | implemented | No binding located in pinned API/README | [08-composition-graph-schema-fixes.md](08-composition-graph-schema-fixes.md), section "2. Content negotiation on GET /wow/spatial/{spatialID}" |
| CM-006 | X-OSL-WoW-Node-Form header and 400/422 refusal codes on POST | implemented | No binding located in pinned API/README | [08-composition-graph-schema-fixes.md](08-composition-graph-schema-fixes.md), section "3. POST body-shape handling and error codes" |
| CM-007 | Spatial schema | implemented | API.yaml:438-468 at d39a1a0 | [08-composition-graph-schema-fixes.md](08-composition-graph-schema-fixes.md), section "2. Content negotiation on GET /wow/spatial/{spatialID}" |
| CM-008 | X-OSL-WoW-Spatial-Form request/response headers | implemented | No binding located in pinned API/README | [08-composition-graph-schema-fixes.md](08-composition-graph-schema-fixes.md), section "2. Content negotiation on GET /wow/spatial/{spatialID}" |

## Draw it

8 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-036 | Node.spatialAssetURI for signed spatial documents (SpatialFabricSubtree) | implemented | API.yaml:492-493 at d39a1a0 | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "Transclusion contract for signed spatial documents" |
| CM-037 | Rendering backend selection for transcluded assets (placement) | implemented | No binding located in pinned API/README | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |
| CM-038 | OpenSpatialAsset signed document type | open-question | OpenSpatialAsset/API.yaml:51-161 at d39a1a0 | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "Open questions for the working group" |
| CM-039 | Node.spatialAssetURI (no format, no media type, no prose) | deferred | API.yaml:492-493 at d39a1a0 | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "spatialAssetURI resolution rule" |
| CM-040 | SpatialFabricSubtree.parallax (backdrop parallax dial) | implemented | No binding located in pinned API/README | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |
| CM-041 | SpatialFabricSubtree.maxDepth (recursion cap) | implemented | No binding located in pinned API/README | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |
| CM-042 | SpatialFabricSubtree.epochTicks (time epoch for time-dependent fabrics) | implemented | No binding located in pinned API/README | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |
| CM-043 | First-party TeleportXR browser rendering | refused | No binding located in pinned API/README | [07-assets-and-the-render-seam.md](07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |

## Cross a portal

18 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-009 | Local destination extension; optional canonical proposal | implemented | API.yaml:409-436 at d39a1a0 | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Optional Portal destination" |
| CM-010 | Candidate visual-continuity profile | drafted | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Crossing continuity (candidate traversal profile)" |
| CM-011 | Local portal-frame pose mapping; arbitrary-frame acceptance open | implemented | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Pose mapping across portal frames (candidate traversal profile)" |
| CM-012 | Portal interaction geometry (Zone) | implemented | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Portal interaction geometry (SHOULD, optional extension)" |
| CM-013 | GET /wow/portal/{portalId} response composite (OSLPortalResponse) | implemented | API.yaml:110-129 at d39a1a0 | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "What Open Spatial Lab built and learned" |
| CM-014 | Node type discrimination (portal vs geometry node) | deferred | API.yaml:471-495 at d39a1a0 | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Node type discrimination (SHOULD)" |
| CM-015 | World-to-world crossing / portal traversal semantics | deferred | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Open questions for the working group" |
| CM-016 | IWPS-shaped Query and Teleport handshake | interpretation | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "What Open Spatial Lab built and learned" |
| CM-017 | Best-effort exit/arrival notifications and correlation id | implemented | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Server-side exit and arrival notifications (protocol sketch)" |
| CM-018 | Source exit-intent removal and finite tombstone; later client departure confirms; no global authority guarantee | implemented | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Presence and authority during crossing (open profile decision)" |
| CM-019 | Portal traversal controller routes and fields | implemented | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "What Open Spatial Lab built and learned" |
| CM-020 | Graph-based destination composition proposal | drafted | Canonical Spatial/root-node access exists; traversal binding remains proposed | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Crossing destination resolution (candidate graph-traversal profile)" |
| CM-021 | Portal scale semantics in a host graph | open-question | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Open questions for the working group" |
| CM-022 | No-reload local visual-rig promotion; not network authority transfer | implemented | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "No-reload crossing (optional continuous-view profile)" |
| CM-023 | Portal traversal direction (PortalTraversal) | implemented | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Portal traversal direction (SHOULD, optional extension)" |
| CM-024 | Native TeleportXR teleport / session behavior | refused | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "What Open Spatial Lab built and learned" |
| CM-025 | Signed-fabric refusal under test-anchor policy; separate portal checks can continue | implemented | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Policy decisions before crossing" |
| CM-026 | Portal trigger volume and plane-crossing detection | implemented | No binding located in pinned API/README | [02-portal-destination-and-traversal.md](02-portal-destination-and-traversal.md), section "Portal trigger model (SHOULD, optional extension)" |

## Carry yourself across

5 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-027 | User response with optional signed portable assertions | implemented | API.yaml:47-66 and 351-380 at d39a1a0 | [03-portable-user-state-and-identity.md](03-portable-user-state-and-identity.md), section "Proposed normative text" |
| CM-028 | UMSignature byte/key verification; not identity or age assurance | implemented | No signing profile in pinned API; OpenUserManifest HEAD already authorizes resource access | [03-portable-user-state-and-identity.md](03-portable-user-state-and-identity.md), section "Proposed normative text" |
| CM-029 | Declared age in local manifest; assurance policy remains separate | implemented | API.yaml:351-380 (no age field) and OpenUserManifest/API.yaml:49-60 (age at L57-58) at d39a1a0 | [03-portable-user-state-and-identity.md](03-portable-user-state-and-identity.md), section "Proposed normative text" |
| CM-030 | Core Requirement: world entry role and embodiment | interpretation | No binding located in pinned API/README | [03-portable-user-state-and-identity.md](03-portable-user-state-and-identity.md), section "What Open Spatial Lab built and learned" |
| CM-031 | DELETE /wow/user/{userId} | deferred | API.yaml:67-85 at d39a1a0 | [03-portable-user-state-and-identity.md](03-portable-user-state-and-identity.md), section "Open questions for the working group" |

## Be there together

4 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-032 | Local WebSocket /events convention; transport/recovery profile unproven | implemented | No binding located in pinned API/README | [05-presence-live-sync-and-persistence.md](05-presence-live-sync-and-persistence.md), section "Real-time event channel" |
| CM-033 | Preview: no additional user, separate authorization | interpretation | README.md:13 is truncated; April 2, 2025 post supplies participation/authorization intent | [05-presence-live-sync-and-persistence.md](05-presence-live-sync-and-persistence.md), section "What Open Spatial Lab built and learned" |
| CM-034 | Follower visibility and departure behavior | open-question | No binding located in pinned API/README | [05-presence-live-sync-and-persistence.md](05-presence-live-sync-and-persistence.md), section "Follower rules" |
| CM-035 | Core Requirement: follow (bare, no aspect) | interpretation | README.md:12 at d39a1a0 | [05-presence-live-sync-and-persistence.md](05-presence-live-sync-and-persistence.md), section "Bare #follow" |

## Trust what you see

5 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-050 | Publisher-declared proof_boundary flags; not test receipts | implemented | No binding located in pinned API/README | [04-provenance-and-signed-subtrees.md](04-provenance-and-signed-subtrees.md), section "Proof-boundary declaration" |
| CM-051 | Required signed-fabric verification; not every portal check fails closed | implemented | No binding located in pinned API/README | [04-provenance-and-signed-subtrees.md](04-provenance-and-signed-subtrees.md), section "Verification on navigation and transclusion (candidate signed-content profile)" |
| CM-052 | GET /wow/world response composite (OSLWorldResponse) | implemented | API.yaml:32-44 at d39a1a0 | [04-provenance-and-signed-subtrees.md](04-provenance-and-signed-subtrees.md), section "Response extension mechanism" |
| CM-053 | Transcluded-fabric verification under a configured test anchor | implemented | No binding located in pinned API/README | [04-provenance-and-signed-subtrees.md](04-provenance-and-signed-subtrees.md), section "Verification of transcluded spatial subtrees (CM-053)" |
| CM-054 | GET /wow/view/{viewId} response composite (OSLViewResponse) | implemented | API.yaml:88-107 at d39a1a0 | [04-provenance-and-signed-subtrees.md](04-provenance-and-signed-subtrees.md), section "Response extension mechanism" |

## Know what conforms

8 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-066 | spatialID and nodeId path parameter declarations on node operations | implemented | API.yaml:156-260 at d39a1a0 | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "3. spatialID parameter declaration" |
| CM-067 | Scene/spatial route and graph-id mapping across four sources | interpretation | README.md:28,31; whitepaper p.29; simpleWorlds schema:132 at 13d2cbe; API.yaml:133,156 at d39a1a0 | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "5. Graph path adjudication" |
| CM-068 | Extension policy (additive, labeled, surfaced, honest) | implemented | No binding located in pinned API/README | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "8. Extension policy (optional)" |
| CM-069 | Behavioral conformance profile; existing type constraints remain testable | open-question | Six schemas accept empty objects and reject wrong object types; no BCP 14 preamble | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "1. RFC 2119 keyword adoption" |
| CM-070 | Bind published Preview intent; complete Persist/Share descriptions | open-question | README.md:13–15; April 2, 2025 post defines preview without additional user and with authorization | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "7. Core Requirement descriptions" |
| CM-071 | Published dotted kind.id example versus local slash parser | interpretation | README.md:11; whitepaper p.28 shows join=view.5845 | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "9. Aspect identifier syntax (optional)" |
| CM-072 | Complete aspect grammar and target-failure binding | open-question | README.md:11–13 and whitepaper p.28 give syntax examples, not a complete algorithm | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "9. Aspect identifier syntax (optional)" |
| CM-073 | GeoPose.position.lan misspelling of longitude | interpretation | API.yaml:294-295 at d39a1a0 | [09-conformance-vocabulary-and-errata.md](09-conformance-vocabulary-and-errata.md), section "6. GeoPose.position.lan correction" |

## Among its neighbours

1 row.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-055 | Core Requirement: persist and share | interpretation | README.md:14-15 at d39a1a0 | [05-presence-live-sync-and-persistence.md](05-presence-live-sync-and-persistence.md), section "Persist and share fragment rule" |

---

## Inventory counts

**Total rows:** 73

**By status:**

- implemented: 40
- interpretation: 14
- open-question: 7
- drafted: 5
- deferred: 4
- refused: 3

**By stage:**

- Find a world: 10
- Place it: 6
- Compose it: 8
- Draw it: 8
- Cross a portal: 18
- Carry yourself across: 5
- Be there together: 4
- Trust what you see: 5
- Know what conforms: 8
- Among its neighbours: 1

**Rows not treated in a numbered document:** 0

---

## Sources

- [Linked Spatial Experiences: The Web of Worlds, April 2, 2025](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/): published units, preview, authorization and aspect intent.

- [OpenSpatialWorld API 0.0.1 at d39a1a0](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml) and [README](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md), checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf): architecture, distinct from the API bindings summarized here.
- Retained Open Spatial Lab surface inventory and local source/receipt set, September 7, 2026. The 73 row identifiers are preserved for traceability. Public reproduction of the exact local source bytes is not established.
- The linked report chapters explain each proposal and its evidence. In particular, CM-018 distinguishes source exit removal, finite tombstones and client confirmation, CM-025/051/053 concern a bounded fabric-verification path, and CM-027/028 do not establish visitor assurance. Chapter 11 carries the separate publication comparison.

## Change log

- 2026-09-07: corrected row meanings, evidence labels and chapter references while preserving all 73 historical identifiers and their inventory classifications.
