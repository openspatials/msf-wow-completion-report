# Discovery and Addressing for Web of Worlds

The Web of Worlds README already defines URL entry, join/follow/preview examples, resource paths, persistence and sharing. What remains is a precise binding for fragment interpretation, endpoint resolution, optional pose deep links and history behavior. The March 31, 2026 whitepaper uses query-plus-fragment examples on printed page 28; those establish architectural intent, not a settled grammar.

Open Spatial Lab developed local address, fragment, discovery and history conventions. The path-prefix rule is a local API-base convention; it does not make query-bearing entry URLs invalid. Named spatial identifiers are a declared implementation divergence from the pinned integer path parameter.

All choices below remain proposals. Case-insensitive fragment matching, typed aspects, named spatial identifiers and representation negotiation need explicit acceptance and migration rules. Source examples and local checks do not establish agreement with a second implementation.

Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).


## What the specification says today

**Fragment verbs.** The README lists fragment verbs in the Core Requirements table (README.md lines 11-13):

> "URL, URL#join, URL#join=aspect.id" / "URL#follow, URL#follow=aspect.id" / "URL#preview, #preview=aspect.id"

Source: [README.md lines 11-13](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L11-L13).

The table lists three verbs (join, follow, preview) with an optional `=aspect.id` parameter. The whitepaper's printed-page-28 example includes `#join=view.5845`, showing a kind, a dot and an identifier. It does not settle a complete grammar: case sensitivity, compound fragments, escaping and unrecognized-fragment behavior remain unbound. The Preview description is truncated mid-clause ("experence world without" -- verbatim).

**The term "aspect."** The word "aspect" appears in the specification in three contexts: as the syntax element `aspect.id` in the fragment examples (README.md lines 11-13), in the API description (API.yaml lines 4-5, "read and modify aspects of the world" and "some aspects of spatial data management"), and in the Optional Feature section (README.md lines 21-24, "Read world status including live aspects" and "Read and write aspects of the world to enable links to sub-elements of the world (e.g user avatar)"). The latter two discuss the concept; the first uses it as a parameter name without definition. The whitepaper's `view.5845` example makes View one published aspect case; the 2025 post also names addressable users and views. Neither source gives a complete aspect-kind registry, identifier encoding or resolution/failure algorithm.

Source: [README.md lines 11-13](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L11-L13); [API.yaml lines 4-5](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L4-L5); [README.md lines 21-24](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L21-L24).

**spatialID type.** API.yaml defines `spatialID` as `type: integer` (API.yaml lines 141-142, 144-146):

> `name: spatialID` / `in: path` / `required: true` / `schema:` / `type: integer`

Source: [API.yaml lines 141-142, 144-146](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L141-L146).

**World URL as path prefix (local interpretation I7).** Open Spatial Lab uses origin/path service bases such as `/w/{worldKey}/` and keeps legacy query entry URLs via redirects. This avoids its text-concatenation bug. It is a local convention, not a proof that query-based world entry is invalid; service bases and entry URLs need separate resolution semantics.

Source: [README.md lines 22-31](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L22-L31).

**GET /.** API.yaml defines `GET /` as returning `text/html` with summary "Provisions a web application to deliver the spatial experience on the given device" and operationId `getDefaultApp` (API.yaml lines 17-29).

Source: [API.yaml lines 17-29](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L17-L29).

**Persist and Share.** The README lists "Persist world: store or bookmark URL" and "Share world: send URL to second user" as Core Requirements (README.md lines 14-15). No further definition of what the URL contains or what "bookmark" means for spatial state.

Source: [README.md lines 14-15](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L14-L15).

**Source boundary.** The reviewed API/README do not define a complete service-discovery or history contract. Their entry and resource examples already constitute addressing provisions, and the whitepaper elaborates delivery. Exact bindings and failure behavior must not be confused with an absence of the underlying concepts.


## What fails without it

**Pose deep links need a shared grammar.** Existing world URLs can already be bookmarked and shared. The reviewed contract does not specify a portable encoding for every node, camera pose and composition breadcrumb, so those richer links may not reproduce the same viewpoint across clients.

**Two implementations parse the same fragment differently.** Browser A treats `#Join` as a join intent; Browser B rejects it because the verb is capitalized. Browser A allows `#join&follow`; Browser B treats it as unrecognized. The specification gives no rules to resolve these, so each implementation invents its own. The result is that a URL with a fragment that works in one browser fails or behaves differently in another.

**A WoW client hijacks non-WoW fragments.** A page with both a WoW world and ordinary HTML section anchors (`#about`, `#contact`) loses its section anchors if the WoW client treats every fragment as a WoW intent. The specification is silent on what to do with unrecognized fragments.

**Named identifiers need mapping or a profile.** The integer path parameter does not accept Open Spatial Lab's named ids directly. A server can map internal names to canonical integers; changing the wire type is an alternative proposal, not a prerequisite for node addressing.

**Blind text concatenation targets the wrong endpoint.** Appending `/wow/world` to `http://host/?role=player` produces a syntactically valid URL whose pathname is `/` and whose query is `?role=player/wow/world`. The suffix was placed inside the query. This proves incorrect composition, not invalid URL syntax or a prohibition on query-bearing entry addresses.

**Deployment-specific endpoints need discovery or a defined base.** Canonical resource paths already exist. To support alternate mounts, clients need a defined API base, a documented mapping or advertised endpoints, rather than guessing how presentation URLs compose with resource paths.

**Back/forward navigation has no definition.** A user who has crossed from World A to World B presses the browser's Back button. The specification says nothing about what a history entry contains, whether the camera pose is restored, whether re-verification is required, or what the distinction between "back" (temporal) and "up" (structural parent) is.


## What Open Spatial Lab built and learned

**Address grammar (OSL-NAV-1, labeled extension).** OSL designed a URL-based addressing scheme that separates the trust-bearing root identifier from presentation parameters and context. The grammar uses query parameters on the hosting page's URL:

- `?fabric=<url>` carries the root world's canonical URL. This identifies the primary content root; indirect service and nested-resource fetches have their own trust boundaries.
- `&via=<url>,<url>` carries an optional descent breadcrumb (advisory, capped at four entries), giving the UP affordance a target even on a cold deep link.
- `&view=` / `&az=` / `&el=` / `&fit=` carry camera pose.
- `#/<node-path>` is a fragment identifier for a place inside the root, using slash-separated node names from the composed walk, sanitized (lowercase, spaces to hyphens), disambiguated by walk-order suffix (`~2`) when names collide.

The design encodes world identity, descent context, camera pose and place. It intends to restore that view; changed content, missing nodes and unavailable roots require defined failure behavior.

Source: `NAVIGATION-ARCHITECTURE.md` lines 139-167.

Claim boundary: this is an OSL extension. The pinned examples do not define this extended pose/breadcrumb grammar. The cited local grammar evidence is distinct from the full pose/breadcrumb design. Independent adoption and full address restoration are not established by those parser checks.

**Fragment grammar interpretation (Interpretations I2, I3, I4).** OSL interpreted the specification's fragment examples as follows:

- Interpretation I3: a fragment is a single `verb[=value]` token, matched case-insensitively. Compound fragments (`#join&follow`) are rejected as unrecognized. OSL is strict in what it emits (always lowercase).
- Interpretation I2: the local parser resolves user/avatar and node identifiers and accepts an optional slash qualifier such as `user/4182`. Its `ASPECT_KIND` and resolver do not support `view` or the published dotted `view.5845` form as typed View resolution. That is a local compatibility gap, not evidence that the publications omit View.
- Interpretation I4: an unrecognized fragment (`#some-anchor`) is not a WoW intent. OSL falls back to the bare-URL join reading and flags `recognized:false` so ordinary anchor behavior is left alone.

Source: `wow-url.mjs` lines 73-84 (I2), 152-164 (I3), 174-186 (I4).

Claim boundary: these are local interpretations of the named verbs and examples. The historical 70-check grammar result is reported, not rerun here, and does not prove that other clients share the parsing rules. Case-insensitivity and fallback behavior remain proposed profile choices.

**spatialID as string (Declared Divergence D4).** OSL types spatialID as `string` instead of the specification's `integer`, allowing identifiers such as `world_id`, `location_id`, `default`, and `world`. The divergence is labeled in the OSL schema with `x-osl-divergence`.

Source: `schema.yaml` lines 210-220.

Claim boundary: this is a declared divergence from the specification. OSL's string identifiers are tested within its own runtime; no conformance claim is made against the specification's integer type.

**World URL as path prefix (Interpretation I7).** OSL requires that a world's URL be an origin-or-path prefix (e.g., `/w/{worldKey}/`), not a query-based URL. The specification's own examples compose resources by path concatenation ("URL/wow/world"), which is well-defined only for path prefixes. OSL's pre-existing entry URLs were query-based and failed this test, which is how the gap was discovered.

Source: `wow-url.mjs` lines 316-328.

Claim boundary: the README examples leave endpoint-base resolution underdefined. The whitepaper's page 28 query/fragment examples are illustrative and partly malformed; they support intended use, not a complete parsing algorithm.

**History semantics (labeled extension).** OSL defined a history entry as `{address (all components), contentHash (sha256 of verified token bytes), poseAtLeave}`. Going back restores the address and the camera pose at the moment of departure. The design requires history activation to retain the binding between verified bytes and the applicable trust policy, including when using a cache. UP (structural parent, from the descent stack or `&via=`) is distinct from BACK (temporal predecessor).

Source: `NAVIGATION-ARCHITECTURE.md` lines 169-187.

Claim boundary: this is an OSL extension. The specification has no text on history. These semantics are recorded as a navigation design; the cited evidence does not establish the full history behavior as a live cross-implementation result.

**Fabric services pointer (labeled extension).** OSL implemented a `services[]` array in the root fabric manifest. Each entry carries a `name` (logical resource), an optional `type` (protocol family, `web-of-worlds`), and an `endpoint` (URL template). The client resolves templates against the manifest base URL. Service names map to WoW resources: `wow` to `/wow/world`, `wow-user` to `/wow/user/{userId}`, `wow-portal` to `/wow/portal/{portalId}`, `wow-view` to `/wow/view/{viewId}`. When `services[]` is absent, the client falls back to hardcoded integer routes.

Source: `live-adapter.js` lines 564-658.

Claim boundary: this is an OSL extension. The reviewed API does not standardize this services-array binding. The pointer is used within OSL's own browser; consumption by a second implementation was not established.

**Well-known discovery endpoint (labeled extension).** OSL defined `/.well-known/spatial-fabric` as a discovery endpoint returning the fabric manifest URL, a region streaming endpoint, and a presence endpoint. This follows the IETF RFC 8615 pattern for well-known URIs.

Source: `airport-lobby-transition.mjs` lines 254-260.

Claim boundary: this is an OSL extension. No IANA registration has been filed. The endpoint is consumed within OSL's own crossing pipeline.

**GET / omitted.** OSL omitted `GET /` (API.yaml lines 17-29) from its contract because OSL serves its client independently of the API endpoint. Including a `text/html` root would be a misleading conformance claim.

Source: `OSL-WOW-CONTRACT.md` line 190.


## Proposed normative text

These are unadopted candidate profile choices, not canonical behavior. Schema/parameter fragments target OpenAPI 3.0.4; address and service examples are labeled explanatory data.

**1. Fragment grammar.**

```
A WoW URL fragment MUST contain at most one verb token.
The verb MUST be one of: join, follow, preview.
Verb matching MUST be case-insensitive.
A verb MAY be followed by =kind.id, where kind is user, view
or node and id is a non-empty resource identifier.
An unrecognized fragment MUST NOT be consumed or rewritten
as a WoW intent. Preserve ordinary page-anchor handling.
A recognized verb with an unresolved or unsupported target
MUST report that target outcome; it MUST NOT silently become
a bare join.
```

Rationale: without these rules, two implementations will parse the same URL differently; unrecognized fragments must be preserved for non-WoW uses of the fragment on the same page.

**2. Aspect definition.**

```
Use the published dotted form kind.id for typed aspects.
The initial proposed kinds are user, view and node.
Resolve view.5845 through the View endpoint for id 5845;
resolve user and node through their selected resource bindings.
For node, the profile must identify the composition graph as well
as the node when the chosen path includes a graph identifier.
```

Rationale: `view.5845` is already a published example. The proposed parser separates the first literal dot from the kind, then decodes the non-empty identifier once; malformed escapes or an unknown kind produce an explicit target error. A View identifier must also satisfy the selected View endpoint's integer path type. Local slash-qualified or unqualified forms may remain in a declared legacy profile; do not guess among resource kinds for a new typed address. Case-insensitive verb matching remains a proposal. Identifier case follows the selected resource contract.

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

Rationale: the optional string-id profile permits named wire identifiers. Canonical integers remain the default; a server can map internal names to integers instead. The group must specify migration and supported-profile negotiation before changing the path type.

**4. Entry URL and API-base resolution.**

Proposed rule: keep the entry/presentation URL separate from the API base. Resolve advertised URI references using RFC 3986 section 5 against a declared base, using platform URL parsing; never append resource path text to an unparsed URL. Define the base's trailing-slash behavior and whether a service path is root-relative or relative to its mount. Preserve query/fragment meaning according to the selected reference, rather than copying or dropping components indiscriminately.

```text
entry URL: https://example.org/w/room/?role=player#join
declared API base: https://example.org/w/room/
service reference: wow/world
resolved endpoint: https://example.org/w/room/wow/world
root-relative reference /wow/world resolves to https://example.org/wow/world
asset reference ../assets/chair.glb resolves to https://example.org/w/assets/chair.glb
```

The fragment is client-side entry state, not part of the HTTP request target. Query parameters belong to the resource that defines them; this example does not propagate the presentation `role` query to a separate API endpoint. Unsupported schemes or unresolved references produce an explicit failure. Relative references are usable when their base and resolution rules are known.

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

Rationale: the extended grammar proposes portable pose/place state beyond the world-link persistence and sharing already named in the README.

**6. History semantics (optional extension).**

```
A client MUST record a history entry on every root-changing
navigation. The entry MUST contain the full address and the
camera pose at departure. For a signed-content profile, going back MUST validate the root
under the applicable byte-binding, trust and freshness policy;
a still-valid cached result MAY be reused. UP (structural parent) MUST be
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

Rationale: discovery or an agreed API base supports alternate mounts; canonical resource paths already provide a baseline.

**8. Well-known discovery endpoint (optional extension).**

```
A world server SHOULD serve /.well-known/spatial-fabric
returning the fabric manifest URL and advertised service
endpoints. Registration with IANA under RFC 8615 SHOULD
be pursued.
```

Rationale: a well-known HTTP resource can bootstrap service discovery from an origin. This is HTTP discovery using an origin that may be resolved through DNS, not DNS-level service discovery.


## Adoption path

**Existing worlds.** The canonical integer spatialID and existing URL entry examples remain supported. A string-id wire profile must define numeric-id migration/mapping; changing a path-parameter type is not automatically compatible. Query-bearing entry URLs remain valid.

**Clients.** Support published dotted View targets under the agreed profile before claiming that entry example. Keep OSL's slash and unqualified forms explicitly legacy; agree a fragment profile before relying on case-insensitive matching or aspect fallback. Resolve entry, service and asset references against their specified bases, preserve query/fragment semantics, and report unresolved targets. Pose-history and breadcrumb support can remain optional.

**Servers.** Advertise API bases or service templates when deployments need discovery. The optional well-known HTTP resource and services array require an agreed shape and error behavior before independent clients can rely on them. They do not require changing canonical node routes.

## Open questions for the working group

- Should `GET /` remain in the standard? It couples the API endpoint to a specific web-app delivery mechanism (`text/html`), which does not apply to all implementations. OSL omitted it. (CM-062)

- The aspect definition (item 2) can go two ways: enumerate the aspects in the specification, or provide a discovery mechanism. Which approach does the group prefer?

- The address grammar (item 5), history semantics (item 6), service discovery pointer (item 7), and well-known endpoint (item 8) are optional extensions proposed from one implementation. Should any of them become required?

- The `#/<node-path>` fragment for places inside a composed world depends on the composition model. If the composition model changes, the fragment grammar changes with it. Should the fragment grammar be defined separately from the composition model, or together?

- Should the IANA registration for `/.well-known/spatial-fabric` be pursued now, or deferred until a second implementation adopts it?


## Sources

- [Linked Spatial Experiences: The Web of Worlds, April 2, 2025](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/): published units, preview, authorization and aspect intent.

- [OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialWorld/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf); relevant printed pages are identified in this chapter or [chapter 10](10-role-and-blind-spots.md).
- [RFC 3986, section 5, reference resolution](https://www.rfc-editor.org/rfc/rfc3986#section-5) and [RFC 8615, Well-Known URIs](https://www.rfc-editor.org/rfc/rfc8615).
- Open Spatial Lab local source snapshot and retained evidence, checked September 7, 2026: NAVIGATION-ARCHITECTURE.md, wow-url.mjs, schema.yaml, OSL-WOW-CONTRACT.md, live-adapter.js and airport-lobby-transition.mjs. Address/history designs and reported grammar checks do not establish a second implementation or universal URL convention. Public reproduction of these exact local bytes is not established.
- [Appendix A](A-completion-map.md) and [Appendix B](B-findings-register.md) preserve the historical surface/finding identifiers.

## Change log

- 2026-09-07: corrected source scope, proposal compatibility and evidence boundaries; updated public citations.
