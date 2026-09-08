# Conformance Vocabulary and Errata

WoWAPI 0.0.1 at d39a1a0 has testable type and endpoint constraints, but its six OpenSpatialWorld resource schemas accept empty objects. Structural validation alone therefore cannot establish a useful world or interoperable behavior. This chapter proposes a behavioral conformance profile and seed corpus, alongside concrete path-parameter, spelling and documentation corrections. BCP 14 wording supports clear requirements; it is not what first gives OpenAPI constraints meaning.

The cited defects can be checked against the pinned sources. Proposed required fields, path migration and longitude conflict handling remain group decisions. Evidence labels distinguish source checks, executed shape examples, local historical receipts and untested behavioral proposals.

**Status:** Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).

---

## What the specification says today

### RFC 2119 keywords

A search for the uppercase keywords MUST, SHOULD, SHALL, MAY, REQUIRED, RECOMMENDED, and OPTIONAL across all three API YAML files returns zero hits (verified: `grep -cE` at commit d39a1a0). The specification contains no RFC 2119 normative-strength keywords.

### Required schema properties

The word "required" appears 10 times in API.yaml. Nine are `required: true` on path parameters (lines 57, 76, 98, 120, 144, 167, 198, 217, 251). One is `required: true` on a `requestBody` (line 226). Zero are schema-level `required` property declarations (verified: `grep -B2 'required: true'` at API.yaml, commit d39a1a0).

Consequence: an empty JSON object `{}` validates against all six main schemas (World, User, View, Portal, Spatial, Node) because none of them declares any property as required.

### Graph paths across the published sources

The README lists the graph endpoints as:

- `URL/wow/scene/` (README line 28)
- `URL/wow/scene/node` (README line 31)

The API.yaml defines the machine-readable endpoints as:

- `/wow/spatial/{spatialID}` (API.yaml line 133)
- `/wow/spatial/{spatialID}/node/{nodeId}` (API.yaml line 156)

The whitepaper's printed-page-29 worked example also uses `/wow/scene/node/35643`. The [simpleWorlds API schema at 13d2cbe](https://github.com/WebOfWorlds/simpleWorlds/blob/13d2cbe/packages/wow-spec/src/schema.yaml#L132) uses `/wow/scene/node/{nodeId}`. Thus the scene family appears in the README, paper and reference schema, while OpenSpatialWorld/API.yaml uses spatial and is the only one with a graph id in the node path. These sources need one agreed route and identity mapping. The paper example also names `name` and `assetURI`; the API names `label`, `names` and `spatialAssetURI`. These field-name differences need an explicit example correction or mapping.

### Missing spatialID parameter on node operations

The path template `/wow/spatial/{spatialID}/node/{nodeId}` (API.yaml line 156) contains two template variables: `spatialID` and `nodeId`. The four operations under this path (POST lines 158-188, GET lines 189-207, PUT lines 208-241, DELETE lines 242-260) each declare only `nodeId` as a parameter. None declares `spatialID`. OpenAPI 3.0.4 requires every template variable in a path to be declared as a parameter; an undeclared variable is an invalid specification (verified: OpenAPI Specification 3.0.4, Section 3.5, "Path Templating").

### DELETE node parameter misspelling

The DELETE operation at lines 242-260 declares its path parameter as `nodeid` (lowercase d, API.yaml line 248). The path template spells it `nodeId` (camelCase). Because OpenAPI matches parameter names to template variables by exact string, `nodeid` does not match `{nodeId}`, leaving both variables undeclared on this operation.

### GeoPose.position.lan

Every GeoPose occurrence in API.yaml spells the longitude field as `lan` (lines 294, 368, 395, 424, 456). The field appears alongside `lat` (latitude) and `h` (height). No field named `lon` or `longitude` exists anywhere in the specification (verified: `grep -c 'lon:' API.yaml` returns 0). The misspelling is carried consistently across all five schemas that embed GeoPose: World, User, View, Portal, and Spatial.

### Core Requirement descriptions

The README's Core Requirements table (lines 9-15) contains three description defects:

- **Preview world** (line 13): the description reads "experence world without" and stops mid-clause. The [Linked Spatial Experiences: The Web of Worlds, April 2, 2025](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/) post already supplies the intended distinction: no additional user is created, while user-based authorization is needed.
- **Persist world** (line 14): the Description cell is empty. Only the Feature column ("store or bookmark URL") is populated.
- **Share world** (line 15): the Description cell is empty. Only the Feature column ("send URL to second user") is populated.

### aspect.id syntax

The Join row (README line 11) lists `URL#join=aspect.id` as a feature. The whitepaper's page-28 example shows `#join=view.5845`, a dotted kind-and-id form. The README does not define its full grammar, encoding or kind registry. The Optional Feature section (README lines 24-25) gives one concrete example, `URL/wow/user/4182`, but does not state the grammar that connects the fragment value to a resource identifier. No failure semantics are defined for an aspect identifier that does not resolve.

---

## What fails without these fixes

**Useful behavior is underconstrained.** The retained September 7 probe confirmed that all six resource schemas accept `{}` and an extra property, while all six reject arrays and Node rejects a string id. Implementations can fail existing structural tests. What is missing is a sufficiently defined behavioral profile and corpus for graph operations, addressing, transition behavior and observable failures. The bounded validator tested the common schema subset, not the full OpenAPI document.

**Path-parameter defects obstruct strict tooling.** The missing spatialID declarations and DELETE nodeId mismatch violate OpenAPI path-template rules. The local contract records a tooling failure from that defect. This report does not claim to have run every named code generator; actual behavior depends on tooling and version.

**The path mapping is unspecified.** A client requesting only `/wow/scene/` can receive a 404 from a server exposing only `/wow/spatial/`. A documented alias or adapter can bridge the routes, so different path strings alone do not prove two complete implementations cannot exchange graph data. The group should select the authoritative mapping and migration behavior.

**The longitude field is misnamed across the standard.** Emitting only `lon` can leave a literal `lan` consumer without longitude. Keeping `lan` requires a mapping when exchanging OGC Basic YPR data. Both choices cost something; neither is right until the standard picks one and provides a transition path.

**Preview intent and its binding are separate.** Open Spatial Lab's no-presence interpretation agrees with the 2025 post's no-additional-user intent. The report must also retain the post's authorization clause. The remaining work is to complete the README and define observable presence, access and failure behavior.

---

## What Open Spatial Lab built and learned

Open Spatial Lab built a local interpretation of the pinned specification and recorded selected ambiguities, divergences and defects. The findings below are drawn from that implementation.

**Parameter repair (labeled divergence D5).** OSL's schema declares `spatialID` as a path parameter on all four node operations and spells `nodeId` with a capital I on DELETE, repairing the two OpenAPI defects. This is a labeled repair of an upstream defect, documented in the OSL contract (`OSL-WOW-CONTRACT.md`), not a silent improvement. The repair was necessary because OSL's validation tooling rejected the upstream spec as invalid OpenAPI (verified: OSL schema.yaml lines 189-194, comment block).

**Graph path decision.** OSL implemented the OpenSpatialWorld/API.yaml path `/wow/spatial/{spatialID}`. The README, whitepaper example and reference schema use the scene family. No simpleWorlds runtime behavior is established here, and OSL's choice does not adjudicate the publications. This decision is recorded in OSL's contract document (`OSL-WOW-CONTRACT.md` lines 49-52, verified).

**Extension policy.** Open Spatial Lab labels its extensions and keeps `standards_conformance: false`. OpenAPI already permits extra response properties; what remains unstated upstream is a shared namespace, semantics and reporting convention. Local declared divergences, such as alternate child representations, must not be disguised as merely additive constraints.

**Bounded conformance claims.** Open Spatial Lab does not assert general WoW conformance. Its reported 70 grammar checks exercise its own interpretations, and the September 7 fixture run passed 44 supplied cases under an intentionally limited evaluator. Neither is a complete behavioral conformance corpus. Existing upstream type constraints remain independently testable.

**Core Requirement interpretations.** OSL completed the Preview description as "without joining" (interpretation I5), interpreted Persist as "bookmark the URL," and interpreted Share as "send the URL." All three are labeled as interpretations, not as spec text. OSL implemented the five URL-fragment Core Requirements against documented interpretations (historical labels I1–I10) rather than guessing silently (verified: WORKING-GROUP-DOSSIER.md section B4.4).

**Aspect identifier resolution.** OSL interpreted `aspect.id` as a placeholder for an aspect's identifier (interpretation I1). The implementation accepts an optional `kind/` qualifier (e.g., `user/4182`, `node/portal-1`) for disambiguation and falls back to probing the known aspect kinds (user first, then node) when no qualifier is given. An unresolvable aspect is treated as a warning; the client proceeds with the intent alone. The local resolver supports user and node, not the whitepaper's typed View example. These are labeled local behaviors, not the proposed dotted profile (verified: wow-url.mjs lines 63-71 and 287-311).

**Longitude misspelling carried verbatim.** OSL's GeoPose schema uses `lan` to match the standard exactly (OSL schema.yaml line 494, comment at line 495: "sic: the standard spells longitude 'lan'"), with a drift guard that fails the build if a payload ever says `lon` (WORKING-GROUP-DOSSIER.md section B4.2). OSL preserved the literal field spelling, on the ground that correcting the spelling unilaterally would break interoperability with any other literal implementation (verified).

**Claim boundary.** Open Spatial Lab demonstrates a locally working interpretation with recorded extensions and repairs. It does not establish that the specification is unimplementable, that no existing constraint can fail, or that a second implementation makes the same choices.

---

## Proposed normative text

All additions are unadopted proposals; schema fragments target OpenAPI 3.0.4. A profile names its required data and behavior separately from optional features.

### 1. RFC 2119 keyword adoption

Add a BCP 14 preamble citing both RFC 2119 and RFC 8174, then use its uppercase requirement terms consistently in the chosen behavioral profile. This is editorial support for clear requirement strength; existing OpenAPI types and parameter rules already have defined semantics.

The seed corpus should include: useful world data; successful and failed node GET/PUT; preservation of accepted children; entry/service/asset resolution; unresolved portal targets; and profile-specific transition, replay and recovery behavior. Each case needs an expected observation and versioned input. Signing vectors, historical local assertions and review counts cannot substitute for those behavioral outcomes.

### 2. Required schema properties

The group should choose required data from an agreed minimal scenario. Do not require georeferencing on every virtual World merely to reject `{}`. A candidate graph-traversal profile can require identifiers and the root pointer while preserving their canonical numeric types:

```yaml
TraversalSpatial:
  allOf:
    - $ref: '#/components/schemas/Spatial'
    - type: object
      required: [id, rootNodeID]
TraversalNode:
  allOf:
    - $ref: '#/components/schemas/Node'
    - type: object
      required: [id]
```

These are profile constraints, not base changes. Canonical `{}` remains valid under the old schemas but fails these examples. Spatial `{"id":1,"rootNodeID":2}` and Node `{"id":2}` pass their respective shapes; a string Node id fails both old and new schemas. The profile still needs to test that rootNodeID actually resolves and that the graph behaves correctly. Required World/User/View/Portal properties remain decisions tied to the intended scenario; optional destination is not promoted to a base requirement here.

### 3. spatialID parameter declaration

The specification MUST declare `spatialID` as a path parameter on all four node operations (POST, GET, PUT, DELETE at `/wow/spatial/{spatialID}/node/{nodeId}`):

```yaml
# To be added to each of the four node operations' parameters lists
- name: spatialID
  in: path
  description: ID of the spatial composition graph
  required: true
  schema:
    type: integer
```

Rationale: OpenAPI 3.0.4 requires every template variable to be declared; the current spec is invalid without this declaration (verified).

### 4. DELETE parameter spelling correction

The DELETE node operation MUST spell its path parameter `nodeId` (camelCase), matching the path template `{nodeId}`:

```yaml
# Line 248: change 'nodeid' to 'nodeId'
- name: nodeId
  in: path
  description: node id to delete
  required: true
  schema:
    type: integer
```

Rationale: OpenAPI matches parameter names to template variables by exact string; `nodeid` does not match `{nodeId}` (verified).

### 5. Graph path adjudication

The working group should choose the graph path and decide whether a graph identifier belongs in it. The README, whitepaper example and reference schema use the scene family; OpenSpatialWorld/API.yaml uses spatial with `{spatialID}`. Update all four sources or publish explicit aliases/version mappings. Also reconcile the paper's `name`/`assetURI` example with the selected node property names. Open Spatial Lab implemented API.yaml; that implementation choice is evidence, not the group's decision.

Rationale: one authoritative mapping avoids ambiguous endpoint discovery. The group may retain aliases during migration; the path choice does not by itself establish full implementation compatibility.

### 6. GeoPose.position.lan correction

Proposed rule: align `lan` to `lon` under the selected OGC GeoPose 1.0 Basic YPR profile. During a declared transition window, accept legacy lan or new lon; if both are supplied, accept only equal numeric values and reject conflict. Emit lon in the new profile. The fixed WGS-84/ENU and ellipsoidal-height semantics remain those of Basic YPR; a different reference frame requires an explicit mapping/profile.

```yaml
# Proposed transition schema fragment
position:
  type: object
  properties:
    lat:
      type: number
    lon:
      type: number
      description: "Longitude. Replaces the previous field name 'lan'."
    lan:
      type: number
      deprecated: true
      description: "Deprecated. Use 'lon'. Accepted during transition."
    h:
      type: number
```

The fragment permits both names but cannot compare two property values using this OpenAPI 3.0.4 subset; conflict rejection is a separately tested semantic rule. Requiring lat/lon/h for an adopted GeoPose profile is distinct from requiring every World to be georeferenced. The group must set the migration window and client/version behavior.

### 7. Core Requirement descriptions

The specification should complete Preview using the 2025 post and add Description cells for Persist world and Share world. Proposed text:

- **Preview world:** "Experience the world without creating an additional user; apply user-based authorization to protected access."
- **Persist world:** "Store or bookmark the world URL for later return."
- **Share world:** "Send the world URL to a second user to invite them."

Rationale: the published preview intent should be stated in the README, while persistence and sharing need a clear restoration/address contract.

### 8. Extension policy (optional)

The specification SHOULD define an extension policy. Proposed text (inferred from Open Spatial Lab's implementation):

- Extensions MUST be additive: an extension MUST NOT remove, rename, or redefine a canonical field.
- Extensions MUST be labeled: each extension field SHOULD carry a tag or namespace prefix that distinguishes it from canonical fields.
- Extensions SHOULD be surfaced: a conformance-reporting mechanism SHOULD list which extensions are active in a response.

This is an optional addition. The specification can function without it, but without a declared policy, implementations extend silently and conformance claims become unreliable (inferred).

### 9. Aspect identifier syntax (optional)

Use the single candidate grammar in [chapter 06](06-discovery-and-addressing.md#proposed-normative-text): `kind.id`, with proposed kinds `user`, `view` and `node`, following the published `view.5845` example. Split at the first literal dot and decode the non-empty identifier once; the selected endpoint constrains that identifier. A node target also needs a graph binding when the route carries a graph id.

An unsupported kind, malformed identifier or unresolved target produces an explicit target outcome while preserving the entry intent; it must not silently become a bare join. Unrecognized non-WoW fragments remain available to the page. OSL's slash-qualified and unqualified forms can remain in a declared legacy profile, with its probing order and warning fallback labeled as local behavior.

Rationale: the publication supplies a useful typed example; an agreed grammar and failure rule make it interoperable without treating the local parser as the source of authority.

---

## Adoption path

**Existing payloads.** Existing type constraints remain in force. Optional additions can preserve old payloads, but new required fields, exact transform lengths or identifier-type changes narrow compatibility. The example traversal schemas are opt-in profiles; requiring World.geoPose is not proposed as a universal fix.

**Clients.** Apply the versioned path mapping and GeoPose migration rules, including the conflict case. A parser accepting both names is not enough to establish a geodetic/local-frame conversion. Respect unresolved-aspect behavior selected by the group rather than assuming Open Spatial Lab's fallback is universal.

**Servers.** Repair path-parameter declarations while preserving canonical parameter types, publish the authoritative route mapping and advertise supported profiles. Reject malformed shapes where required, and test meaningful success/failure behavior beyond schema validation.

## Open questions for the working group

1. **Graph path:** which route and graph-id model should the group choose across the README, whitepaper, reference schema and OpenSpatialWorld/API.yaml? Which aliases, field mappings or version transitions should remain?

2. **Longitude field name:** should the corrected field be `lon` or `longitude`? Is a deprecation period for `lan` acceptable, or must backward compatibility be preserved indefinitely?

3. **Required properties:** which properties beyond `id` should be required on each schema? The proposed graph profile is one bounded example; required data for other resources must follow the agreed scenario, including non-georeferenced worlds.

4. **Conformance test corpus:** does the group intend to publish a set of test vectors (valid and invalid responses) as part of the specification? Open Spatial Lab offers its conformance harness and fixtures as seed material.

5. **Extension policy:** should the specification define a formal extension mechanism, or is a convention (such as `x-` prefixed fields) sufficient?

6. **Aspect identifier syntax:** should the dotted `kind.id` example become the common grammar, with `user`, `view` and `node` kinds? Which legacy slash or unqualified forms should remain, and how will unsupported targets be reported?

---

## Sources

- [Linked Spatial Experiences: The Web of Worlds, April 2, 2025](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/): published units, preview, authorization and aspect intent.

- [OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialWorld/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialAsset/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf); relevant printed pages are identified in this chapter or [chapter 10](10-role-and-blind-spots.md).
- [OpenAPI 3.0.4 Path Templating](https://spec.openapis.org/oas/v3.0.4.html#path-templating) and [Schema Object](https://spec.openapis.org/oas/v3.0.4.html#schema-object).
- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119), [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) and [OGC GeoPose 1.0](https://docs.ogc.org/is/21-056r11/21-056r11.html).
- Open Spatial Lab local source snapshot and retained evidence, checked September 7, 2026: schema.yaml, OSL-WOW-CONTRACT.md, WORKING-GROUP-DOSSIER.md and wow-url.mjs. The September 7 probe tested the canonical shared schema subset; the 44 supplied fixture cases used a limited evaluator, not full OpenAPI or live HTTP validation. Public reproduction of these exact local bytes is not established.
- [Appendix A](A-completion-map.md) and [Appendix B](B-findings-register.md) preserve the historical surface/finding identifiers.

## Change log

- 2026-09-07: corrected source scope, proposal compatibility and evidence boundaries; updated public citations.
