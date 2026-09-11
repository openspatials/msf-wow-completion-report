# Method and Sources

This report compares the published Web of Worlds architecture with a pinned application programming interface (API), draws on a local implementation, and offers explicit proposals for shared bindings. Source inspection, executed tests, historical receipts and proposed behavior are different kinds of evidence. The report does not establish independent cross-engine, network-scale or production acceptance.

## How the comparisons were made

Each comparison starts with the publisher's actual position, including architecture outside the machine-readable API. The current binding is then read in the pinned API files and their resource-specific READMEs. Local evidence can show why a binding matters, but one implementation's convention does not become a standards requirement.

| Evidence class | What it establishes | Boundary |
|---|---|---|
| Published architecture | A goal, relationship, use case or intended behavior at the cited page/section. | Does not establish a complete wire contract, adoption or implementation. |
| API binding | A method, path, field, type, response or README provision at the pinned commit. | Does not supply every behavior mentioned by a publication. |
| Executed local check | A named input produced the recorded result in the stated environment. | Does not establish behavior outside the tested inputs or independent implementations. |
| Historical local receipt | A retained record describes a particular earlier execution. | Is not a fresh rerun, a similarly named current file or a reliability estimate. |
| Proposal or open decision | Candidate text, an optional extension, an informative implementation choice or an acceptance test. | Does not imply group adoption or completion of the test. |

Searches helped locate material. A missing word was not used to prove missing architecture or behavior. Extra properties are permitted by the reviewed resource schemas, and existing type constraints can reject invalid data. `allOf` applies every member's constraints; it has no override order. These interpretations follow the [OpenAPI 3.0.4 Schema Object](https://spec.openapis.org/oas/v3.0.4.html#schema-object).

## Named source set and September 7 baseline versions

The [source-by-source account](11-published-positions-and-current-state.md#source-by-source-coverage-account) maps the complete bodies of the following sources. References linked from those works are background, not automatically additional publications in this comparison.

| Source | Version and check |
|---|---|
| [Full whitepaper](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf) | March 31, 2026; 33 pages. September 7, 13:37 UTC PDF bytes match the earlier same-day substantive capture. SHA-256 `a43200294fb15ca6bfa80f93a1ee810eba2c010f76a88d39affde0cfd179110a`. All 33 pages were read. The source diagrams carrying world, graph, resolver and service relationships were checked visually; the September 7 revision rechecked pages 14, 17, 21, 22, 24 and 27. |
| [Executive summary](https://webofworlds.github.io/initial_MSF_Whitepaper/) | Labelled 2026-Q1; distinct from the full paper. September 7, 13:37 UTC HTML SHA-256 `6d6f04018e186aa41a5ee3d305d96983dec06a20c0ea361ce5fa13cf7ee4da72`. Source repository HEAD was `988f369b0af206de0a7b53e7903dd55e08be80a0`. |
| [Announcement post](https://metaverse-standards.org/news/blog/announcing-the-web-of-worlds-whitepaper-a-concrete-path-to-the-open-metaverse/) | Visible June 2, 2026; publication metadata `2026-06-03T01:40:32+00:00`. September 7, 13:37 UTC HTML SHA-256 `40155229ff087b6a2ac6bfb1f223dda33930cc7f6cbe78109d300425026664c7`. All article sections, including Next Steps, were read. |
| [Linked Spatial Experiences post](https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/) | April 2, 2025; metadata modified September 4, 2025. September 7, 13:37 UTC HTML SHA-256 `5446deb416ea2903640057b53d14d6860505df92ae439a16d2f9242aca73a7f9`. All requirement groups, examples and roadmap text were read. |
| [WebOfWorlds home](https://webofworlds.github.io/) | September 7, 13:37 UTC HTML SHA-256 `52af48d6e82b2f62d3055def154ead83c8e0b7190c130d6a44995c2ef675d3d4`; site repository HEAD `6f14cd37823bd7c4c698f94a97893366e4dc07f3`. Includes both implementation tables and architecture diagrams. March 31 dates the linked paper, not all home-page content. |
| [WoWAPI](https://github.com/WebOfWorlds/WoWAPI/tree/d39a1a009aa4ef8fb6d14aa66d588cffb74c33de) | API version 0.0.1, commit `d39a1a009aa4ef8fb6d14aa66d588cffb74c33de`, May 21, 2026. Upstream HEAD matched in the September 7, 13:38 UTC check. All three API files, five READMEs and architecture diagrams were included. |
| [simpleWorlds](https://github.com/WebOfWorlds/simpleWorlds) | Bounded context: the public README API table and API path declaration at `13d2cbea991e17df0b14857f11f693712e6171cb`, fetched September 7, 20:51 UTC. Only declared paths are compared. The earlier HEAD check returned `d2bda3e2e73097c6e36ae0fd65a935cb3064ce71`; implementation internals and conformance are not assessed at either version. |

The three API files are [OpenSpatialWorld](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a009aa4ef8fb6d14aa66d588cffb74c33de/specification/OpenSpatialWorld/API.yaml), [OpenSpatialAsset](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a009aa4ef8fb6d14aa66d588cffb74c33de/specification/OpenSpatialAsset/API.yaml) and [OpenUserManifest](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a009aa4ef8fb6d14aa66d588cffb74c33de/specification/OpenUserManifest/API.yaml). World declares OpenAPI 3.0.4; Asset and User Manifest declare 3.0.3. The five READMEs are at repository root, `specification/`, and those three resource directories. Fragment examples and optional `/wow/scene/` exposure appear in the [World README](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a009aa4ef8fb6d14aa66d588cffb74c33de/specification/OpenSpatialWorld/README.md#L5-L31), not the top-level README.

The September 7 home capture lists Open-Spatial-Lab among its world implementations. Source checks establish the inspected versions, not delivery of every roadmap milestone or published implementation level. HTML hashes bind exact captures; navigation changes can alter those bytes without changing article text.

The path comparison reads the [World README](https://github.com/WebOfWorlds/WoWAPI/blob/d39a1a009aa4ef8fb6d14aa66d588cffb74c33de/specification/OpenSpatialWorld/README.md#L17-L31), [paper p29](https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf#page=29), and [simpleWorlds API](https://github.com/WebOfWorlds/simpleWorlds/blob/13d2cbea991e17df0b14857f11f693712e6171cb/packages/wow-spec/src/schema.yaml#L124-L228) as distinct source artifacts. All use `scene`; the canonical World YAML uses `spatial` and includes `spatialID`. This comparison reads declarations only. It makes no claim about simpleWorlds handler order, socket validation, persistence or runtime behavior.

## September 11 source refresh and architecture revision

Fresh checks on September 11, 2026 at 00:32–00:34 UTC retrieved 25 public source artifacts and five repository HEAD values. These establish the inspected versions. They do not test the implementations or verify a roadmap milestone. Initial Python HTTPS requests failed local certificate-chain verification; subsequent system-curl requests used normal TLS verification and succeeded. Failed transport attempts supply no source evidence.

| Source | Fresh result |
|---|---|
| WoWAPI | HEAD remains `d39a1a009aa4ef8fb6d14aa66d588cffb74c33de`; the three API files and five READMEs were retrieved from main alongside that matching HEAD check. |
| Whitepaper and summary | Source HEAD remains `988f369b0af206de0a7b53e7903dd55e08be80a0`. The PDF and summary hashes match the baseline table above. The existing substantive page comparison therefore remains applicable to those exact bytes. |
| WebOfWorlds home | Source HEAD remains `6f14cd37823bd7c4c698f94a97893366e4dc07f3`; HTML SHA-256 remains `52af48d6e82b2f62d3055def154ead83c8e0b7190c130d6a44995c2ef675d3d4`. Implementation entries remain publisher declarations. |
| Announcement post | Current HTML SHA-256 `6befd47130c2d7ce7c901c45824a65a21997e13c7cf42e7a4cd046317c800268`. Bytes differ from September 7. The current article, including Next Steps and roadmap language, was read against this report's source-by-source account. Its forward-looking language remains a published position, not proof of delivery. |
| Linked Spatial Experiences post | Current HTML SHA-256 `02c8b291e19b8f19e3608637b902d612782ea08392c24988158265fb60e1ec48`. Bytes differ from September 7. The current principles, requirements, examples and roadmap were read against the comparison. Byte changes are recorded without assuming that only navigation changed. |
| simpleWorlds | HEAD remains `d2bda3e2e73097c6e36ae0fd65a935cb3064ce71`. The older pinned README/API declarations remain the basis of the bounded path comparison; no implementation assessment is added. |
| IWPS | Repository HEAD `f09c0da9e50a8dfe2e49e2266cf397973d41ba8a`; published Base Specification retrieved. It remains a candidate interface reference. |

The refresh also retrieved the cited [Universal Manifest build documentation](https://universalmanifest.net/build/), [GeoPose 1.0 standard](https://docs.ogc.org/is/21-056r11/21-056r11.html), [glTF 2.0 specification](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html), [WebXR Anchors draft](https://immersive-web.github.io/anchors/), [Verifiable Credentials Data Model 2.0](https://www.w3.org/TR/vc-data-model-2.0/), [did:key method](https://w3c-ccg.github.io/did-key-spec/), [HTTP semantics](https://www.rfc-editor.org/rfc/rfc9110) and [BCP 14 clarification](https://www.rfc-editor.org/rfc/rfc8174). Publication status and scope differ: a draft, profile, format specification and project README are not interchangeable standards or implementation receipts. A Universal Manifest profile/version and its compatibility rules would need explicit agreement before adoption by reference.

This revision compares the whole architecture across stationary and distributed entities, client rendering and behavior, ongoing interaction, live state, authority and persistence, human and software actors, discovery, placement and conformance. [Chapter 10](10-role-and-blind-spots.md#the-whole-architecture-and-its-connections) supplies the map and specialist responsibilities. Chapters 03 and 05–09 add the missing operating contracts and candidate acceptance cases. The stationary workshop and ongoing shared-operation scenarios are proposals; they were not implemented or executed by this document revision. The historical portal evidence retains its original scope.

The runtime-comparison corpus and ecosystem entity documents were used as research indexes. Their coverage is unequal: read-only upstream analysis, bounded local checks and retained browser receipts establish different things. The corrected publication comparison in this report supersedes older descriptions that treated WoW as having no normative specification. A specialist's documented feature identifies a possible interface responsibility, not proven interoperability with WoW. No upstream clone was changed and no new architecture implementation is claimed.

## September 11 document validation

The retained OpenAPI 3.0 schema validator was also run against the current report using its preserved validation environment. It passed 26 schema fragments, 90 positive/negative instance cases, four GeoPose schema cases, four proposed longitude-conflict cases, three URI-resolution examples and the units-ratio example. It also verified the 73 selected surface rows, 25 historical findings and their retained status distribution. A first invocation in the default Python environment stopped before validation because its URI-format checker was unavailable; using the existing equipped environment resolved that setup problem without changing assertions or installing a new dependency.

These are document/schema checks. They do not exercise an HTTP service, live-state recovery, cross-engine behavior, cryptographic trust or the new proposed scenarios. The historical implementation results below remain separately dated and bounded.

## Register units and coverage

Document 11 contains **76 active public-source comparison entries**, retaining **80 public identifiers** and **four explicit aliases**. Original public identifiers 1–44 survive; additions are 45–80; aliases are 31→5, 32→19, 34→6 and 35→7. **Five non-public context identifiers, G1–G5, retain their original names but contain no meeting material in this public edition.**

An entry groups related subclaims when they share a binding question and evidence boundary. Distinct API operations and the five spatial-computing operators remain separate. These are comparison entries, not atomic requirements. The page/section account exposes the grouping and accounts for context such as bibliography entries. The former 49-row total mixed public entries and meeting notes; its 20/17/12 classification is retired.

The count does not measure standards completeness. The eleven chapters and two appendices organize the report. Appendix A's selected specification surfaces and Appendix B's findings are documentation units, not independent interoperability outcomes.

## Executed checks on September 7

The checks below ran earlier on September 7. Their retained outcomes were reconciled in the accepted September 7 content baseline. The September 11 revision preserves these limits and rechecks current public sources and document consistency; it does not claim a fresh execution of those behavior/signing suites or a live demonstration.

| Check | Recorded outcome | Evidence boundary |
|---|---|---|
| Canonical World component schemas | All six accepted `{}` and an extra property; all six rejected an array. A Node with string `id` was rejected. | Shared schema subset, not full OpenAPI-document or HTTP validation. Existing types and endpoint constraints still matter. |
| `allOf` counterexample | Neither a number nor a string satisfied conflicting `id` constraints, in either member order. | Demonstrates conjunction, not precedence. |
| Coordinate counterexample | Three power-of-two scaling choices retained 16,384-metre float32 spacing at the tested astronomical distance. Subtracting a nearby origin before casting retained the one-metre difference. | Refutes universal precision restoration by scaling; does not select a renderer strategy. |
| Presence controller in isolation | The controller used an in-memory transport that lost the explicit departure request and then registered at the destination; both simulated registries retained the user. | The probe bypassed the source server’s exit-intent handler. It tests the isolated controller, not the integrated crossing path and not an atomic transfer between servers. |
| Manifest copy/tamper | Original and copied signed bytes verified; altered content failed. | Signed-byte integrity and key consistency, not attested age, holder control or admission. |
| URL composition | A query-bearing URL parsed; blindly appended path text landed in its query. | Wrong endpoint composition, not invalid syntax. Relative references need a defined base. |
| Supplied schema fixtures | 44 of 44 passed. | Limited evaluator and declared-divergence cases, not full OpenAPI or live HTTP validation. |
| Supplied signing vectors | 91 of 91 passed. | Local signing profile and supplied inputs, not general identity or policy assurance. |
| Supplied adversarial/signing-profile cases | 35 supplied expected/resisted cases passed, with two input-discipline boundaries reported. | The log states that safe-integer range and duplicate-key rejection need input controls; those guards are not established by this result. No general security guarantee. |

The source-server inspection adds a separate fact: when an exit-intent with a player identifier is accepted, the server removes that player’s local presence and sets a five-second departure tombstone, which blocks heartbeat upserts during that interval. A failed exit-intent request leaves the client in its source view. If the server removed presence but its response was lost, the client can remain visible locally while absent from the registry until recovery. Destination arrival and registration are separate requests. Expiry and tombstones bound particular local cases; they do not establish global exclusivity under arbitrary network failure.

The inspected source server uses a ten-second default presence lifetime, configurable from one to sixty seconds, and a three-second heartbeat hint. These are implementation settings observed in source, not newly executed failure/recovery results or required WoW timings.

The bounded probes recorded Node v22.22.3 and observation time `2026-09-07T12:21:09.193Z`. Public specification links let readers inspect source claims. Local execution records remain reported evidence where the underlying repository or receipt is not available to an outside reader.

## Historical implementation evidence

| Exact receipt | Supported result | Limitation |
|---|---|---|
| July 5, 2026 three-window record; SHA-256 `ba8449242041cf30401a6751c48cb761cb9a2be3a06f9479f395123c863be2c6` | Assertions passed; navigation counts remained one in the player and both observer windows. | One recorded local visual-continuity scenario, not independent implementations or network reliability. |
| July 11, 2026 record with the same basename; SHA-256 `909a8f8d5475b97f1dff4bace49eea9aa1df30c4d24cdadab03aa9a1d8ecba50` | The same navigation counts were recorded. | Overall assertions were false because three expected conformance declarations did not match. It cannot silently replace the earlier passing receipt. |
| Retained deterministic crossing record; SHA-256 `e268b1d716beb3e1ce7c28e1c3e02d8985d15ef595cc198845ca9d70563c6f42` | 81 passing assertions in a Node-based local check. | No browser execution; some assertions check source-code presence. Neither 81 nor the older “48/48” caption counts independent user journeys. |
| Historical 55-check subtree result, July 11, 2026 | Cumulative 29 previous checks plus 26 new schema/contract and discovery/transform-handover checks. | The scene builder excluded browser DOM, fetch, WebAssembly and renderer execution. These are not 55 cryptographic or rendered-subtree tests. |

Signed-fabric refusal applies to its configured payload and trust anchor. The inspected portal path separately continues after a failed manifest result or arrival notification. Signature validity, trusted assertions, holder control, admission, execution permission and test receipts remain distinct. A capability flag is a declaration even when signed.

## Ecosystem context and private evidence

Private meeting records were not reanalysed for this revision and are not reproduced in the public report. G1–G5 remain reserved references only. Governance claims use published sources or remain open questions; private keyword counts and paraphrases supply no public-source evidence.

The [infrastructure map](https://openspatials.com/msf/map) provides a useful comparison lens. Its selected 102 rows across six subjects describe that corpus. The retained classification has one “yes” (`net.address`), 17 “partial” and 84 “no” cells for WoW; these are labels in that map, not conformance verdicts. A blank cell does not establish worldwide absence, set priority or make WoW responsible for every concern. Adjacent projects retain different evidence levels; documentation about one project is not equivalent to an executed receipt for another.

## Limits and proposal status

This report supplies no new independent engine-to-engine crossing, multi-device localization, distributed replay/recovery, all-format rendering, production trust or security-policy acceptance. It does not assess simpleWorlds internals, execute native TeleportXR or audit every Universal Manifest policy module. The limited simpleWorlds path comparison described above does not extend that boundary. Future tests are acceptance criteria, not completed results.

The [five bounded asks](README.md#five-decisions-for-the-working-group) remain concrete decisions. Optional profiles can be evaluated without adopting one renderer's choices. Compatibility needs explicit old/new examples and failure behavior; retaining a field name does not prove it.

Source and consistency checks improve traceability. Internal review-pass and claim-review tallies do not count independent experiments, independently proven truths or a confidence score. This is a draft for working-group review dated September 7, 2026; proposed text is not adopted policy.

GeoPose interpretation follows [OGC GeoPose 1.0, Requirements 4, 12 and 13](https://docs.ogc.org/is/21-056r11/21-056r11.html): Basic YPR uses the specified WGS-84/ENU frame and ellipsoidal height. Additional metadata does not redefine those semantics. A `lan`/`lon` correction can fix an example’s shape without proving its coordinate values or scene mapping. URI-reference interpretation follows [RFC 3986, section 5](https://datatracker.ietf.org/doc/html/rfc3986#section-5); a URL’s entry query and fragment must not be confused with its service path.
