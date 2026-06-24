---
name: maxq-revision-validation
description: Validate a single SPDM revision for conformance to the revision authoring guideline (guidelines--revision.md), checking every aspect it demands — schema/field rules, the completeness checklist, functional-changes prefix conventions, acceptance-criteria quality, terminology/glossary alignment, the status-lifecycle gate, and dependencies — then writing a findings-first compliance report to .maxqlabs/analysis/. Use when the user wants to check a revision against the guideline or authoring standard, e.g. "does this revision meet the revision guideline?", "validate revision X against the guideline", "voldoet deze revision aan de richtlijn?", "controleer de revision tegen de guideline", or when a revision YAML is open and they ask if it follows the authoring rules. If no revision is named, ask which. This is the guideline-conformance counterpart to maxq-revision-audit (which judges it against current spec truth); prefer it when the user references the guideline, authoring standard, completeness, or "all aspects".
---

# Revision Guideline Validation

A **revision** is the functional-change specification in the SPDM hierarchy (Domain → Feature → **Revision** → Story → Spec Updates → Code). The bar the guideline sets is sharp: a revision is complete only when a development team can generate unambiguous stories from it *without asking the analyst a single clarifying question*. The authoring guideline — `guidelines--revision.md` — codifies every rule that gets a revision to that bar: which fields are required, the analytical areas that must all be covered, the labeled-prefix convention inside `functional-changes`, what makes an acceptance criterion testable, the terminology discipline, and the status-lifecycle gate.

This skill answers one question: **does a given revision meet every aspect of that guideline?** It walks the revision against the guideline aspect by aspect, marks each one Met / Partial / Not met / N/A with the evidence, and writes a per-aspect compliance report. It is **advisory and read-only** — it never edits the revision; it tells the author precisely which aspects fall short and why.

## How this differs from `maxq-revision-audit`

These two are complementary, not interchangeable — keep them distinct:

- **This skill (`maxq-revision-validation`)** is a **conformance check against the authoring guideline itself**. The question is *"does the revision follow the authoring rules?"* — structural, checklist-driven, judged from the guideline + the revision's own text (plus the glossary for the terminology aspect). It does **not** read the solution's component specs or features.
- **`maxq-revision-audit`** is a **senior-FA review against the solution's current spec truth** — it reads the glossary, parent feature, `solution.yaml`, and every component spec to judge functional soundness and gap-closure. It is draft-gated.

Rule of thumb: if the user mentions *the guideline, the authoring standard, completeness, or "all aspects"*, use **this** skill. If they want a soundness/spec-truth review of whether the revision holds up against what's built, use the audit.

## Inputs

- **Revision** (required) — the one revision to validate, by id, filename, or "the open file". Resolve it in Step 1.
- **Glossary path** (optional) — defaults to `solution-definition/glossary.yaml`. The terminology-alignment aspect (Group E) reads it. If the user supplies a different path, use that instead. If the default file is absent, note it and treat terminology aspects as best-effort against `solution.yaml` entity names.

**Validation is not status-gated.** Unlike the audit, this runs on a revision in *any* status — the revision's `status` is itself one of the aspects being validated (the lifecycle gate, Group F). Do not refuse to validate a non-draft revision.

## Step 1 — Resolve the target revision

Revisions normally live in `solution-definition/analysis/revisions/{revision-id}.yaml`, but they can also live inside a review-cycle tree — e.g. `review-cycles/<cycle>/feedback/<date>/<round>/revisions/{…}.yaml`. Consider **both** locations.

- **If the user named a revision** (id, filename, or "the one I have open"), use it — take its **actual path** (it may be in a review-cycle tree). Resolve partial names against the files in both locations.
- **If no revision was given**, do not guess silently. List the available revisions with their `status` and **propose a likely candidate** (an open file at its actual path, else the most recently modified), then ask the user to confirm.

Carry the **actual resolved path** forward — it becomes the report's `Revision file` header and the base for every Location link. Read the target revision fully before validating.

## Step 2 — Resolve and read the live guideline + glossary

The guideline is the authority for *which aspects exist*, so read the live copy each run — the rules evolve, and you must validate against the current version, not a frozen memory.

Resolve `guidelines--revision.md`, preferring **this plugin's own bundled copy** (`$CLAUDE_PLUGIN_ROOT/shared/guidelines/`) so validation is reproducible and works standalone — then fall back to an installed MaxQ marketplace (which may carry a newer copy). Resolve the path like this and pick the first hit:

```bash
# Bundled copy first (always present, version-pinned to this plugin), then the
# active-version maxq-solution-core cache, the marketplace working copy, any plugin copy.
for p in \
  "$CLAUDE_PLUGIN_ROOT"/shared/guidelines/guidelines--revision.md \
  "$HOME"/.claude/plugins/cache/*/maxq-solution-core/*/shared/guidelines/guidelines--revision.md \
  "$HOME"/.claude/plugins/marketplaces/*/plugins/maxq-solution-core/shared/guidelines/guidelines--revision.md \
  "$HOME"/.claude/plugins/**/maxq-*/shared/guidelines/guidelines--revision.md ; do
  [ -f "$p" ] && { echo "$p"; break; }
done
```

If none resolves, ask the user for the guideline path (they may have a working copy of the marketplace repo). Record the resolved guideline path **and** its source in the report header so the validation is reproducible — for the bundled copy note the plugin (`maxq-sdlc-toolkit`); for a marketplace copy read the version from the path (e.g. `…/maxq-solution-core/3.47.3/…`).

Read the guideline fully. Then read the glossary (the resolved glossary path; default `solution-definition/glossary.yaml`) — it is needed for the terminology-alignment aspect only.

> **Two `analysis/` trees — don't conflate them.** Revisions live under `solution-definition/analysis/`. The report you write goes to `.maxqlabs/analysis/` — a *different* directory under the hidden `.maxqlabs/` tooling folder. Read from the former, write to the latter.

## Step 3 — Validate aspect by aspect

Read **`references/validation-rubric.md`** now. It is the operational harness: it enumerates every aspect the guideline defines, grouped exactly as the report is, with *how to verify* each and *how to score* it. The live guideline you read in Step 2 is the authority — if it carries an aspect the rubric hasn't captured yet (the guideline drifted), validate against the guideline and note the drift in the report's Summary.

Walk every aspect and assign each a status with concrete evidence from the revision:

- **Met** — the revision satisfies the aspect; cite the field/line that shows it.
- **Partial** — present but falls short of the guideline's bar (e.g. a one-sentence `description` where the guideline demands 3–5; prefixes used but missing happy-path numbering). Say exactly what's missing.
- **Not met** — the aspect is required and absent or violated. Say what's missing and what the guideline requires.
- **N/A** — the aspect genuinely doesn't apply, with the reason (e.g. UI aspects for a system-to-system revision that declares "no UI surface" in `notes` — which is itself the guideline-prescribed way to mark it).

Be evidence-driven and honest. A Partial that names the precise shortfall is actionable; a vague one is noise. Don't manufacture Not-mets to look thorough, and don't wave through a genuine gap. Judge content, not surface: prefixed labels with empty content is not "Met".

**Checking is exhaustive; reporting is not.** Walking all aspects is how you *find* the gaps and earn confidence that nothing was skipped — but the per-aspect verdicts are mostly an audit trail, not the message. The reader is a functional analyst who wants to know **what is functionally wrong with this revision and how to fix it** — not to scan forty rows of "✅ Met" and "⚪ N/A". So as you walk the aspects, your real job is to **distil the substantive functional pain points**: the gaps that would actually mislead an implementer or leave a story-generator guessing — a scenario with no closing branch, a batch with no decided size cap, an actor whose authorization is unstated, analysis trapped in prose that downstream can't classify. Those, with their *why* and a concrete fix, are what the report leads with (Step 4).

## Step 4 — Write the report

Compute the timestamp:

```bash
date +"%Y-%m-%d-%Hu%M"
```

Write to **`.maxqlabs/analysis/{revision-filename-without-ext}-validation-{yyyy-mm-dd}-{HHuMM}.md`** (the `-validation-` infix keeps it distinct from audit and design-coverage reports for the same revision). Use the **filename stem**, not the bare `revision-id`, so wave-release variants stay distinct. Create `.maxqlabs/analysis/` if it does not exist.

Follow the report template in `references/validation-rubric.md` exactly. The report is **findings-first**: it opens with a functional read and the substantive pain points (each with why-it-matters and a concrete fix), and only then gives the compact conformance coverage as a backstop. The full A–G matrix is *not* the body — clean groups collapse to a single pass-line, and only the actual gaps (Partial / Not met) get rows. This keeps the signal — what to fix and why — from drowning in mechanical "Met / N/A" bookkeeping.

The verdict is **status-aware** because the guideline's full bar is "ready-for-story-creation":

- **Conformant** — every *required* aspect Met (N/A allowed); recommended aspects may carry suggestions. Safe against the guideline.
- **Conformant with recommendations** — all required aspects Met, but one or more recommended aspects are Partial/Not met.
- **Non-conformant** — one or more *required* aspects are Not met or Partial. For a revision whose `status` is `ready-for-story-creation` (or beyond), any unmet completeness-checklist area or the empty-acceptance-criteria gate makes it non-conformant, because it claims a readiness it has not earned.
- **Draft in progress** — when `status: draft` and required aspects are unmet: frame the verdict as a readiness assessment ("N of M aspects met; these remain before it can move to `ready-for-story-creation`") rather than a hard failure, since gaps are expected in a draft. Still list every gap.

## Step 5 — Report back

Tell the user where the report was written and give a two-line summary: the verdict and the single highest-leverage aspect to address. Don't paste the whole report into chat — point them to the file. If they want to apply fixes, hand off to `maxq-revision-define`.
