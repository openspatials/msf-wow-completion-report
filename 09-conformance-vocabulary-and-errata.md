# Conformance Vocabulary and Errata

The Web of Worlds specification at commit d39a1a0 contains zero RFC 2119 keywords, declares no required properties on any schema, contradicts itself between the README and API.yaml on the graph path, carries a misspelling of "longitude" across all five GeoPose occurrences, omits a mandatory OpenAPI parameter on four operations, truncates a Core Requirement description mid-sentence, and leaves two other Core Requirement descriptions empty (all verified). Until these are fixed, "compliant with Web of Worlds" has no testable meaning for any implementer. The standard needs three things: RFC 2119 keywords and required schema properties to make conformance definable, a short errata pass to fix the defects listed here, and a minimal conformance test corpus to make the claim verifiable.

Each erratum below is independently checkable in minutes at the cited commit. Each proposed normative sentence is marked with its confidence: "verified" means we checked the primary source ourselves; "inferred" means our reasoning on top of verified facts.

**Status:** Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).

---

## What the specification says today

### RFC 2119 keywords

A search for the uppercase keywords MUST, SHOULD, SHALL, MAY, REQUIRED, RECOMMENDED, and OPTIONAL across all three API YAML files returns zero hits (verified: `grep -cE` at commit d39a1a0). The specification contains no RFC 2119 normative-strength keywords.

### Required schema properties

The word "required" appears 10 times in API.yaml. Nine are `required: true` on path parameters (lines 57, 76, 98, 120, 144, 167, 198, 217, 251). One is `required: true` on a `requestBody` (line 226). Zero are schema-level `required` property declarations (verified: `grep -B2 'required: true'` at API.yaml, commit d39a1a0).

Consequence: an empty JSON object `{}` validates against all six main schemas (World, User, View, Portal, Spatial, Node) because none of them declares any property as required.

### Graph path: README vs API.yaml

The README lists the graph endpoints as:

- `URL/wow/scene/` (README line 28)
- `URL/wow/scene/node` (README line 31)

The API.yaml defines the machine-readable endpoints as:

- `/wow/spatial/{spatialID}` (API.yaml line 133)
- `/wow/spatial/{spatialID}/node/{nodeId}` (API.yaml line 156)

These are two different path hierarchies. The README uses `scene`; the API.yaml uses `spatial`. Nothing in the repository adjudicates which is normative (verified: no errata file, no changelog, no comment addressing this).

### Missing spatialID parameter on node operations

The path template `/wow/spatial/{spatialID}/node/{nodeId}` (API.yaml line 156) contains two template variables: `spatialID` and `nodeId`. The four operations under this path (POST lines 158-188, GET lines 189-207, PUT lines 208-241, DELETE lines 242-260) each declare only `nodeId` as a parameter. None declares `spatialID`. OpenAPI 3.0.4 requires every template variable in a path to be declared as a parameter; an undeclared variable is an invalid specification (verified: OpenAPI Specification 3.0.4, Section 4.7.1, "Path Templating").

### DELETE node parameter misspelling

The DELETE operation at lines 242-260 declares its path parameter as `nodeid` (lowercase d, API.yaml line 248). The path template spells it `nodeId` (camelCase). Because OpenAPI matches parameter names to template variables by exact string, `nodeid` does not match `{nodeId}`, leaving both variables undeclared on this operation.

### GeoPose.position.lan

Every GeoPose occurrence in API.yaml spells the longitude field as `lan` (lines 294, 368, 395, 424, 456). The field appears alongside `lat` (latitude) and `h` (height). No field named `lon` or `longitude` exists anywhere in the specification (verified: `grep -c 'lon:' API.yaml` returns 0). The misspelling is carried consistently across all five schemas that embed GeoPose: World, User, View, Portal, and Spatial.

### Core Requirement descriptions

The README's Core Requirements table (lines 9-15) contains three description defects:

- **Preview world** (line 13): the description reads "experence world without" and stops mid-clause. The sentence is truncated; "without" has no object.
- **Persist world** (line 14): the Description cell is empty. Only the Feature column ("store or bookmark URL") is populated.
- **Share world** (line 15): the Description cell is empty. Only the Feature column ("send URL to second user") is populated.

### aspect.id syntax

The Join row (README line 11) lists `URL#join=aspect.id` as a feature. The specification does not define the syntax of `aspect.id`: whether it is an opaque string, a typed identifier, a namespaced value, or scoped to a resource kind. The Optional Feature section (README lines 24-25) gives one concrete example, `URL/wow/user/4182`, but does not state the grammar that connects the fragment value to a resource identifier. No failure semantics are defined for an aspect identifier that does not resolve.

---

## What fails without these fixes

**Conformance is undefined.** An implementer who builds a complete Web of Worlds server and passes every response through an OpenAPI validator cannot earn the claim "compliant with Web of Worlds." The validator will accept an empty object for every schema. No implementation can fail a conformance test because no conformance test can distinguish a conformant response from an empty one (verified by Open Spatial Lab's conformance harness: `{}` validates against all six schemas).

**Code generators reject the spec.** OpenAPI code generators and validation tools that enforce the template-variable rule reject the node operations because `spatialID` is undeclared. An implementer using Swagger Codegen, OpenAPI Generator, or similar tooling cannot generate correct client or server stubs for the graph operations without first patching the specification by hand (verified: OpenAPI 3.0.4, Section 4.7.1).

**Interoperability breaks on the graph path.** Two implementations that each follow a different source of truth for the graph path (one following the README's `/wow/scene/`, the other following API.yaml's `/wow/spatial/`) cannot interoperate. A client built against one path will get 404 responses from a server built against the other. The simpleWorlds reference implementation uses `/wow/scene/node/{nodeId}`; Open Spatial Lab uses `/wow/spatial/{spatialID}/node/{nodeId}`. These two implementations cannot exchange graph data today (verified: different URL paths are different endpoints).

**The longitude field is misnamed across the standard.** An implementer who corrects `lan` to `lon` or `longitude` breaks interoperability with every implementation that reads the specification literally. An implementer who reads the specification literally ships a field called `lan` that no geospatial library recognizes. Both choices cost something; neither is right until the standard picks one and provides a transition path.

**Implementers guess the same truncated sentence.** Every implementer of the Preview core requirement must independently guess what "experience world without" means. Open Spatial Lab interpreted it as "without joining" (labeled interpretation I5). Another implementer might interpret it as "without modifying," "without persisting," or "without authentication." Without a complete sentence, interoperability on Preview semantics is accidental.

---

## What Open Spatial Lab built and learned

Open Spatial Lab implemented the Web of Worlds specification at commit d39a1a0 over the course of 2026 and documented every point where the specification was silent, contradictory, or defective. The findings below are drawn from that implementation.

**Parameter repair (labeled divergence D5).** OSL's schema declares `spatialID` as a path parameter on all four node operations and spells `nodeId` with a capital D on DELETE, repairing the two OpenAPI defects. This is a labeled repair of an upstream defect, documented in the OSL contract (`OSL-WOW-CONTRACT.md`), not a silent improvement. The repair was necessary because OSL's validation tooling rejected the upstream spec as invalid OpenAPI (verified: OSL schema.yaml lines 189-194, comment block).

**Graph path decision.** OSL adopted the API.yaml path `/wow/spatial/{spatialID}` as canonical and explicitly rejected the README's `/wow/scene/` path and the simpleWorlds reference implementation's use of `/wow/scene/`. This decision is recorded in OSL's contract document (`OSL-WOW-CONTRACT.md` lines 49-52, verified).

**Extension policy.** Because the specification says nothing about how implementations may extend the standard, OSL established a four-part extension policy: (1) additive only, never remove or rename a canonical field; (2) always labeled with `x-osl-extension: true`; (3) surfaced by a conformance-reporting mechanism, never hidden; (4) `standards_conformance` stays `false` on every `/wow` response, so no cosmetic conformance claim is possible. This policy is binding on all OSL responses (`OSL-WOW-CONTRACT.md` lines 116-125, verified).

**Conformance refusal.** Every OSL `/wow` response carries `standards_conformance: false` by design. OSL refuses to claim conformance because conformance is currently undefined. The URL+fragment grammar is tested by 70 mutation-tested checks, but the harness cannot assert spec conformance because the specification defines no conformance criteria (verified: WORKING-GROUP-DOSSIER.md, B4 offerings).

**Core Requirement interpretations.** OSL completed the Preview description as "without joining" (interpretation I5), interpreted Persist as "bookmark the URL," and interpreted Share as "send the URL." All three are labeled as interpretations, not as spec text. OSL implemented the five URL-fragment Core Requirements against 9 documented interpretations (labeled I1-I10) rather than guessing silently (verified: WORKING-GROUP-DOSSIER.md section B4.4).

**Aspect identifier resolution.** OSL interpreted `aspect.id` as a placeholder for an aspect's identifier (interpretation I1). The implementation accepts an optional `kind/` qualifier (e.g., `user/4182`, `node/portal-1`) for disambiguation and falls back to probing the known aspect kinds (user first, then node) when no qualifier is given. An unresolvable aspect is treated as a warning; the client proceeds with the intent alone. This is a labeled interpretation, not spec-mandated behavior (verified: wow-url.mjs lines 63-71 and 287-311).

**Longitude misspelling carried verbatim.** OSL's GeoPose schema uses `lan` to match the standard exactly (OSL schema.yaml line 494, comment at line 495: "sic: the standard spells longitude 'lan'"), with a drift guard that fails the build if a payload ever says `lon` (WORKING-GROUP-DOSSIER.md section B4.2). OSL chose literal conformance over correctness, on the ground that correcting the spelling unilaterally would break interoperability with any other literal implementation (verified).

**Claim boundary.** OSL's implementation proves that a single team can build a working Web of Worlds server against the current specification, but only by making at least 9 documented interpretive choices and 1 labeled spec repair where the specification is silent, truncated, or defective. OSL does not claim standards conformance and cannot, because conformance is undefined.

---

## Proposed normative text

### 1. RFC 2119 keyword adoption

```yaml
# Specification preamble (proposed addition)
# The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
# "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
# document are to be interpreted as described in RFC 2119.
```

Rationale: without normative-strength keywords, no sentence in the specification distinguishes a mandatory behavior from a suggestion (inferred from the zero-keyword count).

### 2. Required schema properties

Each schema MUST declare its identifying properties as `required`. At minimum:

```yaml
# World schema (proposed addition)
World:
  type: object
  required:
    - content
    - geoPose

# User schema (proposed addition)
User:
  type: object
  required:
    - id
    - name

# View schema (proposed addition)
View:
  type: object
  required:
    - id
    - geoPose

# Portal schema (proposed addition)
Portal:
  type: object
  required:
    - id
    - geoPose

# Spatial schema (proposed addition)
Spatial:
  type: object
  required:
    - id
    - rootNodeID

# Node schema (proposed addition)
Node:
  type: object
  required:
    - id
```

Rationale: without required properties, `{}` validates against every schema and no validator can reject a non-conformant response (verified).

### 3. spatialID parameter declaration

The specification MUST declare `spatialID` as a path parameter on all four node operations (POST, GET, PUT, DELETE at `/wow/spatial/{spatialID}/node/{nodeId}`):

```yaml
# To be added to each of the four node operations' parameters lists
- name: spatialID
  in: path
  description: ID of the spatial composition graph
  required: true
  schema:
    type: integer
```

Rationale: OpenAPI 3.0.4 requires every template variable to be declared; the current spec is invalid without this declaration (verified).

### 4. DELETE parameter spelling correction

The DELETE node operation MUST spell its path parameter `nodeId` (camelCase), matching the path template `{nodeId}`:

```yaml
# Line 248: change 'nodeid' to 'nodeId'
- name: nodeId
  in: path
  description: node id to delete
  required: true
  schema:
    type: integer
```

Rationale: OpenAPI matches parameter names to template variables by exact string; `nodeid` does not match `{nodeId}` (verified).

### 5. Graph path adjudication

The working group MUST adjudicate the README vs API.yaml path contradiction and correct the losing side. Open Spatial Lab's recommendation: the README SHOULD be corrected to reference `/wow/spatial/` instead of `/wow/scene/`, because the API.yaml definition is the machine-readable, generated, and testable artifact.

Rationale: two implementations that chose different paths cannot interoperate (verified).

### 6. GeoPose.position.lan correction

The specification SHOULD correct `lan` to `lon` in the next schema revision. During a transition period, implementations SHOULD accept both `lan` and `lon` as the longitude field name:

```yaml
# Proposed transition schema fragment
position:
  type: object
  properties:
    lat:
      type: number
    lon:
      type: number
      description: "Longitude. Replaces the previous field name 'lan'."
    lan:
      type: number
      deprecated: true
      description: "Deprecated. Use 'lon'. Accepted during transition."
    h:
      type: number
```

Rationale: `lan` is not a recognized abbreviation for longitude in any geospatial standard or library; the misspelling creates friction for every new implementer (verified).

### 7. Core Requirement descriptions

The specification MUST complete the Preview description sentence and MUST add Description cells for Persist world and Share world. Proposed text (inferred from the Feature column and the specification's own structure):

- **Preview world:** "Experience the world without joining as a user; read-only observation."
- **Persist world:** "Store or bookmark the world URL for later return."
- **Share world:** "Send the world URL to a second user to invite them."

Rationale: a truncated sentence and two empty cells force every implementer to guess the same meaning independently (verified).

### 8. Extension policy (optional)

The specification SHOULD define an extension policy. Proposed text (inferred from Open Spatial Lab's implementation):

- Extensions MUST be additive: an extension MUST NOT remove, rename, or redefine a canonical field.
- Extensions MUST be labeled: each extension field SHOULD carry a tag or namespace prefix that distinguishes it from canonical fields.
- Extensions SHOULD be surfaced: a conformance-reporting mechanism SHOULD list which extensions are active in a response.

This is an optional addition. The specification can function without it, but without a declared policy, implementations extend silently and conformance claims become unreliable (inferred).

### 9. Aspect identifier syntax (optional)

The specification SHOULD define the syntax of aspect identifiers used in the URL fragment grammar (`URL#join=aspect.id`). The specification MUST define what a client does when an aspect identifier does not resolve. Proposed text (inferred from Open Spatial Lab's implementation):

- An aspect identifier MAY be qualified with a kind prefix: `user/{id}` or `node/{id}`.
- An unqualified identifier SHOULD be resolved by probing the known resource kinds in a defined order.
- An unresolvable aspect identifier MUST NOT cause a client error; the client SHOULD proceed with the intent alone and MAY issue a warning.

Rationale: without defined syntax, two implementations could parse the same fragment differently; without failure semantics, an unresolvable aspect could crash, warn, or be silently ignored (inferred).

---

## Adoption path

**What stays valid for a minimal world.** An existing implementation that serves the six schemas with their current field names remains valid. The `required` property additions constrain responses that were previously unconstrained, so an implementation that already returns an `id` field on its resources (as the schema structures suggest) will pass the new constraint. The `lan` to `lon` transition preserves backward compatibility during the deprecation window.

**What a client must do.** A conformant client MUST accept both `lan` and `lon` as the longitude field during the transition period. A client MUST use `/wow/spatial/` (not `/wow/scene/`) once the path is adjudicated. A client that parses `aspect.id` fragments SHOULD accept both qualified (`user/4182`) and unqualified (`4182`) forms.

**What a server must do.** A conformant server MUST declare `spatialID` on all node operations in its OpenAPI definition. A server MUST return responses where required properties (at minimum `id` for resources that have one) are present. A server SHOULD emit `lon` as the longitude field and MAY also emit `lan` during the transition period.

---

## Open questions for the working group

1. **Graph path:** which is normative, the README's `/wow/scene/` or the API.yaml's `/wow/spatial/`? Both cannot be right. Open Spatial Lab adopted API.yaml; the simpleWorlds reference implementation followed the README.

2. **Longitude field name:** should the corrected field be `lon` or `longitude`? Is a deprecation period for `lan` acceptable, or must backward compatibility be preserved indefinitely?

3. **Required properties:** which properties beyond `id` should be required on each schema? The proposals above are minimal (inferred from the schema structure); the group may want to require more.

4. **Conformance test corpus:** does the group intend to publish a set of test vectors (valid and invalid responses) as part of the specification? Open Spatial Lab offers its conformance harness and fixtures as seed material.

5. **Extension policy:** should the specification define a formal extension mechanism, or is a convention (such as `x-` prefixed fields) sufficient?

6. **Aspect identifier syntax:** should aspect identifiers be typed (`user/4182`), opaque (`4182`), or both? What is the resolution order when an identifier is ambiguous?

---

## Sources

- WebOfWorlds/WoWAPI, commit d39a1a0, `specification/OpenSpatialWorld/API.yaml`: https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml
- WebOfWorlds/WoWAPI, commit d39a1a0, `specification/OpenSpatialWorld/README.md`: https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md
- OpenAPI Specification 3.0.4, Section 4.7.1 (Path Templating): https://spec.openapis.org/oas/v3.0.4
- RFC 2119, Key words for use in RFCs to Indicate Requirement Levels: https://www.rfc-editor.org/rfc/rfc2119
- Open Spatial Lab, `wow-spec/schema.yaml` (parameter repair and lan misspelling): container-internal; public release at https://github.com/grigb/open-spatial-lab
- Open Spatial Lab, `wow-spec/OSL-WOW-CONTRACT.md` (graph path decision, extension policy): container-internal; public release at https://github.com/grigb/open-spatial-lab
- Open Spatial Lab, `docs/WORKING-GROUP-DOSSIER.md` (conformance analysis, core requirement interpretations): container-internal; public release at https://github.com/grigb/open-spatial-lab
- Open Spatial Lab, `web/wow-url.mjs` (aspect identifier resolution): container-internal; public release at https://github.com/grigb/open-spatial-lab

---

## Change log

- 2026-09-07: first public draft, verified.
