# Portal Destination and the Traversal Protocol

The Web of Worlds specification defines a Portal as a positioned point with no target. A portal that cannot say where it leads cannot connect two worlds, and a standard with no traversal protocol leaves every implementer to invent their own crossing. This document proposes the normative additions that turn the Portal into a link and define what happens when a user walks through it.

The core addition (high confidence, verified against the spec and a working implementation): Portal MUST carry a destination object with at minimum a target world identifier and a target base URL. The traversal protocol additions (medium confidence, proven in one implementation, not yet tested between independent implementations): the standard SHOULD define pose mapping across portal frames, server-side exit and arrival notifications with a correlation identifier, a depart-then-register presence lifecycle, and a trigger geometry model for portal activation. Three items are open questions for the working group: whether to adopt the OMA3 IWPS Query-then-Teleport handshake, what a portal's TRS scale means in a rigid host graph, and whether native TeleportXR teleport is required for conformance.

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

GET /wow/portal/{portalId} returns this schema ([API.yaml lines 109-129](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L109-L129)). The response carries no additional fields beyond the schema above.

The Node schema ([API.yaml lines 471-495](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L471-L495)) has no type or role field. A client reading the composition graph cannot distinguish a portal node from a geometry node without out-of-band knowledge.

The specification is silent on portal traversal. Searches for the following terms in API.yaml at commit d39a1a0 return zero results: `destination`, `target`, `crossing`, `traversal`, `handoff`, `handshake`, `trigger`, `zone`, `prefetch`, `arrival`, `departure`, `exit-intent`, `frame` (one hit at line 324 refers to "Web framework", not portals), `one-way`, `bidirectional`, `scale` (in the portal context). The README ([README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md)) does not mention portals. The World schema in API.yaml includes a `portals.portal_count` field (lines 343-349), but the README's Optional Feature table lists only world, user, and scene endpoints.


## What fails without it

**A portal graph with no links.** The canonical Portal has `id` and `geoPose` but no field that says where the portal leads. An implementer who builds a portal between World A and World B has no conformant way to express "this portal in World A leads to World B." The portal is a positioned point with no arrow. Two independently operated world servers cannot build a linked portal graph from the current schema. (Source: CM-009, verified against the spec.)

**No crossing protocol between worlds.** The specification defines read endpoints for worlds, users, views, portals, and spatial graphs, a delete endpoint for users, and full CRUD for spatial nodes, but defines no protocol for what happens when a user walks through a portal. There is no handshake, no state-transfer contract, no presence lifecycle at the boundary, and no correlation between a departure from one world and an arrival in another. Each implementer must invent a crossing protocol, and two implementations that do so independently will not interoperate. (Source: CM-015, CM-016, CM-017.)

**Arbitrary arrival position.** Without a defined pose mapping between source and target portal frames, the user arrives at an undefined position in the destination world. Camera heading and lateral offset relative to the portal aperture are lost. (Source: CM-011.)

**No trigger semantics.** The specification provides no vocabulary for when or how a portal crossing fires. Without trigger geometry (a volume, an aperture, a plane-crossing test), two implementations will disagree on the moment of activation. (Source: CM-026.)

**Ghost avatars after crossing.** Without a defined presence lifecycle at portal boundaries, a user who crosses from World A to World B can appear in both worlds simultaneously. The source world does not know the user left; the destination world does not know the user arrived. (Source: CM-018.)

**Portal nodes indistinguishable from geometry nodes.** The Node schema has no type or role field. A renderer reading the composition graph cannot tell whether a node should trigger traversal logic or render an asset. (Source: CM-014.)


## What Open Spatial Lab built and learned

Open Spatial Lab (OSL) implemented a portal traversal system as a set of labeled, non-canonical extensions to the Web of Worlds API. Every extension is annotated `x-osl-extension: true` in the OSL schema and every API response carries `standards_conformance: false` (source: OSL schema.yaml, x-osl-extension tags; OSL contract, proof-boundary policy). No conformance is claimed.

**Portal destination.** OSL added a `PortalDestination` schema with `target_world_id`, `target_location_id`, `target_base_url`, and `spatial_fabric_address` (source: OSL schema.yaml lines 697-714). The destination rides on `OSLPortalResponse.destination`, an extension of the canonical Portal response (source: OSL schema.yaml lines 1099-1115). This is the field the canonical Portal does not have. It was proposed upstream as WO-018.

**Portal traversal controller.** OSL implemented a portal traversal controller that manages the full crossing lifecycle: trigger detection, exit-intent notification, pose mapping, root promotion, arrival notification, and presence handoff (source: live-adapter-portal-traversal-controller.mjs). The controller tracks portal frame geometry (position, forward, up, right, width, height, trigger depth), signed plane distance, oval aperture membership, crossing direction, and traversal mode (source: live-adapter-portal-traversal-controller.mjs lines 58-113).

**Pose mapping.** The function `mapTransformBetweenPortalFrames` computes the arrival position and rotation from the source portal frame, target portal frame, and entry transform (source: live-adapter-portal-traversal-controller.mjs lines 620-625). The mapped exit transform is carried in the handoff packet as `avatar_context.portal_frame_transform`. The mapping is pure math with no renderer dependency.

**Trigger geometry.** Portal trigger volumes are oval apertures with width, height, and trigger depth. A crossing commits when the avatar crosses the portal plane from an allowed entry side while inside the oval aperture (source: live-adapter-portal-traversal-controller.mjs lines 272-388). The detection uses signed-distance math against the portal plane.

**Exit-intent and arrival notifications.** The controller posts two server notifications: POST /portal/exit-intent to the source world (returns a handoff_id) and POST /portal/arrival to the target world (carries the full handoff packet). The handoff_id is a correlation key that links exit to arrival (source: live-adapter-portal-traversal-controller.mjs lines 758-776, 978). These are labeled as non-load-bearing for the composition.

**One-avatar-per-world invariant.** Crossing follows depart-then-register: `presence.departPresence` on the source, then `presence.registerPresence` on the destination. The avatar is nulled on departure and re-created on arrival. A `continuity_id` survives the crossing (source: live-adapter-portal-traversal-controller.mjs lines 1192-1196).

**No-reload root promotion.** The child fabric is promoted to the root frame of the same JavaScript execution context via `promoteActiveEndpoint`. No page reload, no `location.assign()`, no URL navigation. The main-frame navigation count stays pinned at 1 (source: live-adapter-portal-traversal-controller.mjs lines 928-1017, which calls `promoteActiveEndpoint` imported from the live adapter module).

**Crossing continuity.** OSL identified four continuity ingredients: (1) prefetched scene swap is one-to-few-frames, (2) pose maps exactly via position subtraction, (3) the parent context does not vanish, (4) visual continuity is measured, not asserted (source: NAVIGATION-ARCHITECTURE.md lines 383-417).

**IWPS-shaped handshake.** OSL built a post-hoc rendering of the crossing onto the OMA3 IWPS v0.3 Query-then-Teleport two-call protocol (source: iwps-query-teleport.mjs lines 1-95). The crossing genuinely makes two POSTs in IWPS order to two separate world servers. However, no IWPS parameter is on the wire: the request body is demo-native snake_case. The module itself documents this as a design study, not an implementation, and `iwps_conformance` stays false (source: iwps-query-teleport.mjs, IWPS_CONFORMANCE descriptor).

**Interaction volumes and traversal direction.** OSL added a `Zone` schema (kind, position, radius_m) and a `PortalTraversal` schema (mode: bidirectional/one-way, allowed_entry_side) as labeled extensions (source: OSL schema.yaml lines 854-880).

**Portal response composite.** OSL returns `OSLPortalResponse`, which extends Portal with `label`, `proof_boundary`, `webofworlds_extension`, and `destination` (source: OSL schema.yaml lines 1099-1115).

**Crossing re-sourced onto the composition graph.** OSL re-sourced the destination scene from the WoW Spatial Composition Graph (/wow/spatial/{id} walk) so that the crossing carries no format-specific dependency. The destination is resolved from Portal.destination (source: deferred-conformance-ledger DCL-011).

**Node type discrimination.** OSL deferred canonical node-type discrimination. Portal destination on graph nodes is carried via a parseable `osl-portal:` spatialAssetURI and a labeled webofworlds_extension with `role:portal` (source: deferred-conformance-ledger DCL-010).

**Evidence and claim boundary.** OSL documents 48 out of 48 portal crossings with continuity preserved (source: repo/open-spatial-lab/docs/WORKING-GROUP-DOSSIER.md). This count is documented by OSL and was not re-run in this verification pass; confidence is medium until re-run (source: proof-ledger.md, "48/48 crossing-continuity proof" entry). All evidence is local proof between two localhost world-server nodes; no claim is made about interoperation with independently operated servers. Every API response carries `standards_conformance: false`.

**Not attempted.** Native TeleportXR teleport was not tested. OSL is a browser-side viewer with application-level portal crossing between local origins. This is a disclosure, not a failure (source: wow-spec-coverage.mjs, entry oos.native_teleport; CM-024).


## Proposed normative text

### Portal destination (MUST)

```yaml
Portal:
  type: object
  required:
    - id
    - destination
  properties:
    id:
      type: string
      format: uri
    geoPose:
      $ref: '#/components/schemas/GeoPose'
    destination:
      type: object
      required:
        - target_world_id
      properties:
        target_world_id:
          type: string
          format: uri
          description: >
            Identifier of the destination world.
        target_base_url:
          type: string
          format: uri
          description: >
            Base URL of the destination world server.
        target_portal_id:
          type: string
          description: >
            Identifier of the linked portal in the destination world.
```

Rationale: without a destination, a portal cannot express where it leads and the graph of worlds has no edges.

A conformant Portal MUST include a `destination` object with at minimum `target_world_id` (string, URI format). A Portal SHOULD include `target_base_url` so the client can resolve the destination server. A Portal MAY include `target_portal_id` to identify the linked portal in the destination world for pose mapping.


### Portal response (MUST)

GET /wow/portal/{portalId} MUST return the `destination` alongside the portal's position. The response SHOULD carry a human-readable `label`.

Rationale: the portal endpoint is the client's only way to learn where a portal leads.


### Pose mapping across portal frames (MUST)

A portal pair (source portal, linked target portal) MUST define an isometry: the user's position and heading relative to the source portal frame MUST map to the corresponding position and heading relative to the target portal frame. The mapping SHOULD preserve lateral offset and facing direction.

Rationale: without a defined pose mapping, the user arrives at an arbitrary position in the destination world.


### Crossing continuity (SHOULD)

A root-changing navigation MUST preserve camera pose continuity: the first frame after a root swap SHOULD render the same view from the same viewpoint, with only the underlying coordinate precision changed. A conformant client MAY implement verify-ahead prefetch to reduce the swap to one-to-few-frames.

Rationale: a visible page-load stutter during portal crossing breaks the spatial illusion.


### Server-side exit and arrival notifications (SHOULD)

A world server SHOULD expose an exit-intent endpoint accepting a portal identifier and returning a handoff correlation identifier. A world server SHOULD expose an arrival endpoint accepting the handoff packet. The handoff identifier MUST be unique per crossing and MUST correlate exit to arrival.

```yaml
paths:
  /wow/portal/exit-intent:
    post:
      requestBody:
        content:
          application/json:
            schema:
              type: object
              required:
                - portal_id
              properties:
                portal_id:
                  type: string
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                required:
                  - handoff_id
                properties:
                  handoff_id:
                    type: string
                    format: uuid
  /wow/portal/arrival:
    post:
      requestBody:
        content:
          application/json:
            schema:
              type: object
              required:
                - handoff_id
              properties:
                handoff_id:
                  type: string
                  format: uuid
```

Rationale: without server notifications, the source world cannot update presence and the crossing has no audit trail.


### One-avatar-per-world invariant (MUST)

A conformant client MUST depart presence from the source world before registering presence in the destination world during a portal crossing. A `continuity_id` SHOULD survive the crossing to enable same-person correlation. A world server SHOULD enforce ghost-free presence via heartbeat TTL.

Rationale: without depart-then-register ordering, a user can appear in two worlds at once.


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
      description: >
        Radius in metres from the portal centre.
```

Rationale: without a trigger volume, two implementations will disagree on when a portal activates.


### Portal trigger model (SHOULD, optional extension)

The standard SHOULD define a portal trigger model: a planar aperture (dimensions, depth) centred on the portal's geoPose. A crossing triggers when an avatar's bounding point crosses the portal plane from an allowed entry side while inside the aperture. The standard MAY define one-way traversal via an `allowed_entry_side` property.

Rationale: a deterministic, testable trigger model is needed for interoperable portal activation.


### Portal traversal direction (SHOULD, optional extension)

A Portal SHOULD support a `traversal` object with `mode` (bidirectional / one-way) and MAY include `allowed_entry_side`.

Rationale: not all portals are bidirectional; spatial narratives require one-way doors.


### No-reload crossing (SHOULD)

The standard SHOULD recommend that a conformant client implement portal crossings as in-memory root promotions rather than page reloads. The crossing MUST preserve the execution context, avatar state, and session cache. The crossing MUST NOT trigger a document navigation.

Rationale: a page reload loses all client state (avatar equipment, presence registration, camera pose, session cache).


### Crossing destination resolution (SHOULD)

A portal crossing SHOULD resolve the destination world by fetching its composition graph (/wow/spatial) and composing a scene from it. The destination address MUST be resolved from the Portal.destination field. The crossing MUST NOT require a format-specific payload when the composition graph is sufficient.

Rationale: the crossing must work between any two conformant world servers, not only those that serve a specific scene format.


### Policy gate before crossing (SHOULD)

A conformant client SHOULD implement a policy evaluation gate before committing a portal crossing. The gate MUST default to deny when the trust state is unknown or verification is incomplete.

Rationale: a crossing that fires in an untrusted context bypasses the trust model.


## Adoption path

**Minimal world (what stays valid).** A world that serves the current Portal schema (id, geoPose) is not broken by these additions. The `destination` field is the only MUST-level addition to the Portal schema. A world that adds `destination` to its Portal responses becomes addressable in a portal graph. All other additions are SHOULD or MAY level and do not affect a minimal world.

**What a client must do.** Read Portal.destination to learn where a portal leads. Implement depart-then-register presence at crossing boundaries. Map pose across portal frames using the isometry defined by the source and target portal geometry.

**What a server must do.** Add `destination` (with at least `target_world_id`) to Portal responses. Optionally implement /wow/portal/exit-intent and /wow/portal/arrival for crossing correlation. Optionally enforce ghost-free presence via heartbeat TTL.


## Open questions for the working group

1. **IWPS adoption.** Should Web of Worlds adopt the OMA3 IWPS v0.3 Query-then-Teleport two-call protocol as the normative traversal handshake? OSL built a post-hoc rendering of its crossing onto IWPS vocabulary, but no IWPS parameter is on the wire and `iwps_conformance` stays false. The two-service topology IWPS assumes (source world, destination world) matches the Web of Worlds model. The gap is that IWPS defines mandatory response fields (`approval`, `destinationUrl`) and a security profile that the current implementation does not serve. (Source: CM-016, R-019.)

2. **Portal scale semantics.** What does a portal's TRS scale mean when the portal is placed in a rigid (isometry-only) host graph? Options: (a) the scale is informational and does not affect the host frame, (b) the scale defines a coordinate-system transition for content crossing the portal, (c) the scale participates in intensity compensation per a defined formula. OSL identified this as a disclosed limitation when a fabric-side portal carries a large scale factor (e.g. 2e8) that the rigid host frame cannot losslessly carry (source: UPSTREAM-REGISTER.md entry C12; CM-021).

3. **Native TeleportXR teleport.** Does conformance require native TeleportXR teleport, or is application-level portal crossing a valid alternative? OSL is a browser-side viewer and did not attempt native TeleportXR teleport. This is a disclosure, not a failure (source: CM-024, wow-spec-coverage.mjs entry oos.native_teleport).

4. **Canonical crossing semantics.** Should the specification define portal traversal semantics (handshake, state transfer, continuity), or is crossing purely implementation-defined? OSL built the full crossing, but the question of how much of it belongs in the standard is a group decision (source: CM-015).


## Sources

- OpenSpatialWorld API specification (commit d39a1a0): [github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml)
- OpenSpatialWorld README (commit d39a1a0): [github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md)
- OSL WoW contract schema: repo/open-spatial-lab/wow-spec/schema.yaml
- OSL portal traversal controller: repo/open-spatial-lab/web/live-adapter-portal-traversal-controller.mjs
- OSL IWPS query-teleport module: repo/open-spatial-lab/web/iwps-query-teleport.mjs
- OSL navigation architecture: repo/open-spatial-lab/docs/NAVIGATION-ARCHITECTURE.md
- OSL deferred conformance ledger: repo/open-spatial-lab/.dev/ai/deferred-conformance-ledger.md
- OSL upstream register: repo/open-spatial-lab/docs/UPSTREAM-REGISTER.md
- OSL spec coverage: repo/open-spatial-lab/web/wow-spec-coverage.mjs
- OSL working group dossier: repo/open-spatial-lab/docs/WORKING-GROUP-DOSSIER.md
- OMA3 IWPS Base Specification v0.3 (referenced via OSL's iwps-query-teleport.mjs conformance descriptor)
- Web of Worlds Completion Map (73 rows, 2026-09-07)
- Findings and Recommendations (2026-09-07)


## Change log

2026-09-07: first public draft, verified.
