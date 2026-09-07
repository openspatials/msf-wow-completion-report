# Presence, Live Sync, and Persistence

The Web of Worlds specification defines a REST API that can describe a world as a snapshot, but it provides no wire contract for real-time events, no session lifecycle semantics, no rules for when a visitor registers presence, and only two one-line rows for persistence and sharing. A second implementation that tried to add multi-user presence or URL-based persistence would have to invent the same contracts Open Spatial Lab invented, with no guarantee of agreement.

The standard needs four additions (each detailed below): a real-time event channel for presence and node changes (inferred from implementation); session lifecycle semantics for the join, follow, and preview verbs (verified in code); completion of the truncated Preview description that governs whether preview visitors create server-side state (verified against the spec text); and a rule that persisted and shared URLs carry their fragment so the user returns to the same world, aspect, and mode (verified in code).

Two further gaps sit at the persistence layer: portable inventory (a user who crosses a portal cannot carry items; inferred from the architecture map) and portable preferences (accessibility settings do not travel between worlds; reported from whitepaper-vs-schema comparison). Both are blind spots with no covering standard anywhere in the standards landscape.

Status: Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).

## What the specification says today

**Session and presence.** The README introduces the API as providing an "Open continuous spatial experience and session for a given device and User-Agent/browser (UA)" ([README.md line 7](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L7)). The word "session" appears once more in API.yaml as the description of the `webXR-immersive` field: "WebXR session profile" ([API.yaml line 321](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L321)). Neither occurrence defines session lifecycle, session creation, session departure, or session state.

**Presence.** The World schema includes a `presence` object with three string fields: `avatar`, `navigation`, and `gravity` ([API.yaml lines 307-315](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L307-L315)). These are string-typed with no further description. There is no protocol, endpoint, or event model for knowing when a user joins, leaves, or moves. The `users` object on World carries `active_user_count` and `total_user_count` ([API.yaml lines 325-335](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L325-L335)), but these are static counters on a GET response, not a live feed.

**Real-time events.** Searching API.yaml and README.md for the terms `websocket`, `real-time`, `realtime`, `event`, and `subscribe` returns zero results across both files. The term `sse` appears only as a substring of unrelated words (`asset`, `AssetURI`), not as a reference to Server-Sent Events. The specification defines no mechanism for pushing updates from server to client.

**Join, follow, preview.** The Core Requirements table defines three entry verbs ([README.md lines 11-13](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L11-L13)):

- Join: "join the world as new or existing user on a given device and UA"
- Follow: "follow the world as new or existing user"
- Preview: "experence world without"

The Preview description is truncated. The sentence ends at "without" with no object. No further text in the repository completes it. Join and Follow both say "as new or existing user"; Preview does not.

**Follower visibility and departure.** The specification does not state whether a follower is visible to other users, whether a follower counts toward `active_user_count`, or what happens when the followed aspect leaves the world. Searching for `depart`, `departure`, `leave`, `visible`, and `follower` across API.yaml and README.md returns zero results.

**Bare #follow.** The Core Requirements table lists `URL#follow` alongside `URL#follow=aspect.id` ([README.md line 12](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L12)). The table provides no description of what `#follow` without an aspect.id means.

**Persist and share.** Two rows in the Core Requirements table ([README.md lines 14-15](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md#L14-L15)):

- Persist world: "store or bookmark URL"
- Share world: "send URL to second user"

Neither row states whether the URL must include its fragment, whether the fragment must round-trip, or whether a browser that strips the fragment on bookmark has broken the contract. Searching for `fragment`, `hash`, `round-trip`, and `restore` returns zero results.

**Inventory.** Searching API.yaml and README.md for `inventory`, `item`, `carry`, `equip`, and `asset` (excluding `spatialAssetURI` and `appearanceURI`) returns zero results for any portable-item vocabulary. No schema in the specification defines what a user can carry between worlds.

**Preferences.** Searching API.yaml and README.md for `preference`, `preferences`, `accessibility`, `setting`, and `settings` returns zero results. The User schema carries `id`, `name`, `AvatarURI`, and `geoPose`, with no field for user preferences.

## What fails without it

**No live world.** An implementer who builds a multi-user spatial experience on the current API gets a REST endpoint that returns a snapshot. When a second user joins, the first user's client has no way to learn about it except by polling GET /wow/world. Node creation, movement, and deletion are likewise invisible until the next poll. The API describes a world; it does not describe a world that is alive.

**Two implementations disagree on preview visibility.** The Preview description is truncated at "experence world without." One implementation may read "without" as "without joining" (no session, no presence). Another may read it as "without cost" or "without full features" and register the preview visitor as present. A preview visitor who appears in one server's user count but not another's breaks interoperability at the most basic level: two clients connected to the same world would disagree on how many people are in it.

**Follower breakage on departure.** When a followed aspect leaves the world, a follower has no defined state to fall back to. One implementation may freeze the camera. Another may switch to a free camera. A third may eject the follower entirely. Without a rule, the follower's experience after departure depends on which server they connected to.

**Bare #follow has no meaning.** A URL with `#follow` and no aspect.id is valid according to the Core Requirements table but has no defined behavior. An implementation must decide whether this means "follow the world's default viewpoint," "follow the world's live state," or "error." Each choice produces a different user experience.

**Bookmarked URLs lose state.** A user who bookmarks `https://example.com/world/#follow=avatar-7` and later restores it expects to return to the same world, following the same aspect, in follow mode. If the specification does not require the fragment to survive, an implementation that strips it on persist or share would silently demote the user to a bare join.

**Portable inventory is absent.** A user who acquires an item in World A and walks through a portal to World B cannot bring the item. The IWPS portal protocol reserves an `assets` parameter but defines no schema for it. Without a portable-inventory contract, every world is a walled garden for carried items.

**Preferences do not travel.** A user who sets accessibility preferences (text size, color contrast, motion reduction) in one world loses them in the next. The WoW whitepaper's "Digital YOU" section claims "Preferences & settings," but the OpenUserManifest schema carries only `name`, `age`, and `avatarAssetURI` (reported: whitepaper-vs-schema comparison from R3 analysis).

## What Open Spatial Lab built and learned

**WebSocket /events channel.** OSL implemented a WebSocket endpoint at `/events` that emits labeled events: `user_joined`, `user_left`, `node_created`, `node_updated`, `node_deleted`. This channel is documented in OSL's contract specification as a "labeled non-canonical realtime convention" and is kept out of the OpenAPI HTTP contract on purpose (verified: `repo/open-spatial-lab/wow-spec/OSL-WOW-CONTRACT.md` lines 127-133). The upstream `simpleWorlds` reference implementation registers the WebSocket handler before its validator, so socket messages are unvalidated; OSL follows the same pattern, implementing realtime as a labeled convention (WO-015). This is a labeled extension, not a canonical feature.

**Presence registration by intent (Interpretation I5).** OSL implemented a rule: join and follow register presence (create a session and a user record); preview does not. The reasoning is documented in source code comments (verified: `repo/open-spatial-lab/web/wow-url.mjs` lines 221-254). For preview, OSL completed the truncated spec sentence as "experience world without joining it," meaning no session, no presence registration, no visibility to other users. OSL states plainly that the words "without joining" are its own, not the specification's. For follow, OSL relied on the fact that the Follow row says "as new or existing user," the same words as Join, concluding that follow is a presence-registered mode with an attached viewpoint rather than a subscribe-only mode.

**Follower visibility and departure fallback.** OSL registers a follower as an ordinary present user (visible to others) and falls back to a free camera if the followed aspect leaves the world (verified: `repo/open-spatial-lab/web/wow-url.mjs` lines 247-249). This is a gap-filling decision, not a spec-stated rule.

**Bare #follow follows the world (Interpretation I6).** OSL interpreted bare `#follow` (no aspect.id) as "follow the world itself": register presence and track the world's live state with a free camera, flagged as following so a follow-target can be attached later without a reload (verified: `repo/open-spatial-lab/web/wow-url.mjs` lines 256-268). The spec's Core Requirements table titles this row "Follow world," and the aspect form is the optional variant.

**Persist/share URLs carry the fragment (Interpretation I8).** OSL's `buildWorldUrl` function serializes the full URL including the fragment, so that bookmarking or sharing returns the user to the same world, aspect, and mode (verified: `repo/open-spatial-lab/web/wow-url.mjs` lines 437-461). A bare join emits the clean URL with no fragment, since the spec lists bare `URL` as the Join feature. This is a labeled interpretation: the spec says "store or bookmark URL" and "send URL to second user," and OSL reads "URL" as including the fragment because a URL includes its fragment by definition (RFC 3986).

**Claim boundary.** All of the above are single-implementation readings of a specification that does not address these topics. None proves that the readings are the only valid ones, and none proves interoperability with a second implementation that made different choices.

## Proposed normative text

### Real-time event channel

A conformant server SHOULD provide a WebSocket or Server-Sent Events endpoint for real-time events.

Rationale: the REST API describes snapshots; a spatial world needs live updates.

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

The specification SHOULD leave the transport choice (WebSocket vs SSE) to the implementer but MUST define the event-type vocabulary so that a client from one implementation can parse events from another.

Rationale: interoperability requires a shared vocabulary; transport flexibility allows servers to match their infrastructure.

### Session lifecycle semantics

The specification SHOULD define explicit state transitions for the three entry verbs:

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
      The user experiences the world without joining it.
      No session is created. The user is not visible to other users
      and is not counted in active_user_count.
  depart:
    description: >
      The user leaves the world. The session is ended and the user
      is removed from presence.
```

Rationale: without these definitions, two implementations will disagree on whether a preview visitor is visible, which breaks the most basic interop test (user count).

The specification MUST complete the Preview description. The current text "experence world without" is truncated and does not state its object.

Rationale: a truncated sentence in a normative table forces every implementer to guess its meaning.

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

The specification SHOULD state that a persisted or shared URL MUST include the fragment.

Rationale: a URL that loses its fragment loses the user's mode and target aspect.

```yaml
# Fragment round-trip rule
PersistShareRule:
  description: >
    A persisted (bookmarked) or shared URL MUST include the
    URL fragment. Restoring or opening the URL MUST return the
    user to the same world, aspect, and mode encoded in the
    fragment. A bare URL (no fragment) is equivalent to #join.
```

Rationale: without this rule, "store or bookmark URL" could be satisfied by an implementation that strips the fragment.

### Portable inventory (optional extension)

This is an optional extension track, not a core requirement.

The specification SHOULD define a portable-inventory schema or adopt one by reference from Universal Manifest's equipped-items vocabulary.

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

Rationale: without an inventory contract, items cannot cross portal boundaries.

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

Rationale: the whitepaper promises portable preferences but the manifest schema does not carry them.

## Adoption path

**What stays valid for a minimal world.** A world that serves only the current REST API (GET /wow/world, GET /wow/user/{id}, and the spatial graph endpoints) remains a valid world. The real-time channel, session lifecycle semantics, and fragment rules are additions, not changes to existing endpoints.

**What a client must do.** A client that wants live presence SHOULD connect to the server's event channel (WebSocket or SSE) after loading the world via REST. A client that bookmarks or shares a URL SHOULD preserve the fragment. A client that opens a `#preview` URL SHOULD NOT register presence or create a session.

**What a server must do.** A server that supports multi-user presence SHOULD implement the event channel with the defined event-type vocabulary. A server SHOULD accept and honor the session lifecycle semantics (join creates presence, preview does not). A server that serves the Core Requirements MUST complete the Preview description and MUST define follower departure behavior.

## Open questions for the working group

1. **Event transport.** Should the standard mandate WebSocket, SSE, or leave it open? Mandating one simplifies interop testing. Leaving it open gives servers flexibility but requires clients to support both.

2. **Preview sentence.** The working group must complete the truncated Preview description ("experence world without"). The reading "without joining" is the most natural completion, but the group must confirm it.

3. **Follower departure.** Should a follower revert to a free camera, be ejected, or freeze? OSL chose revert-to-free-camera. The group should pick one default and state it.

4. **Bare #follow.** Should bare `#follow` follow the world, follow a default viewpoint, or be an error? OSL chose follow-the-world (join with a follow flag).

5. **Inventory scope.** Should inventory be a core feature or an optional extension track? It depends on Portal.destination landing first (a portal without a destination field cannot carry inventory across it).

6. **Preference scope.** At minimum, accessibility preferences (per WCAG) should be portable. Should input preferences and rendering quality also be included, or should those wait for implementation experience?

7. **Session lifecycle and TeleportXR.** TeleportXR's session vocabulary (connect with an opaque identity string) is a useful reference for the live-session contract. Should WoW define session semantics and reference TeleportXR's vocabulary for the transport layer, or should WoW define both?

## Sources

- WebOfWorlds/WoWAPI specification at commit d39a1a0: [API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), [README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md)
- Open Spatial Lab contract specification: `repo/open-spatial-lab/wow-spec/OSL-WOW-CONTRACT.md` lines 127-133
- Open Spatial Lab URL handling and interpretations: `repo/open-spatial-lab/web/wow-url.mjs` lines 221-268, 437-461
- Completion Map rows CM-032, CM-033, CM-034, CM-035, CM-055
- Findings rows R-003, R-004, R-012, R-024
- Findings and Recommendations document rows 3, 4, 12, 24 (blind spots and known gaps)
- RFC 3986 (Uniform Resource Identifier): defines that a URI includes its fragment component
- Proof ledger: `.dev/ai/roles/project-steward/proof-ledger.md` (48/48 crossing-continuity and 55/55 signed-subtree counts are documented by OSL, not re-run in this pass; medium confidence until re-run)

## Change log

- 2026-09-07: first public draft, verified.
