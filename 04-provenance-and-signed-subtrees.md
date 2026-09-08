# Provenance and Signed Subtrees

WoWAPI 0.0.1 defines resources and a composition graph, but its OpenSpatialWorld API does not define durable signed-object provenance or a signed-subtree execution profile. Its object schemas already permit extra properties. The whitepaper discusses existing web authentication/encryption, the asset README names HTTP authentication and single sign-on, and both asset and manifest APIs authorize resource access; neither should be erased by a broader claim that the architecture has no trust mechanisms.

Open Spatial Lab adds publisher-declared capability flags, labeled response extensions and signed-fabric verification against a configured test anchor. The fabric refusal path is useful local evidence. It must be distinguished from the portal path that logs failed manifest verification or arrival notification and can continue composition. Signature validity, capability declarations, proof receipts, identity assurance and execution permission are separate results.

The bounded ask is to open an optional signed-subtree extension track with sample payloads and refusal cases. It must name publisher trust, signed-byte scope, mutable parent transforms, execution permissions and resource limits. The following text is a proposal for that evaluation, not an adopted universal verification requirement.

**Status:** Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).


## What the specification says today

The pinned OpenSpatialWorld API and README have no application-signature profile for their resources. That scoped source finding does not imply that unsigned HTTP responses lack transport integrity: HTTPS authenticates/protects the transport under its trust model, while object signatures can preserve provenance through copying and storage. OpenSpatialAsset and OpenUserManifest both define `HEAD /` for authorization and ETag checks. A 200 response grants access; 403 denies it; 404 can conceal either existence or authorization, and the descriptions instruct clients not to follow redirects. The [asset README](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/README.md#L9-L10) names HTTP authentication and modern single sign-on. These are existing access provisions, not a signed-object provenance or visitor-assurance profile.

The `required` keyword appears ten times in the specification: nine on path parameters (e.g. `userId`, `viewId`, `portalId`, `spatialID`, `nodeId`) and once on a request body (PUT node, line 226). No schema property is marked required (verified).

**GET /wow/world** (lines 32-44) returns the World schema directly:

```yaml
/wow/world:
  get:
    ...
    responses:
      "200":
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/World"
```

[API.yaml lines 32-44](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L32-L44)

The World schema (lines 268 onward) contains `content`, `geoPose`, `presence`, `technology`, `users`, `views`, and `portals`. No named proof boundary or shared extension namespace; extra properties remain permitted. No required properties.

**GET /wow/view/{viewId}** (lines 88-107) returns the View schema:

```yaml
/wow/view/{viewId}:
  get:
    ...
    responses:
      "200":
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/View"
```

[API.yaml lines 88-107](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L88-L107)

The View schema contains `id` (number) and `geoPose`. No named proof boundary or shared extension namespace; extra properties remain permitted.

Portal and User also return their component schemas without named trust metadata. As OpenAPI 3.0.4 defaults `additionalProperties` to true, additional response fields are schema-legal; their shared names and semantics are not defined.

The Node schema (lines 471 onward) defines `spatialAssetURI` as a bare `type: string` with no `format`, no `mediaType` constraint, and no prose. A `.msf` URL in that field is not prohibited, but there is no concept of a signed or verified subtree, no verification requirement, and no trust policy.


## What remains unbound

**Durable object provenance.** A receiver needs to know which bytes were signed and under which publisher/trust policy if it must verify content after caching, mirroring or third-party distribution. Absence of an application signature alone does not prove a man-in-the-middle vulnerability over authenticated HTTPS.

**Claims versus evidence.** A capability flag says what the publisher declares. Signing the flag attributes it to the signer but does not prove that a test passed or a capability works. A conformance claim needs a named profile, implementation/version and relevant receipt or attestation.

**Content execution policy.** An authentic signed subtree can still consume excessive resources or request disallowed capabilities. Publisher verification, authorization to execute, sandboxing, recursion/cost limits and failure handling are distinct controls.

**Extension interoperability.** Existing open objects allow extra fields. A shared namespace and profile convention can reduce collisions and tell consumers how to interpret those fields. It is not needed merely to make extensions schema-legal, and one generic bag still needs namespacing among its members.

## What Open Spatial Lab built and learned

### ProofBoundary schema (CM-050)

Open Spatial Lab defined a ProofBoundary schema with four required boolean flags: `application_level_handoff`, `native_teleportxr_teleport`, `first_party_teleportxr_browser_rendering`, and `standards_conformance`. The schema is strict (`additionalProperties: false`). Every flag is required, and `standards_conformance` is set to `false` on every response in the current version (verified in code; `schema.yaml` lines 670-695).

Each flag is a publisher declaration of a named capability. Its value can be useful for discovery, but the boolean is not an independently checked receipt. Even a signed flag must be evaluated against evidence before being treated as an earned assurance claim.

This is a labeled OSL extension (`x-osl-extension: true`). Its semantics are not defined by the pinned specification.

### Signed-fabric refusal and the separate portal path (CM-051)

The documented signed-fabric navigation pipeline verifies required fabric bytes before promotion and shows a refusal when that verification fails. The stated trust model is the configured test anchor, not operating-system trust or production public-key infrastructure. Navigation caching must preserve the binding between verified bytes and the active payload.

This is not a claim that every navigation check fails closed. In the actual portal controller, arrival-notification failure is logged while composition continues. Manifest-verification failure is surfaced before target promotion, but does not itself stop it. Fabric verification, user assertions and notification delivery are different paths with different policies.

### Response composites with proof boundary and extension point (CM-052, CM-054)

Open Spatial Lab wraps every canonical response in an `allOf` composite that adds a required `proof_boundary` and an optional `webofworlds_extension`. Four composites exist:

- **OSLWorldResponse** (`schema.yaml` lines 1042-1062): `allOf[World, {proof_boundary (required), webofworlds_extension, id, location, session}]`.
- **OSLViewResponse** (`schema.yaml` lines 1086-1097): `allOf[View, {proof_boundary (required), webofworlds_extension}]`.
- **OSLUserResponse** (`schema.yaml` lines 1063-1085): `allOf[User, {proof_boundary (required), webofworlds_extension, open_user_manifest}]`.
- **OSLPortalResponse** (`schema.yaml` lines 1098-1115): `allOf[Portal, {label, proof_boundary (required), webofworlds_extension, destination}]`.

`allOf` applies every member's constraints together. Member order gives no override precedence. A compatible extension can add a required proof-boundary property for the OSL response profile, but cannot change a canonical numeric identifier to a string by putting the string constraint later. The retained probe rejected both numeric and string ids under those conflicting constraints in either order. Omitting an id still passed because neither member required it.

These are labeled OSL response profiles; their semantics are not standardized. Any incompatible constraints remain a schema defect, not an override.

### Verification of transcluded spatial subtrees (CM-053)

Open Spatial Lab defined the SpatialFabricSubtree schema (`schema.yaml` lines 716-850) for nodes that transclude a signed `.msf` spatial fabric. The schema carries a `requireVerified` boolean (default `true`). When `true` or absent, the fabric must verify using RS256 with an x5c certificate chain to a shipped test anchor, or it is refused and not drawn. A visible error and a labeled placeholder are shown on failure. The `false` value is a labeled development escape hatch only and must never be set in a published world.

The verification claim is bounded: "verified" means a valid signature chaining to a shipped test anchor. It is not an operating-system-trust or public-PKI claim (stated in the schema description).

The SpatialFabricSubtree also requires `unitsPerMeter`, `upAxis`, and `placement`, each with no default and a refusal on absence or unrecognized values. This is a labeled divergence from the specification (divergence D8), proposed upstream.

The historical July 11 total was 29 prior checks plus 26 new checks: 16 schema/contract/vocabulary assertions and 10 discovery/transform/placeholder assertions. The scene-builder tests used no DOM, fetch, WebAssembly or renderer. These are useful contract checks, not 55 cryptographic or rendered-subtree trials, and they were not rerun for these edits.


## Proposed normative text

These are unadopted optional-profile proposals targeting OpenAPI 3.0.4. They do not alter the base API by themselves.

### Proof-boundary declaration

```yaml
ProofBoundary:
  type: object
  required:
    - standards_conformance
  properties:
    standards_conformance:
      type: boolean
      description: >
        Publisher-declared conformance claim. This boolean is not a
        test receipt or independent attestation; evaluate it against
        a named profile, implementation version and supporting evidence.
```

An optional response profile may include `proof_boundary` to state publisher-declared capabilities. Receivers must distinguish those declarations from verified receipts or attestations. The field's historical name is retained for compatibility; it does not prove a boundary was enforced.

The set and meaning of flags remain a profile decision. The OSL four-flag shape is one local example. Requiring any flag in the base response would be a compatibility change; the schema above only constrains `standards_conformance` within a present ProofBoundary object.

### Verification on navigation and transclusion (candidate signed-content profile)

For a profile requiring signed content, a client MUST validate the selected signing profile and publisher trust before activating that content. Cached validation may be reused only while it remains bound to the exact bytes and applicable trust/freshness policy. Failure of required verification MUST produce a visible refusal for that content, including during prefetch promotion and nested loading.

```yaml
SignedSubtreePolicy:
  type: object
  properties:
    requireVerified:
      type: boolean
      default: true
      description: >
        Within the proposed signed-content profile, absent or true
        requires verification under its declared publisher-trust policy.
        This does not confer execution permission or visitor assurance.
```

Unsigned leaf assets and generic canonical nodes are not silently brought under this signed-content rule. A development `false` value cannot satisfy the signed-content profile; a deployment must not expose it as an untrusted input that bypasses required verification. The group must separately define allowed execution, resource limits, signed-byte scope and treatment of mutable parent transforms.

### Response extension mechanism

```yaml
WorldResponse:
  allOf:
    - $ref: "#/components/schemas/World"
    - type: object
      properties:
        proof_boundary:
          $ref: "#/components/schemas/ProofBoundary"
```

A response profile may express compatible additional constraints with `allOf`. Every member applies regardless of order. The canonical object schemas already allow extra fields; the proposal supplies shared naming and semantics. Validate extensions against canonical fields instead of treating an `allOf` member as an override.

An optional extension namespace such as `webofworlds_extension` can separate extension data from canonical members. Independent extension authors still need unique names or profile identifiers within that object; a single reserved container alone does not prevent all collisions.


## Adoption path

**Existing responses.** World, View, User and Portal responses remain valid without proof-boundary metadata. Additional fields are already allowed. Optional profile adoption gives fields shared semantics; it is not permission to add fields for the first time.

**Clients.** Treat capability flags as declarations. Verify signed content when the negotiated profile requires it, report its exact publisher-trust boundary, and make separate admission and execution decisions. Do not infer real-world identity or content safety from a valid signature.

**Servers.** Advertise supported profiles and the bytes/claims they cover. Preserve canonical field types in response composites. Attach evidence references when making a tested-conformance claim; a boolean alone is not that evidence.

## Open questions for the working group

1. **Which proof-boundary flags should the standard require?** Open Spatial Lab uses four (application-level handoff, native TeleportXR teleport, first-party TeleportXR rendering, standards conformance). The example above requires `standards_conformance` only inside a present ProofBoundary object; it does not require that object on every response. The working group should decide whether additional flags belong in the base standard or in an extension profile.

2. **What signature profile should the standard recommend?** Open Spatial Lab uses RS256 with an x5c certificate chain to a shipped test anchor for spatial subtrees, and Ed25519 for user identity manifests. A signed-content profile needs to name its algorithms and publisher-trust model. The test-anchor model used by Open Spatial Lab is intentionally limited and would not be appropriate for a production trust infrastructure.

3. **Should the proof boundary be MUST or SHOULD?** Should capability declarations stay in an optional profile, and which evidence should accompany an assurance claim? A required base property would need versioning and a migration rule; flags alone do not supply trust.

4. **How should the extension namespace be governed?** A single `webofworlds_extension` object is simple but risks becoming a dumping ground. The working group could define a registry of named extension profiles, or define rules for vendor-prefixed extension keys.

5. **What is the relationship between proof boundary and the proposed conformance test corpus (document 09)?** The `standards_conformance` flag is meaningful only if the group defines what conformance means and publishes a test corpus. These two efforts should be sequenced together.


## Sources

- [OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialWorld/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenUserManifest/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialAsset/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml#L19-L34), resource authorization and ETag checks.
- [OpenSpatialAsset/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/README.md), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf); relevant printed pages are identified in this chapter or [chapter 10](10-role-and-blind-spots.md).
- [OpenAPI 3.0.4 Schema Object](https://spec.openapis.org/oas/v3.0.4.html#schema-object) and [W3C Verifiable Credentials 2.0 trust model](https://www.w3.org/TR/vc-data-model-2.0/#trust-model).
- Open Spatial Lab local source snapshot and retained evidence, checked September 7, 2026: schema.yaml and NAVIGATION-ARCHITECTURE.md, plus the signed-fabric and portal-controller paths. The July 11 contract total combines 29 prior checks and 26 additions; the scene-builder checks used no DOM, fetch, WebAssembly or renderer. Test-anchor fabric refusal is separate from the continuing portal-notification/manifest path. Public reproduction of these exact local bytes is not established.
- [Appendix A](A-completion-map.md) and [Appendix B](B-findings-register.md) preserve the historical surface/finding identifiers.

## Change log

- 2026-09-07: corrected source scope, proposal compatibility and evidence boundaries; updated public citations.
