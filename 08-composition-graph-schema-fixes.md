# Composition Graph Schema Fixes

The OpenSpatialWorld API defines a spatial composition graph with recursive embedded nodes, a Spatial descriptor endpoint, and a node-creation endpoint. Three fixes are needed before a second implementation can interoperate. First, the standard SHOULD define both an embedded form and a reference (integer-id) form for Node.children and MUST declare which form is used in each response (verified: the canonical schema embeds child Node objects recursively, which prevents O(1) node addressing and requires rewriting an enclosing subtree on any update). Second, GET /wow/spatial/{spatialID} SHOULD support content negotiation between the Spatial descriptor and the root node (verified: the current schema returns only a descriptor, but an implementation that returns the root node directly saves a round-trip and serves renderable content immediately). Third, POST /wow/spatial/{spatialID}/node/{nodeId} MUST NOT silently discard data from a successful (2xx) response, and SHOULD define error codes for body-shape mismatches (verified: without this, a spec-conformant POST with embedded children can be answered 200 OK with the embedded subtree silently deleted). Beyond these three, the standard SHOULD define a named extension point on Node and SHOULD define a cross-world reference construct on Node for sub-world composition.

Status: Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).


## What the specification says today

All citations are from `specification/OpenSpatialWorld/API.yaml` at commit d39a1a0 unless stated otherwise. Line references use the blob URL prefix `https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml`.

**Node.children (lines 484-487).** The Node schema defines children as a recursive reference:

```yaml
children:
  type: array
  items:
    $ref: '#/components/schemas/Node'
```

This is the only form. There is no integer-reference alternative, no flag to select between them, and no constraint on nesting depth.

**GET /wow/spatial/{spatialID} (lines 133-153).** The operation `getSpatialById` returns a `$ref: "#/components/schemas/Spatial"`. The Spatial schema (lines 438-468) is a metadata descriptor: `{id: number, rootNodeID: number, geoPose: {...}}`. It points at a root node by id. It is not the root node.

**POST /wow/spatial/{spatialID}/node/{nodeId} (lines 158-188).** The operation `addNewNodes` accepts `requestBody: type: array, items: $ref Node` and returns `type: array, items: $ref Node`. The request body and response body both use the canonical Node schema with embedded children. No error codes for body-shape mismatches are defined beyond the generic `default: Unexpected error`.

**Node schema (lines 471-496).** The Node schema defines `id` (integer), `label` (string), `names` (array of string), `parent` (integer), `children` (array of Node), `localTransform` (array of number), `spatialAssetURI` (string), and `appearanceURI` (string). No `additionalProperties` constraint is set (open by default in OpenAPI 3.0.4), and no named extension point is defined.

**Content negotiation on GET /wow/spatial/{spatialID}.** The operation defines no query parameter and no request header for selecting between representations. (Search terms: `form`, `accept`, `content-negotiation`, `header`, `X-` in lines 133-153: zero hits.)

**Error codes on POST /wow/spatial/{spatialID}/node/{nodeId}.** The operation defines `200`, `default` only. (Search terms: `400`, `422`, `409`, `embedded`, `flat` in lines 158-188: zero hits.)

**Cross-world references on Node.** The Node schema (lines 471-496) contains no field for referencing a child world, a sub-world, or an external composition graph. The Spatial schema (lines 438-468) references nodes only within a single world by rootNodeID. (Search terms: `sub-world`, `cross-world`, `compose`, `transclusion`, `child_fabric`, `reference_world` across the full 505-line file: zero hits.)

**README path mismatch (errata).** The README (lines 28 and 31) uses `URL/wow/scene/` and `URL/wow/scene/node`. The machine-readable API.yaml (lines 133 and 156) implements `/wow/spatial/{spatialID}` and `/wow/spatial/{spatialID}/node/{nodeId}`. Both cannot be right.


## What fails without these fixes

**Flat-graph implementations cannot interoperate.** The canonical Node.children embeds child Node objects recursively. An implementation that stores a flat graph (each node addressed by integer id) must either reject the canonical form or silently convert it. Without a standard way to declare the serialization form, a client sending embedded children to a flat-graph server will get an unspecified response. Open Spatial Lab hit this: a spec-conformant POST body with embedded children was answered 200 OK with the embedded subtree silently deleted. The data loss was invisible to the caller (verified: OSL schema.yaml lines 316-397, declared divergence D7).

**Clients that need a renderable root hit an extra round-trip.** GET /wow/spatial/{spatialID} returns a Spatial descriptor `{id, rootNodeID, geoPose}`. A client that wants to render the world must then issue a second GET for the root node by its rootNodeID. Without content negotiation, there is no way to request the root node directly from this endpoint. Open Spatial Lab diverged from the standard by returning the root node by default and offering the canonical descriptor on a versioned opt-in (verified: OSL schema.yaml lines 195-287, declared divergence D3).

**Extensions collide with future canonical fields.** The Node schema is open by default (no `additionalProperties` constraint), and no named extension point exists. Two implementations that add ad-hoc properties at the top level of Node have no way to tell canonical fields from extensions. If a future revision of the standard adds a property that matches an extension name, the collision is silent. Open Spatial Lab defined a `webofworlds_extension` bag as a labeled, open extension point to separate the namespaces (verified: OSL schema.yaml lines 984-1033).

**Worlds are flat islands.** The Node schema contains no construct for referencing a child world. A composition-graph standard that cannot express "this node includes that world's subtree" limits every world to a single, self-contained tree. Open Spatial Lab implemented cross-world composition by letting a node reference a child .msf fabric through the `spatial_fabric_subtree` extension; the viewer fetches, verifies, and composes it in the parent's frame. The transclusion/traversal distinction is kept: a subtree reference on Node (`<img src>`) is not the same as a portal destination on Portal (`<a href>`) (verified: OSL WEB-OF-WORLDS.md lines 59-68, 55/55 contract checks as documented by Open Spatial Lab; documented, not re-run in this pass).


## What Open Spatial Lab built and learned

Open Spatial Lab implemented six changes to the composition-graph layer against the canonical spec at commit d39a1a0. Each change is a labeled, non-canonical divergence or extension, declared in the OSL schema and surfaced by the OSL conformance harness. Every `/wow` response carries `standards_conformance: false`.

**Flat-graph node representation (divergence D1).** OSL serves integer id-references instead of embedded Node objects in the children field. The shape is named `OSLFlatNode` and is declared as a non-canonical divergence. The flat graph makes each node addressable at `/wow/spatial/{spatialID}/node/{nodeId}` in O(1) and updatable without rewriting an enclosing subtree. The canonical Node schema is transcribed in the OSL contract but is not served by default.

**Root-node default on GET /wow/spatial/{spatialID} (divergence D3).** OSL returns the root node by default instead of the Spatial descriptor. The canonical descriptor is served on a versioned opt-in: the query parameter `?form=descriptor` or the request header `X-OSL-WoW-Spatial-Form: descriptor`. The response header `X-OSL-WoW-Spatial-Form` echoes which representation was served (enum: `node`, `descriptor`). Existing clients that expected a node are never broken.

**POST body-shape handling (divergence D7).** OSL rejects a canonical embedded-children POST body with HTTP 422 (`embedded_children_not_supported`) by default. On explicit opt-in via the header `X-OSL-WoW-Node-Form: canonical`, OSL flattens the embedded subtree into id-references, stores every node, and returns all stored nodes. Response headers `X-OSL-WoW-Nodes-Stored` and `X-OSL-WoW-Children-Flattened` (described in the schema comments but not formally declared in the response headers block) report exactly what was stored. No request path ends in a silent drop.

**WebOfWorldsExtension bag (extension).** OSL added a labeled, open (`additionalProperties: true`) extension bag named `webofworlds_extension` on OSLFlatNode. It carries `schema`, `geopose_mapping`, `target_world_id`, `target_location_id`, `target_base_url`, `spatial_fabric_address`, `spatial_fabric_subtree`, `portal_id`, `trigger`, `zones`, `traversal`, and `traversal_mode`. The bag separates canonical fields from OSL-specific extensions.

**Content-negotiation query parameter and headers (extension).** `?form=` (enum: `descriptor`, `spatial`, `node`) on GET /wow/spatial/{spatialID} selects the representation. The `X-OSL-WoW-Spatial-Form` header is both a request header (alternative to the query parameter, takes precedence when both are sent) and a required response header (echoes the served form). `X-OSL-WoW-Node-Form` on POST declares the body serialization.

**Cross-world composition (extension).** A node may reference a child .msf fabric through `node.webofworlds_extension.spatial_fabric_subtree`. The subtree contract is strict (`additionalProperties: false`) and requires `fabricURI`, `unitsPerMeter`, `upAxis`, and `placement`, and optionally carries `mediaType`, `parallax`, `requireVerified`, `maxDepth`, and `epochTicks`. The viewer fetches, verifies independently, and composes the child fabric in the parent node's frame. This is transclusion, not traversal: the subtree is drawn in the parent, not navigated to.

**Claim boundary.** All changes are labeled as non-canonical. Open Spatial Lab makes no claim of standards conformance. The evidence counts documented by Open Spatial Lab (48/48 crossing-continuity checks, 55/55 signed-subtree contract checks) are as documented; they were not re-run in this pass.


## Proposed normative text

### 1. Node.children: embedded and reference forms

The standard SHOULD define both an embedded form and a reference form for Node.children.

```yaml
Node:
  type: object
  properties:
    children:
      type: array
      items:
        oneOf:
          - $ref: '#/components/schemas/Node'   # embedded form
          - type: integer                        # reference form (node id)
```

A server MUST declare which form it uses in each response, either by a response header or by a schema discriminator.

Rationale: a flat graph (integer references) enables O(1) node addressing and independent node updates; an embedded graph (recursive Node objects) preserves the current canonical form. Both are valid engineering choices. The standard should allow both and require declaration.

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

A server that supports both forms SHOULD echo the served form in a response header.

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

Rationale: without a named extension point, implementations add ad-hoc top-level properties on Node. A named bag separates canonical from extended fields and prevents collisions with future canonical additions.

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
          description: URI of the child spatial document.
        unitsPerMeter:
          type: number
          description: Scale factor of the child relative to the host.
        upAxis:
          type: string
          enum: [y, z]
          description: Up-axis convention of the child document.
```

A client that fetches a referenced subtree SHOULD verify it independently before composing it in the parent frame.

Rationale: without cross-world references, every world is a flat island. The composition rules (fetch, verify independently, inherit parent frame) follow the pattern implemented by Open Spatial Lab.


## Adoption path

**Minimal world (what stays valid).** A server that serves the current canonical form (embedded Node.children, Spatial descriptor on GET, no extensions) remains valid. None of the proposed changes break existing payloads.

**What a client must do.** A client MUST inspect the response to determine whether children are embedded objects or integer references. If the server provides a `form` response header, the client SHOULD use it. A client SHOULD handle a 422 response to POST by inspecting the error body for the supported forms.

**What a server must do.** A server MUST declare which Node.children form it uses. A server MUST NOT return 200 OK while silently discarding embedded children. A server that supports the cross-world reference construct MUST serve the referenced subtree's metadata (units, up-axis) in the reference object.


## Open questions for the working group

1. Should the embedded form or the reference form be the default for Node.children? The current canonical form is embedded. Changing the default is a breaking change.

2. Should content negotiation on GET /wow/spatial/{spatialID} use a query parameter, an Accept header, or both?

3. Should the extension point on Node be named `extensions` (generic) or carry a namespace prefix (e.g. `webofworlds_extension`)? A generic name is simpler but risks collisions across standards.

4. Should the cross-world reference construct be a first-class property on Node, or should it live inside the extension bag? The former is cleaner for conformance; the latter is less invasive.

5. The README path (`/wow/scene/`) contradicts the API.yaml path (`/wow/spatial/{spatialID}`). Which path is canonical? This is errata and needs adjudication.


## Sources

- OpenSpatialWorld API specification: `specification/OpenSpatialWorld/API.yaml` at commit d39a1a0, WebOfWorlds/WoWAPI, `https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml`
- OpenSpatialWorld README: `specification/OpenSpatialWorld/README.md` at commit d39a1a0, `https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md`
- Open Spatial Lab schema: `repo/open-spatial-lab/wow-spec/schema.yaml` (lines cited in text)
- Open Spatial Lab spec-coverage declarations: `repo/open-spatial-lab/web/wow-spec-coverage.mjs` (lines 317-320)
- Open Spatial Lab working-group document: `repo/open-spatial-lab/docs/working-groups/WEB-OF-WORLDS.md`
- Open Spatial Lab working-group dossier: `repo/open-spatial-lab/docs/WORKING-GROUP-DOSSIER.md`


## Change log

- 2026-09-07: first public draft, verified.
