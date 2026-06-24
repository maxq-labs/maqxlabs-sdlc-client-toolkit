# Validation Rubric

The complete aspect list derived from `guidelines--revision.md`, grouped exactly as the report is. The **live guideline read in Step 2 is the authority** — if it carries an aspect not captured here, validate against it and note the drift. For each aspect: what to check, and how to decide Met / Partial / Not met / N/A.

Each aspect is tagged **[required]** or **[recommended]**. Required aspects drive the verdict; recommended ones surface as suggestions. "Required" means the guideline states it as a must for a *complete* revision (the `ready-for-story-creation` bar) or a schema constraint. A `draft` is allowed to have required gaps — those are reported as "remaining before ready", not hard failures (see the status-aware verdict in SKILL.md Step 4).

Throughout: **judge content, not surface.** A labeled prefix with hollow content is not Met. A field that exists but says nothing ("Allow users to cancel orders") is Partial at best.

---

## Group A — Schema & field conformance

The `Field-by-Field Reference` table in the guideline. These are checkable from the revision file alone.

| Aspect | Check | Scoring |
|--------|-------|---------|
| A1 — Required fields present **[required]** | `revision-id`, `feature-id`, `revision-number`, `description`, `created-at` all present. | Any missing → Not met (one row, list which). |
| A2 — `revision-id` pattern **[required]** | Matches `{feature-id}-rev-{N}`. | Mismatch → Not met. |
| A3 — `revision-number` agreement **[required]** | Integer ≥ 1, and equals the `N` embedded in `revision-id`. | Disagreement or <1 → Not met. (Sequential-vs-siblings is out of scope — single-revision validation.) |
| A4 — `feature-id` populated **[required]** | Non-empty string. (Whether it resolves to a real feature file is the audit's job, not conformance.) | Empty → Not met. |
| A5 — `status` is a valid enum **[required]** | One of `draft \| ready-for-story-creation \| stories-created \| in-progress \| completed \| cancelled`. Absent is allowed (defaults to draft intent) but note it. | Invalid value → Not met. |
| A6 — `title` constraint **[recommended]** | If present, ≤ 80 chars and action-oriented (names the specific change, not "Cancel orders"). | >80 → Partial; vague/generic → Partial. |
| A7 — `created-at` / `updated-at` format **[recommended]** | ISO 8601 UTC date-time. `updated-at` present if content has changed. | Malformed → Partial. |
| A8 — No manually-filled `generated-by` **[recommended]** | `generated-by` is automation-only; fine if present from an agent, but flag if it looks hand-authored/inconsistent. | n/a unless clearly wrong → suggestion. |
| A9 — No custom/unknown top-level fields **[required]** | Schema is `additionalProperties: false`; only known fields. Analytical content must live in `functional-changes`/`notes`, not invented keys. | Unknown key → Not met. |

---

## Group B — Completeness checklist (analytical-area coverage)

The guideline's **Completeness Checklist** — the eighteen analytical areas. Each must contain **specific, actionable content** in its prescribed container (schema field or `functional-changes` prefix), not a placeholder or one-liner. This group asks **"is the area covered at all, with substance?"** (Group D judges *how well*.) Emit one row per area.

For each: **Met** = substantive content in the right container; **Partial** = touched but thin/placeholder, or in the wrong container; **Not met** = absent; **N/A** = genuinely doesn't apply, with reason.

| # | Area | Container the guideline prescribes | N/A only when… |
|---|------|-----------------------------------|----------------|
| B1 | Description **[required]** | `description` field | never (always required) |
| B2 | Rationale **[recommended]** | `rationale` field | — |
| B3 | Terminology & glossary alignment **[required]** | `description`/`notes` + glossary | never (also scored in Group E) |
| B4 | Actors & authorization **[required]** | `description`/`notes` or `BUSINESS RULE BR-N:` | — |
| B5 | Trigger **[required]** | `TRIGGER:` | — |
| B6 | Happy path **[required]** | `HAPPY PATH:` | — |
| B7 | Alternative flows **[recommended]** | `ALTERNATIVE:` | no plausible variations exist |
| B8 | Exception & error flows **[required]** | `EXCEPTION:` | — |
| B9 | Business rules **[required]** | `BUSINESS RULE BR-N:` | no invariants govern the change |
| B10 | Validation rules **[required]** | `VALIDATION:` | no input is accepted |
| B11 | Required data **[required]** | `SYSTEM BEHAVIOR:` / `description` | — |
| B12 | System behavior **[required]** | `SYSTEM BEHAVIOR:` | — |
| B13 | UI & interaction expectations **[required when UI involved]** | `UI:` / `description` / `notes` | no UI surface — **and** `notes` says so explicitly (the guideline-prescribed marker). If UI is involved but unstated → Not met, not N/A. |
| B14 | Status & state changes **[required]** | `STATE CHANGE:` | the change drives no entity-state transition |
| B15 | Integration expectations **[required]** | `INTEGRATION:` | no external system is touched |
| B16 | Audit, logging & traceability **[required]** | `AUDIT:` | — |
| B17 | Notifications **[required]** | `NOTIFICATION:` | the change sends no communication |
| B18 | Acceptance criteria **[required]** | `acceptance-criteria` field | never (always required) |

A claimed N/A must be *defensible* — "no exceptions possible" is rarely true; push back in the note if the area was likely just skipped.

---

## Group C — functional-changes structure & prefix conventions

The guideline's **Functional-Changes as the Structured Container** section.

| Aspect | Check | Scoring |
|--------|-------|---------|
| C1 — Items use labeled prefixes **[required]** | Each `functional-changes[i].change-description` opens with a recognized label: `TRIGGER:`, `HAPPY PATH:`, `ALTERNATIVE:`, `EXCEPTION:`, `BUSINESS RULE BR-N:`, `VALIDATION:`, `SYSTEM BEHAVIOR:`, `STATE CHANGE:`, `INTEGRATION:`, `NOTIFICATION:`, `AUDIT:`, `UI:`. | No prefixes at all → Not met. Some unlabeled / free-prose items → Partial (name them). |
| C2 — Labels used exactly as defined **[required]** | Spelling/casing matches the catalog; no invented labels. | Drifted labels → Partial. |
| C3 — Business rules numbered `BR-N` **[required]** | Each business rule carries a sequential `BR-1`, `BR-2`, … identifier. | Rules present but unnumbered → Partial. |
| C4 — Analytical content lives in the structured containers **[required]** | Substantive analysis is in `functional-changes`/`notes`, not crammed into `description` prose only (rules buried in prose is the named anti-pattern). | Rules/flows only in prose → Partial. |

---

## Group D — Per-area quality rules

The deeper bar from each per-area section of the guideline. Score only areas that are present (an absent area is already Not met in Group B; don't double-penalize — here judge the quality of what *is* there).

| Aspect | Check (guideline section) | Scoring |
|--------|---------------------------|---------|
| D1 — Description depth **[required]** | (Description) Multi-sentence, ≥3–5 sentences covering **what / for whom / under what circumstances / to what end**. A single sentence is never sufficient. | 1 sentence → Not met; present but misses ≥1 of the four facets → Partial. |
| D2 — Rationale substance **[recommended]** | (Rationale) References a real driver — business driver, user pain, regulatory/compliance, or technical debt — and connects to the parent feature's purpose. | Vague/circular → Partial. |
| D3 — Happy-path quality **[required]** | (Happy Path) Numbered steps; each identifies **who acts, what the system does, what data moves, what the user perceives**. | Unnumbered or missing the per-step facets → Partial. |
| D4 — Alternative flows anchored **[recommended]** | (Alternative Flows) Each `ALTERNATIVE:` references the happy-path step where it diverges and still ends in success. | No divergence anchor → Partial. |
| D5 — Exception flows complete **[required]** | (Exception Flows) Each `EXCEPTION:` gives trigger condition + system response + user perception, and distinguishes recoverable vs non-recoverable. | Missing facets / no recoverable-vs-not distinction → Partial. |
| D6 — Business rules well-formed **[required]** | (Business Rules) Each is named, stated as an invariant, and carries a violation **consequence** (intent, not exact error code). | Missing consequence or not invariant-shaped → Partial. |
| D7 — Validation rules well-formed **[required]** | (Validation Rules) Each names field(s), the constraint, and the consequence. | Missing any → Partial. |
| D8 — Bounded-operation triple **[required when a change acts on a set/batch/bulk/loop]** | (Validation Rules → Bounded Operations) Three *independent* constraints decided: **eligibility**, **atomicity** (per-item failure behavior), and **maximum size** (explicit cap, or a deliberate "no cap" with a safety reason). The size cap is the most-missed. | Acts on a set but size cap undecided → Not met (silence ≠ "unlimited"); eligibility or atomicity missing → Partial. N/A if no set/batch operation. |
| D9 — System behavior at the right altitude **[required]** | (System Behavior) Business-level computations/sequencing, **no implementation detail** (no ORM/SQL/framework/HTTP-client names). | Leaks implementation detail → Partial. |
| D10 — UI captures intent not CSS **[required when UI involved]** | (UI) Screens, layout intent, interactive elements, feedback patterns, navigation outcome — behavior, not visual design. | Over-specifies CSS, or under-specifies behavior → Partial. N/A if no UI (must be declared in `notes`). |
| D11 — State changes with side-effects **[required when state transitions]** | (Status & State Changes) From-state, action, to-state, who triggers, side-effects. | Missing facets → Partial. N/A if none. |
| D12 — Integration intent **[required when integration]** | (Integration) Target, direction, protocol, sync/async, failure-handling intent. | Missing facets → Partial. N/A if none. |
| D13 — Audit specifics **[required]** | (Audit) What's recorded, by whom, when, on what entity; correlation-id propagation; retention/compliance driver where relevant. | Generic "log it" → Partial. |
| D14 — Notification specifics **[required when notifying]** | (Notifications) Recipient, trigger condition, channel, content intent. | Missing facets → Partial. N/A if none. |
| D15 — Acceptance-criteria quality **[required]** | (Acceptance Criteria) Each criterion is **observable, unambiguous, independent (one behavior), free of implementation detail**. Prefer Given/When/Then or declarative system statements. | Vague/compound/implementation-bound criteria → Partial (quote the offender). |
| D16 — Acceptance-criteria coverage **[required]** | (Acceptance Criteria → Minimum coverage) At least one AC per happy-path outcome, per major exception, per named business rule, per key integration, per notification; and every AC traces back to a functional change (no orphans either direction). | Uncovered area or orphan AC → Partial (name the gap). |

---

## Group E — Terminology & glossary alignment

The **Terminology and Glossary Alignment** section. Reads the glossary (resolved path; default `solution-definition/glossary.yaml`) and `solution.yaml` entity names.

| Aspect | Check | Scoring |
|--------|-------|---------|
| E1 — Canonical terms used **[required]** | Business terms the revision references match the glossary's canonical term (not a synonym). | Synonym for a defined term → Partial. |
| E2 — Load-bearing terms defined **[recommended]** | Every business term the revision *references* (not just its subject) has a glossary entry; missing ones are candidates to add. | Undefined business term → Partial (suggestion). |
| E3 — No ambiguous / colliding terms **[required]** | No term that is *similar but not identical* to an existing glossary term / `solution.yaml` entity, leaving same-or-different unresolved. | Colliding/ambiguous term → Not met (this is the guideline's most serious terminology defect). |
| E4 — Internal naming consistency **[required]** | One concept named the **same way** across `description`, `functional-changes`, `acceptance-criteria`, aligned with `solution.yaml` entity names. | Synonyms drift across fields → Partial (or Not met if it causes genuine ambiguity). |

Only **business vocabulary** counts (entities, actors, statuses, domain actions). Field names, internal identifiers, technical mechanisms, formats/encodings, and UI/storage artifacts are **not** glossary terms — do not flag them here.

---

## Group F — Status-lifecycle gate

The **Status Lifecycle** section.

| Aspect | Check | Scoring |
|--------|-------|---------|
| F1 — Acceptance-criteria gate **[required]** | A revision **must not** be `ready-for-story-creation` (or beyond) while `acceptance-criteria` is empty or contains only vague statements. | `ready-for-story-creation`+ with empty/vague AC → Not met (blocker — the AC is the gate). |
| F2 — Completeness gate **[required]** | At `ready-for-story-creation`+, **all** completeness-checklist areas (Group B) are addressed. | Any Group B required area unmet at that status → contributes to Non-conformant. |
| F3 — Cancellation reason **[required when status: cancelled]** | A `cancelled` revision documents the reason in `notes`. | `cancelled` with no reason → Not met. N/A otherwise. |

`draft` carries no readiness obligation — F1/F2 are reported against the readiness bar but, for a draft, frame them as "remaining before ready", not failures (see status-aware verdict).

---

## Group G — Dependencies

The **Revision Dependencies** section.

| Aspect | Check | Scoring |
|--------|-------|---------|
| G1 — Dependency id pattern **[recommended]** | Each entry in `dependencies` matches `{feature-id}-rev-{N}`. (Whether the file exists is the audit's structural check, not conformance.) | Malformed id → Partial. N/A if empty. |
| G2 — Dependencies vs soft sequencing **[recommended]** | `dependencies` used for genuine *technical* prerequisites; mere planning/business-relatedness preferences belong in `notes`, not `dependencies`. | Soft sequencing listed as a hard dependency → suggestion. |

---

## Report template

The report is **findings-first**. You walked every aspect (that's how you found the gaps and can be sure nothing was skipped), but the body leads with the **substantive functional pain points and how to fix them** — what a functional analyst actually needs. The exhaustive A–G verdicts become a *compact backstop* at the end: clean groups collapse to a single pass-line, and only real gaps (Partial / Not met) get rows. Mechanical "Met" and "N/A" rows are noise — keep them out of the body and let the counts speak for them.

Write the report with exactly this structure.

```markdown
# Revision Guideline Validation — {revision-filename-without-ext}

- **Revision file:** `{actual-resolved-path}` ← the path resolved in Step 1; every Location link keys off it.
- **Feature:** {feature-id}
- **Status:** {status}
- **Validated:** {yyyy-mm-dd HHuMM}
- **Guideline:** `{resolved-guideline-path}` ({guideline-source} — e.g. bundled `maxq-sdlc-toolkit`, or `maxq-solution-core {version}`)
- **Glossary:** `{resolved-glossary-path}` {or "not found — terminology checked against solution.yaml entities only"}
- **Verdict:** {Conformant | Conformant with recommendations | Non-conformant | Draft in progress — N of M required aspects met}

## Functional read

{Three to five sentences in a functional analyst's voice — not a count recital. What state is this revision in *as a functional specification*? Could a team derive unambiguous stories from it today, or what would they have to invent or come back and ask? Name the single biggest thing standing in the way. State what is genuinely solid so the author knows what not to touch. This is the paragraph the author reads first and may be the only thing a busy reviewer reads — make it carry the message.}

## Key findings

{The substantive functional pain points, ordered by impact — the gaps that would actually mislead an implementer or leave a story-generator guessing. This is the heart of the report. Aim for the handful that matter, not one per unmet aspect; fold related gaps into a single finding (e.g. "the entire structured-analysis layer is missing" rather than fifteen separate "area absent" rows). A clean revision may have zero or one finding — that's a good outcome, not a reason to manufacture more.

Render each finding as a compact table. The "Maps to" cell ties it back to the guideline aspects (so the conformance backstop below is traceable) — but the finding is framed functionally, not as "aspect C1 failed".}

### F1 — {short functional title, e.g. "Analysis is trapped in prose — story generation can't classify it"}

| Field | Detail |
|-------|--------|
| **Severity** | {blocker \| warning \| suggestion} |
| **What's wrong** | {The functional gap, in plain terms. Quote the offending text where it helps.} |
| **Why it matters** | {The downstream functional consequence — what an implementer would have to guess, what scenario goes unhandled, what a story-generator can't derive.} |
| **Suggested fix** | {Concrete and paste-ready wherever possible — show example wording (e.g. the actual `TRIGGER:` / `BUSINESS RULE BR-N:` items to add), not an abstract instruction to "add business rules".} |
| **Maps to** | {aspect ids — e.g. C1, C3, B9, D6} · {linked Location(s) into the revision, paths relative to the report with `../../`, e.g. [functional-changes[0] (L46–72)](../../{revision-path}#L46-L72)} |

---

{repeat per finding. If there are no findings: `_No substantive findings — see conformance coverage below._`}

## Conformance coverage

The full aspect walk, kept compact: a clean group is one line; only Partial / Not met aspects get rows. (Met and N/A are summarised in the counts.) This is the completeness backstop behind the findings above.

| Group | Met | Partial | Not met | N/A | Notes |
|-------|-----|---------|---------|-----|-------|
| A — Schema & field | {n} | {n} | {n} | {n} | {"✅ all met" or the one thing that isn't} |
| B — Analytical-area coverage | {n} | {n} | {n} | {n} | {…} |
| C — functional-changes structure | {n} | {n} | {n} | {n} | {…} |
| D — Per-area quality | {n} | {n} | {n} | {n} | {…} |
| E — Terminology & glossary | {n} | {n} | {n} | {n} | {…} |
| F — Status-lifecycle gate | {n} | {n} | {n} | {n} | {…} |
| G — Dependencies | {n} | {n} | {n} | {n} | {…} |
| **Total** | **{n}** | **{n}** | **{n}** | **{n}** | |

{Then, ONLY for groups that have at least one Partial or Not met, a short table listing just those aspects. Skip clean groups entirely — their pass-line above says all that's needed. Status is 🟡 Partial · ❌ Not met. Location links use `../../`. This is where a reader who wants the audit trail for a specific gap finds it.}

**Gaps in {Group X}:**

| Aspect | Status | Location | Gap |
|--------|--------|----------|-----|
| {id — name} | {🟡\|❌} | [field (L#)](../../{revision-path}#L#) | {one line; how it falls short} |

{repeat the gaps-table per group that has gaps}
```

Two rules that keep the report honest:

- **The verdict follows mechanically from the counts and status** (see SKILL.md Step 4). If there are zero Not-met *required* aspects, it is at least "Conformant with recommendations", never "Non-conformant" — don't let prose tone drift from the tally.
- **Required-aspect denominator.** When the verdict says "N of M required aspects met", M is the count of **[required]** aspects that *apply* (exclude defensible N/A); N is how many of those are Met. State both so the reader can reconcile the number with the coverage table — don't quote a bare fraction the table can't explain.
