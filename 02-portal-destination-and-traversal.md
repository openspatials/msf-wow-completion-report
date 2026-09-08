# Portal Destination and the Traversal Protocol

WoWAPI 0.0.1 defines a positioned Portal but no canonical destination property or interoperable traversal protocol. The whitepaper describes linked spatial resources; the proposal here binds a portal target in the API and identifies the remaining traversal decisions. Existing open objects permit implementation-specific target fields already.

The immediate ask is an optional canonical `destination` field while retaining numeric Portal identifiers. A separate future traversal profile would define target resolution, pose mapping, notifications, presence recovery and activation. The local prototype supplies implementation evidence and failure cases; it does not prove a complete protocol between independently operated worlds.

**Status:** Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).


## What the specification says today

The Portal schema is defined at [API.yaml lines 409-436](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L409-L436):

```yaml
Portal:
  type: object
  properties:
    id:
      type: number
    geoPose:
      type: object
      properties:
        position:
          type: object
          properties:
            lat:
              type: number
            lan:
              type: number
            h:
              type: number
        angles:
          type: object
          properties:
            yaw:
              type: number
            pitch:
              type: number
            roll:
              type: number
```

Two properties: `id` (number) and `geoPose` (position plus angles). No destination, no target, no frame geometry, no traversal mode, no interaction volume.

GET /wow/portal/{portalId} returns this schema ([API.yaml lines 109-129](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L109-L129)). The response schema defines no other members; because the object is open, extra implementation fields are allowed.

The Node schema ([API.yaml lines 471-495](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L471-L495)) has no type or role field. A client reading the composition graph cannot distinguish a portal node from a geometry node without out-of-band knowledge.

The pinned API does not define a portal-traversal binding; the whitepaper names portals linking worlds on printed pages 14 and 21. Searches for the following terms in API.yaml at commit d39a1a0 return zero results: `destination`, `target`, `crossing`, `traversal`, `handoff`, `handshake`, `trigger`, `zone`, `prefetch`, `arrival`, `departure`, `exit-intent`, `frame` (one hit at line 324 refers to "Web framework", not portals), `one-way`, `bidirectional`, `scale` (in the portal context). The README ([README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md)) does not mention portals. The World schema in API.yaml includes a `portals.portal_count` field (lines 343-349), but the README's Optional Feature table lists only world, user, and scene endpoints.


## What fails without it

**No shared destination binding.** The canonical Portal has `id` and `geoPose` but no defined destination member. An extension is schema-legal, yet two implementations need shared semantics to agree which world a portal names and how that identity resolves to endpoints. The architecture can describe linked worlds without this particular API binding being complete. (CM-009.)

**No crossing protocol between worlds.** The specification defines read endpoints for worlds, users, views, portals, and spatial graphs, a delete endpoint for users, and full CRUD for spatial nodes, but defines no protocol for what happens when a user walks through a portal. There is no handshake, no state-transfer contract, no presence lifecycle at the boundary, and no correlation between a departure from one world and an arrival in another. Each implementer must invent a crossing protocol, and independent choices need not interoperate. (Source: CM-015, CM-016, CM-017.)

**Arbitrary arrival position.** Without a defined pose mapping between source and target portal frames, clients can choose different arrival positions. Camera heading and lateral offset have no shared preservation rule. (Source: CM-011.)

**No trigger semantics.** The specification provides no vocabulary for when or how a portal crossing fires. Without trigger geometry (a volume, an aperture, a plane-crossing test), two implementations can disagree on the moment of activation. (Source: CM-026.)

**Ghost avatars after crossing.** Without a defined presence lifecycle at portal boundaries, a user who crosses from World A to World B can appear in both worlds simultaneously. The source world does not know the user left; the destination world does not know the user arrived. (Source: CM-018.)

**Portal nodes indistinguishable from geometry nodes.** The Node schema has no type or role field. A renderer reading the composition graph cannot tell whether a node should trigger traversal logic or render an asset. (Source: CM-014.)


## What Open Spatial Lab built and learned

Open Spatial Lab (OSL) implemented a portal traversal system as a set of labeled, non-canonical extensions to the Web of Worlds API. Every extension is annotated `x-osl-extension: true` in the OSL schema and every API response carries `standards_conformance: false` (source: OSL schema.yaml, x-osl-extension tags; OSL contract, proof-boundary policy). No conformance is claimed.

**Portal destination.** OSL added a `PortalDestination` schema with `target_world_id`, `target_location_id`, `target_base_url`, and `spatial_fabric_address` (source: OSL schema.yaml lines 697-714). The destination rides on `OSLPortalResponse.destination`, an extension of the canonical Portal response (source: OSL schema.yaml lines 1099-1115). This is the field the canonical Portal does not have. It was proposed as a canonical addition.

**Portal traversal controller.** OSL implemented a portal traversal controller that manages the full crossing lifecycle: trigger detection, exit-intent notification, pose mapping, root promotion, arrival notification, and presence handoff (source: live-adapter-portal-traversal-controller.mjs). The controller tracks portal frame geometry (position, forward, up, right, width, height, trigger depth), signed plane distance, oval aperture membership, crossing direction, and traversal mode (source: live-adapter-portal-traversal-controller.mjs lines 58-113).

**Pose mapping.** The function `mapTransformBetweenPortalFrames` computes the arrival position and rotation from the source portal frame, target portal frame, and entry transform (source: live-adapter-portal-traversal-controller.mjs lines 620-625). The mapped exit transform is carried in the handoff packet as `avatar_context.portal_frame_transform`. The mapping is pure math with no renderer dependency.

**Trigger geometry.** Portal trigger volumes are oval apertures with width, height, and trigger depth. A crossing commits when the avatar crosses the portal plane from an allowed entry side while inside the oval aperture (source: live-adapter-portal-traversal-controller.mjs lines 272-388). The detection uses signed-distance math against the portal plane.

**Exit-intent and arrival notifications.** The controller awaits POST /portal/exit-intent at the source before crossing. For a registered player, the request carries `player_id`; the source removes that presence record while issuing the handoff packet. POST /portal/arrival then sends the packet to the target, whose failure does not block visual composition. The target stores the client-supplied `handoff_id`; the inspected arrival handler does not authenticate it against the source. It correlates records, but is not proof that the source authorized a transfer (traversal controller lines 755–776 and 978–989; runtime-state.js lines 2929–2956 and 3048–3079).

**Visual continuity and source-side presence removal.** The source's accepted exit-intent removes the identified player and starts a five-second departure tombstone. A heartbeat for that missing player cannot re-register it while the tombstone is active; an explicit registration can clear the tombstone. The controller later commits the destination scene, sends an idempotent departure confirmation, and registers at the destination. Losing only that later confirmation does not undo the earlier source removal. The retained September 7 client-only probe bypassed this server mechanism, so its two simulated registries are not a result for the two-server crossing (CM-018).

The code separates failure cases:

- If the exit-intent never reaches the source and the request fails, the client remains in the source world.
- If the source accepts it but its reply is lost, the client can remain visually at the source while its presence record is absent. After the tombstone expires, a delivered heartbeat can restore the record.
- If destination registration fails, a later delivered heartbeat can upsert the destination record. The nominal heartbeat interval is three seconds, not a guaranteed network recovery bound.
- A pagehide beacon attempts departure. If a tab crashes or delivery fails, presence expires after its last accepted heartbeat: the default lease is ten seconds, the hidden-tab request is sixty seconds, and server bounds are one to sixty seconds.

These are code-derived failure paths, not a new network test. A heartbeat delayed beyond the five-second tombstone can use the upsert path again; the timer therefore supplies a bounded race guard, not global exclusivity under arbitrary delay. Visual continuity, per-world presence and session authority remain separate results.

**No-reload root promotion.** The controller promotes the target in the same JavaScript execution context without requesting document navigation. The exact July 5 local three-window receipt recorded one main-frame navigation in each of the player, source-observer and target-observer windows. A later July 11 receipt with the same historical name is a different run and failed three stale handshake assertions. Neither receipt proves independent cross-engine interoperability.

**Crossing continuity.** The local design preserves a visual avatar and maps a portal-relative pose during a prefetched scene swap. Visual commit occurs before the remaining arrival/presence work completes. The yaw-oriented demonstration does not establish arbitrary six-degree-of-freedom or scale-changing pose mappings.

**IWPS-shaped handshake.** OSL built a post-hoc rendering of the crossing onto the OMA3 IWPS v0.3 Query-then-Teleport two-call protocol (source: iwps-query-teleport.mjs lines 1-95). The cited local crossing makes two POSTs in IWPS order to two separate world servers. However, no IWPS parameter is on the wire: the request body is demo-native snake_case. The module itself documents this as a design study, not an implementation, and `iwps_conformance` stays false (source: iwps-query-teleport.mjs, IWPS_CONFORMANCE descriptor).

**Interaction volumes and traversal direction.** OSL added a `Zone` schema (kind, position, radius_m) and a `PortalTraversal` schema (mode: bidirectional/one-way, allowed_entry_side) as labeled extensions (source: OSL schema.yaml lines 854-880).

**Portal response composite.** OSL returns `OSLPortalResponse`, which extends Portal with `label`, `proof_boundary`, `webofworlds_extension`, and `destination` (source: OSL schema.yaml lines 1099-1115).

**Crossing re-sourced onto the composition graph.** OSL re-sourced the destination scene from the WoW Spatial Composition Graph (/wow/spatial/{id} walk) so that the crossing carries no format-specific dependency. The destination is resolved from Portal.destination (source: deferred-conformance-ledger DCL-011).

**Node type discrimination.** OSL deferred canonical node-type discrimination. Portal destination on graph nodes is carried via a parseable `osl-portal:` spatialAssetURI and a labeled webofworlds_extension with `role:portal` (source: deferred-conformance-ledger DCL-010).

**Evidence and claim boundary.** The retained crossing receipt contains 81 passing assertions from Node with two local backend instances, including two minted continuity identities. Its browser-machinery checks are source-presence tripwires, not browser execution. It is not a count of completed user journeys or a reliability estimate. Historical browser evidence is distinct from that contract run. No independent end-to-end interoperability claim is made; `standards_conformance` remains false.

**Not attempted.** Native TeleportXR teleport was not tested. OSL is a browser-side viewer with application-level portal crossing between local origins. This is a disclosure, not a failure (source: wow-spec-coverage.mjs, entry oos.native_teleport; CM-024).


## Proposed normative text

All text and fragments below are unadopted proposals. The destination addition is the bounded base ask; the rest belongs to a separately agreed traversal profile. Schema fragments target OpenAPI 3.0.4.

### Optional Portal destination

```yaml
PortalWithDestination:
  allOf:
    - $ref: '#/components/schemas/Portal'
    - type: object
      properties:
        destination:
          $ref: '#/components/schemas/PortalDestination'
PortalDestination:
  type: object
  required: [target_world_id]
  properties:
    target_world_id:
      type: string
      format: uri
      description: Proposed absolute URI identifying the target world; not automatically an API base.
    target_base_url:
      type: string
      description: Optional service-base URI reference resolved against the Portal response retrieval URI.
    target_portal_id:
      type: number
      description: Identifier of a Portal in the destination; preserves the canonical numeric Portal type.
```

The referenced canonical `Portal.id` remains a number. `destination` is optional, so an existing `{"id":7}` remains valid. In the proposed schema a present destination must name a target; `{"destination":{}}` is invalid. This constrains a newly named member and must be versioned if earlier private extensions use that name differently.

### Target resolution and response behavior

Proposed rule: return `destination` when the target is known. Resolve a relative `target_base_url` against the retrieval URI using the declared URI-reference rules, not text concatenation. Map `target_world_id` to the advertised service base through an agreed discovery/profile binding; do not assume every identifier is itself a fetchable API base. The relation between identity and endpoint must be checked according to the selected profile.

An absent destination, unsupported scheme, unresolved identity, failed lookup or conflicting identity/base relation is an unresolved target. The client reports the reason and retains the current scene rather than guessing a world. Query and fragment semantics are preserved as described in [chapter 06](06-discovery-and-addressing.md). A stricter traversal profile could require a destination for traversable portals, but that would be a separate compatibility decision.

### Pose mapping across portal frames (candidate traversal profile)

The profile must define source and target frames, units, rotation convention and whether authored scale changes are allowed. For aligned distance components, use `target = source / sourceUnitsPerMeter * targetUnitsPerMeter`; 100 units at 100 units/metre becomes one unit at one unit/metre. Compose this with the agreed frame transform and any intentional model scale.

A rigid portal profile can require preservation of lateral offset and heading by an isometry. That requirement cannot silently cover scale-changing portals. The local yaw-oriented mapping is useful evidence; an asymmetric six-degree-of-freedom fixture with a stated arrival tolerance is still needed.

### Crossing continuity (candidate traversal profile)

A continuous-view profile should define the expected camera/avatar continuity and permitted visual interruption. Root precision changes and portal relocation are different cases; a portal may intentionally change the view. Prefetch and in-memory promotion are candidate techniques, not requirements for every WoW client.

### Server-side exit and arrival notifications (protocol sketch)

Open Spatial Lab sends exit-intent and arrival notifications correlated by a handoff identifier. The destination currently stores that client-asserted identifier without source authentication. A future profile could authenticate the source claim through a source-signed transfer packet, a source callback or another agreed mechanism; signing only the user manifest does not authenticate the whole handoff. A correlation identifier alone provides no transaction, retry or replay guarantee.

The proposed request content is a numeric `portal_id` at exit and a `handoff_id` at arrival, with source/target identity and pose/state fields bound by the selected profile. Before defining endpoints as interoperable, specify authentication, idempotency, duplicate handling, expiry, failed arrival recovery and observable status. This sketch is not a complete OpenAPI operation. In the actual local crossing path, failed arrival notification is logged and composition continues.

### Presence and authority during crossing (open profile decision)

Distinguish three invariants: one local visual avatar, one active session authority, and per-world presence records. The local source removes an identified player when accepting exit-intent, then uses a finite tombstone against stale heartbeat upserts. The later client departure is a confirmation. This reduces the dropped-confirmation race but does not make source removal and destination registration one distributed transaction.

A candidate presence profile could require source removal when exit-intent is accepted and define the client's departure as idempotent confirmation. It must specify the permitted interval with no presence, tombstone lifetime, delayed-heartbeat behavior, failed-response recovery, retry identity and expiry. Test a lost exit-intent request, a lost reply after source acceptance, a lost destination registration and a delayed heartbeat against independently controlled endpoints. Globally exclusive session authority needs its own ownership protocol and failure model.

Open Spatial Lab also carries `continuity_id` across worlds, using `avatar_id` as a fallback. That linkage supports continuity but can correlate a visitor across destinations. A portable profile must choose crossing-scoped or pairwise identifiers, consent and retention rules under the whitepaper's data-minimization intent; the current identifier is not itself an authenticated identity claim.

### Node type discrimination (SHOULD)

The Node schema SHOULD add a `type` or `role` field to distinguish portal nodes from geometry/asset nodes. Alternatively, the standard MAY define a well-known `spatialAssetURI` scheme for portal nodes.

Rationale: a renderer cannot determine whether a node should trigger traversal logic without a discriminator.


### Portal interaction geometry (SHOULD, optional extension)

A Portal SHOULD include an interaction volume definition specifying at least a trigger radius and optionally a prefetch radius.

```yaml
PortalZone:
  type: object
  properties:
    kind:
      type: string
      enum: [prefetch, trigger]
    radius_m:
      type: number
      minimum: 0
      exclusiveMinimum: true
      description: >
        Radius in metres from the portal centre.
```

Rationale: without a trigger volume, two implementations will disagree on when a portal activates.


### Portal trigger model (SHOULD, optional extension)

The standard SHOULD define a portal trigger model: a planar aperture (dimensions, depth) in a declared local portal frame, mapped from GeoPose when georeferencing is supplied. A crossing triggers when an avatar's bounding point crosses the portal plane from an allowed entry side while inside the aperture. The standard MAY define one-way traversal via an `allowed_entry_side` property.

Rationale: a deterministic, testable trigger model is needed for interoperable portal activation.


### Portal traversal direction (SHOULD, optional extension)

A Portal SHOULD support a `traversal` object with `mode` (bidirectional / one-way) and MAY include `allowed_entry_side`.

Rationale: not all portals are bidirectional; spatial narratives require one-way doors.


### No-reload crossing (optional continuous-view profile)

An in-memory root promotion can preserve the local rig, camera and cache. A continuous-view profile may require the corresponding observable continuity, while leaving the mechanism to the client. A reload can preserve selected state if it is explicitly transferred; it is not forbidden by the base destination addition.

### Crossing destination resolution (candidate graph-traversal profile)

For a profile that renders the destination composition graph, resolve the optional destination using the agreed identity-to-service binding, fetch the canonical Spatial descriptor and its root node, or request an explicitly negotiated alternative. Report missing or unsupported graph/profile data without guessing. Other destination entry modes remain possible.

Rationale: the optional destination field identifies a target; it does not require every world to adopt this graph-rendering traversal profile or a particular scene format.


### Policy decisions before crossing

A profile must separate manifest signature validity, trusted identity assertions, fresh holder control, destination admission and content execution permission. For required signed-fabric verification, failure must refuse that fabric under the configured publisher-trust policy. This does not imply that every identity/notification failure stops the current prototype: its separate arrival path logs failed manifest verification and can still promote the target.

## Adoption path

**Existing worlds.** The optional destination proposal preserves a Portal with numeric `id` and no destination. Such a response does not promise a traversable link. Requiring destination or changing the identifier type would narrow compatibility and is not part of the bounded ask.

**Clients.** Recognize the optional destination, apply the agreed resolution rules, and report unsupported or unresolved targets. Adopt a traversal profile only when its pose, continuity, admission and recovery behavior are agreed and advertised.

**Servers.** Publish known destinations using the negotiated version/profile. Notifications and presence behavior need a shared failure contract before two endpoints can claim interoperable transfer. A heartbeat timeout is eventual cleanup, not exclusive ownership proof.

## Open questions for the working group

1. **IWPS adoption.** Should Web of Worlds adopt the OMA3 IWPS v0.3 Query-then-Teleport two-call protocol as the normative traversal handshake? OSL built a post-hoc rendering of its crossing onto IWPS vocabulary, but no IWPS parameter is on the wire and `iwps_conformance` stays false. The two-service topology IWPS assumes (source world, destination world) matches the Web of Worlds model. The gap is that IWPS defines mandatory response fields (`approval`, `destinationUrl`) and a security profile that the current implementation does not serve. (Source: CM-016, R-019.)

2. **Portal scale semantics.** What does a portal's TRS scale mean when the portal is placed in a rigid (isometry-only) host graph? Options: (a) the scale is informational and does not affect the host frame, (b) the scale defines a coordinate-system transition for content crossing the portal, (c) the scale participates in intensity compensation per a defined formula. OSL identified this as a disclosed limitation when a fabric-side portal carries a large scale factor (e.g. 2e8) that the rigid host frame cannot losslessly carry (source: UPSTREAM-REGISTER.md entry C12; CM-021).

3. **Native TeleportXR teleport.** Does conformance require native TeleportXR teleport, or is application-level portal crossing a valid alternative? OSL is a browser-side viewer and did not attempt native TeleportXR teleport. This is a disclosure, not a failure (source: CM-024, wow-spec-coverage.mjs entry oos.native_teleport).

4. **Canonical crossing semantics.** Should the specification define portal traversal semantics (handshake, state transfer, continuity), or is crossing purely implementation-defined? OSL built a local visual crossing with best-effort notifications and presence; the profile boundary remains a group decision (source: CM-015).


## Sources

- [OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialWorld/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf); relevant printed pages are identified in this chapter or [chapter 10](10-role-and-blind-spots.md).
- OMA3 IWPS Base Specification v0.3, as identified by the retained iwps-query-teleport.mjs conformance descriptor. The local two-call design is not IWPS wire conformance.
- Open Spatial Lab local source snapshot and retained evidence, checked September 7, 2026: schema.yaml, live-adapter-portal-traversal-controller.mjs, live-adapter-presence-controller.mjs, runtime-state.js, iwps-query-teleport.mjs, NAVIGATION-ARCHITECTURE.md, deferred-conformance-ledger.md and wow-spec-coverage.mjs. Source-side exit removal, the five-second tombstone and lease behavior were checked in code; they were not rerun over the network. The retained crossing receipt reports 81 Node assertions with two local backends; browser source-presence checks are not browser runs. The July 5 three-window browser receipt and failed July 11 successor are distinct historical results. Public reproduction of these exact local bytes is not established.
- [Appendix A](A-completion-map.md) and [Appendix B](B-findings-register.md) preserve the historical surface/finding identifiers.

## Change log

- 2026-09-07: corrected source scope, proposal compatibility and evidence boundaries; updated public citations.
