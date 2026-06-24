# MaxQ labs — SDLC Client Toolkit

A [Claude Code](https://code.claude.com) **plugin marketplace** that distributes MaxQ labs SDLC/SPDM skills to analysts and teams.

This repo is both the marketplace and the plugin host. It currently ships one plugin — **`maxq-sdlc-toolkit`** — containing the **`maxq-revision-validation`** skill.

## Install

```text
# Add the marketplace (one-time, per user)
/plugin marketplace add maxq-labs/maqxlabs-sdlc-client-toolkit

# Install the toolkit plugin
/plugin install maxq-sdlc-toolkit@maqxlabs-sdlc-client-toolkit
```

### Enable automatically for a whole project/team

Add this to a project's `.claude/settings.json` so everyone who clones the project gets the toolkit prompted/enabled:

```json
{
  "extraKnownMarketplaces": {
    "maqxlabs-sdlc-client-toolkit": {
      "source": { "source": "github", "repo": "maxq-labs/maqxlabs-sdlc-client-toolkit" }
    }
  },
  "enabledPlugins": { "maxq-sdlc-toolkit@maqxlabs-sdlc-client-toolkit": true }
}
```

## Skills

### `maxq-revision-validation`

Invoked as **`maxq-sdlc-toolkit:maxq-revision-validation`** after install.

Validates a single SPDM **revision** for conformance to the revision authoring guideline ([`guidelines--revision.md`](plugins/maxq-sdlc-toolkit/shared/guidelines/guidelines--revision.md)) — schema/field rules, the completeness checklist, functional-changes prefix conventions, acceptance-criteria quality, terminology/glossary alignment, the status-lifecycle gate, and dependencies. It writes a per-aspect compliance report to `.maxqlabs/analysis/`.

- **Read-only / advisory** — it never edits the revision; it reports which aspects fall short and why.
- **Assumes an SPDM solution layout** — revisions under `solution-definition/analysis/revisions/…` (or a review-cycle tree) and a glossary at `solution-definition/glossary.yaml`.
- **Guideline resolution** — prefers this plugin's bundled `guidelines--revision.md`, falling back to an installed `maxq-labs-build` marketplace copy if present. The report header records which copy was used.

## The guideline

The authoring standard the skill validates against is browsable directly here, no install required:

➡️ [`plugins/maxq-sdlc-toolkit/shared/guidelines/guidelines--revision.md`](plugins/maxq-sdlc-toolkit/shared/guidelines/guidelines--revision.md)

## Layout

```
.claude-plugin/marketplace.json            # marketplace manifest
plugins/maxq-sdlc-toolkit/
├── .claude-plugin/plugin.json             # plugin manifest
├── skills/maxq-revision-validation/       # the skill (SKILL.md + references/)
└── shared/guidelines/                     # guidelines--revision.md (single canonical copy)
```

## License

[MIT](LICENSE) © MaxQ labs
