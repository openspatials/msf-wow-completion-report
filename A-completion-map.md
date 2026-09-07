# Appendix A: Completion Map

73 spec surfaces examined across ten stages of the Web of Worlds journey. By status: 40 implemented, 14 interpretation (Open Spatial Lab chose a reading where the text is ambiguous), 7 open question (the working group must decide), 5 drafted (design documented, not yet live), 4 deferred (identified but not yet built), 3 refused (examined and deliberately not attempted).

By stage: Find a world (10), Place it (6), Compose it (8), Draw it (8), Cross a portal (18), Carry yourself across (5), Be there together (4), Trust what you see (5), Know what conforms (8), Among its neighbours (1).

**Spec commit:** WoWAPI d39a1a0 (2026-05-21). **Row set:** 2026-09-07.

---

## How to read this table

Each row records one surface the specification defines or should define. "Surface" is the schema field, endpoint, or concept. "Status" is what Open Spatial Lab did with it:

- **implemented** -- built and running in Open Spatial Lab.
- **interpretation** -- the specification text is ambiguous or incomplete; Open Spatial Lab chose a reading and documented it.
- **open-question** -- the surface needs a decision from the working group before any implementation can proceed.
- **drafted** -- the design is documented but not yet live in code.
- **deferred** -- identified as needed but not yet built.
- **refused** -- examined and deliberately not attempted, with the reason documented.

"Spec citation" names the API.yaml or README.md line at commit d39a1a0, or states that the specification is silent on this surface. "Treated in" names the numbered document and section where this surface is discussed in full.

---

## Find a world

10 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-056 | Address grammar (the <a href> for worlds) | drafted | spec is silent | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-057 | spatialID path parameter type | implemented | API.yaml:141-146 at d39a1a0 | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-058 | Core Requirement: definition of 'aspect' | interpretation | README.md:21-31 at d39a1a0 | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "What Open Spatial Lab built and learned" |
| CM-059 | Core Requirement: world URL as path prefix | interpretation | README.md:22-25 at d39a1a0 | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "What Open Spatial Lab built and learned" |
| CM-060 | History semantics | drafted | spec is silent | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-061 | Fabric services pointer (manifest services[]) | implemented | spec is silent | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "What Open Spatial Lab built and learned" |
| CM-062 | GET / | refused | API.yaml:17-29 at d39a1a0 | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "Open questions for the working group" |
| CM-063 | Core Requirement: fragment grammar syntax | interpretation | README.md:11-13 at d39a1a0 | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-064 | Core Requirement: unrecognized URL fragments | interpretation | spec is silent | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "Proposed normative text" |
| CM-065 | .well-known/spatial-fabric discovery | implemented | spec is silent | [06-discovery-and-addressing.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md), section "What Open Spatial Lab built and learned" |

## Place it

6 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-044 | Spatial scale / unitsPerMeter for transcluded assets | implemented | spec is silent | [01-coordinate-precision-units-and-extents.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md), section "Transclusion contract (SpatialFabricSubtree or equivalent)" |
| CM-045 | Up-axis convention for transcluded assets (upAxis) | implemented | spec is silent | [01-coordinate-precision-units-and-extents.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md), section "Transclusion contract (SpatialFabricSubtree or equivalent)" |
| CM-046 | World (units, extent, upAxis) | open-question | API.yaml:268-349 at d39a1a0 | [01-coordinate-precision-units-and-extents.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md), section "World resource spatial properties" |
| CM-047 | Scene flatten / dRenderScale | interpretation | spec is silent | [01-coordinate-precision-units-and-extents.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md), section "Per-scene normalization (SHOULD)" |
| CM-048 | Node.localTransform | interpretation | API.yaml:488-491 at d39a1a0 | [01-coordinate-precision-units-and-extents.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md), section "Node.localTransform" |
| CM-049 | Re-root on approach with hysteresis | drafted | spec is silent | [01-coordinate-precision-units-and-extents.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md), section "Re-root hysteresis (MUST for implementations that support proximity transitions)" |

## Compose it

8 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-001 | Node.children | implemented | API.yaml:484-487 at d39a1a0 | [08-composition-graph-schema-fixes.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md), section "1. Node.children: embedded and reference forms" |
| CM-002 | GET /wow/spatial/{spatialID} default response | implemented | API.yaml:133-153 at d39a1a0 | [08-composition-graph-schema-fixes.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md), section "2. Content negotiation on GET /wow/spatial/{spatialID}" |
| CM-003 | POST /wow/spatial/{spatialID}/node/{nodeId} canonical body handling | implemented | API.yaml:158-188 at d39a1a0 | [08-composition-graph-schema-fixes.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md), section "3. POST body-shape handling and error codes" |
| CM-004 | Node extension mechanism (WebOfWorldsExtension bag) | implemented | API.yaml:471-495 at d39a1a0 | [08-composition-graph-schema-fixes.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md), section "4. Named extension point on Node (optional)" |
| CM-005 | GET /wow/spatial/{spatialID} form query parameter | implemented | spec is silent | [08-composition-graph-schema-fixes.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md), section "2. Content negotiation on GET /wow/spatial/{spatialID}" |
| CM-006 | X-OSL-WoW-Node-Form header and 400/422 refusal codes on POST | implemented | spec is silent | [08-composition-graph-schema-fixes.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md), section "3. POST body-shape handling and error codes" |
| CM-007 | Spatial schema | implemented | API.yaml:438-468 at d39a1a0 | [08-composition-graph-schema-fixes.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md), section "2. Content negotiation on GET /wow/spatial/{spatialID}" |
| CM-008 | X-OSL-WoW-Spatial-Form request/response headers | implemented | spec is silent | [08-composition-graph-schema-fixes.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md), section "2. Content negotiation on GET /wow/spatial/{spatialID}" |

## Draw it

8 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-036 | Node.spatialAssetURI for signed spatial documents (SpatialFabricSubtree) | implemented | API.yaml:492-493 at d39a1a0 | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "Transclusion contract for signed spatial documents" |
| CM-037 | Rendering backend selection for transcluded assets (placement) | implemented | spec is silent | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |
| CM-038 | OpenSpatialAsset signed document type | open-question | OpenSpatialAsset/API.yaml:51-161 at d39a1a0 | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "Open questions for the working group" |
| CM-039 | Node.spatialAssetURI (no format, no media type, no prose) | deferred | API.yaml:492-493 at d39a1a0 | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "spatialAssetURI format constraint" |
| CM-040 | SpatialFabricSubtree.parallax (backdrop parallax dial) | implemented | spec is silent | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |
| CM-041 | SpatialFabricSubtree.maxDepth (recursion cap) | implemented | spec is silent | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |
| CM-042 | SpatialFabricSubtree.epochTicks (time epoch for time-dependent fabrics) | implemented | spec is silent | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |
| CM-043 | First-party TeleportXR browser rendering | refused | spec is silent | [07-assets-and-the-render-seam.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md), section "What Open Spatial Lab built and learned" |

## Cross a portal

18 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-009 | Portal destination | implemented | API.yaml:409-436 at d39a1a0 | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Portal destination (MUST)" |
| CM-010 | Crossing continuity (why it does not feel like a page load) | drafted | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Crossing continuity (SHOULD)" |
| CM-011 | Arrival pose mapping from portal frames | implemented | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Pose mapping across portal frames (MUST)" |
| CM-012 | Portal interaction geometry (Zone) | implemented | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Portal interaction geometry (SHOULD, optional extension)" |
| CM-013 | GET /wow/portal/{portalId} response composite (OSLPortalResponse) | implemented | API.yaml:110-129 at d39a1a0 | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Portal response (MUST)" |
| CM-014 | Node type discrimination (portal vs geometry node) | deferred | API.yaml:471-495 at d39a1a0 | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Node type discrimination (SHOULD)" |
| CM-015 | World-to-world crossing / portal traversal semantics | deferred | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Open questions for the working group" |
| CM-016 | IWPS-shaped Query and Teleport handshake | interpretation | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "What Open Spatial Lab built and learned" |
| CM-017 | Exit-intent and arrival notifications with handoff_id correlation | implemented | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Server-side exit and arrival notifications (SHOULD)" |
| CM-018 | One-avatar-per-world invariant | implemented | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "One-avatar-per-world invariant (MUST)" |
| CM-019 | Portal traversal controller routes and fields | implemented | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "What Open Spatial Lab built and learned" |
| CM-020 | World-to-world crossing re-sourced onto the composition graph | drafted | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Crossing destination resolution (SHOULD)" |
| CM-021 | Portal scale semantics in a host graph | open-question | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Open questions for the working group" |
| CM-022 | No-reload in-memory promotion (one-avatar invariant mechanism) | implemented | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "No-reload crossing (SHOULD)" |
| CM-023 | Portal traversal direction (PortalTraversal) | implemented | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Portal traversal direction (SHOULD, optional extension)" |
| CM-024 | Native TeleportXR teleport / session behavior | refused | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "What Open Spatial Lab built and learned" |
| CM-025 | RP1 fail-closed gate at crossing | implemented | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Policy gate before crossing (SHOULD)" |
| CM-026 | Portal trigger volume and plane-crossing detection | implemented | spec is silent | [02-portal-destination-and-traversal.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md), section "Portal trigger model (SHOULD, optional extension)" |

## Carry yourself across

5 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-027 | GET /wow/user/{userId} identity and signature (OSLUserResponse + OpenUserManifest) | implemented | API.yaml:47-66 and 351-380 at d39a1a0 | [03-portable-user-state-and-identity.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md), section "Proposed normative text" |
| CM-028 | User identity verification (UMSignature) | implemented | spec is silent | [03-portable-user-state-and-identity.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md), section "Proposed normative text" |
| CM-029 | User.age and OpenUserManifest | implemented | API.yaml:351-380 (no age field) and OpenUserManifest/API.yaml:49-60 (age at L57-58) at d39a1a0 | [03-portable-user-state-and-identity.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md), section "Proposed normative text" |
| CM-030 | Core Requirement: world entry role and embodiment | interpretation | spec is silent | [03-portable-user-state-and-identity.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md), section "What Open Spatial Lab built and learned" |
| CM-031 | DELETE /wow/user/{userId} | deferred | API.yaml:67-85 at d39a1a0 | [03-portable-user-state-and-identity.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md), section "Open questions for the working group" |

## Be there together

4 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-032 | WS /events realtime convention | implemented | spec is silent | [05-presence-live-sync-and-persistence.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md), section "Real-time event channel" |
| CM-033 | Core Requirement: preview | interpretation | README.md:13 at d39a1a0 | [05-presence-live-sync-and-persistence.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md), section "What Open Spatial Lab built and learned" |
| CM-034 | Follower visibility and departure behavior | open-question | spec is silent | [05-presence-live-sync-and-persistence.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md), section "Follower rules" |
| CM-035 | Core Requirement: follow (bare, no aspect) | interpretation | README.md:12 at d39a1a0 | [05-presence-live-sync-and-persistence.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md), section "Bare #follow" |

## Trust what you see

5 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-050 | Proof boundary on /wow responses | implemented | spec is silent | [04-provenance-and-signed-subtrees.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/04-provenance-and-signed-subtrees.md), section "Proof-boundary declaration" |
| CM-051 | Trust boundary at every jump | implemented | spec is silent | [04-provenance-and-signed-subtrees.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/04-provenance-and-signed-subtrees.md), section "Verification on navigation" |
| CM-052 | GET /wow/world response composite (OSLWorldResponse) | implemented | API.yaml:32-44 at d39a1a0 | [04-provenance-and-signed-subtrees.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/04-provenance-and-signed-subtrees.md), section "Response extension mechanism" |
| CM-053 | Asset verification / trust for transcluded content (requireVerified) | implemented | spec is silent | [04-provenance-and-signed-subtrees.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/04-provenance-and-signed-subtrees.md), section "Verification of transcluded content" |
| CM-054 | GET /wow/view/{viewId} response composite (OSLViewResponse) | implemented | API.yaml:88-107 at d39a1a0 | [04-provenance-and-signed-subtrees.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/04-provenance-and-signed-subtrees.md), section "Response extension mechanism" |

## Know what conforms

8 rows.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-066 | spatialID and nodeId path parameter declarations on node operations | implemented | API.yaml:156-260 at d39a1a0 | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "3. spatialID parameter declaration" |
| CM-067 | Scene vs Spatial path naming contradiction | interpretation | README.md:28,31 and API.yaml:133 at d39a1a0 | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "5. Graph path adjudication" |
| CM-068 | Extension policy (additive, labeled, surfaced, honest) | implemented | spec is silent | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "8. Extension policy (optional)" |
| CM-069 | Conformance vocabulary (zero RFC 2119 keywords, no required properties) | open-question | spec is silent | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "1. RFC 2119 keyword adoption" |
| CM-070 | Core Requirement: Preview description truncated; Persist and Share have no description | open-question | README.md:13-15 at d39a1a0 | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "7. Core Requirement descriptions" |
| CM-071 | Core Requirement: aspect.id syntax in URL#join=aspect.id | interpretation | README.md:11 at d39a1a0 | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "9. Aspect identifier syntax (optional)" |
| CM-072 | aspect.id syntax and failure semantics | open-question | README.md:11-13 name aspect.id; silent on its syntax and failure semantics | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "9. Aspect identifier syntax (optional)" |
| CM-073 | GeoPose.position.lan misspelling of longitude | interpretation | API.yaml:294-295 at d39a1a0 | [09-conformance-vocabulary-and-errata.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md), section "6. GeoPose.position.lan correction" |

## Among its neighbours

1 row.

| Id | Surface | Status | Spec citation | Treated in |
|---|---|---|---|---|
| CM-055 | Core Requirement: persist and share | interpretation | README.md:14-15 at d39a1a0 | [05-presence-live-sync-and-persistence.md](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md), section "Persist and share fragment rule" |

---

## Counts

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

- WoWAPI repository at commit d39a1a0 (2026-05-21)
- Open Spatial Lab completion map (73 rows, 2026-09-07)
- Open Spatial Lab evidence ledgers: schema coverage, specification-to-implementation delta, crossing and navigation (internal; cited by row in the completion map)
- The ten numbered documents in the completion report set

---

## Change log

- 2026-09-07: first public draft, verified twice.
