# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**CP11** — Immigration Filing & Status Update Drafting Skill. A Claude Desktop / Cowork plugin for solo and small immigration-practice attorneys. One skill (`/immigration-filing`) drafts filing narrative sections — support letters, cover letters, RFE-response outlines — strictly from attorney-supplied case facts and the firm's own filing templates, and drafts a client status-update email whenever the attorney reports a case-status change. It never looks up a case's status with USCIS, never logs into a USCIS account, and never files, submits, or sends anything itself — every output is a draft for the attorney to review, finalize, and send. There is no runtime code, no MCP server, no connector, and no backend. The product is entirely content: a markdown skill file and a reference doc.

The leak it plugs: immigration attorneys routinely juggle 50 to 200+ pending cases at once, with clients checking in anxiously on status they can't see for themselves, and the narrative drafting behind every filing — support letters, cover letters, RFE responses, plus the client status-update emails each status change generates — eats hours that don't scale with caseload. Immigration-specific hallucination and sanctions risk is elevated in this practice area (fabricated citations and invented facts have already drawn real sanctions attention), which is exactly why this skill drafts strictly from what the attorney supplies and never writes legal argument, cites law, predicts an outcome, or computes a deadline from its own knowledge — the discipline behind that design mirrors the verified-source grounding rule in the catalog's legal-research-memo skill (see PAC-A-3's governance rationale for this track), applied here to filing narrative drafting instead of case-law research.

Landing page: `protomated.com/templates/immigration-filing-drafter/` (WordPress — managed outside this repo).

## Working a new ticket (this repo is a per-ticket template)

This repo is not owned by any single ticket — it's reused for every plugin in the PAC catalog. Each new build **replaces the current plugin's content in place**, same repo, same git history, no "repo is now for ticket X" migration. **Never write the word "replace" (or otherwise describe the swap) in a commit message, README, RELEASE.md, or code comment.** Every build should read as if it were authored fresh for its own ticket, not as a diff against whatever was here before. There is also an invocable `/work-pac-ticket` skill (`.claude/skills/work-pac-ticket/`) that runs this recipe end to end for a given ticket number.

**Starting a ticket:**

1. Look up the ticket in Nifty — project niceId `PAC` (project id `FivxoeVG9E`). Fetch the ticket by `niceId`, then its parent epic (`parentTaskId`, normally PAC-61) and any linked/dependency tasks — a single ticket's description is often incomplete without the epic's framing.
2. Read the three fixed reference docs before drafting anything; they don't change per ticket: `docs/NTC-A-1.md` (n8n track onboarding — shared pillar/leak-test framing), `docs/PAC-A-3.md` (Claude plugin track onboarding — the "assisted draft, always reviewed" rule this whole catalog runs on), and `docs/how-to-add-a-new-lead-magnet-template.md` (how the landing page gets published afterward, and why the build format decides the compliance note).
3. Confirm the ticket's plugin name, skill slug, and CP number (from the ticket title/custom fields) with the user before rewriting if any of them are ambiguous — don't guess.
4. If the ticket description or comments point at another catalog plugin for shared design rationale, resolve it — however it's referenced: an explicit `PAC-N`; a CP-number shorthand ("CP7," "C7" — every ticket title in this catalog follows `CP<N>: <Display Name>`, searchable via Nifty full-text search scoped to this project when only the shorthand or name is given); or just the plugin's name with no number at all. Once resolved to a PAC-N, read its `Plugin Repo/Marketplace URL` custom field, or ask the user for that ticket's repo URL/path if the field is empty or the search is ambiguous. Judge incidental mentions (a doc citation, a "replaces X" note) separately — those don't need resolving. Treat whatever you do pull in as read-only research — never a dependency, submodule, or copied file in this repo.

**What gets rewritten vs. left alone:**

Rewritten per ticket: the ticket-specific sections of this file (`What this repo is` above, and the skill/compliance sections below — keep sections like this one and "Commit style"), root `README.md`, `RELEASE.md`, `package.json`, `plugin/.claude-plugin/plugin.json`, `plugin/manifest.json`, `plugin/CONNECTORS.md`, `plugin/README.md`, `plugin/prompts/system-prompt.md`, `plugin/skills/<old-slug>/` → `plugin/skills/<new-slug>/` (delete the old directory rather than leaving both), `tests/skills/<old-slug>.md` + its fixture folder → `tests/skills/<new-slug>.md` + fixtures, `.github/workflows/release.yml` (zip filename + release title).

Left alone: `plugin/LICENSE`, root `LICENSE`, `docs/NTC-A-1.md`, `docs/PAC-A-3.md`, `docs/how-to-add-a-new-lead-magnet-template.md`, `.github/workflows/validate.yml`, `.claude/skills/`.

**Before committing:** run `npm run clean && npm run build` — it should validate, pack, and checksum cleanly — then grep the tree for the previous plugin's name/slug/skill terminology to catch anything left behind. Once the user confirms the build is ready, commit as a single `Build PAC-<N> CP<M>: <Display Name> Skill plugin` commit per "Commit style" below — same standing rule as everywhere else in this project: don't commit unless the user has asked for it. Do not push, tag, or run `npm run release` without the user's explicit go-ahead either.

## Repo layout

```
.claude/skills/work-pac-ticket/SKILL.md   Runs "Working a new ticket" above end to end — stable, not rewritten per ticket
plugin/           The installable plugin (packaged into .zip bundle)
  .claude-plugin/plugin.json   Manifest validated by scripts/validate-plugin.mjs
  .mcp.json                    Empty — filesystem access is Cowork's implicit attached-folder model, not a connector
  manifest.json                Plugin display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/immigration-filing/
    SKILL.md                              The single skill; YAML frontmatter + markdown body
    reference/filing-drafting-reference.md Narrative-type structure, status-update trigger categories, and the `[NEEDS: ...]` placeholder convention
scripts/
  validate-plugin.mjs          Validates plugin/ structure before packing
docs/
  NTC-A-1.md, PAC-A-3.md      Engineer onboarding reference docs
```

## Commands

All commands run from the repo root.

```bash
# Validate plugin structure (manifest, skill dirs, SKILL.md presence)
npm run validate

# Full build: validate → pack → SHA-256 → artifact
npm run build

# Pack only (skips validate)
npm run pack

# Cut a GitHub release (runs build first; requires RELEASE.md at repo root)
npm run release

# Remove build artifacts
npm run clean

# List plugin files (excludes node_modules)
npm run tree
```

## Plugin format

The bundle format is `.zip`. It uses the **plugin variant** (not standalone) — no bundled MCP server, no connectors. Plugin name: `immigration-filing-drafter`, current version: `1.0.0`.

Two manifests serve different purposes:
- `plugin/.claude-plugin/plugin.json` — the identity manifest the validator and Claude Desktop read (`name` must be kebab-case)
- `plugin/manifest.json` — display metadata only (no `server` block — this is a plugin variant, not standalone)

The validator (`scripts/validate-plugin.mjs`) checks:
- `.claude-plugin/plugin.json` is valid JSON with a kebab-case `name`
- Each `skills/*/` subdirectory contains a `SKILL.md`
- `agents/`, `commands/`, `hooks/` (if present) contain files with the expected extension

## Skill: /immigration-filing

The single skill drafts two kinds of output, always from what the attorney supplies — case facts and, where relevant, the firm's own filing template, typed into chat or attached as a workspace folder:

1. Confirms which output is needed — a filing narrative section (support letter, cover letter, or RFE-response outline) or a client status-update email — unless it's already clear from what was attached or asked.
2. **Filing narrative section:** if the firm has its own template, populates its structure with supplied case facts rather than redesigning it; if no template is attached, asks whether the firm has one before falling back to a generic structure, and says plainly when it's using a generic one.
3. Never invents a case fact — a name, date, relationship, or filing-history detail — not present in the attached case folder or chat input; a missing fact is left as an explicit `[NEEDS: ...]` placeholder, and the rest of the section is still drafted around it.
4. Never writes the persuasive legal argument for a filing, and never cites a statute, regulation, or case law, from its own knowledge — organizes only the facts and evidence supplied into the structure the firm's template calls for; anything requiring legal-authority content beyond what was supplied is left as `[NEEDS: attorney's legal argument/citation]`.
5. Never calculates or states a specific USCIS filing or response deadline from general processing rules or an RFE's issue date; a deadline appears only when the attorney states that exact date, otherwise it's flagged for the firm's own docketing/calendaring system to confirm.
6. **Client status-update email:** drafts a plain-English email from a case-status change the attorney reports — never something the skill looks up or infers itself — and never predicts a case outcome or adds legal characterization beyond what was reported.
7. Never certifies, assesses, or predicts anything about the case's legal merits, strength, or likelihood of success — that judgment stays with the attorney.
8. Never accesses, logs into, checks the status of, or submits anything to USCIS, and never sends a client email itself — the skill produces chat text only; the attorney or firm staff files and sends everything.
9. Presents every draft with the compliance header and footer as chat-level text around it — never embedded inside the copyable draft block, since that block is what the attorney copies straight into a USCIS filing package or a client email.
10. Iterates on corrections and `[NEEDS: ...]` placeholder fill-ins as many times as needed; never marks a draft "final," "ready to file," or "sent" — that's the attorney's own action.

Each `SKILL.md` has YAML frontmatter:
```yaml
---
name: skill-name
description: shown to attorney in /skills list
argument-hint: "[hint shown in Claude Desktop]"
---
```

## Compliance constraints — non-negotiable

These rules are enforced in `prompts/system-prompt.md` and `SKILL.md`. Do not weaken them:

1. **Review gate**: Claude must present every draft with the compliance header and footer as chat-level text around it, never inside the draft block, and never call a draft "final," "ready to file," or "sent" without the attorney's own action.
2. **No case-outcome prediction**: the skill never predicts whether a case will be approved, denied, or otherwise resolved favorably, and never assesses how strong a case is — neither in a filing narrative nor in a client status-update email.
3. **No legal argument or citation from model knowledge**: the skill never writes the persuasive legal argument for a filing, and never cites a statute, regulation, or case law, from its own training knowledge. Anything beyond what the firm's template, case folder, or chat input supplied is left as an explicit `[NEEDS: attorney's legal argument/citation]` placeholder — never filled in from general immigration-law knowledge.
4. **No deadline calculation**: the skill never computes or states a specific USCIS filing or response deadline from general processing rules or an RFE's issue date. A deadline appears in a draft only when the attorney states that exact date; otherwise it's flagged (`[NEEDS: response deadline — confirm with your docketing system]`) for the firm's own docketing system to confirm.
5. **No USCIS interaction**: the skill never accesses, logs into, checks the case status of, or submits, files, or sends anything to a USCIS account or system, and never sends a client-facing email itself. A status-update email is drafted only from what the attorney or firm staff reports happened.
6. **No facts invented**: a case fact — name, date, relationship, filing-history detail, evidence description — not present in the attached case folder or chat input is flagged as `[NEEDS: ...]`, never guessed or filled in with a "typical" case detail.
7. **Ambiguity resolution**: which draft type is needed is confirmed when unclear; a missing firm template is asked about before the skill falls back to a generic structure; a status-update trigger reported without a stated next step is asked about, not assumed.
8. **Plan-tier warning**: The system prompt must warn that real case data — A-numbers, dates of birth, immigration or persecution history — should only be used on Claude for Work, Claude Team, or Claude Enterprise, or the Claude API under a signed Data Processing Agreement (DPA), never consumer-tier Claude (claude.ai Personal / Pro).

## Internal QA fixtures — tests/skills/

`tests/skills/<skill-name>.md` is the internal QA testing guide for a skill — a standing convention for every plugin built in this repo, alongside (not replacing) the end-user testing guide in `plugin/README.md`. The difference:

- `plugin/README.md` — ships inside the plugin zip, short scenarios with pasted one-liners, aimed at an attorney verifying the install.
- `tests/skills/<skill-name>.md` — internal only, not packaged, uses real attached-folder fixtures under `tests/skills/<skill-name>/` for the case-facts-and-template path, since this skill's primary input is an attached case folder, not a single file to read cold. Deeper checks (e.g., compliance-wrapper placement, the `[NEEDS: ...]` placeholder rule, legal-argument and deadline refusal, outcome-prediction refusal) belong here even when they overlap with `plugin/README.md`'s scenarios.

All fixture data must be clearly synthetic — fictional firms, clients, matter numbers, and A-numbers. Never use real client or matter data, even anonymized real data, without checking with Dele first. This is especially important for this skill: case facts can include A-numbers, dates of birth, and persecution history, so fixture hygiene matters more here than on lower-sensitivity plugins in this catalog.

## Commit style

Do not include `Co-Authored-By` attribution lines in commit messages.

## Canonical plugin description

Used in `plugin/.claude-plugin/plugin.json` and any marketing copy — keep consistent:

> An immigration filing drafting assistant that turns attorney-supplied case facts and the firm's own filing templates into narrative filing sections — support letters, cover letters, RFE-response outlines — and client status-update emails triggered by a reported case-status change. Never looks up or submits anything to USCIS, never invents a case fact, never predicts an outcome, and never computes a deadline; you review and finalize every draft before filing or sending.

## Testing

Testing is manual inside Claude Desktop / Cowork — there is no test runner. The `plugin/README.md` is the canonical testing guide. It contains:
- Setup steps (build → install → optionally attach a test case folder → verify skill loads)
- 10 specific test inputs with exact text to paste and what to check for each

Key scenarios that must pass: a filing narrative drafted from a firm template (populates it, doesn't redesign it), no firm template attached (skill asks before falling back to generic), missing facts flagged as `[NEEDS: ...]` rather than invented, a legal-argument section left as a placeholder rather than drafted from the skill's own knowledge, a deadline never computed, a client status-update email that doesn't predict an outcome or add legal characterization, an outcome-prediction request declined, a USCIS-lookup-or-submission request declined, confirmation gate (nothing marked final/filed/sent until the attorney says so), and a revision loop that fills a placeholder without disturbing the rest of the draft.

## Notes

- `plugin/.mcp.json` is `{}` — this plugin requires no connector. Drafting runs in chat, with an optional case folder (case facts and the firm's own filing template) attached via Cowork's implicit attached-workspace-folder model, which needs no separate config.
- `plugin/manifest.json` has no `server` block — the plugin variant does not require one. Do not add one.
- `plugin/README.md` and `plugin/CONNECTORS.md` are end-user documentation included in the ZIP bundle; they are not internal developer docs.
- The root `.mcp.json` is gitignored — it holds workspace-level Claude Code MCP credentials and is not part of the plugin artifact.
- `npm run release` passes `--notes-file RELEASE.md` to `gh release create` — create/update `RELEASE.md` at repo root before running it.
- Package scripts use `$npm_package_name` and `$npm_package_version` — keep the `name` field in `package.json` in sync with the plugin slug.
