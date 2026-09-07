# Provenance and Signed Subtrees

The Web of Worlds specification defines a live composition graph of worlds, nodes, portals, users, and views, but it contains no vocabulary for proving that any of those objects are genuine. Every response is unsigned JSON. There is no proof boundary, no signature profile, no verification requirement, and no extension point where an implementation could carry trust metadata alongside canonical fields. A conformant client today cannot distinguish a verified world from one that has been tampered with in transit, and a conformant server cannot declare which of its own claims it has actually earned.

Open Spatial Lab addressed this by adding four mechanisms, each labeled as a non-canonical extension: a ProofBoundary schema on every response, a fail-closed trust boundary on every navigation event, a requireVerified flag on transcluded spatial subtrees, and an allOf response composite that carries the proof boundary and an extension point alongside canonical fields. These mechanisms have been running in production code (verified in code). The 55/55 signed-subtree contract check count is documented by Open Spatial Lab but was not re-run in this verification pass (reported; confidence medium until re-run).

The working group should adopt a proof-boundary declaration on every response as a SHOULD-level requirement, define a verification model for transcluded spatial content with fail-closed as the default, require re-verification on every root-changing navigation event, and provide an additive extension mechanism on response schemas.

**Status:** Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).


## What the specification says today

The specification is silent on provenance, trust, verification, signatures, integrity, and proof. The following searches against `specification/OpenSpatialWorld/API.yaml` (505 lines, commit d39a1a0) each returned zero results: `provenance`, `trust`, `signature`, `proof`, `verify`, `integrity`, `signed`, `ETag`, `MUST`, `SHOULD`, `SHALL` (verified). The README (`specification/OpenSpatialWorld/README.md`) returns zero results for the same terms (verified).

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

The World schema (lines 268 onward) contains `content`, `geoPose`, `presence`, `technology`, `users`, `views`, and `portals`. No proof boundary. No extension point. No required properties.

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

The View schema contains `id` (number) and `geoPose`. No proof boundary. No extension point.

The Portal and User response schemas follow the same pattern: each returns its bare schema with no trust metadata and no extension point.

The Node schema (lines 471 onward) defines `spatialAssetURI` as a bare `type: string` with no `format`, no `mediaType` constraint, and no prose. A `.msf` URL in that field is not prohibited, but there is no concept of a signed or verified subtree, no verification requirement, and no trust policy.


## What fails without it

**A client cannot tell verified content from tampered content.** Every GET response is bare JSON with no signature and no declaration of what was actually checked. If a proxy, a CDN, or a man-in-the-middle alters a World response, the client has no way to detect the change. A world that honestly lacks a capability (native TeleportXR transport, for example) looks identical to one that silently claims to have it. This matters most at portal crossings: a destination world could claim any capability to attract users, and the source world has no machine-readable signal to verify.

**Navigation can silently serve stale or tampered content.** When a user presses back, follows a deep link, or crosses a portal, the client fetches a new response. Without a verification step at each navigation event, content that has changed upstream (or been tampered with) is rendered without question. A history entry that points to a world whose content has been replaced since the user last visited it is silently accepted. Prefetched content that fails a later check has no mechanism to be refused.

**Transcluded spatial subtrees have no trust chain.** The Node schema allows `spatialAssetURI` to point to any URL, including a signed spatial document. But the specification defines no verification model for that content. A node could transclude an arbitrary spatial document with no signature check, no chain of trust, and no refusal on failure. In a composition graph where worlds include content from other origins, this is a content-injection vector: a child subtree from a compromised or malicious source would be composed and drawn without any check.

**Response schemas have no extension point.** An implementation that wants to carry additional metadata (a proof boundary, session state, extension data) alongside canonical fields has no defined mechanism. The canonical schemas are flat objects with no `allOf` composition and no reserved extension namespace. An implementation must either break the schema contract or invent an ad hoc convention with no interoperability guarantee.


## What Open Spatial Lab built and learned

### ProofBoundary schema (CM-050)

Open Spatial Lab defined a ProofBoundary schema with four required boolean flags: `application_level_handoff`, `native_teleportxr_teleport`, `first_party_teleportxr_browser_rendering`, and `standards_conformance`. The schema is strict (`additionalProperties: false`). Every flag is required, and `standards_conformance` is set to `false` on every response in the current version (verified in code; `schema.yaml` lines 670-695).

The design principle is that each flag declares a specific capability the server either has or does not have, and the server must state the truth. A flag set to `false` is not a deficiency report; it is an honesty declaration. The `standards_conformance: false` value is carried on every response precisely because the implementation has not earned that claim and will not assert it prematurely.

This is a labeled OSL extension (`x-osl-extension: true`). It is not granted by the specification.

### Fail-closed trust boundary on every navigation event (CM-051)

Open Spatial Lab implemented fail-closed verification on every navigation event: click-to-enter, proximity commit, typed address, back/forward/up, and prefetch. A history entry is an address, not a scene; going back re-runs the full verification pipeline. Prefetched-but-unverified content is never promoted to the active scene. Verification failure produces a visible refusal page with the history intact (verified in code; `NAVIGATION-ARCHITECTURE.md` lines 188-205).

Fail-open root loading was a bug (fixed by work order WO-061) and was never reintroduced. The `?verify=structured` parameter is a labeled development escape hatch, never a default, and never reachable from a link.

This is a client-side behavior with no corresponding specification requirement.

### Response composites with proof boundary and extension point (CM-052, CM-054)

Open Spatial Lab wraps every canonical response in an `allOf` composite that adds a required `proof_boundary` and an optional `webofworlds_extension`. Four composites exist:

- **OSLWorldResponse** (`schema.yaml` lines 1042-1062): `allOf[World, {proof_boundary (required), webofworlds_extension, id, location, session}]`.
- **OSLViewResponse** (`schema.yaml` lines 1086-1097): `allOf[View, {proof_boundary (required), webofworlds_extension}]`.
- **OSLUserResponse** (`schema.yaml` lines 1063-1085): `allOf[User, {proof_boundary (required), webofworlds_extension, open_user_manifest}]`.
- **OSLPortalResponse** (`schema.yaml` lines 1098-1115): `allOf[Portal, {label, proof_boundary (required), webofworlds_extension, destination}]`.

In each case, the canonical schema is preserved as the first element of the `allOf` array, and the extension members are additive. The `proof_boundary` is required on every response. This pattern keeps the canonical fields intact while providing a structured place for trust metadata and implementation-specific extensions.

These are labeled OSL extensions. They are not granted by the specification.

### Verification of transcluded spatial subtrees (CM-053)

Open Spatial Lab defined the SpatialFabricSubtree schema (`schema.yaml` lines 716-850) for nodes that transclude a signed `.msf` spatial fabric. The schema carries a `requireVerified` boolean (default `true`). When `true` or absent, the fabric must verify using RS256 with an x5c certificate chain to a shipped test anchor, or it is refused and not drawn. A visible error and a labeled placeholder are shown on failure. The `false` value is a labeled development escape hatch only and must never be set in a published world.

The verification claim is bounded: "verified" means a valid signature chaining to a shipped test anchor. It is not an operating-system-trust or public-PKI claim (stated in the schema description).

The SpatialFabricSubtree also requires `unitsPerMeter`, `upAxis`, and `placement`, each with no default and a refusal on absence or unrecognized values. This is a labeled divergence from the specification (divergence D8), proposed upstream.

55 signed-subtree contract checks are documented by Open Spatial Lab (reported; not re-run in this verification pass; confidence medium until re-run).


## Proposed normative text

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
        Whether this response was produced by an implementation that has
        passed the conformance test corpus for this endpoint.
```

Every GET response under `/wow/` SHOULD include a `proof_boundary` object declaring the implementation's verified capabilities. Rationale: a machine-readable honesty declaration lets a receiving world or client distinguish earned claims from unearned ones without out-of-band knowledge.

Implementations MAY add additional boolean flags to the proof boundary beyond `standards_conformance` to declare specific capabilities. Rationale: the four-flag model Open Spatial Lab uses (application-level handoff, native TeleportXR teleport, first-party TeleportXR rendering, standards conformance) proved useful, but the minimal required set for the standard is `standards_conformance` alone.

### Verification on navigation

A conformant client MUST verify the cryptographic envelope (when present) on every root-changing navigation event, including back, forward, deep link, and prefetch promotion. Rationale: a history entry is an address, not a cached scene; content that changed upstream must be re-verified.

A prefetched-but-unverified root MUST NOT be promoted to the active scene. Rationale: prefetch is an optimization, not an exemption from verification.

Verification failure MUST produce a visible refusal with the navigation history intact, not a silent fallback to unverified content. Rationale: silent fallback to unverified content is the exact failure mode this requirement exists to prevent.

### Verification of transcluded content

```yaml
Node:
  properties:
    requireVerified:
      type: boolean
      default: true
      description: >
        When true or absent, a transcluded spatial document referenced
        by spatialAssetURI MUST be cryptographically verified before
        rendering. Unverified content MUST be refused, not silently drawn.
```

The standard SHOULD define a verification model for transcluded spatial content with fail-closed as the default. Rationale: a spatial world that draws untrusted content from another origin without verification is a content-injection vector.

Implementations MAY set `requireVerified: false` as a development escape hatch. Such an implementation MUST NOT claim its subtree is verified. Rationale: the escape hatch exists for local development; it must not be reachable in production.

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

Every response schema (World, View, User, Portal) SHOULD support additive extension via `allOf` composition, preserving canonical fields in the first position. Rationale: without a defined extension point, implementations that need to carry trust metadata must break the schema contract or invent incompatible ad hoc conventions.

An extension namespace (`webofworlds_extension` or equivalent) MAY be defined as an optional container for implementation-specific members. Rationale: a single reserved key prevents collision between independent extensions.


## Adoption path

**A minimal world that does not use signed subtrees or extensions** needs no changes. The proof boundary is SHOULD-level, and a server that omits it is non-conformant in that respect but still produces valid World, View, User, and Portal responses. A client that receives a response with no `proof_boundary` treats it as unknown provenance.

**A client** must: (1) parse the `proof_boundary` object when present and surface its flags (at minimum `standards_conformance`); (2) verify the cryptographic envelope on every root-changing navigation event when the response carries one, refusing on failure; (3) refuse to render transcluded content that fails verification when `requireVerified` is `true` or absent.

**A server** must: (1) include a `proof_boundary` on every GET response under `/wow/` with at minimum `standards_conformance` set to the honest value; (2) use `allOf` composition to carry the proof boundary alongside canonical schema fields; (3) set `requireVerified` honestly on any node that transcludes a spatial document.


## Open questions for the working group

1. **Which proof-boundary flags should the standard require?** Open Spatial Lab uses four (application-level handoff, native TeleportXR teleport, first-party TeleportXR rendering, standards conformance). The minimal proposal above requires only `standards_conformance`. The working group should decide whether additional flags belong in the base standard or in an extension profile.

2. **What signature profile should the standard recommend?** Open Spatial Lab uses RS256 with an x5c certificate chain to a shipped test anchor for spatial subtrees, and Ed25519 for user identity manifests. The standard needs to name at least one algorithm and trust anchor model. The test-anchor model used by Open Spatial Lab is intentionally limited and would not be appropriate for a production trust infrastructure.

3. **Should the proof boundary be MUST or SHOULD?** A MUST requirement forces every implementation to carry honesty flags from day one. A SHOULD requirement lets minimal implementations omit them. The choice turns on how quickly the working group wants machine-readable trust to become baseline.

4. **How should the extension namespace be governed?** A single `webofworlds_extension` object is simple but risks becoming a dumping ground. The working group could define a registry of named extension profiles, or define rules for vendor-prefixed extension keys.

5. **What is the relationship between proof boundary and the proposed conformance test corpus (document 09)?** The `standards_conformance` flag is meaningful only if the group defines what conformance means and publishes a test corpus. These two efforts should be sequenced together.


## Sources

- `specification/OpenSpatialWorld/API.yaml` at commit d39a1a0, WebOfWorlds/WoWAPI main. [Preview](https://redocly.github.io/redoc/?url=https://raw.githubusercontent.com/WebOfWorlds/WoWAPI/refs/heads/main/specification/OpenSpatialWorld/API.yaml). Lines cited: 32-44, 88-107, 268 onward, 471 onward.
- `specification/OpenSpatialWorld/README.md` at commit d39a1a0.
- `repo/open-spatial-lab/wow-spec/schema.yaml`: ProofBoundary (lines 670-695), SpatialFabricSubtree (lines 716-850), OSLWorldResponse (lines 1042-1062), OSLUserResponse (lines 1063-1085), OSLViewResponse (lines 1086-1097), OSLPortalResponse (lines 1098-1115).
- `repo/open-spatial-lab/docs/NAVIGATION-ARCHITECTURE.md`: trust boundary table and re-verification semantics (lines 188-205).
- Completion Map rows: CM-050, CM-051, CM-052, CM-053, CM-054.
- Findings row: R-006.


## Change log

- 2026-09-07: first public draft, verified.
