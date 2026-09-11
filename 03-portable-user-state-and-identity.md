# Portable User State and Identity

WoWAPI 0.0.1 defines a User resource and a separate OpenUserManifest resource, but does not bind signed assertions, holder control and destination admission into one cross-world assurance profile. The March 31, 2026 whitepaper already discusses existing web authentication/encryption and user-controlled disclosure. This chapter proposes optional portable assertions, an explicit assurance policy for age restrictions, and defined entry roles. A valid signature proves integrity of the signed bytes under a key; it does not by itself establish truthful age, real-world identity or permission to enter.

**Specification examined at commit d39a1a0 (WebOfWorlds/WoWAPI main, checked 2026-09-07).**


## What the specification says today

The User schema occupies lines 351--380 of API.yaml. Its complete property set is:

- `id` (number)
- `name` (string)
- `AvatarURI` (string)
- `geoPose` (object: position.lat, position.lan, position.h, angles.yaw, angles.pitch, angles.roll)

(Source: `specification/OpenSpatialWorld/API.yaml`, lines 351--380, commit d39a1a0.)

No property is marked `required`. The `AvatarURI` field is a bare string with no format constraint.

The separate OpenUserManifest 0.0.1 specification uses OpenAPI 3.0.3. Its `UserManifest.content` object contains `name`, `age` and `avatarAssetURI`. `HEAD /` authorizes access to that manifest resource and checks its ETag: 200 means access allowed, other responses mean unauthorized, and redirects are not followed. These are existing resource-access semantics. They do not establish a visitor's identity, holder control, truthful age or destination admission. OpenSpatialWorld User does not reference this resource.

The World schema declares `age_restriction`, but OpenSpatialWorld does not bind it to a verified visitor-age assertion or to an admission procedure. The separate manifest age is a numeric claim, not age assurance by itself.

The spec defines `DELETE /wow/user/{userId}` (lines 67--85, operationId `deleteUserById`) with responses 200 ("User deleted"), 400 ("Invalid user value"), and a default catch-all ("Unexpected error"). No request body, no required properties.

The README Core Requirements table says visitors join a world "as new or existing user on a given device and UA" (README.md, line 11). The spec never states what role or embodiment that visitor starts in.

**Source boundary.** The reviewed OpenSpatialWorld API and README contain no user-signature profile or holder-challenge/admission binding. Keyword absence is not absence of identity architecture: the full whitepaper discusses web authentication and selective disclosure, and OpenUserManifest already has resource authorization as described above.


## What fails without it

**No interoperable visitor-assurance binding.** A User id and name do not tell a receiver who asserted them, whether the assertion is trusted for a purpose, or whether the presenter currently controls the subject key. Worlds can use existing authentication systems, but the reviewed API does not specify how to carry and evaluate those results during crossing.

**An age field is not age assurance.** The separate manifest provides a place for a declared age, but neither that number nor its signature establishes its truth. A destination needs a chosen issuer/assurance policy, holder binding where required, and an admission rule. A self-declared value can support a low-assurance policy only when it is labeled as such.

**A client joining a world does not know what the visitor is.** The README says a visitor joins "as new or existing user" but never says whether the visitor is embodied, what avatar they wear by default, or whether they are a spectator, a player, or something else. Two implementations could assign different default states to the same join URL, producing different visible behaviour for the same action.


## What Open Spatial Lab built and learned

Open Spatial Lab extended the User resource with three additions, each labeled as a non-canonical extension (`x-osl-extension: true`) and served alongside the canonical User properties without removing any of them.

**1. OSLUserResponse and OpenUserManifest (CM-027, CM-029).** The extended GET /wow/user/{userId} response wraps the canonical User in an OSLUserResponse that adds a `proof_boundary`, a `webofworlds_extension`, and an `open_user_manifest`. The manifest carries name, age, and avatarAssetURI (matching the name in the WoW OpenUserManifest sub-specification, which differs from the OpenSpatialWorld User schema's AvatarURI). The manifest is the content that gets signed. When a signer is available, the `signature` field holds a UMSignature block; when no signer is available, it holds null, which the schema documents as "honest unsigned/unverified degradation" (source: `schema.yaml`, lines 959--982 and 1064--1084).

**2. UMSignature byte-integrity verification (CM-028).** Profile A uses Ed25519 over JCS-RFC8785 canonical bytes and binds key resolution/consistency to a `did:key` identifier. Its schema carries algorithm, canonicalization, keyRef, publicKeySpkiB64, created and value. It is separate from the `.msf` RS256/x5c signing profile. September 7 runs passed 91 supplied signing vectors and 35 supplied adversarial/profile cases. The retained cases expose raw JSON parsing and canonicalization input assumptions. Profile A rejects unsafe signing-input numbers by default during verification; signing's numeric guard is optional. Strict text parsing, including duplicate-key rejection, is a separate integration requirement. A bounded probe verified a self-asserted age of 99 and a copied manifest, and rejected altered signed content. These results establish their signature-profile scope, not age truth, fresh holder control or a general security guarantee.

**3. World-entry interpretation I10 (CM-030).** OSL interpreted a bare /w/{world} URL as a world entry that boots an embodied player role. An explicit ?role= parameter overrides this default. The interpretation follows from the README's Core Requirement that visiting a world URL means joining the world "as new or existing user," combined with the judgment that OSL's legacy inspector windows (?role=source or ?role=target) are developer surfaces and must not be the default (source: `wow-url.mjs`, lines 534--540).

**The claim boundary:** these are non-canonical local extensions, with `standards_conformance: false`. Other Universal Manifest policy modules exist, but the cited signing checks do not bind them into visitor assurance or admission. In the local portal controller, a failed manifest-verification result is logged and does not itself prevent target promotion. The signed-fabric refusal path is separate, as described in [chapter 04](04-provenance-and-signed-subtrees.md).

**DELETE /wow/user/{userId} (CM-031).** The retained Open Spatial Lab contract and coverage evidence record this operation as contracted but not served, with a local 404. This is a limit of that implementation; it is not a claim about all other implementations, which were not tested.


## Proposed normative text

All additions below are unadopted proposals. Schema fragments target OpenAPI 3.0.4; the existing OpenUserManifest document remains OpenAPI 3.0.3. No signature choice or assurance policy is adopted by this report.

**N1. Optional portable assertion manifest on User.**

A future User extension could support an optional `identity_manifest` property for portable assertions. The name is illustrative; it does not replace canonical User.id or adopt an assurance policy.

```yaml
User:
  type: object
  properties:
    identity_manifest:
      type: object
      description: >
        A signed portable assertion document. Signature verification
        checks signed bytes and key consistency; trust in claims,
        holder control and admission require a separate profile.
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

Rationale: a signed assertion can be checked without a bilateral authentication channel, but the receiver still selects which issuers and claims to trust. A copied valid document does not prove that its presenter controls the subject key. The profile must also bind signed-byte scope, holder challenges, freshness/replay rules and selective disclosure before making an identity-assurance claim.

**N2. Identity signature profile.**

The standard could define an optional signature profile for portable assertions. Ed25519, JCS-RFC8785 and did:key are the local candidate. The fragment below records its proposed metadata shape; schema validation alone does not verify a signature or its signed-byte scope.

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

Rationale: these choices match the local signature candidate. JCS-RFC8785 defines a deterministic signing representation; the selected did:key method derives a key from its identifier. Neither a key format nor canonicalization establishes issuer trust.

**N3. Age field on User.**

A profile may carry an optional declared `age`, but should first resolve whether to reuse `UserManifest.content.age`, reference a purpose-specific credential, or duplicate a value on User. If both User and manifest values appear, the assurance profile must identify the authority and handle conflict. A self-declared age does not establish eligibility for age-restricted content.

```yaml
User:
  type: object
  properties:
    age:
      type: number
      description: >
        The user's declared age. When a World declares age_restriction,
        an admission policy may evaluate the assertion at its
        declared assurance level; this value does not prove age.
```

Rationale: the existing age restriction needs a defined admission policy, trusted assertion source and data-minimization rule. Adding a number only provides data representation. An over-threshold assertion may avoid disclosing exact age if the chosen profile supports it.

**N4. Default role and embodiment for world entry.**

The specification SHOULD define the default embodiment and role of a visitor who joins a world via a bare URL (no fragment, no query parameter). At minimum, the spec SHOULD state whether a bare-URL visitor is embodied (has a visible avatar) or is a spectator.

Rationale: two implementations that assign different default roles to the same join URL will produce visibly different behaviour for the same action, breaking interoperability at the most basic interaction: entering a world.

**N5. DELETE behavior clarification (optional editorial proposal).**

`userId` is already a required path parameter in the pinned API. No new requirement is needed to make it required. The group may clarify how missing users differ from malformed identifiers and whether deletion is idempotent; the current 200, 400 and default responses do not fully specify those cases. A proposed 404 rule would need to be distinguished from Open Spatial Lab's unsupported-route 404.

## Permission lifetime during ongoing operation

Portable identity is useful before, during and after a crossing. A person or software actor can remain in one world while their permission to read, modify or execute changes. A versioned profile must therefore bind more than an entry presentation. The following extends the proposal; it is not behavior established by the pinned OpenUserManifest schema or local signing vectors.

Keep the actor, operator, resource publisher and state authority distinct. A manifest can reference evidence about them. An authorization service or destination policy then decides which actor may perform which operation on which resource, for how long and under what conditions. If an actor is delegated to act for another party, the request must bind that delegation and its scope. A self-declared software type does not establish the delegation.

The profile should define expiry and revocation at resource reads, action requests, subscriptions and execution boundaries. A client holding a previously valid statement must not assume an indefinite grant. Define the permitted caching/freshness interval, the result when status cannot be checked and the cleanup required when a grant ends. Revocation cannot erase content already disclosed; it controls subsequent access and actions under the chosen contract.

For Universal Manifest adoption by reference, select the exact version and define the visitor/actor data actually needed. Preserve the existing manifest resource-access and ETag behavior through an explicit compatibility mapping. Keep private keys with the holder; a world receives the agreed presentation and evidence, not custody of those keys. The equipped-item, preference and delegation bindings must be evaluated explicitly instead of inferred from the existence of a general envelope.

Acceptance should compare an authorized request, a validly signed but out-of-scope request, a stale or revoked grant, and an unsupported proof method. A software actor and a human-controlled client should receive the profile's stated outcomes for equivalent grants. These are proposed behavioral tests; copying valid signed bytes, passing a HEAD request or setting a capability flag cannot satisfy them.

## Adoption path

**Existing users.** Optional assertion fields do not require a minimal world to adopt signing. Existing canonical numeric ids remain valid. Missing signatures mean no claim under this signature profile; HTTPS transport and any separately authenticated session still have their own properties.

**Receivers.** Report separate results for signed-byte integrity, issuer trust for the claim, holder control, freshness, admission and execution permission. A valid signature is input to policy, not automatic admission. The W3C Verifiable Credentials trust model likewise leaves issuer trust to the verifier.

**Issuers and holders.** The whitepaper's User Digital Wallet on printed page 15 stores the user's private keys and credentials, manages consent and builds presentations. Use that as the published user-controlled custody model. A server-held key per user is a different, custodial model: it makes a server assertion and requires the receiver to establish trust in that server. Neither model makes self-asserted name or age true. Key custody, recovery, consent, holder challenges and disclosure rules remain explicit profile decisions.

**Age restrictions.** Define the assurance requirement and authority before enforcing a restriction. Handle absent, conflicting, expired and insufficiently assured assertions explicitly. The sample age field and signing vectors do not choose that policy.

## Open questions for the working group

1. Should the identity manifest be an optional canonical field on User, or should it ride in a named extension point (as OSL implemented it)? The first is simpler for consumers; the second preserves a clean separation between the canonical schema and identity-layer additions.

2. Should the spec mandate a single signature algorithm (Ed25519) or define a negotiation mechanism for future algorithms? A fixed suite simplifies the initial profile. Negotiation needs explicit version and downgrade protection rules; a fixed algorithm alone does not establish general security.

3. Should `age` be a self-declared number, or should the spec point to a verifiable age-credential format (such as a W3C Verifiable Credential with an age claim)? A self-declared number is simple but trivially falsifiable; a credential can support stronger assurance only under an accepted issuer, holder and verification policy.

4. Should the default role for bare-URL world entry be defined in the spec or left to each world to declare? If worlds declare it, the spec needs a `default_role` field on World.

5. The DELETE /wow/user/{userId} operation is defined in the spec but the retained OSL evidence records it as unserved; other implementations were not assessed. Should the working group confirm this operation is intended to remain in the spec, or should it be deferred to a future extension?


## Sources

- [OpenSpatialWorld/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenSpatialWorld/README.md](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenSpatialWorld/README.md), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [OpenUserManifest/API.yaml](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a0/specification/OpenUserManifest/API.yaml), WoWAPI 0.0.1 at d39a1a0, checked September 7, 2026.
- [Web of Worlds whitepaper, March 31, 2026](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf); relevant printed pages are identified in this chapter or [chapter 10](10-role-and-blind-spots.md).
- [W3C Verifiable Credentials 2.0 trust model](https://www.w3.org/TR/vc-data-model-2.0/#trust-model).
- [RFC 8785, JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785) and [W3C DID Core 1.0](https://www.w3.org/TR/did-core/).
- Open Spatial Lab local source snapshot and retained evidence, checked September 7, 2026: schema.yaml, wow-url.mjs, wow-spec-coverage.mjs and um-signature-profile-a.mjs. September 7 runs passed 91 signing vectors and 35 supplied adversarial/profile cases, with two input-discipline limits. These are signature-profile checks, not identity, age or admission proof. Public reproduction of these exact local bytes is not established.
- [Appendix A](A-completion-map.md) and [Appendix B](B-findings-register.md) preserve the historical surface/finding identifiers.

## Change log

- 2026-09-07: corrected source scope, proposal compatibility and evidence boundaries; updated public citations.
