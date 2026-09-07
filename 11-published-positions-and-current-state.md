# Published Positions and Current State

The Web of Worlds publications and the 2026-08-24 working-group meeting declare 49 positions: 4 vision principles, 5 core concepts, 5 core requirements, 16 API operations, 4 statements on the GitHub Pages home, 1 statement in the announcement post, 9 requirements from the initial blog post, and 5 governance structures. Of these, 20 are present in the specification text at commit d39a1a0 (the 16 API operations, the asset content-negotiation surface the whitepaper names, and three GitHub Pages statements about views, the composition graph, and registered model types), 17 are named but not yet defined (the README names them, a schema exists but carries no behavioural contract, or a publication declares them and the API text partially covers them), and 12 are aspirations with no specification text at all (the four principles, decentralized architecture, AI integration, and six requirements from the blog post that no API endpoint or schema addresses). For an implementer, the 20 present items are buildable from the specification alone. The 17 named-only items require inventing contracts. The 12 aspirations require inventing the entire mechanism.

This document lists every declaration from the Web of Worlds publications in the reference set and the five governance structures described at the 2026-08-24 working-group meeting, shows what the specification text says at commit d39a1a0, and identifies which of our ten documents treats each one.

## Status

Publications checked on 2026-09-07.

- **Whitepaper** (2026-Q1): live page byte-identical to the 2026-06-23 capture (file comparison; verified 2026-09-07T07:05Z and again at 08:12Z). Source repository HEAD 988f369b, unchanged since the capture.
- **WoWAPI specification** at commit d39a1a0 (2026-05-21): upstream HEAD d39a1a0, equal to cited commit (verified by git ls-remote, 2026-09-07T07:05Z).
- **simpleWorlds** reference implementation: HEAD d2bda3e vs cited 13d2cbe. This package cites simpleWorlds only for its URL path choice and, in document 10, for its two licence files; its code is not assessed.
- **GitHub Pages home** (dated 2026-03-31): text changed since 2026-07-01 capture. Two changes to the visible text: "official Spatial Computing WG" became "new Spatial Computing WG"; the implementations table replaced HTMLModeWrapper with Open-Spatial-Lab. The MSF Project slides link target also changed. Updated capture saved as 2026-09-07-webofworlds-github-pages-home.html (the live page was byte-identical to it at 2026-09-07T08:12Z). Quotes below use the 2026-09-07 text where it differs.
- **MSF announcement post** "Announcing the Web of Worlds whitepaper" (published 2026-06-03): article text identical to 2026-06-23 capture; only site-wide CSS and navigation menu items changed (verified 2026-09-07T07:05Z).
- **MSF post** "Linked spatial experiences: the Web of Worlds" (published 2025-04-02, modified 2025-09-04): article text identical to 2026-06-23 capture; only site-wide CSS and navigation menu items changed (verified 2026-09-07T07:05Z).

## How to read this document

Each row below shows one declared position from a Web of Worlds publication. The four columns are:

- **Declared**: the exact text from the publication, or a summary or paraphrase where the section says so.
- **Source**: which publication, its version or date, and a public URL.
- **In the text today**: what the specification text at commit d39a1a0 contains. "Present" means the API text defines this with an operation or schema (line number given). "Partial" means a schema or operation exists but does not carry the behavioural contract the declaration describes (line number given). "Absent" means the API text and README contain no reference (search terms given). Search terms were matched as whole words, case-insensitive; acronyms (AI, DID, SSO, TLS, MITM, CLA, SDO, HTTPS, JSON-LD) were matched case-sensitive; `scalab` is a stem and `create.*view` a pattern.
- **Our document**: which of the ten documents in this package treats this item, with a link.

The three classifications:

- **Specified** (20): the API text defines the operation or schema with enough detail that an implementer can build it from the specification alone.
- **Named only** (17): a publication names the feature or a schema exists, but the specification text does not define the behaviour, the contract, or the required fields.
- **Aspiration** (12): the publications declare a vision or requirement that no API operation, schema, or README text addresses.


## Whitepaper vision principles

The whitepaper (2026-Q1) declares four Open Web Platform principles under "Vision." None appears in any API file. The entries below summarize the whitepaper text; colons replace the original formatting.

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| 1 | Universality: Accessible on any device | Whitepaper, 2026-Q1 ([page](https://webofworlds.github.io/initial_MSF_Whitepaper/)) | Absent. Search terms: `universality`, `adaptation`, `capability` (0 hits each in API.yaml and README.md). The term `device` returns 1 hit in API.yaml (L21, in the GET / summary, not a specification of the feature) and 2 in README.md (L7, the sentence under the Core Requirements heading, and L11, the Join world row; neither specifies the feature). | [01 Coordinate Precision, Units, and World Extents](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md); [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |
| 2 | Interoperability: Consistent cross-platform experiences | Whitepaper, 2026-Q1 ([page](https://webofworlds.github.io/initial_MSF_Whitepaper/)) | Absent. Search terms: `interoperability`, `cross-platform`, `consistent` (0 hits each in API.yaml). | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |
| 3 | Decentralization: No central authority | Whitepaper, 2026-Q1 ([page](https://webofworlds.github.io/initial_MSF_Whitepaper/)) | Absent. Search terms: `decentralization`, `distributed`, `federation`, `DID` (0 hits each in API.yaml). | [03 Portable User State and Identity](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md); [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |
| 4 | Accessibility: Inclusive by design | Whitepaper, 2026-Q1 ([page](https://webofworlds.github.io/initial_MSF_Whitepaper/)) | Absent. Search terms: `accessibility`, `inclusive`, `a11y` (0 hits each in API.yaml, 0 in README.md). | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |

Classification: all four are aspirations.


## Whitepaper core concepts

The whitepaper (2026-Q1) declares five core concepts under headings below the Vision list. The entries below summarize the whitepaper text; colons and commas replace the original formatting.

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| 5 | Linked Spatial Experiences: Worlds are addressable via URIs (like web pages), Navigation through portals and links, Forms a network of connected virtual environments | Whitepaper, 2026-Q1 ([page](https://webofworlds.github.io/initial_MSF_Whitepaper/)) | Partial. Portal schema exists ([API.yaml L409-436](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L409-L436)) but carries only `id` and `geoPose`. No target URI, no link destination, no navigation protocol. | [02 Portal Destination and Traversal](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md); [06 Discovery and Addressing](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md) |
| 6 | Shared Spatial Assets: Modular, reusable 3D components, Built on existing standards (e.g. gltf, x3d, usd), Delivered via standard web infrastructure | Whitepaper, 2026-Q1 ([page](https://webofworlds.github.io/initial_MSF_Whitepaper/)) | Present. OpenSpatialAsset API defines content negotiation across 21 IANA-registered model types ([OpenSpatialAsset/API.yaml L35-161](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml#L35-L161)) and an Asset metadata schema. | [07 Assets and the Render Seam](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md) |
| 7 | Shared User Manifest (“Digital YOU”): A portable, user-controlled identity layer. Includes: Avatar and identity, Preferences and settings, Assets (wearables, credentials, NFTs). Powered by: Decentralized Identifiers (DIDs), Verifiable Credentials, Digital wallets | Whitepaper, 2026-Q1 ([page](https://webofworlds.github.io/initial_MSF_Whitepaper/)) | Partial. OpenUserManifest API exists ([OpenUserManifest/API.yaml L32-60](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml#L32-L60)) but the UserManifest schema defines only three fields under a `content` object: `name` (string), `age` (number), `avatarAssetURI` (string). No preferences, settings, wearables, credentials, NFTs, DIDs, Verifiable Credentials, or wallets. | [03 Portable User State and Identity](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md) |
| 8 | Decentralized Architecture: No platform lock-in, User-owned data, Distributed storage and identity | Whitepaper, 2026-Q1 ([page](https://webofworlds.github.io/initial_MSF_Whitepaper/)) | Absent. Search terms: `distributed`, `federation`, `DID`, `lock-in`, `user-owned` (0 hits each across all three API files). The APIs define centralized REST endpoints. | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |
| 9 | AI Integration: AI agents operate alongside humans. Enables: Automation, Spatial reasoning, Human-AI collaboration | Whitepaper, 2026-Q1 ([page](https://webofworlds.github.io/initial_MSF_Whitepaper/)) | Absent. Search terms: `AI`, `agent`, `automation`, `reasoning` (0 hits each in all three API files; README.md L7 has `User-Agent`, the HTTP user agent, not an AI agent). | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |

Classification: concept 6 (Shared Spatial Assets) is specified. Concepts 5 and 7 are named only. Concepts 8 and 9 are aspirations.


## README Core Requirements

The OpenSpatialWorld README ([README.md L7-15](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L7-L15)) declares "Core Requirements and Feature" as a table with five rows.

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| 10 | "Join world: URL, URL#join, URL#join=aspect.id: join the world as new or existing user on a given device and UA" | README.md L11, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L11)) | Partial. GET / serves text/html ([API.yaml L17-29](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L17-L29)), but the `#join` fragment behaviour is not defined anywhere in API.yaml. Search terms in API.yaml: `join` (1 hit in info.description L5, not a specification of the fragment verb), `fragment` (0 hits). | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md); [06 Discovery and Addressing](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md) |
| 11 | "Follow world: URL#follow, URL#follow=aspect.id: follow the world as new or existing user" | README.md L12, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L12)) | Absent. Search terms in API.yaml: `follow` (0 hits), `fragment` (0 hits). No endpoint or fragment-handling behaviour defined. | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md); [06 Discovery and Addressing](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md) |
| 12 | "Preview world: URL#preview, #preview=aspect.id: experence world without" | README.md L13, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L13)) | Absent. The description is truncated mid-sentence in the source (verbatim: "experence world without"). Search terms in API.yaml: `preview` (0 hits), `fragment` (0 hits). | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md); [06 Discovery and Addressing](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md); [09 Conformance Vocabulary and Errata](https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md) |
| 13 | "Persist world: store or bookmark URL" | README.md L14, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L14)) | Absent. Defined as browser-side bookmarking. No server-side persistence API. Search terms in API.yaml: `persist` (0 hits), `bookmark` (0 hits). The description column is empty in the source table. | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md) |
| 14 | "Share world: send URL to second user" | README.md L15, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L15)) | Absent. Defined as external URL sharing. No sharing endpoint. Search terms in API.yaml: `share` (1 hit in info.description L5, not a sharing endpoint), `send` (0 hits). The description column is empty in the source table. | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md) |

Classification: row 10 (Join) is named only (the GET / endpoint exists but the fragment verb is not defined). Rows 11-14 are named only (the README names them but no API endpoint or behaviour is defined).


## API.yaml declared surfaces

The three API files define 16 operations across three services. All 16 are present in the specification text with defined request/response shapes.

### OpenSpatialWorld API (v0.0.1)

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| 15 | "GET /: Provisions a web application to deliver the spatial experience on the given device" | API.yaml L17-29, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L17-L29)) | Present. Returns text/html. Operation: getDefaultApp. | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md) |
| 16 | "GET /wow/world: Returns a World status as a single data state" | API.yaml L32-44, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L32-L44)) | Present. World schema at L268-349 includes content, geoPose, presence, technology, users, views, portals. | [01 Coordinate Precision, Units, and World Extents](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md) |
| 17 | "GET /wow/user/{userId}: Returns a User status" | API.yaml L47-66, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L47-L66)) | Present. User schema at L351-380: id, name, AvatarURI, geoPose. | [03 Portable User State and Identity](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md) |
| 18 | "DELETE /wow/user/{userId}: Deletes a user" | API.yaml L67-85, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L67-L85)) | Present. Returns 200 on success, 400 on invalid user value. | [03 Portable User State and Identity](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md) |
| 19 | "GET /wow/view/{viewId}: Returns a Viewpoint status" | API.yaml L88-107, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L88-L107)) | Present. View schema at L382-407: id, geoPose. | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md) |
| 20 | "GET /wow/portal/{portalId}: Returns a Portal status" | API.yaml L110-129, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L110-L129)) | Present. Portal schema at L409-436: id, geoPose. No destination URI or link target. | [02 Portal Destination and Traversal](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md) |
| 21 | "GET /wow/spatial/{spatialID}: Returns a single Spatial status as a single data state" | API.yaml L133-153, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L133-L153)) | Present. Spatial schema at L438-468: id, rootNodeID, geoPose. | [08 Composition Graph Schema Fixes](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md) |
| 22 | "POST /wow/spatial/{spatialID}/node/{nodeId}: Create and add new nodes." | API.yaml L156-188, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L156-L188)) | Present. Accepts array of Node objects under parent nodeId. | [08 Composition Graph Schema Fixes](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md) |
| 23 | "GET /wow/spatial/{spatialID}/node/{nodeId}: Returns a single Node tree as a single data state" | API.yaml L189-207, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L189-L207)) | Present. Returns Node tree from nodeId. | [08 Composition Graph Schema Fixes](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md) |
| 24 | "PUT /wow/spatial/{spatialID}/node/{nodeId}: Update an existing node." | API.yaml L208-241, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L208-L241)) | Present. Full node replacement. | [08 Composition Graph Schema Fixes](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md) |
| 25 | "DELETE /wow/spatial/{spatialID}/node/{nodeId}: Deletes a node" | API.yaml L242-260, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L242-L260)) | Present. Returns 200 on success, 400 on invalid node value. | [08 Composition Graph Schema Fixes](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md) |

### OpenSpatialAsset API (v0.0.1)

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| 26 | "HEAD /: Used to authorize resources and simultaneously check whether ETag still matches cached ETag." | OpenSpatialAsset/API.yaml L19-34, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml#L19-L34)) | Present. Returns 200 with ETag, 403 Forbidden, or 404 Not Found. | [07 Assets and the Render Seam](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md) |
| 27 | "GET /: Used to negotiate and fetch a data instance as resources." (content negotiation across 21 model types) | OpenSpatialAsset/API.yaml L35-161, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml#L35-L161)) | Present. 21 IANA-registered model types including gltf-binary, gltf+json, x3d+xml, vnd.usda, vnd.usdz+zip, step, vnd.collada+xml, vrml. | [07 Assets and the Render Seam](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md) |
| 28 | "GET /wow/asset: Returns a asset status as a single data state" | OpenSpatialAsset/API.yaml L162-179, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml#L162-L179)) | Present. Asset schema includes content (label, age_restriction, license, cost, version) and geoPose. | [07 Assets and the Render Seam](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md) |

### OpenUserManifest API (v0.0.1)

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| 29 | "HEAD /: Used to authorize resources and simultaneously check whether ETag still matches cached ETag." | OpenUserManifest/API.yaml L16-31, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml#L16-L31)) | Present. Same authorization pattern as OpenSpatialAsset. | [03 Portable User State and Identity](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md) |
| 30 | "GET /: Returns a user manifest as a single data state" | OpenUserManifest/API.yaml L32-60, commit d39a1a0 ([link](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml#L32-L60)) | Present. UserManifest schema has three fields under a content object: name (string), age (number), avatarAssetURI (string). | [03 Portable User State and Identity](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md) |

Classification: all 16 API operations are specified. The operations define request/response shapes; the schemas they reference are present but carry no required properties, no RFC 2119 keywords, and no behavioural contracts. These gaps are addressed in documents 01-09.


## GitHub Pages home

The WebOfWorlds GitHub Pages home (dated 2026-03-31, [page](https://webofworlds.github.io/)) restates the whitepaper's positions with additional detail on spatial worlds, user manifest, and spatial assets. Most declarations overlap with the whitepaper core concepts already listed above; this section covers the additional or differently-worded declarations.

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| 31 | "Portals stores links to external and independent world instance" | GitHub Pages home, 2026-03-31 ([page](https://webofworlds.github.io/)) | Partial. Portal schema exists ([API.yaml L409-436](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L409-L436)) with id and geoPose, but no destination URI or link target. The portal cannot store a link to an external world. | [02 Portal Destination and Traversal](https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md) |
| 32 | "Views to expose a viewing pose or camera to the consumer" | GitHub Pages home, 2026-03-31 ([page](https://webofworlds.github.io/)) | Present. View schema at [API.yaml L382-407](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L382-L407) defines id and geoPose, accessible via GET /wow/view/{viewId}. | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md) |
| 33 | "Spatial Composition Graph hieratical structure to manage any number of spatial assets links" (verbatim, including "hieratical") | GitHub Pages home, 2026-03-31 ([page](https://webofworlds.github.io/)) | Present. Spatial and Node schemas with CRUD operations at [API.yaml L133-260](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L133-L260). Node.spatialAssetURI carries the asset link. | [08 Composition Graph Schema Fixes](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md) |
| 34 | "RFC2077 standard and its associated model content registrations have formed a robust foundation on the existing web" | GitHub Pages home, 2026-03-31 ([page](https://webofworlds.github.io/)) | Present. OpenSpatialAsset API negotiates IANA-registered model media types, the model top-level type that RFC 2077 defined; its externalDocs (L13-15) point to the IANA list ([OpenSpatialAsset/API.yaml L35-161](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml#L35-L161)). | [07 Assets and the Render Seam](https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md) |

Classification: rows 32, 33, 34 are specified (present). Row 31 is named only (partial).


## MSF announcement post

"Announcing the Web of Worlds whitepaper: a concrete path to the open metaverse" (published 2026-06-03, [post](https://metaverse-standards.org/news/blog/announcing-the-web-of-worlds-whitepaper-a-concrete-path-to-the-open-metaverse/)). This post largely restates the whitepaper and names the three APIs. One declaration adds specificity beyond the whitepaper.

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| 35 | "Open User Manifest API (The “Digital YOU”): Powers a portable, user-controlled identity layer. Utilizing JSON-LD, Decentralized Identifiers (DIDs), and Verifiable Credentials, users can carry their avatars, preferences, and assets across different virtual worlds." | Announcement post, 2026-06-03 ([post](https://metaverse-standards.org/news/blog/announcing-the-web-of-worlds-whitepaper-a-concrete-path-to-the-open-metaverse/)) | Partial. OpenUserManifest API exists ([OpenUserManifest/API.yaml L32-60](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml#L32-L60)) but the schema defines three fields only (name, age, avatarAssetURI). Search terms for the declared scope, in the three API files and README.md: `JSON-LD` (0 hits in the API files; 1 hit in README.md L21 as an example container for world status, not for the user manifest), `DID` (0 hits), `credential` (0 hits), `wallet` (0 hits), `preference` (0 hits). | [03 Portable User State and Identity](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md) |

Classification: named only (the API exists but the declared scope far exceeds the defined surface). The announcement's other positions (three APIs, OWP values, AI collaboration) overlap with whitepaper entries 6, 7, 8, 9 above.


## MSF post: "Linked spatial experiences: the Web of Worlds"

Published 2025-04-02, modified 2025-09-04 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)). This is the earlier post that defined the core requirements before the whitepaper. It carries nine requirements that the whitepaper and the announcement post do not restate in the same terms.

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| 36 | "Capability to handle billions of addressable spatial data states" | Linked-spatial post, 2025-04-02 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)) | Absent. Search terms: `billion`, `scale`, `scalab`, `capacity` (0 hits each in API.yaml and README.md). The API uses integer IDs with no stated range. | [01 Coordinate Precision, Units, and World Extents](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md); [08 Composition Graph Schema Fixes](https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md) |
| 37 | "Experience consistency, e.g., in view and navigation parameters, units, physics" | Linked-spatial post, 2025-04-02 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)) | Absent. Search terms: `units` (0 hits), `physics` (0 hits), `consistency` (0 hits), `navigation` (1 hit: World.presence.navigation, a bare string with no definition at [API.yaml L312](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L312)). | [01 Coordinate Precision, Units, and World Extents](https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md) |
| 38 | "Seamless shared multi-user and multi-device scenarios" | Linked-spatial post, 2025-04-02 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)) | Absent. Search terms: `multi-user` (0 hits), `multi-device` (0 hits), `shared` (0 hits in API.yaml). World.users carries active_user_count and total_user_count as static counters ([API.yaml L325-335](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L325-L335)) but no multi-user protocol. | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md) |
| 39 | "Support for mixed and dynamic user and device configurations e.g., desktop, mobile, and immersive devices" | Linked-spatial post, 2025-04-02 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)) | Absent. Search terms: `mobile` (0 hits), `desktop` (0 hits), `immersive` (1 hit: World.technology.webXR-immersive, a bare string at [API.yaml L319-321](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L319-L321) with description "WebXR session profile" but no device-adaptation mechanism). | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |
| 40 | "World-agnostic user identification and data authentication (e.g., SSO)" | Linked-spatial post, 2025-04-02 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)) | Absent. Search terms: `SSO` (0 hits), `authentication` (0 hits), `identity` (0 hits), `login` (0 hits) in all three API files. HEAD / on OpenSpatialAsset and OpenUserManifest returns 200/403/404 but defines no authentication mechanism. | [03 Portable User State and Identity](https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md) |
| 41 | "Ability to jump to predefined viewpoints in worlds" | Linked-spatial post, 2025-04-02 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)) | Partial. View schema exists with geoPose ([API.yaml L382-407](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L382-L407)), but no "jump" or navigation mechanism is defined. Search terms: `jump` (0 hits), `navigate` (0 hits), `teleport` (0 hits). | [06 Discovery and Addressing](https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md) |
| 42 | "Creation and sharing of new viewpoints" | Linked-spatial post, 2025-04-02 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)) | Absent. GET /wow/view/{viewId} exists but no POST or PUT for views. Search terms: `create.*view` (0 hits), no write operation on the view path. | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md) |
| 43 | "Security, e.g., protection against Man in the Middle attacks" | Linked-spatial post, 2025-04-02 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)) | Absent. Search terms: `security` (0 hits), `MITM` (0 hits), `TLS` (0 hits), `HTTPS` (0 hits in API.yaml). The OpenSpatialWorld test server URL uses `http://` not `https://` ([API.yaml L9](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L9)); the OpenSpatialAsset and OpenUserManifest server URLs use `https://` (L10 in each file). | [04 Provenance and Signed Subtrees](https://github.com/openspatials/msf-wow-completion-report/blob/main/04-provenance-and-signed-subtrees.md) |
| 44 | "Automatic user ID controlled join/rejoin management" | Linked-spatial post, 2025-04-02 ([post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/)) | Absent. The User schema exists but no session lifecycle, no join/rejoin protocol, no user-creation endpoint. Search terms: `rejoin` (0 hits), `session` (1 hit: "WebXR session profile", not session management). | [05 Presence, Live Sync, and Persistence](https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md) |

Classification: rows 36, 37, 38, 39, 40, 43 are aspirations. Rows 41, 42, 44 are named only.


## Governance (from working-group meetings)

Five governance structures were described at the 2026-08-24 working-group meeting. None has a formal document. The transcript is not public; the entries below paraphrase what was said and give the transcript line for the record.

| # | Declared | Source | In the text today | Our document |
|---|----------|--------|-------------------|--------------|
| G1 | An initiative hub, modelled on the Khronos structure, to manage external open source projects; the hub is to live in the Metaverse Standards Forum infrastructure and link to the projects | 2026-08-24 working-group meeting, transcript lines 109 and 605 | Absent. Search terms: `hub`, `initiative`, `governance` (0 hits each in all three API files and README.md). | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |
| G2 | A business council on the Forum side, brokering between developer communities and standards bodies; the meeting noted that nothing is written yet on how it is built | 2026-08-24 working-group meeting, transcript lines 141, 613, and 655 | Absent. Search terms: `business council`, `council`, `business` (0 hits each in all three API files and README.md). | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |
| G3 | An external maintainer council that manages the maintainers who work on the code and the APIs; its relationship to the hub was described as still open | 2026-08-24 working-group meeting, transcript lines 145-147 (cues 36-37) | Absent. Search terms: `maintainer`, `council` (0 hits each in all three API files and README.md). The term `external` has 1 hit in README.md (L27, a scene part linked in an external world, not a governance body) and appears as the OpenAPI key `externalDocs` at OpenSpatialAsset/API.yaml L13. The council is external to the Metaverse Standards Forum. | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |
| G4 | An open source licence with a contributor licence agreement (CLA), stated as the one already used for the API and the two open source implementations | 2026-08-24 working-group meeting, transcript line 157 (cue 39) | Absent. Search terms: `license` (2 hits: World.content.license at [API.yaml L278](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L278) and Asset.content.license at [OpenSpatialAsset/API.yaml L190](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml#L190), both content properties, not a project licence declaration), `CLA` (0 hits). Apache-2.0 appears in the repository LICENSE file but not in the specification text. | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |
| G5 | A point of contact that brokers between the developer communities and the standards development organizations (SDOs) | 2026-08-24 working-group meeting, transcript line 213 (cue 53) | Absent. Search terms: `SDO`, `liaison`, `broker` (0 hits each in all three API files and README.md). No liaison protocol defined. | [10 Role and Blind Spots](https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md) |

Classification: all five are named only (verbally declared, no written document).


## Where the publications and the text agree

The specification text and the publications agree in these areas, and the specification carries enough detail to build from:

- The three-API structure (OpenSpatialWorld, OpenSpatialAsset, OpenUserManifest) matches the three core pillars the announcement post names (Open Spatial World, Open User Manifest, Open Spatial Asset) and the whitepaper's core concepts of linked spatial experiences, shared spatial assets, and the shared user manifest. Each API exists with defined operations.
- The spatial composition graph has full CRUD: GET, POST, PUT, DELETE on nodes under a spatial root. The recursive Node schema with parent, children, localTransform, spatialAssetURI, and appearanceURI carries the structure the whitepaper names.
- The OpenSpatialAsset content-negotiation surface covers 21 IANA-registered model types, fulfilling the whitepaper's "Built on existing standards (e.g. gltf, x3d, usd )" declaration (quoted as written).
- The GeoPose structure appears on World, User, View, Portal, Spatial, and Asset, providing the spatial anchoring the architecture diagrams show.
- The ETag-based authorization on OpenSpatialAsset and OpenUserManifest (HEAD / returning 200/403/404) provides a starting point for the access control the publications describe.


## Where the text has not yet caught up with the publications

Seventeen declarations are named in the publications but not yet defined in the specification text. Twelve more are aspirations with no specification text at all.

Named-only items that could become specified with targeted additions:

- Portal destination: the portal has a position but cannot say where it leads.
- Fragment verbs (join, follow, preview): named in the README table, not defined in the API.
- User manifest scope: the API exists but carries three fields where the publications describe a full identity layer with DIDs, credentials, and wallets.
- Viewpoint navigation: views can be read but not created, and no "jump to viewpoint" mechanism exists.
- Session lifecycle: the README declares "continuous spatial experience and session" but no session lifecycle is defined.
- Governance: five structures were described in a working-group meeting; none has a formal document.

Aspirations that would require new specification work:

- The four vision principles (universality, interoperability, decentralization, accessibility) have no specification text of any kind.
- AI integration has no API surface.
- Multi-user/multi-device, SSO, experience consistency, security, and billions-scale addressing were declared in the 2025 blog post and have no specification text.


## Open questions for the working group

1. Which of the 17 named-only items does the working group intend to specify next? The portal destination and the fragment verb grammar are the smallest additions with the largest impact for implementers.
2. The linked-spatial-experiences post (2025) declares requirements that the whitepaper (2026) does not repeat: experience consistency, security, billions-scale addressing, multi-user/multi-device. Are these still active requirements?
3. The GitHub Pages implementations table now lists Open-Spatial-Lab (Apache-2.0, Level 5, gltf-binary) where HTMLModeWrapper previously appeared. What are the level definitions, and is the working group maintaining a compliance matrix?
4. The governance structures described on 2026-08-24 (initiative hub, business council, maintainer council, licence/CLA, SDO liaison) have no formal document. When is a governance document expected?


## Sources

### Publications (the reference set)

- Whitepaper: "initial Web of World Whitepaper", 2026-Q1. Page: https://webofworlds.github.io/initial_MSF_Whitepaper/ . PDF: https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf . Source: https://github.com/WebOfWorlds/initial_MSF_Whitepaper (HEAD 988f369b on 2026-09-07).
- MSF announcement: "Announcing the Web of Worlds whitepaper: a concrete path to the open metaverse", 2026-06-03. https://metaverse-standards.org/news/blog/announcing-the-web-of-worlds-whitepaper-a-concrete-path-to-the-open-metaverse/
- MSF post: "Linked spatial experiences: the Web of Worlds", 2025-04-02. https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/
- GitHub Pages home, dated 2026-03-31. https://webofworlds.github.io/
- Specification: https://github.com/WebOfWorlds/WoWAPI at commit d39a1a0 (2026-05-21), equal to upstream HEAD on 2026-09-07.
- Reference implementation (named only, not assessed): https://github.com/WebOfWorlds/simpleWorlds at commit 13d2cbe (HEAD d2bda3e on 2026-09-07).

### Specification files cited

- OpenSpatialWorld/API.yaml: https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml
- OpenSpatialWorld/README.md: https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md
- OpenSpatialAsset/API.yaml: https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml
- OpenUserManifest/API.yaml: https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml

### Our documents

- 01: https://github.com/openspatials/msf-wow-completion-report/blob/main/01-coordinate-precision-units-and-extents.md
- 02: https://github.com/openspatials/msf-wow-completion-report/blob/main/02-portal-destination-and-traversal.md
- 03: https://github.com/openspatials/msf-wow-completion-report/blob/main/03-portable-user-state-and-identity.md
- 04: https://github.com/openspatials/msf-wow-completion-report/blob/main/04-provenance-and-signed-subtrees.md
- 05: https://github.com/openspatials/msf-wow-completion-report/blob/main/05-presence-live-sync-and-persistence.md
- 06: https://github.com/openspatials/msf-wow-completion-report/blob/main/06-discovery-and-addressing.md
- 07: https://github.com/openspatials/msf-wow-completion-report/blob/main/07-assets-and-the-render-seam.md
- 08: https://github.com/openspatials/msf-wow-completion-report/blob/main/08-composition-graph-schema-fixes.md
- 09: https://github.com/openspatials/msf-wow-completion-report/blob/main/09-conformance-vocabulary-and-errata.md
- 10: https://github.com/openspatials/msf-wow-completion-report/blob/main/10-role-and-blind-spots.md

## Change log

2026-09-07: first public draft, verified twice.
