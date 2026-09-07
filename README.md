# Web of Worlds Completion Report

Short address: https://openspatials.com/msf/wow/completion-report/ (redirects here).

One team built a working Web of Worlds implementation and ran it against the specification at commit d39a1a0. The implementation proved that the composition graph works: worlds compose, portals connect them, users cross between them with no page reload, and signed spatial documents verify and render in place. It also proved that the specification does not yet carry the contracts that make any of this interoperable. There are no units, no coordinate frame, no portal destination, no traversal protocol, no presence wire contract, no identity verification, no conformance vocabulary, and no RFC 2119 keywords. An empty JSON object validates against every schema. A second implementation that followed the specification alone would have to invent the same contracts independently, with no guarantee of agreement.

This package contains ten documents, two appendices, and a cross-reference document (document 11), covering specification gaps, the full surface inventory, findings, and a comparison of published positions against the specification text. Each document opens with what the spec says today (verified against the canonical repository), what fails without the fix, what the implementation built and learned, proposed normative text with schema fragments, and open questions for the working group. Together the documents propose 68 normative additions and raise 53 open questions. Every load-bearing claim carries its confidence (verified, reported, or inferred) and its source. Each of the ten numbered documents was checked by two independent adversarial verifiers and a confirm pass; 609 claims were checked, 63 findings were raised, and 61 were resolved across the ten (the two unresolved items are minor citation corrections that do not affect substantive claims). Document 11 and the two appendices were verified in two further passes, each followed by fixes and a confirm check; the counts are in METHOD-AND-SOURCES.md.

## How to read these documents

Each document stands alone. Pick the topic that matters to your work and read that document; you do not need to read the others first. Where one document depends on a fact stated in another, it names the document and the section. The documents share one skeleton (answer, what the spec says, what fails, what was built, proposed text, open questions, sources) so you can enter any of them at the same place.

## Documents

1. **Coordinate Precision, Units, and World Extents.** The spec declares no units, no up-axis, no extent, and no matrix format; proposes six additions that make spatial composition deterministic between independent implementations. 7 proposals.

2. **Portal Destination and the Traversal Protocol.** The spec defines a portal with no target and no crossing protocol; proposes a destination field, pose mapping, server-side notifications, and traversal semantics. 13 proposals.

3. **Portable User State and Identity.** The spec carries a four-property User with no identity verification; proposes a signed identity manifest, an age field, and default-role semantics. 5 proposals.

4. **Provenance and Signed Subtrees.** The spec has no trust vocabulary; proposes a proof-boundary declaration on every response, fail-closed verification on navigation, and a verification model for transcluded content. 4 proposals.

5. **Presence, Live Sync, and Persistence.** The spec defines a REST snapshot with no real-time channel, no session lifecycle, and a truncated Core Requirement; proposes event-channel semantics, session lifecycle rules, and fragment round-trip requirements. 7 proposals.

6. **Discovery and Addressing.** The spec shows fragment verbs with no grammar and no addressing scheme; proposes a fragment grammar, a URL-as-path-prefix rule, an address grammar, and service discovery. 8 proposals.

7. **Assets and the Render Seam.** The spec offers 21 asset formats with no baseline and no transclusion contract; proposes a required baseline format, a typed transclusion contract for signed spatial documents, and an engine-internal scope statement. 4 proposals.

8. **Composition Graph Schema Fixes.** The spec embeds child nodes recursively with no alternative and no error codes for shape mismatches; proposes both embedded and reference forms, content negotiation, and cross-world references. 5 proposals.

9. **Conformance Vocabulary and Errata.** The spec contains zero RFC 2119 keywords, zero required schema properties, a path contradiction between README and API, a misspelled longitude field, and truncated Core Requirement descriptions; proposes a conformance vocabulary and an errata pass. 9 proposals.

10. **Web of Worlds among Its Neighbours: Role and Blind Spots.** Maps the spec against 102 interop sockets from the MSF infrastructure architecture; proposes a precision vocabulary, a shared-anchor vocabulary, portable inventory, user preferences, agent identity, and governance. 6 proposals.

11. **Published Positions and Current State.** Lists every declaration from the Web of Worlds publications and the 2026-08-24 working-group meeting, shows what the specification text contains at commit d39a1a0 for each one, and identifies which of our documents treats it. 49 declarations: 20 specified, 17 named only, 12 aspirations.

## Appendices

- **A-completion-map.md.** The 73 specification surfaces with their status and the document that covers each one.
- **B-findings-register.md.** The 25 findings from the implementation effort, each with a recommendation, an evidence level, and the document that treats it.

## Reference set

The following publications form the reference set for this package. Every claim is checked against these sources at the versions and dates listed.

- **Whitepaper**: "initial Web of World Whitepaper", 2026-Q1. https://webofworlds.github.io/initial_MSF_Whitepaper/ . Source: https://github.com/WebOfWorlds/initial_MSF_Whitepaper .
- **MSF announcement post**: "Announcing the Web of Worlds whitepaper: a concrete path to the open metaverse", 2026-06-03. https://metaverse-standards.org/news/blog/announcing-the-web-of-worlds-whitepaper-a-concrete-path-to-the-open-metaverse/
- **MSF post**: "Linked spatial experiences: the Web of Worlds", 2025-04-02. https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/
- **GitHub Pages home**, dated 2026-03-31. https://webofworlds.github.io/
- **Specification**: https://github.com/WebOfWorlds/WoWAPI at commit d39a1a0 (2026-05-21).
- **Reference implementation** (named only): https://github.com/WebOfWorlds/simpleWorlds at commit 13d2cbe.

## Status

Draft for working-group review. All documents verified against the specification at commit d39a1a0 (WebOfWorlds/WoWAPI main, confirmed equal to upstream HEAD on 2026-09-07). Method and verification details are in METHOD-AND-SOURCES.md.

Date: 2026-09-07.

## How to respond

Open one issue per document. Each document is a self-contained set of proposals and can be discussed independently. Reference the proposal numbers (the numbered items under "Proposed normative text" in each document) and the open questions by number.

## Author & Contact

**Grig Bilham**, Open Spatial Lab  
*Co-chair, Metaverse Standards Forum (MSF) Infrastructure Working Group*

- **GitHub:** [@grigb](https://github.com/grigb)
- **Feedback & Collaboration:** Please open an issue in this repository ([github.com/openspatials/msf-wow-completion-report/issues](https://github.com/openspatials/msf-wow-completion-report/issues)) for chapter-specific technical discussions.
- **Working Group Inquiries:** Members of the Metaverse Standards Forum can also reach Grig directly via MSF Working Group channels and Member Portal.

## Licence

Licence to be decided by the author. Do not redistribute without permission until a licence is declared.

## Change log

- 2026-09-07: first public draft, verified twice.
