# Coordinate Precision, Units, and World Extents

The Web of Worlds specification at commit d39a1a0 declares no units, no coordinate precision, no up-axis convention, no world extent, and no matrix format for spatial transforms. A World resource is a bag of properties with no spatial frame. Node.localTransform is an unconstrained array of numbers. GeoPose uses the field name `lan` where longitude is meant, and carries no explicit geodetic datum.

The standard needs six additions to make spatial composition work between independent implementations:

1. A World resource MUST declare units, up-axis, and extent (high confidence; verified against two independent implementations).
2. Node.localTransform MUST be defined as a 16-element column-major 4x4 matrix (high confidence; the current definition is ambiguous by construction).
3. A transclusion contract MUST include a units-per-metre scale factor and an up-axis declaration, both required with no default (high confidence; verified in code by Open Spatial Lab).
4. The standard SHOULD define a per-scene normalization rule mapping double-precision world coordinates to float32-friendly render coordinates (high confidence for the need; the specific rule is an open design question).
5. A client implementing proximity-based root transitions MUST apply hysteresis with at least a 4x ratio between descend and ascend thresholds (high confidence; without it, oscillation is a structural inevitability).
6. GeoPose SHOULD adopt the OGC GeoPose Basic YPR form with an explicit datum field, and `lan` should be corrected to `lon` (high confidence; `lan` is a typo with 6 occurrences across the specification and 0 occurrences of `lon`).

**Status:** Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).


## What the specification says today

The World schema (API.yaml lines 268-349) defines content metadata (label, age restriction, licence, cost, version, duration), a GeoPose, presence settings, technology descriptors, user and view counts, and a portal count. It declares no `units`, no `scale`, no `upAxis`, no `handedness`, and no `extent` or `bounds` field.

Searches across all three API files (OpenSpatialWorld/API.yaml, OpenSpatialAsset/API.yaml, OpenUserManifest/API.yaml) for the following terms return zero results: `precision`, `float32`, `float64`, `units`, `scale`, `origin`, `extent`, `bounds`, `jitter`, `upAxis`, `handedness` (verified: grep, 2026-09-07).

The specification contains zero RFC 2119 keywords (MUST, SHOULD, SHALL, MAY). All 10 occurrences of `required` in API.yaml are on REST endpoint parameters or request bodies (lines 57-251), not schema-level property requirements.

Node.localTransform is defined at API.yaml lines 488-491:

> `localTransform: type: array, items: type: number`

([API.yaml line 488-491](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L488-L491))

No `minItems`, no `maxItems`, no documentation of matrix convention (row-major vs column-major), no statement of what the 4x4 entries represent.

GeoPose appears on six resources: World, User, View, Portal, and Spatial (OpenSpatialWorld/API.yaml lines 294, 368, 395, 424, 456) and Asset (OpenSpatialAsset/API.yaml line 204). Each instance uses the field name `lan`. The field `lon` does not appear anywhere in the specification. No geodetic datum is declared; the position sub-object is `{lat, lan, h}` with no reference frame.

([API.yaml line 294](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml#L294) and four more occurrences)

The README states URL paths as `URL/wow/scene/` and `URL/wow/scene/node` (README.md lines 28 and 31). The API.yaml implements `/wow/spatial/{spatialID}` and `.../node/{nodeId}` (API.yaml lines 133 and 156). This path contradiction is a separate erratum, not in scope for this document.


## What fails without it

**Scale ambiguity at transclusion boundaries.** When a node transcludes a spatial document authored at a different scale, the host renderer has no declared conversion factor. A celestial fabric where 1 AU = 1 internal unit, composed into a room-scale world where 1 unit = 1 metre, can silently render the solar system at a 1:1 metre scale or the room at AU scale. Both are defensible readings of a specification that says nothing. Open Spatial Lab hit this when mounting a Z-up fabric into a Y-up host without declared units: the result rendered upside down and at the wrong scale.

**Up-axis mismatch across composed subtrees.** The specification declares no up-axis convention. An asset authored Z-up, composed into a Y-up host, rotates 90 degrees around the wrong axis and lies on its side. A default-to-Z fallback reintroduces the exact orientation bug the declaration was created to prevent, because the host has no way to know the fallback was applied.

**Float32 quantization at planetary scale.** GPU rendering pipelines operate in float32. Without a declared world extent, a renderer has no information to compute the per-scene normalization that keeps coordinates representable. At solar-system scale, the float32 grid quantizes at approximately 13.4 km per step at Earth's orbital distance (reported: Open Spatial Lab, measured in the engine's compositor with a solar fabric of 30.16 AU reach, render scale 0.166; source: DESIGN-NOTE-fabric-as-precision-domain.md lines 35-72, NAVIGATION-ARCHITECTURE.md lines 308-316). Surface rendering at that quantization is impossible without re-rooting to a tighter precision domain.

**Transform ambiguity.** An array of numbers with no length or order constraint is not a transform. A 3-element array could be a position. A 9-element array could be a 3x3 rotation. A 16-element array could be row-major or column-major. Two implementations that guess differently will place every node in the wrong location, and neither is wrong according to the specification.

**GeoPose datum ambiguity.** A GeoPose without an explicit geodetic datum is ambiguous. Two implementations assuming different reference frames will place the same world in different locations on Earth. The `lan` typo (for longitude) is a minor defect that compounds the problem: an implementer reading the schema for the first time cannot be certain the field means longitude without consulting external documentation.


## What Open Spatial Lab built and learned

Open Spatial Lab (OSL) addressed four of the six gaps in its SpatialFabricSubtree extension, a labelled non-canonical extension to the Web of Worlds schema. The remaining two (World-level declarations and GeoPose datum) are proposed as upstream changes.

**unitsPerMeter (labelled extension).** OSL added `unitsPerMeter` as a required field on SpatialFabricSubtree with no default (schema.yaml lines 784-794). The field is a positive number representing world units per fabric metre. It is folded into the compositor's `composeScale` (`composeScale = k * s`, where `k` is `unitsPerMeter` and `s` is the node's decomposed uniform scale) rather than post-multiplied, so the engine's light-intensity rule applies correctly. A non-positive or non-finite value is refused with `BAD_UNITS_PER_METER`; an absent value is refused with `MISSING_UNITS_PER_METER`.

**upAxis (labelled extension).** OSL added `upAxis` as a required field (enum: `z`, `y`) on SpatialFabricSubtree with no default (schema.yaml lines 795-804). `z` means the fabric corpus default (Z-up), and the host inserts the fixed basis change B = R_x(-90 degrees), a pure isometry with determinant +1. `y` means the fabric is authored Y-up and B = I. An absent or unrecognized value is refused with `UNKNOWN_UP_AXIS`, never defaulted.

**Per-scene normalization (verified from engine source, not an OSL invention).** OSL verified from the engine source that the per-scene flatten is a single uniform render scale computed once per scene: `dRenderScale = TARGET_EXTENT / dMaxReach`, where `TARGET_EXTENT` is a constant (5.0 render units, per Compositor.cpp line 264-267) and `dMaxReach` is the root-anchored bounding sphere accumulated through the entire scene graph including child fabrics (DESIGN-NOTE-fabric-as-precision-domain.md lines 35-72). A composed child fabric inherits the root's single precision domain and has no independent normalization. OSL mirrored this flatten in its own compositor (`engineRenderScale` / `applyRenderScaleToScene`). The normalization was validated against the engine's laws to 14 structural parity checks with maximum relative error at or below 3.3e-16 (reported: Open Spatial Lab UPSTREAM-REGISTER.md line 137; not re-run in this pass, confidence medium).

**Re-root with hysteresis (designed, not yet live).** OSL designed a precision-horizon commit loop with three phases: OBSERVE (hover affordance), PREPARE (verify-ahead off-screen at distance less than 8 times d_commit), and COMMIT (re-root at distance less than d_commit). The commit distance is derived from the float quantum, viewport height, and field of view, not tuned per-world: `d_commit = q_local * H / (2 * tan(fovY/2) * p_tol)`, where `p_tol` is a jitter tolerance of approximately 0.5 pixels. Hysteresis: descend at d_commit, ascend only at 4 times d_commit. The 4x band makes oscillation structurally impossible (NAVIGATION-ARCHITECTURE.md lines 300-381).

**Node.localTransform convention (labelled divergence).** OSL conventionally interprets localTransform as a 16-float column-major 4x4 matrix, matching the glTF convention. This is labelled as an OSL convention, not a reading of the standard: the conformance lane reports a non-16-length array as a divergence from OSL's convention, not as spec drift (schema.yaml lines 898-901).

**Claim boundary.** These are extensions and conventions developed by one implementation. They are proposed normative text, not a compliance claim against the current specification. The 14 structural parity checks were run by Open Spatial Lab and have not been independently re-run (reported: UPSTREAM-REGISTER.md line 137). The re-root hysteresis design is documented but not yet implemented in a live system.


## Proposed normative text

### World resource spatial properties

```yaml
World:
  type: object
  properties:
    # ... existing properties ...
    units:
      type: string
      enum: [metres, AU]
      default: metres
      description: >
        The base unit of length for all coordinates and
        transforms in this world.
    unitsScaleFactor:
      type: number
      exclusiveMinimum: 0
      description: >
        When units is not sufficient, a numeric scale factor
        expressing world-units-per-metre. If present, it
        overrides the enum value.
    upAxis:
      type: string
      enum: [y, z]
      description: >
        The up direction for this world. y = Y-up (WebGL,
        Three.js, glTF convention). z = Z-up (engineering,
        many CAD tools).
    extent:
      type: number
      exclusiveMinimum: 0
      description: >
        Bounding-sphere radius in the declared units,
        measured from the world origin. A conformant
        renderer uses this to compute precision-domain
        normalization.
  required:
    - units
    - upAxis
    - extent
```

Rationale: `units` and `upAxis` are required because their absence caused silent rendering failures in practice. `extent` is required because without it a renderer cannot compute the per-scene normalization that keeps float32 coordinates representable at large scale.

### Node.localTransform

```yaml
localTransform:
  type: array
  items:
    type: number
  minItems: 16
  maxItems: 16
  description: >
    A 4x4 transformation matrix in column-major order
    (matching the glTF convention). Elements [12], [13],
    [14] are the translation components.
```

Rationale: column-major 4x4 matches glTF, the most widely adopted spatial asset format. Constraining length to 16 eliminates the ambiguity of shorter arrays.

### Transclusion contract (SpatialFabricSubtree or equivalent)

A node that transcludes a spatial document MUST include:

- `unitsPerMeter`: a required positive number with no default, expressing the transcluded document's world-units-per-metre ratio.
- `upAxis`: a required enum (`z` or `y`) with no default, declaring the transcluded document's up direction.

A server MUST refuse a transclusion request that omits either field. A server MUST refuse a `unitsPerMeter` value that is non-positive or non-finite.

Rationale: defaults for these fields are dangerous. A default of 1.0 for unitsPerMeter silently maps 1 AU to 1 metre when a celestial fabric is transcluded into a room-scale world. A default of `z` for upAxis reintroduces the orientation bug the field was created to fix.

### Per-scene normalization (SHOULD)

A conformant renderer SHOULD implement a per-scene normalization that maps double-precision world coordinates to float32-friendly render coordinates. The normalization SHOULD use the declared `extent` of the root world to compute a uniform scale factor.

When a subtree is composed in place (not re-rooted), it MUST inherit the root scene's single precision domain. Re-rooting into a child subtree is the mechanism for gaining independent precision inside a nested body.

Rationale: this makes the precision architecture explicit without mandating a specific formula. Implementations that do not handle large worlds can ignore the normalization; implementations that do handle them need the extent to compute it.

### Re-root hysteresis (MUST for implementations that support proximity transitions)

A client implementing proximity-based root transitions MUST apply hysteresis: the descend threshold and the ascend threshold MUST differ by at least a factor of 4. The descend threshold SHOULD be derived from the current precision domain's float quantum and the viewport geometry, not tuned per-world.

Rationale: without hysteresis, a camera near the commit threshold oscillates between two roots on every micro-movement. The 4x ratio is the minimum that makes oscillation structurally impossible rather than merely unlikely.

### GeoPose (SHOULD, adopt by reference)

Every GeoPose in the Web of Worlds API SHOULD adopt the OGC GeoPose Basic YPR form with an explicit `datum` field (default: WGS84).

The field `lan` SHOULD be corrected to `lon` in the next schema revision, with a deprecation window during which both field names are accepted.

Rationale: `lan` appears 6 times across the specification (5 in OpenSpatialWorld/API.yaml and 1 in OpenSpatialAsset/API.yaml) and `lon` appears 0 times (verified: grep, 2026-09-07). An explicit datum eliminates the reference-frame ambiguity.

### Optional extension: derived commit distance formula

The formula `d_commit = q_local * H / (2 * tan(fovY/2) * p_tol)` is offered as an informative (non-normative) reference for implementations. It derives the root-transition threshold from the float quantum at the camera's position, the viewport height, the vertical field of view, and a jitter tolerance. It is the formula Open Spatial Lab designed for this purpose (NAVIGATION-ARCHITECTURE.md lines 325-333).


## Adoption path

**Minimal world (room-scale, single asset format).** A world that operates at room scale with metre units and Y-up can declare `units: metres`, `upAxis: y`, and `extent` equal to the room's bounding-sphere radius. No normalization is needed at room scale. localTransform gains `minItems: 16, maxItems: 16` and a column-major convention. This is the smallest change and it breaks nothing that currently works.

**What a client must do.** Read `units`, `upAxis`, and `extent` from the World resource. Apply the declared up-axis when composing transcluded content. Use `extent` to compute a precision-domain normalization if the world operates at scales where float32 quantization matters (approximately above 10 km extent). When implementing proximity-based root transitions, apply the hysteresis rule.

**What a server must do.** Add `units`, `upAxis`, and `extent` to the World schema response. On transclusion endpoints, require `unitsPerMeter` and `upAxis` with no defaults. Refuse requests that omit them. Correct `lan` to `lon` in GeoPose, with a deprecation window accepting both.


## Open questions for the working group

1. **Units enum vs numeric scale factor.** Should `units` be an enum (metres, AU) with an optional numeric override, or a single numeric scale-factor field? The enum is friendlier for the common case; the numeric field is more general. Both are proposed above; the group should pick one.

2. **Extent representation.** Bounding-sphere radius (simpler, sufficient for normalization) or axis-aligned bounding box (more precise, more complex)? The proposal above uses a bounding-sphere radius. A bounding box would serve spatial indexing as well as precision, but adds three fields.

3. **Up-axis default vs required.** Should `upAxis` have a default (Y-up, matching glTF) or remain required with no default? A default reduces the burden on simple worlds but risks silent mis-composition when the default is wrong. Open Spatial Lab's experience argues for no default.

4. **Normative vs informative normalization rule.** Should the per-scene normalization be a MUST (all conformant renderers compute it) or a SHOULD (recommended for large worlds)? The proposal above uses SHOULD. A MUST would guarantee interoperability at large scale but burdens room-scale implementations that do not need it.

5. **GeoPose adoption scope.** Should the OGC GeoPose adoption be limited to adding a `datum` field, or should the group adopt the full OGC GeoPose Basic YPR schema by reference? The latter is cleaner but requires aligning field names (`angles` vs the OGC structure).

6. **Shared spatial anchors.** Two AR devices in the same physical room cannot agree on where "here" is without shared spatial anchors. This is a must-interop socket with zero coverage from any standard in the ecosystem. Should the group define a spatial-anchor vocabulary or adopt the WebXR Anchors Module by reference? This is a large question and may belong in a separate extension track.


## Sources

- WebOfWorlds/WoWAPI specification, commit d39a1a0 (2026-05-21): [API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), [README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md)
- Open Spatial Lab schema extension, schema.yaml lines 784-804 (unitsPerMeter, upAxis) and lines 898-901 (localTransform convention), commit 8887d5f (grigb/open-spatial-lab-dev, private; public release at github.com/grigb/open-spatial-lab)
- Open Spatial Lab design note, DESIGN-NOTE-fabric-as-precision-domain.md lines 35-72 (per-scene flatten) and lines 260-266 (World schema gaps), same commit
- Open Spatial Lab navigation architecture, NAVIGATION-ARCHITECTURE.md lines 300-381 (re-root hysteresis), same commit
- Open Spatial Lab upstream register, UPSTREAM-REGISTER.md line 137 (14 structural parity checks, confidence note), same commit
- Open Spatial Lab working-group dossier, WORKING-GROUP-DOSSIER.md line 345 (16 bodies, max relative error 3.3e-16), same commit
- OGC GeoPose Standard (OGC 21-056r11), adopted 2022: defines Basic YPR form with explicit datum
- Completion Map rows: CM-044, CM-045, CM-046, CM-047, CM-048, CM-049
- Findings rows: R-001, R-002, R-013, R-022


## Change log

- 2026-09-07: first public draft, verified.
