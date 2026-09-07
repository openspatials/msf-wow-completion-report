# Method and Sources

## The implementation as the source of the findings

Every finding in this package originates from building a working Web of Worlds implementation. Open Spatial Lab built a working implementation against the Web of Worlds specification at commit d39a1a0 over the course of 2026: a composition graph, portal traversal with no page reload, signed spatial documents verified fail-closed, a real-time presence channel, a URL fragment grammar, and a service discovery mechanism. The findings are the specification silences and defects that implementation encountered, documented with the extensions and conventions that were built to fill them.

The implementation is the primary source. Where the specification is silent, the finding reports what the implementation had to build to keep working. Where the specification contradicts itself, the finding reports the contradiction with the line numbers. Where the specification carries a defect (a misspelling, a missing parameter, a truncated sentence), the finding reports the defect and its downstream effect. No finding is asserted from speculation alone; each traces to a concrete code path, a schema field, a test count, or a verified keyword search against the canonical specification.

## Specification commit and upstream check

The specification was examined at commit d39a1a0 of the WebOfWorlds/WoWAPI repository. On 2026-09-07, the upstream main branch HEAD was confirmed equal to d39a1a0 by `git ls-remote` (verified). Every spec citation in these documents is anchored to that commit with a GitHub blob link.

The whitepaper page was checked on the same date: the live page was byte-identical to the local capture (file comparison, verified). The simpleWorlds reference implementation has moved (HEAD d2bda3e vs the cited 13d2cbe); this package cites simpleWorlds only for its choice of URL path (`/wow/scene/` vs the API's `/wow/spatial/`) and, in document 10, for its two licence files; its code is not assessed.

## The standards map as the scope lens

The MSF Infrastructure Working Group architecture map (openspatials.com/msf/map, HTTP 200 on 2026-09-07) provided the scope lens. Its companion data (102 interop sockets across six MSF subjects) was used in document 10 to map which sockets Web of Worlds covers, which it partially covers, and which are blind spots. This framing keeps the report's scope aligned with the working group's own architecture rather than the author's preferences.

## Transcripts as discussion evidence

Five meeting transcripts from the MSF Web of Worlds working group were used as discussion evidence. The transcripts provided topic counts (how many times a subject was discussed and by whom), direct quotes attributed to specific participants in their own words, and the verbal commitment for this talk. Dates of the meetings used: 2026-06-01, 2026-06-15, 2026-06-29, 2026-07-13, and 2026-08-24. No participant is quoted except in their own words from the transcript. Governance discussion frequency (599 hits across the meetings) is cited as evidence that governance is the most-discussed and least-codified topic.

## Verification

Each of the ten documents was verified by two independent adversarial passes before the author's name was attached:

- **Spec lens.** Every specification quote was checked byte-exact against API.yaml and README.md at commit d39a1a0. Every line number was confirmed. Every "silent" claim (a keyword with zero hits) was re-searched. Every link was followed.

- **Implementation lens.** Every claim about the Open Spatial Lab codebase was checked against the source files cited. People were quoted only in their own words from the transcripts. The claim boundary (local proof, no conformance claim, every response carries `standards_conformance: false`) was confirmed present and consistent. Numbers (48/48 crossing-continuity, 55/55 signed-subtree contract checks, 22-check schema validation) were checked against their documented sources and attributed to the codebase that produced them. Numbers documented by Open Spatial Lab but not re-run in this pass are marked as reported with medium confidence.

A fix pass applied the verifiers' findings; a confirm pass re-checked only the items raised. One round; a second only for a named defect.

**Document 11 and the appendices.** Document 11 and appendices A and B were written after the ten documents and verified in two passes. The first pass checked 178 claims and 34 section pointers and raised 11 findings (line-number offsets in document 11, a word count, and the meeting dates); all were applied and confirmed. The final pass re-ran every upstream freshness check (a live fetch of every page and `git ls-remote` of every repository, 2026-09-07T08:12Z, same results), checked 567 claims and all 98 section pointers, and raised 48 findings (5 major: the declaration count, transcript quotation beyond the recorded ledger, one wrong zero-hit search, a licence remark about the reference implementation, and the statement about how simpleWorlds is cited; 43 minor); all were applied and confirmed.

**Totals across the ten documents:** 609 claims checked, 63 findings raised, 61 resolved, 2 unresolved. The two unresolved items are minor citation corrections: one in document 07 (a search-term list that included a term with one hit among terms with zero hits, which was corrected in the document text but flagged late in the confirm pass) and one in document 09 (a line-range citation for an extension policy section). Neither affects a substantive claim.

## What was deliberately excluded

**Reference-implementation defects.** The simpleWorlds reference implementation's own code defects were left out of these documents. The report covers specification defects, not implementation bugs. Where simpleWorlds makes a different path choice than the API (`/wow/scene/` vs `/wow/spatial/`), the contradiction is noted as a specification erratum, not an implementation defect.

**Private evidence tiers.** The Open Spatial Lab codebase is in a private repository (public release planned at github.com/grigb/open-spatial-lab). Schema lines, design-note sections, and architecture documents are cited by path and line number. The citations are verifiable once the repository is public. Until then, they are reported evidence, not verified by an outside reader.

**Unverified claims.** Any claim that could not be confirmed by the verification passes was removed or rewritten as an open question. Nothing unverified survives in the final documents.

**Evidence counts not re-run.** The 48/48 crossing-continuity count and the 55/55 signed-subtree contract check count are documented by Open Spatial Lab and attributed to that codebase. They were not re-run as part of this verification. Their confidence is marked as medium until re-run.

**Demo material.** No live demonstration is included in this package. The demo recordings, launcher scripts, and demo-plan documents produced during preparation are retained as internal evidence. The findings stand on the specification text and the code, not on a live performance.

## Reference set of Web of Worlds publications

Document 11 (Published Positions and Current State) uses the following publications as its reference set. Each is cited with its version or date, capture date, and freshness check result.

- **Whitepaper**: "initial Web of World Whitepaper", 2026-Q1. Page: https://webofworlds.github.io/initial_MSF_Whitepaper/ . PDF: https://webofworlds.github.io/initial_MSF_Whitepaper/gen/MSF-3DWebInterop_WoWWhitepaper.pdf . Source: https://github.com/WebOfWorlds/initial_MSF_Whitepaper (HEAD 988f369b on 2026-09-07). Local capture: 2026-06-23. Freshness check 2026-09-07T07:05Z, repeated 08:12Z: live page byte-identical to the capture.

- **MSF announcement post**: "Announcing the Web of Worlds whitepaper: a concrete path to the open metaverse", published 2026-06-03 (page metadata; the page displays Jun 2, 2026). https://metaverse-standards.org/news/blog/announcing-the-web-of-worlds-whitepaper-a-concrete-path-to-the-open-metaverse/ . Local capture: 2026-06-23. Freshness check 2026-09-07T07:05Z, repeated 08:12Z: article text identical; site-wide CSS and navigation menu items changed.

- **MSF post**: "Linked spatial experiences: the Web of Worlds", published 2025-04-02, modified 2025-09-04. https://metaverse-standards.org/news/blog/linked-spatial-experiences-the-web-of-worlds/ . Local capture: 2026-06-23. Freshness check 2026-09-07T07:05Z, repeated 08:12Z: article text identical; site-wide CSS and navigation menu items changed.

- **GitHub Pages home**, dated 2026-03-31. https://webofworlds.github.io/ . Local capture: 2026-07-01. Freshness check 2026-09-07T07:05Z, repeated 08:12Z: two changes to the visible text since capture, plus one changed link target. (1) "official Spatial Computing WG" became "new Spatial Computing WG". (2) Implementations table: HTMLModeWrapper replaced by Open-Spatial-Lab (Apache-2.0, Level 5, gltf-binary). (3) The MSF Project slides link points to a different presentation. Updated capture saved as 2026-09-07-webofworlds-github-pages-home.html; the live page was byte-identical to it at 08:12Z.

- **Specification**: https://github.com/WebOfWorlds/WoWAPI at commit d39a1a0 (2026-05-21). Freshness check 2026-09-07T07:05Z, repeated 08:12Z: upstream HEAD d39a1a0, equal to cited commit (verified by git ls-remote).

- **Reference implementation** (named only, not assessed): https://github.com/WebOfWorlds/simpleWorlds at commit 13d2cbe. Freshness check 2026-09-07T07:05Z, repeated 08:12Z: HEAD d2bda3e, moved past cited commit. Cited for its URL path choice and, in document 10, for its two licence files; its code is not assessed.

## Appendices

- **A-completion-map.md**: the 73 specification surfaces with their status and the document that covers each one.
- **B-findings-register.md**: the 25 findings from the implementation effort, each with a recommendation, an evidence level, and the document that treats it.

## Dates

- Specification commit d39a1a0: 2026-05-21.
- Upstream freshness check (`git ls-remote`): 2026-09-07 (07:05Z, repeated 08:12Z).
- Whitepaper text check: 2026-09-07.
- GitHub Pages home freshness check: 2026-09-07 (changed; updated capture saved).
- MSF announcement post freshness check: 2026-09-07 (article text identical).
- MSF linked-spatial-experiences post freshness check: 2026-09-07 (article text identical).
- Standards map availability check (HTTP 200): 2026-09-07.
- Meeting transcripts used: 2026-06-01, 2026-06-15, 2026-06-29, 2026-07-13, 2026-08-24.
- Documents written and verified: 2026-09-07.
- Final verification pass over document 11 and the appendices: 2026-09-07.
- This method document: 2026-09-07.

## Change log

- 2026-09-07: first public draft, verified twice.
