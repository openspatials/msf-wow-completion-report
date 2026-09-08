# Coordinate Precision, Units, and World Extents

WoWAPI 0.0.1 at commit d39a1a0 leaves the local coordinate and transform conventions needed for composition incompletely bound in its schemas. The March 31, 2026 whitepaper already discusses units and origins on printed page 7. The missing work is to bind that architectural intent to interoperable fields, frame mappings and observable error limits.

This chapter offers six proposals, not adopted requirements:

1. Define local coordinate units, axes and origin/frame relationships. Keep world extent optional in the base proposal.
2. Define the matrix size, storage order, multiplication convention and transform direction for `Node.localTransform`.
3. Define source-to-target unit conversion separately from intentional model scaling at transclusion boundaries.
4. Agree an error budget and test fixtures; leave numerical representation and renderer normalization to implementations.
5. Keep precision-root switching and the proposed 4× hysteresis band as an informative Open Spatial Lab design.
6. Align the GeoPose field names and semantics with the selected OGC profile, including a migration rule for `lan` and `lon`.

**Status:** source and local evidence checked September 7, 2026. All proposed schemas in this chapter target OpenAPI 3.0.4. They are fragments for a future version/profile, not a complete replacement API or group decision.

## What the specification says today

The [World schema](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L268-L349) defines metadata, GeoPose, presence settings, technology descriptors and counts. It has no local-units, local-axis, handedness or extent properties. This is a statement about the pinned API binding, not an absence claim about the whitepaper's spatial architecture.

`Node.localTransform` is an array of numbers at [lines 488–491](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L488-L491). It has no length bound, matrix storage order or transform-direction description. Existing numeric and object types still impose constraints; the absence of BCP 14 keywords does not remove OpenAPI semantics.

Six inline GeoPose definitions use `position.lan`: five in OpenSpatialWorld and one in OpenSpatialAsset. They do not explicitly bind a version of OGC GeoPose or the transformation into local scene coordinates. The [OGC GeoPose 1.0 Basic YPR profile](https://docs.ogc.org/is/21-056r11/21-056r11.html) fixes WGS-84 and a local tangent East-North-Up frame; height is ellipsoidal height in metres. Its extension properties do not make that frame configurable.

The README, whitepaper example and reference implementation schema use `scene` paths; the API uses `spatial` paths. Their mapping is a separate group decision, covered in [chapter 09](09-conformance-vocabulary-and-errata.md).

## What fails without a shared mapping

**Scale ambiguity.** A source using 100 local units per metre and a target using one unit per metre need a declared conversion. An astronomical model may also be intentionally reduced to fit a room. Physical-unit conversion and that authored reduction are different operations.

**Axis and rotation ambiguity.** A Z-up source placed in a Y-up host can be incorrectly oriented if the basis mapping is omitted. Up-axis alone is insufficient for arbitrary frames: handedness, axis directions, rotation order and the relation between local and geodetic frames also matter. A scale failure does not by itself explain an orientation failure.

**Local detail lost at large absolute coordinates.** In the retained September 7 numerical probe, float32 spacing near one astronomical unit was 16,384 metres at scales 1, 2^-20 and 2^-40 after converting back to world units. Casting large absolute positions first lost a one-metre offset; subtracting a nearby origin before casting preserved it. Uniform scaling alone does not restore relative precision. It can help range and clipping, but camera-relative calculation, origin rebasing and split/high-precision representations are alternatives that do not require a semantic world-root change.

**Transform ambiguity.** A three-element array, a 3×3 rotation and a 4×4 matrix can all satisfy the current array shape. Two implementations can therefore read a valid value differently. The group needs an explicit transform contract and asymmetric fixtures, not just a new array length.

**GeoPose-to-local ambiguity.** Adopting a geodetic profile does not eliminate conversion into a scene frame. That conversion still needs an origin, orientation and units. A different geodetic reference system requires an explicitly different profile or transformation, not a `datum` field added to Basic YPR.

## What Open Spatial Lab built and learned

These are local implementation facts or documented designs. They do not establish interoperability between independent engines.

**Existing `unitsPerMeter` field.** Open Spatial Lab's SpatialFabricSubtree uses this name for `k`, host world units per fabric metre, folded into `composeScale = k * s`, where `s` is the node's uniform scale. Its tabletop example uses `k = 0.5 / 149597870700` to show one astronomical unit as half a metre in a metre-based room. This existing field includes the chosen model reduction. It is not automatically the same quantity as the proposed per-world local-units ratio below. Missing, non-positive and non-finite values are refused by the implementation.

**Existing `upAxis` field.** The local contract requires `z` or `y` with no default. For Z-up fabric in its Y-up host, the implementation uses `R_x(-90 degrees)`; for Y-up it uses the identity. These are mappings for those known frames, not a complete six-degree-of-freedom frame specification.

**Engine normalization.** The cited engine source uses `dRenderScale = TARGET_EXTENT / dMaxReach`, with a target of five render units and a root-anchored bounding sphere. Open Spatial Lab mirrors this flattening and its light-scaling rules. The historical source reports 14 structural parity checks with maximum relative error at or below 3.3e-16. Those are structural numerical checks, not a fresh rendered-surface or universal-precision proof.

**Precision-root design.** The navigation design has OBSERVE, PREPARE and COMMIT phases, with prefetch below eight times a candidate commit distance and ascent at four times the descent distance. The proposed expression `d_commit = q_local * H / (2 * tan(fovY/2) * p_tol)` estimates screen-space error using float spacing, viewport height, vertical field of view and pixel tolerance. It assumes a projection and consistent units. Threshold updates, noise, motion and coordinate conversion remain part of its analysis. The 4× band is a candidate policy; it does not prove oscillation impossible. The design was not demonstrated as a live precision-transition loop in the cited evidence.

**Matrix convention.** Open Spatial Lab interprets `localTransform` as a 16-element column-major 4×4 matrix. Its checks label that as an implementation convention, not a requirement imposed by the pinned API.

## Proposed normative text

The following fragments are unadopted proposals. Uppercase requirement words apply only if the group adopts the named field or profile. OpenAPI 3.0.4 uses `minimum: 0` with boolean `exclusiveMinimum: true` for a positive number.

### Optional World coordinate metadata

```yaml
WorldCoordinateMetadata:
  type: object
  properties:
    unitsPerMeter:
      type: number
      minimum: 0
      exclusiveMinimum: true
      description: Local coordinate units per physical metre; no inferred default.
    upAxis:
      type: string
      enum: [y, z]
      description: Local up-axis; a full frame mapping is still required.
    handedness:
      type: string
      enum: [right, left]
      description: Local frame handedness; no inferred default or complete basis mapping.
    extent:
      type: number
      minimum: 0
      exclusiveMinimum: true
      description: Optional bounding-sphere radius in local units about the declared local origin.
```

This metadata proposal leaves all four properties optional. A future composition profile could require a known frame and unit ratio before composition; it would need to name how those are obtained and what happens when they are unknown. It must not silently assume metres, handedness or one renderer's axis convention. An up-axis and handedness still do not identify the horizontal axes or origin; a composition profile must supply the full basis and origin mapping. Extent can assist bounds and loading decisions without prescribing a precision algorithm.

### Unit conversion and intentional scale

For a distance in aligned source and target frames:

```text
target = source / sourceUnitsPerMeter * targetUnitsPerMeter
100 source units / 100 units per metre * 1 unit per metre = 1 target unit
```

Frame rotation/translation and any intentional model scale are separate. For a source already expressed in fabric metres, Open Spatial Lab's existing `k` combines target units per metre with the model reduction; its separate node scale `s` is then applied. The proposed World metadata must not be read as a drop-in reinterpretation of that existing field.

### Candidate localTransform profile

```yaml
LocalTransform:
  type: array
  items:
    type: number
  minItems: 16
  maxItems: 16
  description: >
    Proposed 4x4 column-major matrix mapping node-local column vectors
    into the parent frame. Elements 12, 13 and 14 hold translation.
    Compose a child using parentWorldMatrix * childLocalMatrix.
```

This is a candidate convention, selected for agreement with the existing local implementation. Fixing length rejects arrays accepted by the current API, so it belongs in an explicit version/profile with a migration rule. A 16-number identity passes this shape; a three-number array fails. Shape validation does not establish invertibility, acceptable scale, units or numerical error.

### Informative precision strategies

An implementation may normalize, use camera-relative coordinates, rebase its render origin or use higher-precision representations. A conformance profile should instead state an observable placement/error budget for agreed fixtures. No universal root extent, root-change algorithm, light compensation formula or hysteresis ratio is proposed as a base requirement.

### GeoPose adoption by reference

Proposed wording: adopt the OGC GeoPose 1.0 Basic YPR field names and fixed frame semantics, then define the mapping into local scene coordinates. For longitude migration, emit `lon` in the new profile; accept legacy `lan` during a declared compatibility window; accept both only when the values agree, and reject a conflict. This is proposed conflict handling, not current upstream behavior. Non-georeferenced virtual worlds need not acquire a GeoPose merely to satisfy this proposal. A complete WoW-shaped specimen with all position and angle members fails the official Basic YPR schema when longitude is spelled `lan` and passes after that member becomes `lon`. This is a specimen-level shape check: the current WoW schemas also allow missing members that Basic YPR requires. The official schema permits extra properties, so extra `datum` metadata is not automatically schema-invalid; it cannot redefine Basic YPR's fixed frame.

## Adoption path

**Existing payloads.** Adding optional metadata preserves payloads that omit it. Validating a named property constrains its value when present. Requiring units, an axis or a fixed transform length would narrow the accepted set and must be versioned or negotiated. The metadata example `{"unitsPerMeter":100,"upAxis":"y","handedness":"right"}` is valid without extent; zero or negative unitsPerMeter is invalid under this proposal.

**Client behavior.** Resolve both frames and their unit ratios before composing content under a future composition profile. If the information is unavailable, report unresolved placement or use only an explicitly agreed fallback. Select the numerical strategy that meets the profile's error budget.

**Server behavior.** Advertise the version/profile, emit known coordinate metadata, and retain the agreed legacy behavior during migration. Do not invent extent or georeferencing data merely to satisfy a shape check.

## Open questions for the working group

1. Which initial cross-implementation scenario defines the frame and precision contract: room-scale placement, astronomical viewing, georeferenced scenes, or separate profiles?
2. Is `unitsPerMeter` the preferred field name, and how will the different meaning of the existing local composition field be distinguished?
3. Which origin, handedness, axis and rotation conventions are mandatory in each profile? What asymmetric six-degree-of-freedom fixture and placement tolerance will test them?
4. Should optional extent be a sphere or a box, and for which loading or indexing use case?
5. What is the `lan` acceptance window, and does the group accept the proposed conflict rule and Basic YPR reference?
6. For shared multi-device localization, which anchor discovery, localization and permissions contract should be referenced? No such binding was located in the reviewed WoW sources. A stable local WebXR anchor alone does not settle shared localization.

## Sources

- [OpenSpatialWorld API 0.0.1 at d39a1a0](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf), printed page 7: units and origin architecture.
- [OGC GeoPose 1.0, 21-056r11](https://docs.ogc.org/is/21-056r11/21-056r11.html), published September 8, 2023: Requirements 4, 12 and 37 and section 9.2.
- [OpenAPI 3.0.4 Schema Object](https://spec.openapis.org/oas/v3.0.4.html#schema-object).
- Open Spatial Lab local source snapshot checked September 7, 2026: `schema.yaml` lines 784–804; `NAVIGATION-ARCHITECTURE.md` precision design; `DESIGN-NOTE-fabric-as-precision-domain.md` and the historical structural-parity receipt. Public reproduction of these exact local bytes is not established.
- Numerical counterexample retained September 7, 2026, Node 22.22.3: three float32 scale cases and origin subtraction. This is a bounded numerical probe, not a renderer benchmark.
- [Appendix A](A-completion-map.md): CM-044 through CM-049. [Appendix B](B-findings-register.md): R-001, R-002, R-013 and R-022.

## Change log

- 2026-09-07: corrected the units direction, precision claims, optional extent, GeoPose profile and migration guidance. Preserved local implementation evidence with its limits.
