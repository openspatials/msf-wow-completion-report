# Composition Graph Schema Fixes

OpenSpatialWorld 0.0.1 defines a composition graph with embedded child nodes, a Spatial descriptor and direct per-node GET/PUT operations. Open Spatial Lab chose a flat reference form and a root-response shortcut. These are optional representation/efficiency proposals, not prerequisites for addressing or updating a node. The actionable issues are explicit negotiation, preservation of accepted graph data, path-parameter errata and a shared binding for the external references already described in the whitepaper.

Status: Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).


## What the specification says today

All citations are from `specification/OpenSpatialWorld/API.yaml` at commit d39a1a0 unless stated otherwise. The [pinned API source](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml) supplies the cited line numbers.

**Node.children (lines 484-487).** The Node schema defines children as a recursive reference:

```yaml
children:
  type: array
  items:
    $ref: '#/components/schemas/Node'
```

This is the only declared child-item form. `children` is optional, so the schema accepts an omitted or empty children member. `parent` already uses an integer reference. There is no defined child-id alternative, depth selector or complete shallow-read/update contract.

**GET /wow/spatial/{spatialID} (lines 133-153).** The operation `getSpatialById` returns a `$ref: "#/components/schemas/Spatial"`. The Spatial schema (lines 438-468) is a metadata descriptor: `{id: number, rootNodeID: number, geoPose: {...}}`. It points at a root node by id. It is not the root node.

**POST /wow/spatial/{spatialID}/node/{nodeId} (lines 158-188).** The operation `addNewNodes` accepts `requestBody: type: array, items: $ref Node` and returns `type: array, items: $ref Node`. The request body and response body both use the canonical Node schema with embedded children. No error codes for body-shape mismatches are defined beyond the generic `default: Unexpected error`.

**Individual-node GET and PUT (lines 189–241).** GET addresses a node by `nodeId` and describes a Node tree response; PUT accepts and returns Node. The recursive `children` property permits embedded subtrees, but its optionality does not specify whether omission means a leaf, a shallow read, preserved children or deletion during update. The API does not require every GET or PUT to contain a whole subtree, nor bind those omitted-child semantics.

**Node schema (lines 471-496).** The Node schema defines `id` (integer), `label` (string), `names` (array of string), `parent` (integer), `children` (array of Node), `localTransform` (array of number), `spatialAssetURI` (string), and `appearanceURI` (string). No `additionalProperties` constraint is set (open by default in OpenAPI 3.0.4), and no named extension point is defined.

**Content negotiation on GET /wow/spatial/{spatialID}.** The operation defines no query parameter and no request header for selecting between representations. (Search terms: `form`, `accept`, `content-negotiation`, `header`, `X-` in lines 133-153: zero hits.)

**Error codes on POST /wow/spatial/{spatialID}/node/{nodeId}.** The operation defines `200`, `default` only. (Search terms: `400`, `422`, `409`, `embedded`, `flat` in lines 158-188: zero hits.)

**Cross-world references on Node.** The pinned Node schema has no dedicated external-node reference binding beyond its asset/address fields. The March 31, 2026 whitepaper names Data Inline on printed page 21 and live internal/external node references on page 22, building on the Anchor/Inline lineage discussed on page 10. The missing work is an interoperable encoding and resolution/composition contract, not invention of the concept.

**Graph-path disagreement (errata).** The README (lines 28 and 31) uses the `scene` family. The whitepaper's printed-page-29 example uses `/wow/scene/node/35643`, and the [simpleWorlds API schema at 13d2cbe](https://github.com/WebOfWorlds/simpleWorlds/blob/13d2cbe/packages/wow-spec/src/schema.yaml#L132) uses `/wow/scene/node/{nodeId}`. OpenSpatialWorld/API.yaml uses `/wow/spatial/{spatialID}/node/{nodeId}` and alone among these sources adds a graph identifier to that path. The group should select the route and graph-identity mapping; machine readability alone does not settle the choice. The whitepaper example also uses `name` and `assetURI`, whereas the API names `label`, `names` and `spatialAssetURI`.


## What fails without these fixes

**OSL's adapter once lost embedded children.** A canonical embedded POST was reported as successful while a subtree was discarded. This is an implementation/adapter defect, not a necessary consequence of embedded serialization. A server can index nodes in a map, accept/store every embedded child and serialize a tree on request. The canonical API already provides per-node GET and PUT at `/wow/spatial/{spatialID}/node/{nodeId}`. Missing error vocabulary does not authorize success with data loss.

**Clients that need a renderable root hit an extra round-trip.** GET /wow/spatial/{spatialID} returns a Spatial descriptor `{id, rootNodeID, geoPose}`. A client that wants to render the world must then issue a second GET for the root node by its rootNodeID. Without content negotiation, there is no way to request the root node directly from this endpoint. Open Spatial Lab diverged from the standard by returning the root node by default and offering the canonical descriptor on a versioned opt-in (verified: OSL schema.yaml lines 195-287, declared divergence D3).

**Extensions collide with future canonical fields.** The Node schema is open by default (no `additionalProperties` constraint), and no named extension point exists. Two implementations that add ad-hoc properties at the top level of Node have no way to tell canonical fields from extensions. If a future revision of the standard adds a property that matches an extension name, the collision is silent. Open Spatial Lab defined a `webofworlds_extension` bag as a labeled, open extension point to separate the namespaces (verified: OSL schema.yaml lines 984-1033).

**External-node encoding needs agreement.** The whitepaper's external-node architecture is not fully bound in the pinned YAML. Open Spatial Lab supplies one signed `.msf` transclusion extension. An inert asset reference, external composition graph, executable signed fabric and portal destination are distinct cases; the extension is one candidate, not the sole composition answer.


## What Open Spatial Lab built and learned

Open Spatial Lab implemented six changes to the composition-graph layer against the canonical spec at commit d39a1a0. Each change is a labeled, non-canonical divergence or extension, declared in the OSL schema and surfaced by the OSL conformance harness. Every `/wow` response carries `standards_conformance: false`.

**Flat-graph node representation (divergence D1).** Open Spatial Lab serves integer child references in OSLFlatNode. Its internal map can support efficient lookup/update, but so can a server that emits embedded trees. Direct node endpoints are canonical already. Wire form does not determine storage complexity; the flat response is a declared divergence from the current embedded response.

**Root-node default on GET /wow/spatial/{spatialID} (divergence D3).** OSL returns the root node by default instead of the Spatial descriptor. The canonical descriptor is served on a versioned opt-in: the query parameter `?form=descriptor` or the request header `X-OSL-WoW-Spatial-Form: descriptor`. The response header `X-OSL-WoW-Spatial-Form` echoes which representation was served (enum: `node`, `descriptor`). This preserves that local node-response expectation; a canonical client expecting a Spatial descriptor still needs an explicit adapter or negotiated response.

**POST body-shape handling (divergence D7).** OSL rejects a canonical embedded-children POST body with HTTP 422 (`embedded_children_not_supported`) by default. On explicit opt-in via the header `X-OSL-WoW-Node-Form: canonical`, OSL flattens the embedded subtree into id-references, stores every node, and returns all stored nodes. Response headers `X-OSL-WoW-Nodes-Stored` and `X-OSL-WoW-Children-Flattened` (described in the schema comments but not formally declared in the response headers block) report exactly what was stored. The recorded repair preserves accepted children in the exercised paths; it is not a proof about every possible request.

**WebOfWorldsExtension bag (extension).** OSL added a labeled, open (`additionalProperties: true`) extension bag named `webofworlds_extension` on OSLFlatNode. It carries `schema`, `geopose_mapping`, `target_world_id`, `target_location_id`, `target_base_url`, `spatial_fabric_address`, `spatial_fabric_subtree`, `portal_id`, `trigger`, `zones`, `traversal`, and `traversal_mode`. The bag groups OSL extension fields separately; independent authors still need unique names or profile identifiers inside it.

**Content-negotiation query parameter and headers (extension).** `?form=` (enum: `descriptor`, `spatial`, `node`) on GET /wow/spatial/{spatialID} selects the representation. The `X-OSL-WoW-Spatial-Form` header is both a request header (alternative to the query parameter, takes precedence when both are sent) and a required response header (echoes the served form). `X-OSL-WoW-Node-Form` on POST declares the body serialization.

**Cross-world composition (extension).** A node may reference a child .msf fabric through `node.webofworlds_extension.spatial_fabric_subtree`. The subtree contract is strict (`additionalProperties: false`) and requires `fabricURI`, `unitsPerMeter`, `upAxis`, and `placement`, and optionally carries `mediaType`, `parallax`, `requireVerified`, `maxDepth`, and `epochTicks`. The viewer fetches, verifies independently, and composes the child fabric in the parent node's frame. This is transclusion, not traversal: the subtree is drawn in the parent, not navigated to.

**Claim boundary.** These are local divergences and extensions with `standards_conformance: false`. The retained crossing assertion receipt and July 11 mixed contract-check receipt validate their stated local scopes, not independent graph exchange. A second independently authored consumer of the same pinned graph remains a separate acceptance test.


## Proposed normative text

All additions below are unadopted candidate profiles targeting OpenAPI 3.0.4. The canonical default remains embedded children and a Spatial descriptor. A client must opt into an alternate representation; accepting old payloads under a new schema does not mean an old client can read new responses.

### 1. Node.children: embedded and reference forms

Evaluate an optional reference-form profile while preserving canonical embedded children. Keep the representations separate so a response cannot silently mix child objects and ids:

```yaml
ReferenceNode:
  type: object
  properties:
    id:
      type: integer
    label:
      type: string
    names:
      type: array
      items: {type: string}
    parent:
      type: integer
    children:
      type: array
      items: {type: integer}
    localTransform:
      type: array
      items: {type: number}
    spatialAssetURI:
      type: string
    appearanceURI:
      type: string
```

This separate proposal preserves the other canonical field types and replaces only `children` semantics. It is not an `allOf` override of canonical Node: applying both embedded-object and integer-child constraints would reject non-empty arrays. An empty array is structurally valid in either form, so negotiated metadata must identify the form.

The protocol should declare and echo the requested representation, define unsupported-form errors, and retain canonical responses by default. In the proposed tree profile, a server MUST reject a child reference that creates a cycle and define missing-reference, duplicate-parent and partial-update behavior before accepting a mutation. The integer-array shape cannot enforce those graph-wide rules; they need the stored graph and behavioral fixtures. Lookup/update complexity is an internal storage choice, not a reason that embedded form cannot work.

### 2. Content negotiation on GET /wow/spatial/{spatialID}

The standard SHOULD support a `form` query parameter or an Accept-based mechanism on GET /wow/spatial/{spatialID} to negotiate between the Spatial descriptor and the root node.

```yaml
parameters:
  - name: form
    in: query
    required: false
    schema:
      type: string
      enum: [descriptor, node]
```

A server that supports both forms SHOULD echo the served form under the agreed negotiation binding. An unsupported request must produce a documented outcome; request/header precedence and cache variation must be defined. The canonical descriptor remains the default in this proposal.

Rationale: the Spatial descriptor is a metadata pointer; the root node is renderable content. Different clients need different forms, and a server that returns a non-canonical default needs a way to offer the canonical form alongside it.

### 3. POST body-shape handling and error codes

A server MUST NOT silently discard data from a successful (2xx) response to POST /wow/spatial/{spatialID}/node/{nodeId}.

The standard SHOULD define error codes for body-shape mismatches:

```yaml
responses:
  '422':
    description: >-
      The request body shape does not match the server's expected
      node form. The response body names the mismatch and the
      supported forms.
```

A server that accepts both embedded and flat bodies SHOULD define a request header or parameter to declare the body form.

Rationale: silent data loss on a 200 OK is the worst failure mode in this system. A 422 with a named mismatch is recoverable; a silent drop is not.

### 4. Named extension point on Node (optional)

The standard SHOULD define a named extension property on Node:

```yaml
Node:
  type: object
  properties:
    extensions:
      type: object
      additionalProperties: true
```

Rationale: open objects already allow extension properties. A named bag reduces collisions with canonical members; names or profile identifiers inside the bag still need a collision rule.

### 5. Cross-world reference construct on Node (optional)

The standard SHOULD define a cross-world reference construct on Node so that a node can include another world's subtree in the composition graph.

```yaml
Node:
  type: object
  properties:
    subtreeReference:
      type: object
      properties:
        uri:
          type: string
          description: URI reference of the child spatial document, resolved against the declared base.
        unitsPerMeter:
          type: number
          minimum: 0
          exclusiveMinimum: true
          description: Child-local coordinate units per physical metre; model scale is separate.
        upAxis:
          type: string
          enum: [y, z]
          description: Up-axis convention of the child document.
        handedness:
          type: string
          enum: [right, left]
          description: Child frame handedness; resolve the full basis and origin from metadata or the selected profile.
```

A future external-node profile must specify identity/resolution, source and target frames, cycle policy and resource bounds. The proposed child-reference tree profile rejects cycles; more general external-link semantics require their own rule. For distance conversion, divide source units by sourceUnitsPerMeter and multiply by targetUnitsPerMeter. Verification is required where a signed-content profile says so; generic external references are not automatically signed or executable.

Rationale: this supplies one possible binding for the Data Inline and external-node architecture already published on whitepaper pages 21–22. Open Spatial Lab's independently verified fabric is a local candidate; generic external-node, asset and executable-document cases must remain distinct.


## Adoption path

**Existing implementations.** Canonical embedded children, numeric identifiers and Spatial-descriptor responses remain the default. A flat internal store can serve them through an adapter. Optional negotiated forms do not authorize unsolicited incompatible responses to old clients.

**Clients.** Request only forms they support and verify the server's selected form. Do not guess form from an empty children array. Handle explicit refusal of unsupported POST shapes; never treat missing accepted children as successful preservation.

**Servers.** Preserve all accepted graph data, validate before committing, and expose recoverable errors for unsupported operations. Define negotiation and conflict precedence before claiming alternate forms interoperate. Add external references only under an agreed resolution/frame/execution profile.

## Open questions for the working group

1. Should the embedded form or the reference form be the default for Node.children? The current canonical form is embedded. Changing the default is a breaking change.

2. Should content negotiation on GET /wow/spatial/{spatialID} use a query parameter, an Accept header, or both?

3. Should the extension point on Node be named `extensions` (generic) or carry a namespace prefix (e.g. `webofworlds_extension`)? A generic name is simpler but risks collisions across standards.

4. Should the cross-world reference construct be a first-class property on Node, or should it live inside the extension bag? The former is cleaner for conformance; the latter is less invasive.

5. Which route and graph-identity model should the group adopt? The README, whitepaper example and reference schema use the scene family; OpenSpatialWorld/API.yaml uses spatial with a graph id. Which aliases, field-name mappings and migration rules connect them?


## Sources

- [OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialWorld/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf); relevant printed pages are identified in this chapter or [chapter 10](10-role-and-blind-spots.md).
- [OpenAPI 3.0.4 Schema Object and composition](https://spec.openapis.org/oas/v3.0.4.html#schema-object).
- Open Spatial Lab local source snapshot and retained evidence, checked September 7, 2026: schema.yaml, wow-spec-coverage.mjs, WEB-OF-WORLDS.md and the retained local graph/contract receipts. Canonical node routes were read directly; local alternate representations do not prove independent graph exchange. Public reproduction of these exact local bytes is not established.
- [Appendix A](A-completion-map.md) and [Appendix B](B-findings-register.md) preserve the historical surface/finding identifiers.

## Change log

- 2026-09-07: corrected source scope, proposal compatibility and evidence boundaries; updated public citations.
