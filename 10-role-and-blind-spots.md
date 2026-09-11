# Web of Worlds among its neighbors: role and open questions

WoWAPI 0.0.1 defines resources, composition-graph operations and asset negotiation. Its whitepaper describes units and origins, internal and external node references, portable user information, human and AI access, and selective feature implementation. The local implementation exposed incomplete bindings for composition, client behavior, shared state and world transitions. This report connects those findings to the whole architecture and proposes contracts and tests for ongoing operation as well as traversal.

The questions below concern coordinate agreement, shared spatial anchors, portable inventory, preferences, software-agent declarations and governance. They do not establish that no standard addresses these subjects, or that Web of Worlds should own all of them. The selected infrastructure map contains 102 rows across six reviewed subjects. Those rows describe that corpus; their empty cells are neither worldwide absence proofs nor measures of standards completion.

**Source boundary:** WoWAPI commit d39a1a0, rechecked September 11, 2026, and the March 31, 2026 whitepaper. The technical chapters distinguish published architecture, API bindings, local implementation evidence and unadopted proposals. [Chapter 11](11-published-positions-and-current-state.md) gives the publication comparison; [Appendix A](A-completion-map.md) and [Appendix B](B-findings-register.md) preserve historical row identifiers.

## The whole architecture and its connections

The system must support places and objects that stay where they are, people and software that act on them, and navigation between contexts. Traversal is one operation within that system. The following map describes required responsibilities and proposed binding work; it is not a claim that a complete open metaverse has been implemented or that every concern belongs in the base Web of Worlds API.

| Area | Responsibility | Connection to the rest of the system |
|---|---|---|
| Worlds and distributed entities | Identify graphs, objects, instances and referenced content | Discovery resolves them; placement relates their frames; clients load them; authority controls their changes. |
| Clients and behavior | Render supported content, interpret input, request actions and reflect updates | Content profiles define meaning; state services accept changes; permissions bound execution and access. |
| People and software actors | Present identity/evidence, preferences and requested capabilities | Manifest and authorization profiles connect the actor to resource access, action decisions and revocation. |
| Authority, live state and storage | Decide accepted changes, distribute revisions and preserve promised state | Clients submit actions and resynchronize; durable stores define recovery; remote inclusion preserves each service's responsibility. |
| Cross-world connections | Resolve entry, include remote content or navigate to another context | Addressing, graph identity, frames, portable data and participation must agree at each boundary. |
| Conformance and coordination | Name profiles, versions, owners and observable outcomes | Independent implementations run shared cases for ongoing operation and traversal. |

Three flows connect the areas. **Resolve → place → present** brings a stationary entity into a client's view. **Input → authorize → apply → publish** changes shared state while everyone remains in that world. **Persist → reconnect → restore** establishes what survives interruption. A portal transition combines these flows with destination admission and a change of participation context. Treating the transition as the whole system would leave the ordinary operating loop unexplained.

The report's ten technical subject chapters cover these areas from different angles. The chapter order is a reference structure, not a requirement that every client perform every step or that a stationary object undergo traversal. [Appendix A](A-completion-map.md) retains the earlier selected surface inventory, and [Appendix B](B-findings-register.md) retains the earlier findings. Their historical counts are not a completeness test for the architecture.

### What Web of Worlds supports, should bind and should reference

**Supports today:** the published world/resource architecture, graph and node operations, asset negotiation, user-manifest resource access and entry intentions. The whitepaper also describes broader composition, human/AI and selective-implementation goals. These are different evidence classes: published intent is not necessarily a complete wire binding.

**Should bind for a selected profile:** world and object identity, resource and service resolution, frame conventions, required behavior, versioned cross-layer references, compatibility and observable failure. The group should define which parts a participant must implement and how unsupported parts are reported. These responsibilities apply to the stationary operating loop as well as navigation.

**Should reference or delegate:** content formats and renderers, identity/credential mechanisms, authorization and policy systems, live-session transports, storage implementation and specialist localization. Referencing a system does not remove the need to specify the handoff to it. It establishes which specification owns each meaning and how the chosen profile uses that meaning.

### Specialist interfaces and accountable owners

| Interface to settle | Work supplied by the specialist | Binding and decision for this group |
|---|---|---|
| Portable information | A selected Universal Manifest version and its schema/verification rules | Define a visitor/actor profile, the relationship to OpenUserManifest access, disclosure and compatibility; adoption remains proposed. |
| Identity and authorization | Identifier methods, credential trust, delegated permissions and revocation | Identify the required assurance, resource/action scope and denied/expired outcome; a declaration is not a grant. |
| Spatial content and client host | A content format, behavior model and executable-host contract | State supported capabilities, representation negotiation, lifecycle, fallback and observable meaning. |
| Live state and sessions | Snapshot/update, input, ordering and recovery facilities | Name the authority, profile/version, discovery and access binding; compare stationary shared-state outcomes. |
| Durable storage | A service's write and restart guarantees | State what an acknowledgement promises and how clients retrieve that result; no database choice is imposed. |
| Earth placement and shared anchors | GeoPose semantics or a selected localization service | Bind local frames, precision, anchor namespace, permission and lifetime; a local pose is not shared localization. |
| Portal coordination | A selected transition specification, such as OMA3 IWPS | Bind destination resolution, admission, portable references and interrupted outcomes without assuming compliance. |
| Conformance and maintenance | Versioned requirements, fixtures and independent implementers | Agree scope, acceptance evidence, compatibility and the responsible standards/maintainer contacts. |

The ecosystem corpus is useful because it separates roles that can otherwise all be called “servers”: a signed-resource host, a manifest resolver, a mutable scene service and a live-session runtime. Likewise, the browser/composition engine and the rendering backend are not the same role. Their evidence is unequal. Historical RP1/Artemis receipts exercise named paths; much of the neighboring runtime material is a documentation read; Universal Manifest's equipped-item mapping remains a profile proposal. None of those sources establishes that the whole combination interoperates.

### People and software in ongoing operation

Human and software actors both need discoverable actions and explicit outcomes. Their declarations can differ, but neither receives authority merely by appearing in a world. A software actor may act for an operator under a limited grant. The profile must bind who issued the grant, which resources/actions it covers, when it expires, how it is revoked, and whether the current request is made under that grant. An advertised capability such as navigation is neither a grant nor a proof of competence.

Apply the same distinctions during continued operation: read an object, request a mutation, subscribe to restricted state, execute a subtree and leave a session are separate operations. Permission can change while the actor remains present. The state authority must enforce the current grant, and the client must receive a defined refusal or revocation outcome. A signed manifest and a fresh holder challenge can supply evidence without deciding that policy.

The proposed workshop test uses one human-controlled client and one software actor, each requesting an operation on the same stationary object. An allowed action changes its accepted revision. An out-of-scope action and an action after revocation do not. Both clients recover the promised durable state after reconnecting. This tests the human/AI intent against observable behavior and complements the avatar, hat and hammer crossing case. [Chapter 05](05-presence-live-sync-and-persistence.md#ongoing-operation-actions-authority-and-durable-state) provides the state contract, [chapter 03](03-portable-user-state-and-identity.md#permission-lifetime-during-ongoing-operation) the evidence/grant distinction, and [chapter 09](09-conformance-vocabulary-and-errata.md#behavioral-coverage-beyond-portal-traversal) the combined acceptance matrix.

## What the sources define

**Resources and operations.** The three API documents define eight named resource schemas and sixteen operations: eleven in OpenSpatialWorld, three in OpenSpatialAsset and two in OpenUserManifest. OpenSpatialWorld already provides individual-node GET and PUT. Its six resource schemas accept empty objects but reject some wrong types. Open objects permit extra fields. These are real capabilities and constraints, even though useful behavioral conformance needs a more specific profile.

**Coordinates.** Whitepaper page 7 discusses units and origins. The pinned World schema does not bind local units, axes, handedness or extent; Node.localTransform is a number array without a length or matrix convention. OGC GeoPose Basic YPR fixes WGS-84, East-North-Up and ellipsoidal height; it is not a configurable reference-frame field. Local scene conversion still needs an explicit mapping. [Chapter 01](01-coordinate-precision-units-and-extents.md) proposes that agreement without prescribing a renderer.

**Composition and delivery.** Whitepaper page 22 describes internal and external node references, and page 28 uses query/fragment delivery examples. The YAML and README provide graph and entry routes, but leave important reference-resolution and alternate-representation bindings open. Flat storage can serve embedded serialization; a reference form is an optional wire proposal. [Chapter 06](06-discovery-and-addressing.md) separates entry URLs from service bases, and [chapter 08](08-composition-graph-schema-fixes.md) preserves canonical node access.

**Portable information and assurance.** Whitepaper page 15 describes user-controlled disclosure and public/private manifest information. OpenUserManifest defines name, age and avatarAssetURI, plus resource-access authorization and ETag behavior. A signature profile can protect bytes, but issuer trust, holder control and destination admission are separate. [Chapter 03](03-portable-user-state-and-identity.md) keeps those distinctions explicit.

**Software agents and implementation scope.** Whitepaper page 25 describes common human/AI access and spatial operators; page 26 describes selective feature implementation and future levels. A dedicated agent-declaration, delegation or cross-world permission binding was not located in the pinned APIs. These published aims must not be described as concepts the initiative never considered.

**Governance.** Whitepaper page 30 describes an X3D profile and standards-body adoption path. That is relevant organizational context. A current charter, contribution policy or liaison document should be located in the initiative's authoritative sources before asserting an organizational gap.

## Questions exposed by this review

**Coordinate agreement.** Different units, origins and axes require a mapping; its absence can cause incorrect scale or orientation. Uniform shrinking does not recover detail lost when large absolute values are first cast to float32. Extent may aid loading and bounds; it is not a universal precision prerequisite. A profile should specify frame transforms and an error budget, leaving camera-relative coordinates, origin rebasing, normalization and other numerical choices to implementers.

**Shared spatial anchors.** Two devices need a common localization result to align content in the same physical room. No complete shared-anchor discovery, localization, permissions and lifetime binding was located in the reviewed WoW sources. The WebXR Anchors Module exposes tracked anchor spaces; that alone is not a protocol for sharing and resolving the same anchor between devices. This is a scoped question, not a claim that the problem is globally unaddressed or outside all web capabilities.

**Portable inventory.** No carried-item exchange binding was located in the pinned WoW APIs. A useful contract needs asset references, rights/provenance, consent and destination acceptance. Item transfer can be developed independently of Portal.destination; crossing is one consumer. The review did not establish that every world prevents users from carrying items.

**User preferences.** The whitepaper names preferences/settings, while the pinned manifest schema does not provide a preference vocabulary. A receiving world must define supported settings and unsupported-request behavior. Data exchange alone does not prove accessible behavior. Personal information also needs a disclosure policy.

**Software-agent declarations.** A destination may need to distinguish a declared software actor, its operator and requested actions. A self-declared type or capability list is not verified identity or permission. The group should decide whether this API refers to a delegated authorization profile or only advertises descriptive metadata.

**Governance evidence.** Contributors need usable pointers to ownership, licensing, contribution and decision rules. The next step is a scoped document lookup or a pointer from the initiative. No worldwide absence, comparative discussion-frequency claim or unresolved license blocker follows from the selected map or transcript keywords.

## What Open Spatial Lab built and learned

**Local transclusion metadata.** SpatialFabricSubtree requires unitsPerMeter and upAxis for its known fabric/host frames. Its unitsPerMeter means host units per fabric metre and can include intentional model reduction. It differs from the per-world local-units ratio proposed in chapter 01. Missing metadata was refused in the local contract. This supports explicit mapping; it does not prove that two fields cover arbitrary frames.

**Local rendering choices.** Open Spatial Lab uses a 16-element column-major matrix convention and particular normalization, light compensation and composition backends. Precision-root transitions and the 4× hysteresis band are documented designs with assumptions. They are informative choices, not mandatory standards behavior or a proved universal minimum.

**Local crossing and signing evidence.** The retained Node crossing receipt contains 81 assertions from two local backends, including source-presence checks for browser code. The July 5 browser receipt is separate; the later July 11 receipt with the same name failed three stale handshake expectations. Signed-manifest vectors check byte/key consistency; signed-fabric refusal is bounded by a configured test anchor. Accepted source exit-intent removes the identified player before crossing and starts a finite tombstone; later client departure is confirmation. Lost replies, destination registration and delayed heartbeats have the distinct code-derived outcomes in chapter 02. None of these results establishes global presence exclusivity or independent cross-engine interoperability.

**Bindings to build or assess.** Shared multi-device anchoring, a portable inventory/preference protocol and a dedicated software-agent authorization binding were not demonstrated by the cited local work. Their feasibility, ownership and standards dependencies remain separate questions. Governance drafting is outside the implementation's scope. No keyword count proves a charter absent.

## Proposed text and profile boundaries

Every addition here is unadopted. First choose a minimal cross-implementation scenario, delegated layers, required data and observable success/failure outcomes. The five immediate asks remain limited to optional Portal.destination, a behavioral conformance profile and seed corpus, scene/spatial path mapping, lan/lon migration, and an optional signed-subtree evaluation track. Accepting them does not adopt every extension in this report.

### World coordinate vocabulary

Use the single proposal in [chapter 01, Optional World coordinate metadata](01-coordinate-precision-units-and-extents.md#optional-world-coordinate-metadata): unitsPerMeter is local coordinate units per physical metre, with no inferred default; upAxis, handedness and extent are optional metadata. Full basis/origin mapping remains explicit. Extent remains optional in the base proposal. A future composition profile can require a known frame and ratio, but must define how they are obtained and what happens when unavailable.

For aligned distance components, `target = source / sourceUnitsPerMeter * targetUnitsPerMeter`. Thus 100 source units at 100 units/metre becomes one target unit at one unit/metre. Frame translation/rotation and intentional model scale are separate. Requiring fields or fixing a 16-element matrix length narrows the currently accepted payload set and needs a declared version/profile. Missing metadata is not permission to assume metre-scale Y-up.

### Shared spatial anchors

Candidate direction: define how an agreed shared-anchor service is referenced, rather than equating a local tracked pose with cross-device localization. A profile must name the anchor namespace, frame, discovery/localization protocol, confidence/error meaning, permissions, lifetime and failure behavior. Its acceptance fixture should have two independent devices resolve the same physical anchor and report placement error.

This report does not select a provider or offer a schema that appears sufficient before those semantics are chosen. WebXR tracked anchors can be one client-side input; shared discovery and localization require their own binding.

### Portable inventory

Use the optional [InventoryItem sketch in chapter 05](05-presence-live-sync-and-persistence.md#portable-inventory-optional-extension) as the common candidate vocabulary. Its item_id, asset_uri, licence and provenance fields are illustrative data, not ownership or usage authorization. A versioned Universal Manifest profile can be evaluated once its exact schema and rights semantics are named. The draft imposes no new required User or manifest properties.

### User preferences

Use the optional [UserPreferences sketch in chapter 05](05-presence-live-sync-and-persistence.md#portable-preferences-optional-extension). Its text_scale, reduced_motion, high_contrast, input and rendering_quality members remain candidates. A profile must state consent, supported-setting discovery and the observable effect of accepted settings. Existing payloads remain valid without these members; the exchange is separate from accessibility conformance.

### Software-agent declarations

This optional metadata fragment targets OpenAPI 3.0.4. It is added through a compatible User extension; it does not replace canonical numeric User.id.

```yaml
AgentDeclaration:
  type: object
  properties:
    actorType:
      type: string
      enum: [human, software-agent]
      description: Self-declared actor type; absent means undeclared.
    capabilities:
      type: array
      items:
        type: string
      description: Declared capabilities, not granted permissions or verified test results.
```

No default implies that an undeclared actor is human. Both fields are optional; `{"actorType":"software-agent","capabilities":["navigation"]}` passes the sketch, while an unrecognized actorType fails. Authentication, operator identity, delegation, consent, issuer trust and permission checks remain external to this shape. The group must decide what assurance is required before using it for admission or execution.

### Governance document references

Candidate request: publish authoritative pointers to the initiative's decision process, maintainer responsibilities, contribution/license policy and external standards-body relationships. First locate and evaluate existing documents, including the whitepaper's adoption plan. Draft only missing rules after the responsible group confirms its scope. The report makes no license determination from a keyword table.

## Adoption path

**Existing payloads and clients.** Optional metadata can leave existing payloads valid. It does not guarantee that clients understand new semantics, and a supported profile must not be silently downgraded when required behavior is unknown. OpenAPI's acceptance of extra fields is a shape rule, not a universal instruction to ignore every unsupported capability safely.

**Profile participants.** Declare the version and supported profile. Establish frame/unit information before placement, negotiate graph and asset representations, and apply agreed target-resolution and failure rules. Keep per-world presence, visual continuity and session authority distinct. Require separate trust/admission/execution policies wherever the profile needs them.

**Working group.** Choose a minimal scenario and independent consumer test before making a broad conformance claim. Preserve the five bounded asks and optional tracks. New required properties, identifier-type changes, mandatory formats or a default graph-form change require explicit compatibility decisions.

## Open questions for the working group

1. Which minimal scenario defines the first behavioral profile, and which identity, live-session, rendering and anchor layers does it reference?
2. Which frame, units and matrix conventions are required, and what asymmetric placement fixture and error budget test them?
3. Does the anchor need include cross-device discovery/localization, and which existing service or standard supplies it?
4. Which exact inventory and preference profile should be evaluated, with what consent, rights and destination-support rules?
5. What actor/operator/delegation assurance is needed for software participants beyond self-declared capabilities?
6. Where are the authoritative governance, contribution/license and standards-body coordination documents, and which missing parts need group action?
7. How will API version, document version and optional profile identifiers be distinguished during migration?

## Sources

- [Linked Spatial Experiences: The Web of Worlds, April 2, 2025](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/): published units, preview, authorization and aspect intent.

- [OpenSpatialWorld API 0.0.1 at d39a1a0](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), [OpenSpatialAsset API](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialAsset/API.yaml), [OpenUserManifest API](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml) and [OpenSpatialWorld README](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md), rechecked September 11, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf), printed pages 7, 15, 22, 25, 26, 28 and 30.
- [OGC GeoPose 1.0](https://docs.ogc.org/is/21-056r11/21-056r11.html), Basic YPR frame and extension requirements.
- [WebXR Anchors Module](https://immersive-web.github.io/anchors/), draft consulted September 7, 2026: tracked anchor spaces; no cross-device acceptance result is claimed here.
- [OpenAPI 3.0.4 Schema Object](https://spec.openapis.org/oas/v3.0.4.html#schema-object) and [W3C Verifiable Credentials 2.0 trust model](https://www.w3.org/TR/vc-data-model-2.0/#trust-model).
- Selected local infrastructure map and topic inventory, September 7, 2026: 102 authored map rows and keyword-discovery material. Public reproduction of these exact local artifacts is not established. Neither establishes worldwide absence or comparative discussion frequency.
- Open Spatial Lab source snapshot and retained local numerical, crossing, contract and signing receipts, rechecked September 11, 2026. Exact proof classes and limits appear in chapters 01–09; no fresh network, renderer or cross-engine acceptance test is claimed.

## Change log

- 2026-09-07: replaced global absence and completion claims with scoped source findings; aligned coordinate, optional-profile, trust, presence and evidence boundaries with chapters 01–09.
