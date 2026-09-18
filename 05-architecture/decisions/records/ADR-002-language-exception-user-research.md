# ADR-002 — Language Exception for User-Research Artifacts

| Field | Value |
|-------|-------|
| **ID** | ADR-002 |
| **Date** | 2026-09-16 |
| **Status** | Accepted |
| **Authors** | Fabian Trujillo —  Project leader |
| **Reviewers** | Alexandra, Félix, Fabián, Julián — Development team |
| **Supersedes / Related to** | Extends ADR-001 (Documentation Language) |

---

## Context

ADR-001 establishes English as the mandatory language for all documentation and code,
with no exceptions listed. However, two artifacts in `03-product/` were produced entirely
in Spanish:

- `03-product/interview_resi-complex.md`
- `03-product/resi-complex-tech-watch.md`

These are not specification or reference documents — they are **primary research
artifacts**: raw interview transcripts and technology-watch notes gathered directly from
real, Spanish-speaking Colombian users (residents, administrators, board members) during
discovery. Translating them on capture would have meant paraphrasing informants in real
time, risking loss of the exact wording, tone, and nuance that make qualitative research
usable as evidence — including direct quotes that later justify product decisions in
`vision.md` and `problem-framing.md`.

Without an explicit exception, these two files are a standing violation of ADR-001 and a
documentation-rules failure ("mixing languages within the same category is grounds for PR
rejection"). Leaving this undecided means every future PR review has to improvise a
judgment call.

**Known constraints:**
- Interviews were already conducted and recorded in Spanish; the source data itself is Spanish.
- The target users of resi-complex are Spanish-speaking; discovery must reflect their actual words.
- Team capacity does not currently include time for professional translation of research artifacts.

---

## Decision

**We decided:** Raw, primary user-research artifacts (interview transcripts, field notes,
technology-watch findings sourced from Spanish-speaking informants) are exempt from
ADR-001 and may remain in Spanish. All other documentation — specs, ADRs, glossary,
domain model, requirements, code, and any *derived/synthesized* artifact (e.g. `vision.md`,
`problem-framing.md`) — remains in English per ADR-001 without exception.

**Justification:** The purpose of ADR-001 is to eliminate translation friction between
code and the documents that describe it. Research transcripts are not consumed by
tooling, are not referenced by code, and their evidentiary value depends on preserving
the informant's original words. Translating them would trade fidelity for a consistency
rule that doesn't serve its original purpose in this case. Any finding that needs to
inform the system must still be synthesized into English in a derived document — the
exception applies only to the raw artifact, never to what's built from it.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **B — Exempt raw research artifacts (chosen)** | Preserves informant fidelity; zero rework; keeps ADR-001 intact for everything else | Two files break the "all docs in English" rule at a glance; reviewers must recognize the exception | — (chosen) |
| A — Translate both files to English | Full ADR-001 compliance; single language across repo | Risks distorting direct quotes; costs translation effort; source recordings are Spanish anyway, so English becomes a second-hand copy | Fidelity loss outweighs consistency gain for evidentiary artifacts |
| C — Keep in Spanish with no ADR (status quo) | No effort required | Leaves an undocumented, silent violation; each PR reviewer has to guess whether it's acceptable | Contradicts `documentation-rules.md`; not auditable |

---

## Consequences

**Positive:**
- Research fidelity preserved; quotes remain verifiable against source language.
- ADR-001 stays simple and enforceable for every other artifact category.
- Reviewers now have a documented, bounded exception instead of an ad hoc exemption.

**Negative / Trade-offs:**
- Non-Spanish-speaking contributors (or AI tooling) can't read these two files directly.
- Slight inconsistency in "browse the repo" experience — most `.md` files are English, these two are not.

**Impact on the system:**
- Affected services: none (documentation-only).
- Documents that must be updated:
  - `00-governance/documentation-rules.md` — add a line referencing this exception.
  - `00-governance/git-conventions.md` / PR checklist, if it currently flags any non-English `.md` file — must allow-list `03-product/interview_resi-complex.md` and `03-product/resi-complex-tech-watch.md`.
  - Filename `interview_resi-complex.md` still needs a **separate** fix for kebab-case (`interview-resi-complex.md`) — that is a naming issue, not a language issue, and this ADR does not resolve it.

---

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Future contributor adds another Spanish file "because these two are allowed" | Medium | Medium | Exception is scoped explicitly to *raw research artifacts*, not to `03-product/` as a folder; state this scope clearly in `documentation-rules.md` |
| Non-Spanish speaker needs a specific insight from these files | Low | Low | Key findings must already be synthesized into English in `vision.md` / `problem-framing.md` per this ADR's own decision clause |

---

## References

- Extends: `05-architecture/decisions/records/ADR-001-idioma-documentacion.md`
- Documentation rules → `00-governance/documentation-rules.md`
- Affected files: `03-product/interview_resi-complex.md`, `03-product/resi-complex-tech-watch.md`