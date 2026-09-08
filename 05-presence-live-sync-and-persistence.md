# Presence, Live Sync, and Persistence

The pinned OpenSpatialWorld API and README define snapshot resources and graph operations, but do not bind a live event transport, session lifecycle or presence-registration procedure. Persistence and sharing each have a short README row. The whitepaper supplies broader architectural intent. A second implementation that tried to add multi-user presence or URL-based persistence would have to invent the same contracts Open Spatial Lab invented, with no guarantee of agreement.

Four proposed bindings would make the reviewed entry and live-state behavior more precise: a negotiated event transport and recovery contract, session/presence lifecycle semantics, alignment of the truncated Preview description with the published preview-and-authorization intent, and preservation of address components on persist/share. Event names alone are insufficient. These are unadopted candidates drawn from one implementation.

Portable inventory and preferences are separate extension questions. The reviewed API lacks those bindings, while the whitepaper describes portable user information and user-controlled disclosure. This does not establish that no other standard addresses them, or that they depend technically on Portal.destination adoption.

Status: Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).

## What the specification says today

**Session and presence.** The README introduces the API as providing an "Open continuous spatial experience and session for a given device and User-Agent/browser (UA)" ([README.md line 7](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L7)). The word "session" appears once more in API.yaml as the description of the `webXR-immersive` field: "WebXR session profile" ([API.yaml line 321](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L321)). Neither occurrence defines session lifecycle, session creation, session departure, or session state.

**Presence.** The World schema includes a `presence` object with three string fields: `avatar`, `navigation`, and `gravity` ([API.yaml lines 307-315](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L307-L315)). These are string-typed with no further description. There is no protocol, endpoint, or event model for knowing when a user joins, leaves, or moves. The `users` object on World carries `active_user_count` and `total_user_count` ([API.yaml lines 325-335](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L325-L335)), but these are static counters on a GET response, not a live feed.

**Real-time events.** Searching API.yaml and README.md for the terms `websocket`, `real-time`, `realtime`, `event`, and `subscribe` returns zero results across both files. The term `sse` appears only as a substring of unrelated words (`asset`, `AssetURI`), not as a reference to Server-Sent Events. The specification defines no mechanism for pushing updates from server to client.

**Join, follow, preview.** The Core Requirements table defines three entry verbs ([README.md lines 11-13](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L11-L13)):

- Join: "join the world as new or existing user on a given device and UA"
- Follow: "follow the world as new or existing user"
- Preview: "experence world without"

The pinned README description stops at "without". The [Linked Spatial Experiences: The Web of Worlds, April 2, 2025](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/) post supplies the missing context: preview creates no additional user, while user-based authorization is still needed. The README should carry that intent into its entry contract. Join and Follow both say "as new or existing user"; the preview rule distinguishes user creation from authorization.

**Follower visibility and departure.** The specification does not state whether a follower is visible to other users, whether a follower counts toward `active_user_count`, or what happens when the followed aspect leaves the world. Searching for `depart`, `departure`, `leave`, `visible`, and `follower` across API.yaml and README.md returns zero results.

**Bare #follow.** The Core Requirements table lists `URL#follow` alongside `URL#follow=aspect.id` ([README.md line 12](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L12)). The table provides no description of what `#follow` without an aspect.id means.

**Persist and share.** Two rows in the Core Requirements table ([README.md lines 14-15](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L14-L15)):

- Persist world: "store or bookmark URL"
- Share world: "send URL to second user"

Neither row states whether the URL must include its fragment, whether the fragment must round-trip, or whether a browser that strips the fragment on bookmark has broken the contract. Searching for `fragment`, `hash`, `round-trip`, and `restore` returns zero results.

**Inventory.** Searching API.yaml and README.md for `inventory`, `item`, `carry`, `equip`, and `asset` (excluding `spatialAssetURI` and `appearanceURI`) returns zero results for any portable-item vocabulary. No schema in the specification defines what a user can carry between worlds.

**Preferences.** Searching API.yaml and README.md for `preference`, `preferences`, `accessibility`, `setting`, and `settings` returns zero results. The User schema carries `id`, `name`, `AvatarURI`, and `geoPose`, with no field for user preferences.

## What fails without it

**Live updates lack an agreed binding.** The API supplies snapshots and graph operations. An implementation can add polling or an event service, but the reviewed contract does not define discovery, event transport, snapshot revision, ordering, replay or reconnect behavior. Independently chosen mechanisms need not converge on the same world state.

**Preview intent needs an API binding.** The 2025 post already separates preview from creation of an additional user and retains user-based authorization. The incomplete README does not bind that intent to presence counts, participation records or authorization failures. Clients should not infer that preview grants anonymous access to protected resources.

**Follower breakage on departure.** When a followed aspect leaves the world, a follower has no defined state to fall back to. One implementation may freeze the camera. Another may switch to a free camera. A third may eject the follower entirely. Without a rule, the follower's experience after departure depends on which server they connected to.

**Bare #follow has no meaning.** A URL with `#follow` and no aspect.id is valid according to the Core Requirements table but has no defined behavior. An implementation must decide whether this means "follow the world's default viewpoint," "follow the world's live state," or "error." Each choice produces a different user experience.

**Bookmarked URLs lose state.** A user who bookmarks `https://example.com/world/#follow=avatar-7` and later restores it expects to return to the same world, following the same aspect, in follow mode. If the specification does not require the fragment to survive, an implementation that strips it on persist or share would silently demote the user to a bare join.

**Inventory transfer is not bound here.** No interoperable carried-item contract was located in the reviewed WoW API. Other identity or item systems can carry data, but the group still needs to decide whether and how this API refers to them, including permissions, provenance and destination acceptance. This is a scoped binding gap, not a worldwide absence finding.

**Preference exchange is not bound here.** The whitepaper describes preferences/settings and selective disclosure. The pinned UserManifest.content schema has name, age and avatarAssetURI, without a preference vocabulary. A portable preference contract must also specify consent and what a destination supports; serialization alone does not ensure accessibility behavior.

## What Open Spatial Lab built and learned

**WebSocket /events channel.** Open Spatial Lab documents a labeled non-canonical WebSocket channel with `user_joined`, `user_left`, `node_created`, `node_updated` and `node_deleted`. This is a local convention, not an interoperable transport/recovery profile. No fresh multi-user network run or inspection of another implementation's internals is claimed here.

**Presence registration by intent (Interpretation I5).** OSL implemented a rule: join and follow register presence (create a session and a user record); preview does not. The reasoning is documented in source code comments (verified: `wow-url.mjs` lines 221-254). For preview, OSL uses no participation session, presence registration or visibility to other users. That local interpretation agrees with the 2025 post's no-additional-user intent. The cited entry parser does not implement the post's user-based authorization requirement; authorization remains a separate binding. For follow, OSL relied on the fact that the Follow row says "as new or existing user," the same words as Join, concluding that follow is a presence-registered mode with an attached viewpoint rather than a subscribe-only mode.

**Follower visibility and departure fallback.** OSL registers a follower as an ordinary present user (visible to others) and falls back to a free camera if the followed aspect leaves the world (verified: `wow-url.mjs` lines 247-249). This is a gap-filling decision, not a spec-stated rule.

**Bare #follow follows the world (Interpretation I6).** OSL interpreted bare `#follow` (no aspect.id) as "follow the world itself": register presence and track the world's live state with a free camera, flagged as following so a follow-target can be attached later without a reload (verified: `wow-url.mjs` lines 256-268). The spec's Core Requirements table titles this row "Follow world," and the aspect form is the optional variant.

**Persist/share URLs carry the fragment (Interpretation I8).** OSL's `buildWorldUrl` function serializes the full URL including the fragment, so that bookmarking or sharing returns the user to the same world, aspect, and mode (verified: `wow-url.mjs` lines 437-461). A bare join emits the clean URL with no fragment, since the spec lists bare `URL` as the Join feature. This is a labeled interpretation: the spec says "store or bookmark URL" and "send URL to second user," and OSL reads "URL" as including the fragment because a URL includes its fragment by definition (RFC 3986).

**Claim boundary.** These are local interpretations and implementation facts. The source server removes an identified player at accepted exit-intent, before visual crossing; its five-second tombstone blocks immediate stale-heartbeat upserts. The client's later depart call is a confirmation, followed by destination registration. The client-only failure probe omitted the source exit-intent handler and is not a two-server crossing result. Lost requests, lost replies, late heartbeats and lease expiry have distinct effects, described in [chapter 02](02-portal-destination-and-traversal.md). Session authority, visual continuity and per-world occupancy remain separate.

## Proposed normative text

All additions are unadopted proposals. Schema fragments target OpenAPI 3.0.4; the session/options mappings below are explanatory data sketches, not OpenAPI Schema Objects. The group must select a coherent profile before these words become requirements.

### Real-time event channel

A future live-state profile should provide endpoint discovery and at least one common mandatory transport binding, or negotiation that selects a mutually supported binding. WebSocket and Server-Sent Events are candidates; independently allowing either one does not ensure two implementations can connect.

Rationale: a live-state profile needs defined update behavior alongside the existing snapshot API.

At minimum, the following event types SHOULD be defined:

```yaml
# Event types for real-time presence and node lifecycle
EventType:
  type: string
  enum:
    - user_joined
    - user_left
    - node_created
    - node_updated
    - node_deleted
```

The vocabulary above is only a starting set. A profile must additionally define event payloads, world/session scope, authentication and authorization, event/revision identifiers, snapshot-to-stream ordering, duplicate handling, resume/expiry behavior and resynchronization when history is unavailable. Agree one dropped-event, duplicate-event and reconnect fixture before calling the channel interoperable.

Rationale: a shared name does not establish whether an event is new, duplicated or already reflected in the loaded snapshot. Those observable behaviors determine whether clients converge after failure.

### Session lifecycle semantics

Candidate lifecycle semantics follow. In this sketch, session means a participation/presence record; preview can still require an authenticated transport or private-resource authorization. The group must define departure acknowledgement, lease expiry, retry and recovery separately. These entries do not specify a distributed ownership transaction.

```yaml
# Session lifecycle states
SessionLifecycle:
  join:
    creates_session: true
    registers_presence: true
    description: >
      The user enters the world as a new or existing participant.
      A session is created and the user is visible to other users.
  follow:
    creates_session: true
    registers_presence: true
    description: >
      The user enters the world as a new or existing participant
      with their viewpoint attached to a target aspect.
      The user is visible to other users.
  preview:
    creates_session: false
    registers_presence: false
    description: >
      No additional user or participation session is created.
      The visitor is not counted in active_user_count.
      User-based authorization still applies to protected access;
      an authenticated access session is separate from presence.
  depart:
    description: >
      The user leaves the world. The session is ended and the user
      is removed from presence.
```

Rationale: without these definitions, two implementations will disagree on whether a preview visitor is visible, which breaks the most basic interop test (user count).

Proposed correction: complete the Preview description using the 2025 post's no-additional-user and authorization distinction. Define how that maps to presence counts and access-denied responses.

Rationale: the published intent exists; the README and observable entry behavior should agree with it.

### Follower rules

The specification SHOULD state that a follower is visible to other users (presence-registered).

Rationale: the Follow row says "as new or existing user," which implies presence.

The specification MUST define the behavior when the followed aspect leaves the world or becomes unavailable. Options for the working group:

```yaml
# Follower departure behavior (group must pick one)
FollowerDepartureBehavior:
  option_a: "Revert to free camera in the same world"
  option_b: "Eject the follower from the world (trigger depart)"
  option_c: "Freeze the follower's last viewpoint until a new target is available"
```

Rationale: without a rule, a follower whose target departs enters an undefined state.

### Bare #follow

The specification SHOULD define the behavior of `URL#follow` (no aspect.id). Options for the working group:

```yaml
# Bare #follow behavior (group must pick one)
BareFollowBehavior:
  option_a: "Follow the world's live state with a free camera (join with a follow flag)"
  option_b: "Follow the world's default viewpoint"
  option_c: "Treat as an error; #follow requires an aspect.id"
```

Rationale: the Core Requirements table lists `URL#follow` as valid but does not define its meaning.

### Persist and share fragment rule

Proposed rule: preserve the complete resolved entry URL, including query and fragment semantics. Restoration should re-enter the encoded world, aspect and mode when still available; an expired or missing aspect requires a defined failure/fallback rule. A bookmark does not freeze mutable world content.

Rationale: a URL that loses its fragment loses the user's mode and target aspect.

```yaml
# Fragment round-trip rule
PersistShareRule:
  description: >
    A persisted (bookmarked) or shared URL MUST include the
    URL fragment. Restoring or opening the URL requests the
    encoded world, aspect and mode; unavailable targets use
    the profile's explicit resolution/failure behavior. A bare URL (no fragment) is equivalent to #join.
```

Rationale: without this rule, "store or bookmark URL" could be satisfied by an implementation that strips the fragment.

### Portable inventory (optional extension)

This is an optional extension track, not a core requirement.

The specification SHOULD define a portable-inventory schema or evaluate a named, versioned Universal Manifest equipped-items profile by reference.

```yaml
# Portable inventory item (optional extension)
InventoryItem:
  type: object
  properties:
    item_id:
      type: string
      description: "Unique identifier for the item"
    asset_uri:
      type: string
      format: uri
      description: "URI of the item's asset"
    licence:
      type: string
      description: "Licence under which the item may be used"
    provenance:
      type: string
      format: uri
      description: "Origin world or authority that issued the item"
```

Rationale: a shared item reference can support transfer, but receiving permission and continued use require the chosen rights/admission contract. Inventory exchange can be developed independently of portal traversal.

### Portable preferences (optional extension)

This is an optional extension track, not a core requirement.

The specification SHOULD define a preference vocabulary on the user manifest.

```yaml
# User preferences (optional extension, minimum set)
UserPreferences:
  type: object
  properties:
    accessibility:
      type: object
      properties:
        text_scale:
          type: number
          minimum: 0
          exclusiveMinimum: true
          description: "Multiplier for default text size"
        reduced_motion:
          type: boolean
          description: "User prefers reduced motion"
        high_contrast:
          type: boolean
          description: "User prefers high contrast"
    input:
      type: object
      properties:
        dominant_hand:
          type: string
          enum: [left, right]
    rendering_quality:
      type: string
      enum: [low, medium, high]
      description: "Preferred rendering quality tier"
```

Rationale: the whitepaper describes portable preferences; the pinned manifest schema does not bind that vocabulary.

## Adoption path

**Existing implementations.** These proposals do not change the base endpoint schemas. A chosen live-state or entry profile adds behavior that existing implementations may not support; it must be advertised and tested. Serving selected routes alone is not a new claim of full conformance.

**Clients adopting a future profile.** Negotiate a supported live-state binding, establish snapshot/revision state, then apply events under its replay/resync rules. Preserve entry URL components on persist/share. Apply the agreed preview participation rule without bypassing resource authorization.

**Servers adopting a future profile.** Advertise transport and recovery capabilities, implement the agreed join/follow/preview behavior, and define acknowledgement and expiry of presence. Do not imply that a successful destination registration proves the source record was removed.

## Open questions for the working group

1. **Event transport.** Should the standard mandate WebSocket, SSE, or leave it open? Mandating one simplifies interop testing. Alternatives require explicit common support or negotiation plus an unsupported-binding outcome; transport names alone are insufficient.

2. **Preview binding.** How should the 2025 post's no-additional-user and user-based-authorization intent map to presence counts, access sessions and failure responses in the API?

3. **Follower departure.** Should a follower revert to a free camera, be ejected, or freeze? OSL chose revert-to-free-camera. The group should pick one default and state it.

4. **Bare #follow.** Should bare `#follow` follow the world, follow a default viewpoint, or be an error? OSL chose follow-the-world (join with a follow flag).

5. **Inventory scope.** Should inventory be a core feature or an optional extension track? The data and permission contract can be developed independently of Portal.destination; portal transfer is one consumer.

6. **Preference scope.** At minimum, accessibility preferences (per WCAG) should be portable. Should input preferences and rendering quality also be included, or should those wait for implementation experience?

7. **Session lifecycle and TeleportXR.** TeleportXR's session vocabulary (connect with an opaque identity string) is a useful reference for the live-session contract. Should WoW define session semantics and reference TeleportXR's vocabulary for the transport layer, or should WoW define both?

## Sources

- [Linked Spatial Experiences: The Web of Worlds, April 2, 2025](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/): published units, preview, authorization and aspect intent.

- [OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialWorld/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenUserManifest/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf); relevant printed pages are identified in this chapter or [chapter 10](10-role-and-blind-spots.md).
- [RFC 3986, URI syntax and resolution](https://www.rfc-editor.org/rfc/rfc3986).
- Open Spatial Lab local source snapshot and retained evidence, checked September 7, 2026: OSL-WOW-CONTRACT.md and wow-url.mjs, including entry interpretations and URL serialization. The source exit-intent removal and finite tombstone were checked in runtime-state.js. The retained September 7 in-memory failure probe uses only the client presence controller and bypasses that source handler; it is not a two-server result or proof of exclusive session authority. Public reproduction of these exact local bytes is not established.
- [Appendix A](A-completion-map.md) and [Appendix B](B-findings-register.md) preserve the historical surface/finding identifiers.

## Change log

- 2026-09-07: corrected source scope, proposal compatibility and evidence boundaries; updated public citations.
