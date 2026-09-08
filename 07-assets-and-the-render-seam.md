# Assets and the Render Seam

OpenSpatialAsset 0.0.1 lists 21 model media types for content negotiation, while OpenSpatialWorld gives nodes a spatialAssetURI. The asset README also discusses HTTP authentication, modern single sign-on, ETag change detection, optional volume/GeoPose/format metadata and a Model Fragment URI opportunity. These provisions should be retained. The remaining proposals concern an agreed minimum asset profile, URI-reference resolution and an optional signed executable-subtree contract; they do not mandate one renderer or claim the architecture lacks external nodes.

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

**The READMEs and whitepaper.** OpenSpatialWorld lists scene/resource entry paths. OpenSpatialAsset describes negotiation, access and metadata conventions, and the March 31, 2026 whitepaper describes internal/external node references on printed page 22. These are architectural and resource-level provisions; a signed executable-subtree profile still needs a specific binding for trust, frames, execution and limits.

The asset GET response also provides a `Content-Disposition` filename-extension hint when `Content-Type` is absent ([API lines 39–48](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml#L39-L48)). Its `HEAD /` authorization and ETag checks are separate from payload signatures. The listed media types and negotiation mechanism are useful existing capabilities. A shared asset profile would specify which representation participants must support, how to report no common representation, and which content classes need a different profile.


## What fails without the fix

**No guaranteed common representation.** Two participants with disjoint supported formats can fail negotiation even though each implements a listed format. This motivates a minimum representation for an agreed content/use-case profile, not a demand to convert every spatial document into GLB.

**No shared transclusion-frame binding.** A source and host need unit and basis mappings for composition. Open Spatial Lab's missing-field failures exposed those needs in its Z-up fabric/Y-up host. The existing open node schema permits metadata, but does not assign shared transclusion semantics or require the local backend-selection field.

**Independent canvases do not share depth.** Two stacked WebGL canvases do not automatically produce shared depth ordering. Open Spatial Lab offers an in-room Three.js path with common depth and approximate shading, and a Filament backdrop path without shared occlusion. Those are its two backends, not an exhaustive law of GPU architecture. Other engines can choose different compositing arrangements; an interoperability profile should state observable occlusion requirements where needed.

**Resource limits need an execution profile.** Recursive or cyclic references can exhaust resources. A signed-subtree profile must define cycle handling and client-enforced limits, including depth, bytes, time and capabilities as appropriate. Publisher-supplied limits cannot raise the client's own bounds, and a signature does not make execution safe.

**Engine behavior needs a profile boundary.** The pinned OpenSpatialWorld API does not bind physics, audio spatialization or input-device mapping. Searched terms with zero hits in that file: `physics`, `audio`, `input`. (`gravity` appears once at line 314 as a string property under `World.presence`, alongside `avatar` and `navigation`; it is metadata, not a physics simulation surface.) The [2025 post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/) already names experience consistency in units and physics. Missing API terms do not erase that intent or establish an exclusion; the group needs to select which observable behaviors its profiles bind or delegate.


## What Open Spatial Lab built and learned

The historical eight-row asset inventory records six local implementation rows. Those labels describe that selected inventory, not asset conformance or complete rendering proof. Every extension is marked `x-osl-extension: true` and `x-osl-divergence: D8` in the schema. No conformance claim is made; `standards_conformance` stays `false` on every response.

**The SpatialFabricSubtree contract** (schema.yaml, lines 716-852, labeled divergence D8). OSL adopts `spatialAssetURI` as the address and adds a strict typed contract at `node.webofworlds_extension.spatial_fabric_subtree`. The local contract has four required fields, covering address, scale, axis mapping and backend selection:

- `fabricURI` (string): the signed `.msf` spatial document to transclude. Resolved and verified fail-closed; an unverified, unfetchable, or tampered fabric is refused at every nesting level.
- `unitsPerMeter` (number, must be positive): world units per fabric metre. No default. The compositor folds this into the scale composition rather than post-multiplying, so the engine's own light-intensity rule applies correctly. For the intentional tabletop model, k = 0.5 / 149597870700 host units per fabric metre. That model reduction must be explicit.
- `upAxis` (enum: z, y): declared, never sniffed from the content, never defaulted. The host applies a fixed basis change (a pure isometry) for Z-up content; Y-up content passes through unchanged. Without it, a Z-up fabric in a Y-up host renders on its side.
- `placement` (enum: in-room, backdrop): selects the rendering backend. `in-room` uses a single Three.js scene graph with one shared depth buffer (depth-correct compositing, approximate shading). `backdrop` uses a Filament stacked layer (measured parity, no depth compositing). No default, because the two backends make different honesty claims about what they draw.

Five optional fields address narrower problems:

- `mediaType` (string): a provisional, unregistered label (`application/msf+jws`). Advisory only; the discriminator is the presence of the contract object, not the media type. No IANA registration is claimed.
- `requireVerified` (boolean, default true): fail-closed by default. When true (or absent), the fabric must verify its signature (RS256 + x5c chain to a shipped test anchor) or be refused and not drawn. False is a labeled development escape hatch only.
- `parallax` (number, 0 to 1, default 0): backdrop only. Controls the degree of camera-translation response for a layer with no shared depth buffer. The default (0, orientation-locked) is the honest behaviour for such a layer.
- `maxDepth` (integer, minimum 0): per-subtree recursion cap for nested child fabrics. Always clamped by the engine's absolute maximum depth; this dial can only lower the cap, never raise it.
- `epochTicks` (number): the epoch at which the fabric's map.wasm is run. Fabrics are time-dependent by design (orbits move), so the epoch must be stated rather than implied by wall clock. Without it, two clients viewing the same fabric at different times see different states.

**Evidence and its limits.** The July 11 local contract receipt totals 29 prior checks and 26 additions, including schema/vocabulary, discovery, transform handover and placeholders. Its scene builder used no DOM, fetch, WebAssembly or renderer. The historical total is not a set of cryptographic trials or rendered-fidelity tests; it was not rerun for these edits.

**Asset ingest.** OSL treats `spatialAssetURI` as a glTF/GLB URL by convention and loads real assets through the existing Three.js GLTFLoader stack (deferred conformance ledger, DCL-005 and DCL-008). This works because every asset in the demo world happens to be glTF. The convention is not stated in the spec and would break for any non-glTF asset.

**TeleportXR rendering.** OSL makes no claim about TeleportXR rendering. Every pixel on screen is drawn by OSL's own Three.js and Filament-web renderer. No TeleportXR renderer is embedded, wrapped, or called. This boundary is disclosed, not a failure (wow-spec-coverage.mjs, lines 291-303).

**Claim boundary.** The local failures support explicit frame/scale contracts and honest backend capability reporting. They do not prove that Open Spatial Lab's four required properties are a necessary or sufficient universal contract. In particular, its existing unitsPerMeter is host units per fabric metre including model reduction; the proposed world-level units ratio in chapter 01 has a separate meaning.


## Proposed normative text

These are unadopted optional-profile proposals. Schema fragments target OpenAPI 3.0.4. Requiring new properties within an opted-in profile does not make them base API requirements.

### Baseline asset format

Candidate decision: for an agreed profile of static mesh assets, require both producer and consumer support for `model/gltf-binary` when the asset can be faithfully represented by that profile. Define profile capabilities, unsupported features and negotiation failure before adoption. A client may offer other formats through its Accept header. If no supported representation exists, use a declared unavailable/unsupported outcome; do not pretend a lossy GLB conversion represents executable or otherwise incompatible content.

This proposal does not require a GLB form of every spatialAssetURI. Signed executable documents and other non-mesh content need their own semantics or explicit fallback.

### spatialAssetURI resolution rule

```yaml
spatialAssetURI:
  type: string
  description: >
    Proposed URI reference for a spatial asset, resolved against the
    declared asset base (or, when that profile specifies it, the
    containing Node response retrieval URI) using RFC 3986 section 5.
    Negotiate a supported representation; report unresolved or unsupported content.
```

A relative path or filename is not inherently invalid. The profile must define its base, allowed schemes and resolution/failure rules. A generic string schema cannot enforce all of that behavior; the URL examples in chapter 06 exercise resolution separately.

### Transclusion contract for signed spatial documents

```yaml
# Add to OpenSpatialWorld Node extension vocabulary
SpatialDocumentTransclusion:
  type: object
  required:
    - documentURI
    - unitsPerMeter
    - upAxis
  properties:
    documentURI:
      type: string
      description: >
        URI reference of the signed document, resolved against the declared base.
        A client MUST verify the document's signature before
        rendering. An unverified document MUST NOT be drawn.
    unitsPerMeter:
      type: number
      minimum: 0
      exclusiveMinimum: true
      description: >
        Document-local coordinate units per physical metre. No default.
        Convert source distances by dividing by this value and
        multiplying by the host unitsPerMeter; model scale is separate.
    upAxis:
      type: string
      enum: [y, z]
      description: >
        The document's up-axis. REQUIRED. No default.
        Resolve the full source and host bases before composition;
        up-axis alone does not identify handedness or horizontal axes.
    handedness:
      type: string
      enum: [right, left]
      description: >
        Optional explicit frame handedness. When omitted, it must be
        obtained from the declared frame/profile, never guessed.
    placement:
      type: string
      enum: [in-room, backdrop]
      description: >
        Optional informative OSL backend hint. 'in-room' means the document
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
        The selected time profile must define tick units, epoch origin
        and behavior when time is not supplied; this sketch does not.
```

This candidate transclusion profile requires a document address and known local units/up-axis. A full frame mapping, including known handedness, is still needed; obtain it from explicit metadata or the declared frame/profile, and report unresolved placement when neither supplies it. `placement` and `parallax` describe optional local rendering hints, not a required architecture. A renderer may ignore those hints; this sketch does not establish shared-occlusion conformance. `unitsPerMeter` uses the proposal's local-units-per-metre definition: 100 source units at 100 units/metre becomes one host unit at one unit/metre. The existing OSL `k` additionally contains intentional model reduction. Signed-byte scope, mutable parent transforms, publisher trust, execution permissions, cycles and limits remain explicit profile decisions.

### Engine-internal scope statement

Proposed scope statement: WoW defines the agreed interchange behavior; rendering, physics, audio and input mechanisms remain implementation choices unless a named profile requires a specific observable result. Existing standards may be referenced for those domains after the group chooses the boundary. This report does not claim that a keyword search proves the architecture excludes them, or that one library provides every required binding.

## Adoption path

**Existing worlds.** Current media negotiation and node assets remain valid. The static-mesh baseline is a candidate optional profile with an explicit supported-content scope. Adding required data or restricting formats in the base API would need a separate compatibility decision.

**Clients.** Resolve URI references against the agreed base, negotiate a representation and report unsupported content. Under a signed-subtree profile, verify the required payload, apply the declared frame mapping and enforce local execution/resource policy. A valid signature alone is not permission to execute.

**Servers.** Advertise supported representations and profiles. Supply the metadata required by an adopted profile. Do not claim fidelity for content converted into an inadequate representation. Backend hints remain optional and do not choose another implementation's renderer.

## Open questions for the working group

1. **Signed-document media type.** Should the standard define a media type for signed spatial documents (such as `application/spatial+jws` or `model/spatial+jws`)? An IANA registration would need to be pursued. OSL uses the provisional, unregistered label `application/msf+jws`. The presence of the transclusion contract object is the discriminator today, not the media type.

2. **Rendering conformance.** Does Web of Worlds conformance require a specific rendering pipeline, or is any web renderer that consumes the composition graph valid? OSL draws every pixel with its own Three.js/Filament-web renderer and makes no claim about TeleportXR rendering.

3. **Baseline format beyond glTF.** Which asset use case should a baseline cover, and is GLB an adequate common representation for that case? What happens when content cannot be represented faithfully? Each additional required format raises the implementation floor.

4. **Physics, audio, and input: scope statement vs. extension track.** Should the scope exclusion be a final statement, or should the standard reserve an extension point for future physics/audio/input vocabularies? The current recommendation is a scope statement only.


## Sources

- [OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialAsset/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialAsset/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/README.md), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf); relevant printed pages are identified in this chapter or [chapter 10](10-role-and-blind-spots.md).
- [OpenAPI 3.0.4 Schema Object](https://spec.openapis.org/oas/v3.0.4.html#schema-object).
- Open Spatial Lab local source snapshot and retained evidence, checked September 7, 2026: schema.yaml, deferred-conformance-ledger.md and wow-spec-coverage.mjs. The July 11 mixed contract total is 29 prior checks plus 26 additions, not rendered-subtree or cryptographic trials. No fresh renderer run is claimed. Public reproduction of these exact local bytes is not established.
- [Appendix A](A-completion-map.md) and [Appendix B](B-findings-register.md) preserve the historical surface/finding identifiers.

## Change log

- 2026-09-07: corrected source scope, proposal compatibility and evidence boundaries; updated public citations.
