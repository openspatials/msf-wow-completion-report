# Assets and the Render Seam

The Web of Worlds specification defines content negotiation across 21 registered model media types (OpenSpatialAsset) and gives every scene-graph node a `spatialAssetURI` field (OpenSpatialWorld), but it specifies neither what that URI resolves to, how the resolved content is rendered, nor what happens when the content is a signed spatial document rather than an inert 3D asset. A conformant server can serve any of the 21 types; a conformant client has no way to know which ones to expect, no vocabulary for how to present them, and no concept of a document that carries its own executable scene graph. Three additions close these gaps (all verified in code): a baseline required asset format, a typed transclusion contract for signed spatial documents, and an explicit scope statement for engine-internal surfaces the standard should not reach. A fourth item, a registered media type for signed spatial documents, is an open question for the working group and for IANA.

**Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).**


## What the specification says today

**Node.spatialAssetURI** is declared as a bare string with no constraints:

```yaml
spatialAssetURI:
  type: string
```

(OpenSpatialWorld/API.yaml, line 492-493, [blob link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L492-L493).)

The field carries no `format` (such as `uri`), no `mediaType`, no `enum`, and no prose describing what it resolves to. Searched terms with zero hits in OpenSpatialWorld/API.yaml: `format`, `mediaType`, `media_type`, `render`, `placement`, `signed`, `physics`, `audio`.

**OpenSpatialAsset content negotiation** lists 21 model media types on the root GET `/` endpoint (OpenSpatialAsset/API.yaml, lines 52-161, [blob link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml#L52-L161)). The types range from `model/3mf` through `model/x3d+xml`. No type is marked required. No rendering semantics are defined for any of them. Searched terms with zero hits in OpenSpatialAsset/API.yaml: `signed`, `signature`, `jws`, `verified`, `render`, `placement`.

**The README** describes the URL entry points (`URL/wow/scene/`, `URL/wow/scene/node`) but offers no guidance on what a client does with the asset a node references (OpenSpatialWorld/README.md, [blob link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md)).

In short: the specification tells a client where to find an asset and offers 21 possible formats, but says nothing about which format to expect, how to render it, or what to do when the asset is not an inert mesh but a signed, executable spatial document.


## What fails without the fix

**No baseline format, no interop.** Two conformant servers can each serve a format the other's clients cannot consume. Server A serves `model/vnd.usdz+zip`; Server B serves `model/gltf-binary`. A client built for one will show nothing when it visits the other. The 21-type content-negotiation list becomes a menu nobody can order from, because no minimum order is defined.

**No transclusion contract, catastrophic defaults.** A node that transcludes an external spatial document through `spatialAssetURI` has no way to state the document's unit scale, up-axis, or rendering backend. Open Spatial Lab hit this directly: a Z-up fabric mounted into a Y-up host renders on its side; a fabric authored in astronomical units (1 AU per unit) mounted into a 1:1-metre room renders the solar system at Earth-to-Sun distance per metre. Both are valid readings of a bare string field with no metadata. The spec permits the address; it does not carry the contract the address needs.

**No depth-buffer truth.** Two stacked WebGL canvases cannot share a depth buffer. A client that transcludes external spatial content must choose between depth-correct compositing (one shared scene graph, approximate shading) and measured-parity rendering (a stacked layer, no occlusion). The choice is forced by GPU architecture, not preference, and the specification has no field to record it. Without one, a client either guesses or ignores the problem; both produce incorrect visuals.

**No recursion cap.** A spatial document that transcludes other spatial documents can create unbounded nesting chains. Without a per-transclusion recursion limit, a world that references itself (directly or through a chain) exhausts memory or enters an infinite render loop.

**No scope boundary for engine internals.** The specification is silent on physics, audio spatialization, and input device mapping. Searched terms with zero hits in OpenSpatialWorld/API.yaml: `physics`, `audio`, `input`. (`gravity` appears once at OpenSpatialWorld/API.yaml line 314 as a string property under `World.presence`, alongside `avatar` and `navigation`; it is a world metadata field, not a physics simulation surface.) An implementer cannot tell whether these surfaces are out of scope by design or simply not written yet.


## What Open Spatial Lab built and learned

Open Spatial Lab's implementation addresses six of the eight rows in this group through a labeled, non-canonical extension. Every extension is marked `x-osl-extension: true` and `x-osl-divergence: D8` in the schema. No conformance claim is made; `standards_conformance` stays `false` on every response.

**The SpatialFabricSubtree contract** (schema.yaml, lines 716-852, labeled divergence D8). OSL adopts `spatialAssetURI` as the address and adds a strict typed contract at `node.webofworlds_extension.spatial_fabric_subtree`. The contract has four required fields, each required because its absence caused a concrete failure:

- `fabricURI` (string): the signed `.msf` spatial document to transclude. Resolved and verified fail-closed; an unverified, unfetchable, or tampered fabric is refused at every nesting level.
- `unitsPerMeter` (number, must be positive): world units per fabric metre. No default. The compositor folds this into the scale composition rather than post-multiplying, so the engine's own light-intensity rule applies correctly. Without it, a tabletop orrery with 1 AU = 0.5 m in a 1:1-metre room is a valid reading and a catastrophic rendering.
- `upAxis` (enum: z, y): declared, never sniffed from the content, never defaulted. The host applies a fixed basis change (a pure isometry) for Z-up content; Y-up content passes through unchanged. Without it, a Z-up fabric in a Y-up host renders on its side.
- `placement` (enum: in-room, backdrop): selects the rendering backend. `in-room` uses a single Three.js scene graph with one shared depth buffer (depth-correct compositing, approximate shading). `backdrop` uses a Filament stacked layer (measured parity, no depth compositing). No default, because the two backends make different honesty claims about what they draw.

Five optional fields address narrower problems:

- `mediaType` (string): a provisional, unregistered label (`application/msf+jws`). Advisory only; the discriminator is the presence of the contract object, not the media type. No IANA registration is claimed.
- `requireVerified` (boolean, default true): fail-closed by default. When true (or absent), the fabric must verify its signature (RS256 + x5c chain to a shipped test anchor) or be refused and not drawn. False is a labeled development escape hatch only.
- `parallax` (number, 0 to 1, default 0): backdrop only. Controls the degree of camera-translation response for a layer with no shared depth buffer. The default (0, orientation-locked) is the honest behaviour for such a layer.
- `maxDepth` (integer, minimum 0): per-subtree recursion cap for nested child fabrics. Always clamped by the engine's absolute maximum depth; this dial can only lower the cap, never raise it.
- `epochTicks` (number): the epoch at which the fabric's map.wasm is run. Fabrics are time-dependent by design (orbits move), so the epoch must be stated rather than implied by wall clock. Without it, two clients viewing the same fabric at different times see different states.

**Evidence and its limits.** OSL's signed-subtree contract checks report 55/55 passing (reported: OSL's own working-group dossier; not re-run in this pass). This count covers the contract shape and verification pipeline, not visual output fidelity. The 55/55 figure was documented by Open Spatial Lab and has not been independently re-run for this report.

**Asset ingest.** OSL treats `spatialAssetURI` as a glTF/GLB URL by convention and loads real assets through the existing Three.js GLTFLoader stack (deferred conformance ledger, DCL-005 and DCL-008). This works because every asset in the demo world happens to be glTF. The convention is not stated in the spec and would break for any non-glTF asset.

**TeleportXR rendering.** OSL makes no claim about TeleportXR rendering. Every pixel on screen is drawn by OSL's own Three.js and Filament-web renderer. No TeleportXR renderer is embedded, wrapped, or called. This boundary is disclosed, not a failure (wow-spec-coverage.mjs, lines 291-303).

**Claim boundary.** OSL proves that a typed transclusion contract is necessary and that the four required fields (fabricURI, unitsPerMeter, upAxis, placement) each prevent a concrete rendering failure. OSL does not prove that these four fields are sufficient, that the field names are optimal, or that the extension vocabulary should be adopted unchanged.


## Proposed normative text

### Baseline asset format

```yaml
# Add to OpenSpatialAsset or to a conformance profile document
baseline_asset_format:
  description: >
    A conformant server MUST serve at least model/gltf-binary
    for every asset referenced by Node.spatialAssetURI.
    model/gltf+json, model/vnd.usdz+zip, and model/x3d+xml
    are OPTIONAL.
```

Rationale: glTF is the most widely supported 3D asset format on the web; requiring one baseline prevents content-negotiation deadlock.

### spatialAssetURI format constraint

```yaml
# Replace in OpenSpatialWorld Node schema
spatialAssetURI:
  type: string
  format: uri
  description: >
    A URI referencing a spatial asset. A client MUST support
    model/gltf-binary. A client SHOULD use HTTP content negotiation
    (Accept header) when fetching the URI.
```

Rationale: a bare `type: string` with no `format` lets any string through, including relative paths and bare filenames that cannot be resolved.

### Transclusion contract for signed spatial documents

```yaml
# Add to OpenSpatialWorld Node extension vocabulary
SpatialDocumentTransclusion:
  type: object
  required:
    - documentURI
    - unitsPerMeter
    - upAxis
    - placement
  properties:
    documentURI:
      type: string
      format: uri
      description: >
        The signed spatial document to transclude.
        A client MUST verify the document's signature before
        rendering. An unverified document MUST NOT be drawn.
    unitsPerMeter:
      type: number
      exclusiveMinimum: 0
      description: >
        World units per document metre. REQUIRED. No default.
        A client MUST use this value to compose the document's
        scale into the host scene.
    upAxis:
      type: string
      enum: [y, z]
      description: >
        The document's up-axis. REQUIRED. No default.
        A client MUST apply a basis change when the document's
        up-axis differs from the host scene's up-axis.
    placement:
      type: string
      enum: [in-room, backdrop]
      description: >
        REQUIRED. No default. 'in-room' means the document
        shares the host scene's depth buffer (depth-correct
        compositing). 'backdrop' means the document renders
        as a separate layer with no shared depth buffer.
    parallax:
      type: number
      minimum: 0
      maximum: 1
      default: 0
      description: >
        OPTIONAL. Backdrop only. Controls the degree of
        camera-translation response. 0 = orientation-locked
        (the honest default for a layer with no shared depth
        buffer). MAY be ignored for in-room placement.
    maxDepth:
      type: integer
      minimum: 0
      description: >
        OPTIONAL. Per-transclusion recursion cap for nested
        spatial documents. A client MUST clamp this value
        to its own engine-level maximum depth.
    epochTicks:
      type: number
      description: >
        OPTIONAL. The epoch at which the document's
        time-dependent content SHOULD be evaluated.
        When absent, the client MAY use wall-clock time.
```

Rationale: each required field prevents a concrete rendering failure documented in OSL's implementation.

### Engine-internal scope statement

The following sentence SHOULD appear in the specification's scope section:

> Physics simulation, audio spatialization, and input device mapping are out of scope. These surfaces are adopted by reference from engine-level standards and specifications (glTF extensions for physics, Web Audio API for spatialization, WebXR Input for device mapping). A conformant Web of Worlds implementation MUST NOT require a specific physics, audio, or input pipeline.

Rationale: silence is ambiguous; an explicit scope statement tells implementers these surfaces are intentionally absent.


## Adoption path

**For a minimal world (leaf assets only):** a server declares `model/gltf-binary` support for every asset node. A client fetches `spatialAssetURI` with an `Accept: model/gltf-binary` header. No transclusion contract is needed. This path requires only the baseline-format addition and the `format: uri` constraint on `spatialAssetURI`.

**What a client must do:** support `model/gltf-binary` at minimum. When a node carries a transclusion contract, verify the signed document before rendering, apply the declared `unitsPerMeter` and `upAxis`, and select the rendering backend from `placement`. Clamp `maxDepth` to the client's own engine limit.

**What a server must do:** serve at least `model/gltf-binary` for every `spatialAssetURI`. When a node transcludes a signed spatial document, populate the transclusion contract with all four required fields. Omitting any required field is a schema error.


## Open questions for the working group

1. **Signed-document media type.** Should the standard define a media type for signed spatial documents (such as `application/spatial+jws` or `model/spatial+jws`)? An IANA registration would need to be pursued. OSL uses the provisional, unregistered label `application/msf+jws`. The presence of the transclusion contract object is the discriminator today, not the media type.

2. **Rendering conformance.** Does Web of Worlds conformance require a specific rendering pipeline, or is any web renderer that consumes the composition graph valid? OSL draws every pixel with its own Three.js/Filament-web renderer and makes no claim about TeleportXR rendering.

3. **Baseline format beyond glTF.** Should the standard require support for a second format (USD, X3D) in addition to glTF, or is one baseline sufficient? Each additional required format raises the implementation floor.

4. **Physics, audio, and input: scope statement vs. extension track.** Should the scope exclusion be a final statement, or should the standard reserve an extension point for future physics/audio/input vocabularies? The current recommendation is a scope statement only.


## Sources

- OpenSpatialWorld/API.yaml at commit d39a1a0: [github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml)
- OpenSpatialAsset/API.yaml at commit d39a1a0: [github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml)
- OpenSpatialWorld/README.md at commit d39a1a0: [github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md)
- Open Spatial Lab schema (SpatialFabricSubtree): `repo/open-spatial-lab/wow-spec/schema.yaml`, lines 716-852
- Open Spatial Lab deferred conformance ledger (DCL-005, DCL-008): `repo/open-spatial-lab/.dev/ai/deferred-conformance-ledger.md`
- Open Spatial Lab spec coverage (first-party renderer boundary): `repo/open-spatial-lab/web/wow-spec-coverage.mjs`, lines 291-303
- Open Spatial Lab working-group dossier (55/55 contract checks): `repo/open-spatial-lab/docs/WORKING-GROUP-DOSSIER.md`

## Change log

- 2026-09-07: first public draft, verified.
