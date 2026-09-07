# Portable User State and Identity

The Web of Worlds specification defines a User resource with four properties (id, name, AvatarURI, geoPose) and no mechanism for verifying who a user is across worlds. The standard needs three additions to make user state portable: an optional signed identity manifest on the User resource so a receiving world can verify a visitor without a shared authentication backend; an age field on User so age-gated content (already declared on World) can be enforced; and a defined default role and embodiment for visitors who enter a world via a bare URL. Each addition is specified below with proposed normative text and schema fragments. The signature profile and manifest shape are high confidence (verified against a running implementation with a passing conformance suite). The default-role proposal is medium confidence (one implementation's interpretation of a silent spec, not yet tested by a second).

**Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).**


## What the specification says today

The User schema occupies lines 351--380 of API.yaml. Its complete property set is:

- `id` (number)
- `name` (string)
- `AvatarURI` (string)
- `geoPose` (object: position.lat, position.lan, position.h, angles.yaw, angles.pitch, angles.roll)

(Source: `specification/OpenSpatialWorld/API.yaml`, lines 351--380, commit d39a1a0.)

No property is marked `required`. The `AvatarURI` field is a bare string with no format constraint.

The WoWAPI repository also contains a separate OpenUserManifest sub-specification (`specification/OpenUserManifest/API.yaml`) that defines a UserManifest schema with name (string), age (number), and avatarAssetURI (string). The OpenSpatialWorld User schema does not reference the OpenUserManifest, and the OpenUserManifest defines no signature, identity verification, or required properties.

The World schema declares `age_restriction` (number, line 276) but the User schema has no `age` field. A conformant world can declare an age restriction that no conformant user can satisfy.

The spec defines `DELETE /wow/user/{userId}` (lines 67--85, operationId `deleteUserById`) with responses 200 ("User deleted"), 400 ("Invalid user value"), and a default catch-all ("Unexpected error"). No request body, no required properties.

The README Core Requirements table says visitors join a world "as new or existing user on a given device and UA" (README.md, line 11). The spec never states what role or embodiment that visitor starts in.

**Terms searched with zero results across OpenSpatialWorld/API.yaml and OpenSpatialWorld/README.md:** identity (0, 0), manifest (0, 0), signature (0, 0), DID (0, 0), did:key (0, 0), Ed25519 (0, 0), consent (0, 0), portable (0, 0), credential (0, 0). The OpenSpatialWorld sub-specification is entirely silent on identity verification, portable credentials, and cryptographic signing for users. (The OpenUserManifest sub-specification in the same repository is described above.)


## What fails without it

**A user crossing between worlds cannot prove their identity.** When a visitor arrives from World A at World B, the receiving world sees a User object with a server-local integer id and a name string. Neither field is verifiable. World B must either trust World A's claim on faith, build a private authentication channel to World A, or treat every visitor as unknown. None of these scales. The first two create bilateral coupling between every pair of worlds; the third defeats the purpose of carrying a user identity at all.

**Age-gated content cannot be enforced through the OpenSpatialWorld API.** The OpenUserManifest sub-specification includes an age field on UserManifest, but the OpenSpatialWorld User schema does not reference it, so a conformant OpenSpatialWorld server has no API path to that value. A world that sets age_restriction to 18 has no conformant way to check whether a visitor meets the restriction through the OpenSpatialWorld endpoints alone (verified: API.yaml line 276 declares age_restriction; lines 351--380 contain no age property; OpenUserManifest/API.yaml defines age on UserManifest but is not referenced by the User schema).

**A client joining a world does not know what the visitor is.** The README says a visitor joins "as new or existing user" but never says whether the visitor is embodied, what avatar they wear by default, or whether they are a spectator, a player, or something else. Two implementations could assign different default states to the same join URL, producing different visible behaviour for the same action.


## What Open Spatial Lab built and learned

Open Spatial Lab extended the User resource with three additions, each labeled as a non-canonical extension (`x-osl-extension: true`) and served alongside the canonical User properties without removing any of them.

**1. OSLUserResponse and OpenUserManifest (CM-027, CM-029).** The extended GET /wow/user/{userId} response wraps the canonical User in an OSLUserResponse that adds a `proof_boundary`, a `webofworlds_extension`, and an `open_user_manifest`. The manifest carries name, age, and avatarAssetURI (matching the name in the WoW OpenUserManifest sub-specification, which differs from the OpenSpatialWorld User schema's AvatarURI). The manifest is the content that gets signed. When a signer is available, the `signature` field holds a UMSignature block; when no signer is available, it holds null, which the schema documents as "honest unsigned/unverified degradation" (source: `repo/open-spatial-lab/wow-spec/schema.yaml`, lines 959--982 and 1064--1084).

**2. UMSignature identity verification (CM-028).** The signature uses Ed25519 over JCS-RFC8785 (RFC 8785 JSON Canonicalization Scheme) canonical bytes, bound to a `did:key` DID URL. The schema defines six fields: algorithm (must be "Ed25519"), canonicalization (must be "JCS-RFC8785"), keyRef (a did:key DID URL), publicKeySpkiB64 (base64 SPKI DER Ed25519 public key), created (ISO 8601), and value (base64url Ed25519 signature over the canonicalized signing input). This is called "UM Signature Profile A" and is separate from the .msf RS256/x5c engine spine used for signed spatial documents (source: `repo/open-spatial-lab/wow-spec/schema.yaml`, lines 932--957). The signing suite has a passing conformance check (91 checks, 35 attack vectors resisted; reported: `repo/open-spatial-lab/docs/ecosystem/layers/universalmanifest.md`, section 4).

**3. World-entry interpretation I10 (CM-030).** OSL interpreted a bare /w/{world} URL as a world entry that boots an embodied player role. An explicit ?role= parameter overrides this default. The interpretation follows from the README's Core Requirement that visiting a world URL means joining the world "as new or existing user," combined with the judgment that OSL's legacy inspector windows (?role=source or ?role=target) are developer surfaces and must not be the default (source: `repo/open-spatial-lab/web/wow-url.mjs`, lines 534--540).

**The claim boundary:** these are labeled non-canonical extensions verified in one implementation. They do not carry a conformance claim against the Web of Worlds specification (every OSL response includes `standards_conformance: false`). The signature suite aligns to the UniversalManifest Signature Profile A specification and has its own conformance checks, but UM itself has not been adopted by any other project in the ecosystem (reported: `repo/open-spatial-lab/docs/ecosystem/layers/universalmanifest.md`, section 2).

**DELETE /wow/user/{userId} (CM-031).** OSL contracted this operation in its schema but does not serve it on the wire. A live DELETE /wow/user/1 returns 404 (verified: `repo/open-spatial-lab/web/wow-spec-coverage.mjs`, lines 256--274). This is the only defined spec operation that OSL does not serve. The gap is acknowledged and tracked.


## Proposed normative text

**N1. Optional signed identity manifest on User.**

A User resource SHOULD support an optional `identity_manifest` property containing a signed identity document.

```yaml
User:
  type: object
  properties:
    identity_manifest:
      type: object
      description: >
        A signed identity document for cross-world verification.
        When present, the receiving world can verify the user's
        identity without a shared authentication backend.
      properties:
        name:
          type: string
        age:
          type: number
        avatarAssetURI:
          type: string
          format: uri
        signature:
          $ref: '#/components/schemas/IdentitySignature'
```

Rationale: a world that receives a visitor from another world needs a way to verify the visitor's identity claim without contacting the origin world. An attached signed manifest, verified against a public key, satisfies this without bilateral coupling.

**N2. Identity signature profile.**

The standard SHOULD define a signature profile for user identity verification. The profile SHOULD use Ed25519 with JCS-RFC8785 canonicalization and a did:key identifier.

```yaml
IdentitySignature:
  type: object
  required:
    - algorithm
    - canonicalization
    - keyRef
    - value
  properties:
    algorithm:
      type: string
      enum: [Ed25519]
    canonicalization:
      type: string
      enum: [JCS-RFC8785]
    keyRef:
      type: string
      description: A did:key DID URL identifying the signer.
    publicKeySpkiB64:
      type: string
      description: Base64-encoded SPKI DER Ed25519 public key.
    created:
      type: string
      format: date-time
    value:
      type: string
      description: Base64url Ed25519 signature over the JCS-RFC8785 signing input.
```

Rationale: Ed25519 is widely implemented, compact, and does not require a certificate authority. JCS-RFC8785 provides deterministic JSON canonicalization for signing. did:key allows offline resolution of the signer's public key.

**N3. Age field on User.**

User SHOULD include an optional `age` property (number) so that age-gated worlds (those declaring `age_restriction` on the World resource) can enforce their restriction.

```yaml
User:
  type: object
  properties:
    age:
      type: number
      description: >
        The user's declared age. When a World declares age_restriction,
        a conformant server MAY reject users whose age is below
        the restriction.
```

Rationale: the World schema already declares `age_restriction`. Without a corresponding field on User, the restriction is unenforceable through the API.

**N4. Default role and embodiment for world entry.**

The specification SHOULD define the default embodiment and role of a visitor who joins a world via a bare URL (no fragment, no query parameter). At minimum, the spec SHOULD state whether a bare-URL visitor is embodied (has a visible avatar) or is a spectator.

Rationale: two implementations that assign different default roles to the same join URL will produce visibly different behaviour for the same action, breaking interoperability at the most basic interaction: entering a world.

**N5. Required properties and RFC 2119 keywords on DELETE /wow/user/{userId} (optional, spec hygiene).**

The DELETE /wow/user/{userId} operation SHOULD declare `userId` as a required path parameter using RFC 2119 language. A conformant server MUST return 404 when the user does not exist (not 400, which the current spec returns for "Invalid user value" without defining what makes a value invalid).

Rationale: the current response set (200, 400, and a default catch-all) does not distinguish a missing user from an invalid parameter. A 404 for a missing user is standard HTTP semantics.


## Adoption path

**For a minimal world (no portal crossings, no signed identity):** nothing changes. The User schema gains optional properties. An existing server that returns only id, name, AvatarURI, and geoPose remains valid.

**For a client that consumes identity manifests:** the client checks whether `identity_manifest` is present on a User response. When present, it verifies the signature against the keyRef. When absent, the client treats the user as unverified (the current implicit state, now made explicit). The client does not need to implement signing, only verification.

**For a server that issues identity manifests:** the server generates or stores an Ed25519 key pair per user, canonicalizes the manifest content with JCS-RFC8785, signs it, and attaches the result as `identity_manifest.signature`. The server also declares a did:key identifier for the user's public key.

**For age gating:** a server that declares `age_restriction` on its World reads the `age` field from the visiting user's manifest (or from the User resource directly). If the field is absent, the server decides its own policy (admit, reject, or prompt). The spec does not mandate a specific enforcement mechanism beyond making the field available.


## Open questions for the working group

1. Should the identity manifest be an optional canonical field on User, or should it ride in a named extension point (as OSL implemented it)? The first is simpler for consumers; the second preserves a clean separation between the canonical schema and identity-layer additions.

2. Should the spec mandate a single signature algorithm (Ed25519) or define a negotiation mechanism for future algorithms? A single algorithm is simpler and avoids downgrade attacks; a negotiation mechanism accommodates future cryptographic changes.

3. Should `age` be a self-declared number, or should the spec point to a verifiable age-credential format (such as a W3C Verifiable Credential with an age claim)? A self-declared number is simple but trivially falsifiable; a verifiable credential is stronger but adds a dependency.

4. Should the default role for bare-URL world entry be defined in the spec or left to each world to declare? If worlds declare it, the spec needs a `default_role` field on World.

5. The DELETE /wow/user/{userId} operation is defined in the spec but no known implementation serves it. Should the working group confirm this operation is intended to remain in the spec, or should it be deferred to a future extension?


## Sources

- WebOfWorlds/WoWAPI, commit d39a1a0 (2026-05-21): `specification/OpenSpatialWorld/API.yaml`, `specification/OpenSpatialWorld/README.md`. Repository: https://github.com/WebOfWorlds/WoWAPI
- Open Spatial Lab schema extensions: `repo/open-spatial-lab/wow-spec/schema.yaml` (lines 932--957, 959--982, 1064--1084)
- Open Spatial Lab world-entry interpretation: `repo/open-spatial-lab/web/wow-url.mjs` (lines 534--540)
- Open Spatial Lab spec-coverage audit: `repo/open-spatial-lab/web/wow-spec-coverage.mjs` (lines 256--274)
- UniversalManifest layer document: `repo/open-spatial-lab/docs/ecosystem/layers/universalmanifest.md`
- Open Spatial Lab working-group dossier: `repo/open-spatial-lab/docs/WORKING-GROUP-DOSSIER.md`
- Web of Worlds Completion Map (2026-09-07): rows CM-027 through CM-031
- Findings and Recommendations (2026-09-07): rows R-020 (avatar.body), R-021 (identity.root)


## Change log

2026-09-07: first public draft, verified.
