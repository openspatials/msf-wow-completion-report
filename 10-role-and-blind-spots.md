# Web of Worlds among its neighbours: role and blind spots

The Web of Worlds specification defines a spatial composition graph and a REST surface for worlds, nodes, portals, users, and views. It does not define the live-session verbs that would let two running worlds agree about anything: no crossing protocol, no presence wire contract, no coordinate vocabulary, no portable inventory, no preference envelope, no agent identity, and no governance document. Of the 102 interop sockets on the MSF Infrastructure Working Group architecture map, Web of Worlds claims 1 outright, claims 17 partially, and is silent on 84 (verified: R1 socket map, 102 rows, 2026-09-07). The composition graph is specified; the composition protocol is not.

Five blind spots and one known gap carry the highest cost for implementers. Each is a must-interop socket with zero coverage from any standard in the ecosystem. The proposed additions below are graded MUST (the spec cannot describe a working system without it), SHOULD (an implementer will hit a wall, but a minimal world can survive without it), or MAY (useful but genuinely optional). Every claim of "zero coverage" was verified by searching the R1 socket map for covering standards and finding none (verified: R1, 2026-09-07). Every claim of "silent" was verified by keyword search across all three API files at commit d39a1a0 (verified: grep against OpenSpatialWorld/API.yaml, OpenSpatialAsset/API.yaml, OpenUserManifest/API.yaml, 2026-09-07).

**Status line:** Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).


## What the specification says today

The specification defines eight schemas across three API files: World, User, View, Portal, Spatial, and Node in OpenSpatialWorld; Asset in OpenSpatialAsset; and UserManifest in OpenUserManifest. It exposes sixteen operations (eleven in OpenSpatialWorld, three in OpenSpatialAsset, two in OpenUserManifest). None of the schemas declares a `required` properties array; nine of the ten `required` annotations in API.yaml are path parameters on endpoints (lines 57, 76, 98, 120, 144, 167, 198, 217, 251); the tenth (line 226) marks a request body as required. No RFC 2119 keyword (MUST, SHOULD, SHALL) appears anywhere in the three API files (verified: `grep -c` for each keyword returns 0).

**On precision and coordinates:** API.yaml contains zero occurrences of `precision`, `float32`, `float64`, `units`, `scale`, `origin`, `extent`, `bounds`, `jitter`, `upAxis`, or `destination` (verified: case-insensitive grep, each term returns 0 hits). The World schema (API.yaml lines 268-349) declares content metadata (label, age_restriction, license, cost, version, duration), a geoPose, presence fields, technology fields, user counts, view counts, and portal counts. It declares no spatial units, no up-axis convention, no handedness, and no extent. Node.localTransform (API.yaml lines 488-491) is defined as `type: array, items: type: number` with no length constraint and no major-order convention.

**On shared spatial anchors:** The R1 socket map lists `geo.anchor-shared` as a must-interop socket. Web of Worlds has no claim on it. Zero MSF subjects cover it. Zero other standards cover it (verified: R1 socket row for geo.anchor-shared, 2026-09-07). This is the single must-interop socket with no coverage from any source.

**On portable inventory:** The R1 socket map lists `persist.inventory` as a must-interop socket. Web of Worlds has no claim. Five MSF subjects touch it (wow, iwps, um, omb, teleportxr) but none defines a portable inventory contract (verified: R1 socket row, 2026-09-07). The IWPS specification reserves an assets parameter but defines no schema for what it carries.

**On user preferences:** The R1 socket map lists `persist.prefs` as a must-interop socket. Web of Worlds has no claim. Four MSF subjects touch it but none specifies a preference contract (verified: R1 socket row, 2026-09-07). The whitepaper lists "Preferences & settings" under the Digital YOU concept, but the OpenUserManifest schema defines only three properties: `name` (string), `age` (number), and `avatarAssetURI` (string) (verified: OpenUserManifest/API.yaml lines 49-60, commit d39a1a0).

**On AI agent identity:** The R1 socket map lists `logic.agent` as a must-interop socket. Web of Worlds has no claim. Five of the six MSF subjects touch it (wow, iwps, um, omb, teleportxr) but none defines an agent protocol (verified: R1 socket row, 2026-09-07). The whitepaper states that AI agents operate alongside humans. The three API files define no agent identity type, no capability declaration, and no agent-to-agent protocol.

**On governance:** The R2 discussion-evidence report records 599 transcript hits for governance across five meetings, more than any technical topic (verified: R2, 2026-09-07). The API files contain zero governance definitions. Neither the README nor the whitepaper contains a governance reference. Five governance structures are named in meeting transcripts (initiative hub, business council, maintainer council, licence/CLA policy, SDO liaison protocol) and none is written as a formal document (verified: R3 rows gov-initiative-hub, gov-business-council, gov-maintainer-council, gov-license-cla, gov-sdo-liaison, all classified named-only).


## What fails without it

**Precision and coordinates.** An implementer composing two worlds at different spatial scales has no vocabulary to declare what "one unit" means. A Z-up spatial fabric mounted into a Y-up host renders upside down. A planetary-scale scene quantizes to visible jitter on float32 GPU pipelines because the world declares no extent and the renderer has no information to compute a normalization scale. Open Spatial Lab hit this when mounting a Z-up orrery fabric into the Y-up Three.js host: without declared unitsPerMeter and upAxis, the solar system rendered sideways at the wrong scale (verified: OSL implementation of SpatialFabricSubtree, CM-044 and CM-045).

**Shared spatial anchors.** Two AR devices in the same physical room cannot agree on where "here" is. Without a shared-anchor vocabulary, co-located augmented reality experiences do not align. No standard in the MSF ecosystem and no standard outside it covers this socket (documented: R1 geo.anchor-shared row shows zero coverage from any source).

**Portable inventory.** A user who acquires an item in World A and walks through a portal to World B cannot bring the item. The portal boundary has no inventory-transfer vocabulary. Every world is a walled garden for carried items (documented: R1 persist.inventory row, five MSF subjects touch the socket, none specifies a contract).

**User preferences.** A user who sets accessibility preferences (text size, colour contrast, motion reduction) in one world loses them in the next. The manifest promises portable preferences and delivers three fields (documented: R3 concept-user-manifest row, gap between whitepaper claim and schema).

**AI agent identity.** An implementer building an AI guide for a museum world has no way to declare the agent's permissions, distinguish it from a human user, or make its capabilities visible to other worlds. The whitepaper promises AI integration; the API defines none (documented: R3 concept-ai-integration row, aspiration classification).

**Governance.** The most discussed topic across all five recorded meetings has no written output. An implementer choosing whether to build on Web of Worlds cannot find a governance charter, a contributor licence agreement, a maintainer council charter, or an SDO liaison protocol.


## What Open Spatial Lab built and learned

**Precision vocabulary (labeled extension).** Open Spatial Lab added `unitsPerMeter` (number, required, no default) and `upAxis` (enum: z or y, required, no default) as required fields on the SpatialFabricSubtree schema. Both are labeled non-canonical extensions (`x-osl-extension: true`). The design decision to refuse absent values rather than default them was deliberate: a default silently reintroduces the bug the field exists to prevent. OSL also identified that the canonical World schema declares no units, scale, upAxis, handedness, or extent (CM-046). Node.localTransform is conventionally interpreted as a 16-float column-major 4x4 matrix, but this is labeled as an OSL convention, not the standard (CM-048).

**Claim boundary:** OSL proved that declaring units and up-axis on a transclusion contract prevents the sideways-render bug for its own fabric corpus. It did not prove that the vocabulary is sufficient for all coordinate systems or for non-metric unit conventions.

**Shared anchors, portable inventory, user preferences, and AI agent identity were not attempted.** OSL is a browser-side implementation. Shared spatial anchoring requires device-level AR capabilities outside the web sandbox. Portable inventory and preferences require a cross-world contract that depends on Portal.destination, which is itself an extension. Agent identity is an organizational design problem, not an implementation task at this stage. These four gaps are recorded as blind spots, not as implementation failures.

**Governance was not attempted.** Governance codification is an organizational task outside the scope of a reference implementation. The evidence for the gap is the R2 transcript count (599 hits) and the R3 classification (all five governance structures are named-only).


## Proposed normative text

### World precision vocabulary

```yaml
# Addition to the World schema (OpenSpatialWorld/API.yaml)
World:
  type: object
  properties:
    units:
      type: string
      enum: [metres]
      default: metres
      description: >
        The base unit of measurement for all coordinates in this world.
    unitsPerMeter:
      type: number
      exclusiveMinimum: 0
      description: >
        Scale factor from world units to metres. 1.0 when units are metres
        at 1:1 scale.
    upAxis:
      type: string
      enum: [y, z]
      description: >
        The axis that points away from the ground.
    extent:
      type: number
      exclusiveMinimum: 0
      description: >
        Radius in world units of the smallest sphere centred at the origin
        that contains all addressable content.
  required: [units, upAxis]
```

Rationale for each field:

- `units` MUST be declared. Without it, metre scale is an assumption that is never stated.
- `upAxis` MUST be declared. Without it, a Z-up world renders upside down in a Y-up host.
- `unitsPerMeter` SHOULD be declared. Without it, a compositor mounting a child world cannot compute the correct scale.
- `extent` SHOULD be declared. Without it, a renderer has no information to size the float32 normalization domain.

```yaml
# Tighten Node.localTransform
Node:
  type: object
  properties:
    localTransform:
      type: array
      items:
        type: number
      minItems: 16
      maxItems: 16
      description: >
        Column-major 4x4 affine transformation matrix from this node's
        local space to its parent's space.
```

Rationale: Node.localTransform SHOULD be defined as a 16-element column-major 4x4 matrix. An array of numbers with no length or order convention is ambiguous.

### Shared spatial anchors

```yaml
# New resource on the World endpoint (extension track)
SpatialAnchor:
  type: object
  properties:
    id:
      type: string
    pose:
      $ref: '#/components/schemas/GeoPose'
    confidence:
      type: number
      minimum: 0
      maximum: 1
    provider:
      type: string
      description: >
        Identifier of the anchoring system (e.g. "webxr-anchors",
        "arcore", "arkit").
```

Rationale: WoW SHOULD define a spatial-anchor vocabulary or adopt the emerging WebXR Anchors Module by reference. This is a greenfield problem the group can own. This is an optional extension; a world that does not support AR anchoring omits it.

### Portable inventory

```yaml
# Addition to the UserManifest schema or a new PortableInventory schema
InventoryItem:
  type: object
  properties:
    id:
      type: string
    assetURI:
      type: string
      format: uri
    licence:
      type: string
      description: >
        SPDX licence identifier or URI to a licence document.
    provenance:
      type: string
      format: uri
      description: >
        URI of the world where this item was acquired.
  required: [id, assetURI]
```

Rationale: WoW SHOULD define a portable-inventory schema (item reference, licence, provenance) or adopt by reference from the Universal Manifest's equipped-items vocabulary. This is an optional extension; a minimal world that does not support item transfer omits it.

### User preferences

```yaml
# Addition to the UserManifest schema
Preferences:
  type: object
  properties:
    accessibility:
      type: object
      properties:
        textScale:
          type: number
          minimum: 0.5
          maximum: 4.0
        highContrast:
          type: boolean
        reduceMotion:
          type: boolean
    inputPreferences:
      type: object
      description: >
        Client-side input configuration. Schema intentionally open
        to avoid prescribing controller layouts.
    renderQuality:
      type: string
      enum: [low, medium, high]
```

Rationale: WoW SHOULD define a preference vocabulary on the user manifest. At minimum: accessibility settings (per WCAG), input preferences, and rendering quality. This is an optional extension.

### AI agent identity

```yaml
# Addition to the User schema (extension track)
User:
  type: object
  properties:
    agentType:
      type: string
      enum: [human, agent]
      default: human
    capabilities:
      type: array
      items:
        type: string
      description: >
        Declared capabilities of an agent user (e.g. "navigation",
        "translation", "content-generation").
```

Rationale: WoW SHOULD define an agent identity type on the User resource (human vs agent, capability declaration) or explicitly defer to a future extension track with a timeline. The whitepaper SHOULD NOT promise AI integration without specifying when and how.

### Governance document

No schema fragment applies. WoW SHOULD write a governance document covering:

- The initiative hub structure and its relationship to the MSF infrastructure.
- The business council charter and membership criteria.
- The maintainer council charter, scope, and relationship to the initiative hub.
- The licence and contributor licence agreement policy.
- The SDO liaison protocol for coordinating with external standards bodies.

Rationale: governance is the most discussed topic (599 transcript hits, more than any other individual topic) and none of it is codified. The licence discrepancy is a concrete blocker for any organization evaluating adoption.


## Adoption path

**What stays valid for a minimal world.** A server that implements the current eight schemas and sixteen operations remains conformant. Every addition proposed above is additive. The precision vocabulary (units, upAxis) is the only MUST-level change; a minimal world that operates at metre scale with Y-up can declare `units: metres, upAxis: y` and nothing else changes. Shared anchors, portable inventory, user preferences, and agent identity are optional extensions that a minimal world omits.

**What a client must do.** A client that receives a World response with `units` and `upAxis` MUST use them for coordinate composition. A client that receives a World response without `units` SHOULD treat the world as metre-scale Y-up and log a warning. A client that encounters a `SpatialAnchor`, `InventoryItem`, `Preferences`, or `agentType` field it does not support MUST ignore the field without error, per standard OpenAPI forward-compatibility.

**What a server must do.** A server MUST add `units` and `upAxis` to its World response. A server SHOULD add `extent` and `unitsPerMeter` when the world operates at non-unit scale. A server MAY add spatial anchors, inventory, preferences, and agent identity when the world supports those features. A server MUST resolve the licence discrepancy before the governance document is published.


## Open questions for the working group

1. Should `units` support an extensible enum beyond `metres` (e.g. astronomical units, feet), or should all non-metre worlds express scale through `unitsPerMeter` alone?

2. Should Node.localTransform be row-major or column-major? glTF uses column-major; USD uses row-major. The choice is arbitrary but must be stated.

3. Should the spatial-anchor vocabulary live on the World resource, on a new /wow/anchor endpoint, or be adopted entirely by reference from WebXR Anchors?

4. Should portable inventory ride on the user manifest (alongside the avatar) or on a separate /wow/inventory endpoint that a world can opt into?

5. Should the governance document live in the WoWAPI repository or in a separate governance repository? The answer determines who has commit access to it.

6. Should the working group adopt a formal versioning policy (SemVer on the OpenAPI version field) before or after the conformance RFC lands?


## Sources

- R1: Socket map (102 rows). Repository-relative: `.dev/ai/subtask-comms/2026-09-07-03-19-33Z-WO-SCRAP-20260907-019-R1-md.md`
- R2: Discussion evidence (12 topics). Repository-relative: `.dev/ai/subtask-comms/2026-09-07-03-19-33Z-WO-SCRAP-20260907-019-R2-md.md`
- R3: Claimed scope vs defined surface (35 rows). Repository-relative: `.dev/ai/subtask-comms/2026-09-07-03-19-33Z-WO-SCRAP-20260907-019-R3-md.md`
- COMPANION.md: Infrastructure WG architecture map companion text. Repository-relative: `repo/msf-wg-tool/infrastructure-wg/model/COMPANION.md`
- OpenSpatialWorld API.yaml, commit d39a1a0. Public: `https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml`
- OpenUserManifest API.yaml, commit d39a1a0. Public: `https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml`
- OpenSpatialWorld README.md, commit d39a1a0. Public: `https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md`
- Completion Map: 73 rows. Repository-relative: `.dev/ai/reports/2026-09-07-02-53-07Z-web-of-worlds-completion-map.json`
- Findings and Recommendations. Repository-relative: Desktop delivery at `FINDINGS-AND-RECOMMENDATIONS.md`
- OSL schema.yaml (not public; extension labels visible at lines 697-714, 784-810, 895-910)
- OSL DESIGN-NOTE-fabric-as-precision-domain.md (not public; cited sections at lines 35-72, 250-280)

## Change log

2026-09-07: first public draft, verified; 2026-09-07 steward edit: remarks on the reference implementation licence files removed.
