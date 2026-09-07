# Discovery and Addressing for Web of Worlds

The Web of Worlds specification needs a URL-based addressing scheme, a URL fragment grammar, a service discovery mechanism, and history semantics. Without these, two browsers cannot exchange a link to the same place in the same world, cannot agree on what `#join` means, cannot find a world server's endpoints without hardcoding them, and cannot implement back/forward navigation.

Open Spatial Lab built all four and tested them against the specification at commit d39a1a0. The address grammar (OSL-NAV-1) and the service discovery pointer are labeled extensions. The fragment grammar interpretation and the world-URL-as-path-prefix interpretation are compatible readings of the specification's own examples, carried to their consequences. The spatialID type change from integer to string is a declared divergence.

The confidence of each proposal below varies. The fragment grammar, the spatialID type, and the world-URL-as-path-prefix rule follow from what the specification already shows and can be adopted with minimal design work (verified: OSL implementation and mutation tests). The address grammar, history semantics, service discovery pointer, and well-known endpoint are new material with no specification text to anchor them (inferred: from the implementation and from web-platform conventions).

Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).


## What the specification says today

**Fragment verbs.** The README lists fragment verbs in the Core Requirements table (README.md lines 11-13):

> "URL, URL#join, URL#join=aspect.id" / "URL#follow, URL#follow=aspect.id" / "URL#preview, #preview=aspect.id"

Source: [README.md lines 11-13](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L11-L13).

The table lists three verbs (join, follow, preview) with an optional `=aspect.id` parameter. It gives no syntax definition: no case-sensitivity rule, no compound-fragment rule, no statement on what happens with an unrecognized fragment. The Preview description is truncated mid-clause ("experence world without" -- verbatim).

**The term "aspect."** The word "aspect" appears in the specification in three contexts: as the syntax element `aspect.id` in the fragment examples (README.md lines 11-13), in the API description (API.yaml lines 4-5, "read and modify aspects of the world" and "some aspects of spatial data management"), and in the Optional Feature section (README.md lines 21-24, "Read world status including live aspects" and "Read and write aspects of the world to enable links to sub-elements of the world (e.g user avatar)"). The latter two discuss the concept; the first uses it as a parameter name without definition. No definition of what an aspect is, which resources count as aspects, or what the `aspect.id` syntax resolves to.

Source: [README.md lines 11-13](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L11-L13); [API.yaml lines 4-5](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L4-L5); [README.md lines 21-24](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L21-L24).

**spatialID type.** API.yaml defines `spatialID` as `type: integer` (API.yaml lines 141-142, 144-146):

> `name: spatialID` / `in: path` / `required: true` / `schema:` / `type: integer`

Source: [API.yaml lines 141-142, 144-146](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L141-L146).

**World URL as path prefix.** The README shows resource composition by path concatenation: "URL/wow/world" (line 22), "URL/wow/user/4182" (line 25), "URL/wow/scene/" (line 28), "URL/wow/scene/node" (line 31). The specification never states that the world URL must be a path prefix rather than a query-based URL.

Source: [README.md lines 22-31](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L22-L31).

**GET /.** API.yaml defines `GET /` as returning `text/html` with summary "Provisions a web application to deliver the spatial experience on the given device" and operationId `getDefaultApp` (API.yaml lines 17-29).

Source: [API.yaml lines 17-29](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L17-L29).

**Persist and Share.** The README lists "Persist world: store or bookmark URL" and "Share world: send URL to second user" as Core Requirements (README.md lines 14-15). No further definition of what the URL contains or what "bookmark" means for spatial state.

Source: [README.md lines 14-15](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L14-L15).

**Silence.** The following terms return zero hits in both API.yaml and README.md: `address`, `history`, `well-known`, `discovery`, `services`, `fabric`, `fragment`, `case`, `bookmark` (zero in API.yaml; one in README.md at line 14, the bare word "bookmark" with no definition). The specification has no text on addressing schemes, navigation history, service discovery, or fragment grammar rules.


## What fails without it

**Two browsers cannot share a link to the same place.** A user in Browser A looks at a specific node inside a composed world at a specific camera angle. There is no specification-defined way to encode that state as a URL. The user cannot send a link to Browser B that opens the same world, at the same node, from the same camera angle. Bookmarks and deep links are impossible. The "Persist world" and "Share world" Core Requirements (README.md lines 14-15) are unimplementable without an address grammar that captures world identity, pose, and place.

**Two implementations parse the same fragment differently.** Browser A treats `#Join` as a join intent; Browser B rejects it because the verb is capitalized. Browser A allows `#join&follow`; Browser B treats it as unrecognized. The specification gives no rules to resolve these, so each implementation invents its own. The result is that a URL with a fragment that works in one browser fails or behaves differently in another.

**A WoW client hijacks non-WoW fragments.** A page with both a WoW world and ordinary HTML section anchors (`#about`, `#contact`) loses its section anchors if the WoW client treats every fragment as a WoW intent. The specification is silent on what to do with unrecognized fragments.

**An integer spatialID cannot address named graphs.** Open Spatial Lab addresses graphs by human-readable string identifiers (`world_id`, `location_id`, `default`, `world`). The specification's `type: integer` (API.yaml line 146) rejects these. An implementation that uses named identifiers cannot conform.

**A query-based world URL breaks resource composition.** If a world URL is `http://host/?role=player`, appending `/wow/world` by the specification's own concatenation pattern produces `http://host/?role=player/wow/world`, which is not a valid URL. The specification's resource composition model (README.md lines 22-31) works only when the world URL is a path prefix.

**A client cannot find endpoints without hardcoding them.** If each world server mounts its WoW resources at different paths, a client must know those paths in advance. Without a manifest-level service pointer or a well-known discovery endpoint, every client hardcodes every server's layout.

**Back/forward navigation has no definition.** A user who has crossed from World A to World B presses the browser's Back button. The specification says nothing about what a history entry contains, whether the camera pose is restored, whether re-verification is required, or what the distinction between "back" (temporal) and "up" (structural parent) is.


## What Open Spatial Lab built and learned

**Address grammar (OSL-NAV-1, labeled extension).** OSL designed a URL-based addressing scheme that separates the trust-bearing root identifier from presentation parameters and context. The grammar uses query parameters on the hosting page's URL:

- `?fabric=<url>` carries the root world's canonical URL. This is the only component that crosses a trust boundary.
- `&via=<url>,<url>` carries an optional descent breadcrumb (advisory, capped at four entries), giving the UP affordance a target even on a cold deep link.
- `&view=` / `&az=` / `&el=` / `&fit=` carry camera pose.
- `#/<node-path>` is a fragment identifier for a place inside the root, using slash-separated node names from the composed walk, sanitized (lowercase, spaces to hyphens), disambiguated by walk-order suffix (`~2`) when names collide.

A full deep link encodes the world identity, the descent context, the camera pose, and the place. Loading one lands the viewer at the same position looking the same way.

Source: `repo/open-spatial-lab/docs/NAVIGATION-ARCHITECTURE.md` lines 139-167.

Claim boundary: this is an OSL extension. The specification defines no addressing scheme. The grammar is tested within OSL's own navigation pipeline; no second implementation has adopted it.

**Fragment grammar interpretation (Interpretations I2, I3, I4).** OSL interpreted the specification's fragment examples as follows:

- Interpretation I3: a fragment is a single `verb[=value]` token, matched case-insensitively. Compound fragments (`#join&follow`) are rejected as unrecognized. OSL is strict in what it emits (always lowercase).
- Interpretation I2: an aspect is any addressable sub-element that the specification's own Optional Feature section exposes by URL: a user/avatar (`/wow/user/{id}`) and a scene node. An optional `kind/` qualifier is accepted. No third id-space is invented.
- Interpretation I4: an unrecognized fragment (`#some-anchor`) is not a WoW intent. OSL falls back to the bare-URL join reading and flags `recognized:false` so ordinary anchor behavior is left alone.

Source: `repo/open-spatial-lab/web/wow-url.mjs` lines 73-84 (I2), 152-164 (I3), 174-186 (I4).

Claim boundary: these are interpretations of the specification's examples, not extensions. They are verified in code with 70 mutation-tested checks (reported: R-014 row). OSL did not invent the verbs or the `aspect.id` syntax; both come from the specification's README.

**spatialID as string (Declared Divergence D4).** OSL types spatialID as `string` instead of the specification's `integer`, allowing identifiers such as `world_id`, `location_id`, `default`, and `world`. The divergence is labeled in the OSL schema with `x-osl-divergence`.

Source: `repo/open-spatial-lab/wow-spec/schema.yaml` lines 210-220.

Claim boundary: this is a declared divergence from the specification. OSL's string identifiers are tested within its own runtime; no conformance claim is made against the specification's integer type.

**World URL as path prefix (Interpretation I7).** OSL requires that a world's URL be an origin-or-path prefix (e.g., `/w/{worldKey}/`), not a query-based URL. The specification's own examples compose resources by path concatenation ("URL/wow/world"), which is well-defined only for path prefixes. OSL's pre-existing entry URLs were query-based and failed this test, which is how the gap was discovered.

Source: `repo/open-spatial-lab/web/wow-url.mjs` lines 316-328.

Claim boundary: this is an interpretation. The specification's examples imply path concatenation but never state the constraint. OSL's legacy query-form URLs still work via a redirect.

**History semantics (labeled extension).** OSL defined a history entry as `{address (all components), contentHash (sha256 of verified token bytes), poseAtLeave}`. Going back restores the address and the camera pose at the moment of departure. Re-verification runs on every history activation via the session cache (fast but never unverified). UP (structural parent, from the descent stack or `&via=`) is distinct from BACK (temporal predecessor).

Source: `repo/open-spatial-lab/docs/NAVIGATION-ARCHITECTURE.md` lines 169-187.

Claim boundary: this is an OSL extension. The specification has no text on history. The semantics are implemented within OSL's navigation pipeline; no second implementation has tested them.

**Fabric services pointer (labeled extension).** OSL implemented a `services[]` array in the root fabric manifest. Each entry carries a `name` (logical resource), an optional `type` (protocol family, `web-of-worlds`), and an `endpoint` (URL template). The client resolves templates against the manifest base URL. Service names map to WoW resources: `wow` to `/wow/world`, `wow-user` to `/wow/user/{userId}`, `wow-portal` to `/wow/portal/{portalId}`, `wow-view` to `/wow/view/{viewId}`. When `services[]` is absent, the client falls back to hardcoded integer routes.

Source: `repo/open-spatial-lab/web/live-adapter.js` lines 564-658.

Claim boundary: this is an OSL extension. The specification has no manifest or service-discovery concept. The pointer is used within OSL's own browser; no second implementation consumes it.

**Well-known discovery endpoint (labeled extension).** OSL defined `/.well-known/spatial-fabric` as a discovery endpoint returning the fabric manifest URL, a region streaming endpoint, and a presence endpoint. This follows the IETF RFC 8615 pattern for well-known URIs.

Source: `repo/open-spatial-lab/web/airport-lobby-transition.mjs` lines 254-260.

Claim boundary: this is an OSL extension. No IANA registration has been filed. The endpoint is consumed within OSL's own crossing pipeline.

**GET / omitted.** OSL omitted `GET /` (API.yaml lines 17-29) from its contract because OSL serves its client independently of the API endpoint. Including a `text/html` root would be a misleading conformance claim.

Source: `repo/open-spatial-lab/wow-spec/OSL-WOW-CONTRACT.md` line 190.


## Proposed normative text

**1. Fragment grammar.**

```
A WoW URL fragment MUST contain at most one verb token.
The verb MUST be one of: join, follow, preview.
Verb matching MUST be case-insensitive.
A verb MAY be followed by =aspect.id, where aspect.id
identifies a sub-element of the world.
An unrecognized fragment MUST NOT be treated as a WoW intent;
the client MUST fall back to the bare-URL join behavior.
```

Rationale: without these rules, two implementations will parse the same URL differently; unrecognized fragments must be preserved for non-WoW uses of the fragment on the same page.

**2. Aspect definition.**

```
The specification SHOULD define the term "aspect" and enumerate
which resources are aspects, or provide a discovery mechanism
for aspect kinds.
```

Rationale: `aspect.id` appears in the fragment grammar but has no definition; implementations cannot resolve it without knowing which resources count as aspects.

**3. spatialID type.**

```yaml
parameters:
  - name: spatialID
    in: path
    required: true
    schema:
      type: string
```

```
spatialID SHOULD be typed as string to allow both numeric
and named identifiers.
```

Rationale: an integer type rejects human-readable and semantically meaningful graph identifiers.

**4. World URL as path prefix.**

```
A world URL MUST be an origin-or-path prefix.
The Optional Feature resources compose onto the world URL
by path concatenation.
A query-based URL is not a valid world URL.
```

Rationale: the specification's own examples (README.md lines 22-31) compose resources by appending `/wow/world` to the world URL; this is well-defined only for path prefixes.

**5. Address grammar (optional extension).**

```
A client that supports spatial deep linking SHOULD encode
the world address as URL query parameters on the hosting
page's URL:
  ?fabric=<world-url>   -- the root world (trust boundary)
  &via=<url>[,<url>]*   -- optional descent breadcrumb (advisory)
  &view=<v>&az=<a>&el=<e>&fit=<f> -- optional camera pose
  #/<node-path>         -- optional place inside the root
```

Rationale: without a defined address grammar, Persist and Share (README.md lines 14-15) are unimplementable.

**6. History semantics (optional extension).**

```
A client MUST record a history entry on every root-changing
navigation. The entry MUST contain the full address and the
camera pose at departure. Going back MUST re-verify the root
(from cache if unchanged). UP (structural parent) MUST be
distinct from BACK (temporal predecessor).
```

Rationale: the browser's Back button is the most trained navigation gesture on the web; spatial browsers must integrate with it.

**7. Service discovery pointer (optional extension).**

```yaml
services:
  - name: wow
    type: web-of-worlds
    endpoint: /wow/world
  - name: wow-user
    type: web-of-worlds
    endpoint: /wow/user/{userId}
```

```
A world manifest SHOULD include a services array with typed
endpoint templates. Each entry MUST have a name (logical resource),
a type (protocol family), and an endpoint (URL template resolved
against the manifest base URL).
```

Rationale: without a discovery mechanism, clients must hardcode endpoint paths for every server.

**8. Well-known discovery endpoint (optional extension).**

```
A world server SHOULD serve /.well-known/spatial-fabric
returning the fabric manifest URL and advertised service
endpoints. Registration with IANA under RFC 8615 SHOULD
be pursued.
```

Rationale: DNS-level service discovery lets a client bootstrap from a domain name alone, following the IETF well-known URI pattern.


## Adoption path

**A minimal world** (one scene, no portals, no crossing) needs only items 1, 3, and 4 from the proposed text: the fragment grammar, the spatialID type as string, and the world URL as a path prefix. These are small changes to the existing specification text and do not add new endpoints or schemas. A minimal world does not need the address grammar, history semantics, service discovery pointer, or well-known endpoint.

**A client** that wants to support deep linking, bookmarks, and browser back/forward additionally adopts items 5 and 6 (address grammar and history semantics). A client that wants to discover endpoints from a manifest additionally adopts item 7. A client that wants domain-level bootstrapping additionally adopts item 8.

**A server** that wants to advertise its endpoints in a manifest adds `services[]` entries (item 7). A server that wants domain-level discovery serves `/.well-known/spatial-fabric` (item 8). Neither requires changes to the existing WoW API routes.


## Open questions for the working group

- Should `GET /` remain in the standard? It couples the API endpoint to a specific web-app delivery mechanism (`text/html`), which does not apply to all implementations. OSL omitted it. (CM-062)

- The aspect definition (item 2) can go two ways: enumerate the aspects in the specification, or provide a discovery mechanism. Which approach does the group prefer?

- The address grammar (item 5), history semantics (item 6), service discovery pointer (item 7), and well-known endpoint (item 8) are optional extensions proposed from one implementation. Should any of them become required?

- The `#/<node-path>` fragment for places inside a composed world depends on the composition model. If the composition model changes, the fragment grammar changes with it. Should the fragment grammar be defined separately from the composition model, or together?

- Should the IANA registration for `/.well-known/spatial-fabric` be pursued now, or deferred until a second implementation adopts it?


## Sources

- WebOfWorlds/WoWAPI specification at commit d39a1a0: `specification/OpenSpatialWorld/API.yaml`, `specification/OpenSpatialWorld/README.md`. Preview: https://redocly.github.io/redoc/?url=https://raw.githubusercontent.com/WebOfWorlds/WoWAPI/refs/heads/main/specification/OpenSpatialWorld/API.yaml
- Open Spatial Lab navigation architecture: `repo/open-spatial-lab/docs/NAVIGATION-ARCHITECTURE.md`
- Open Spatial Lab WoW URL parser: `repo/open-spatial-lab/web/wow-url.mjs`
- Open Spatial Lab WoW schema (with declared divergences): `repo/open-spatial-lab/wow-spec/schema.yaml`
- Open Spatial Lab WoW contract: `repo/open-spatial-lab/wow-spec/OSL-WOW-CONTRACT.md`
- Open Spatial Lab live adapter (service discovery): `repo/open-spatial-lab/web/live-adapter.js`
- Open Spatial Lab airport-lobby-transition (well-known endpoint): `repo/open-spatial-lab/web/airport-lobby-transition.mjs`
- IETF RFC 8615: Well-Known URIs. https://www.rfc-editor.org/rfc/rfc8615


## Change log

- 2026-09-07: first public draft, verified.
