---
name: work-pac-ticket
description: Build or rebuild this repo's Claude plugin for a PAC catalog ticket — fetches the ticket and its parent epic/linked tasks from Nifty, reads the fixed onboarding docs, and rewrites the plugin per this repo's per-ticket template pattern. Trigger on "work on PAC-N," "build PAC-N," "pull the next catalog ticket," or similar.
argument-hint: "[PAC ticket number, e.g. PAC-52; optionally add a cross-ref repo URL/path if the ticket references another CP's build]"
---

# /work-pac-ticket — Build a PAC catalog ticket in this repo

Runs CLAUDE.md's "Working a new ticket" recipe end to end for a given ticket number. That section is the source of truth for what gets rewritten vs. left alone and the no-mention-of-"replace" rule — read it first if it isn't already in context. This skill is the trigger and the Nifty/orchestration steps around it.

## Invocation

```
/work-pac-ticket PAC-52
```

Or from a natural request: "work on PAC-52," "build PAC-52," "pull the next catalog ticket."

To point at a cross-referenced ticket's build directly instead of waiting on the automatic lookup in step 3, add it inline:

```
/work-pac-ticket PAC-54, C7 is at https://github.com/protomated/claude-c7-repo
```

## Steps

1. **Resolve the ticket number.** Take it from the argument; ask if it isn't stated anywhere.
2. **Fetch full ticket context from Nifty** (project niceId `PAC`, id `FivxoeVG9E`):
   - The ticket itself: `tasks_query {resource: "task", operation: "list", projectId: "FivxoeVG9E", niceId: <N>}`. This MCP server rejects `niceId_in` with a comma list — one `niceId` per call, so fetch the ticket and its parent/linked tasks as separate calls, not batched.
   - Its parent epic (`parentTaskId` from the ticket's own record, normally PAC-61) for the track-level framing — a ticket's own description is often incomplete without it.
   - Any linked or dependency tasks, plus its discussion/comments, for anything not already captured in the description or custom fields (Build Format, Tier, Pillar, Practice Area, Complexity, Catalog Channel, Upsell Target).
3. **Resolve any other plugin the ticket points to for design context — by PAC-N, by CP-number, or by name alone.** Don't pattern-match on one fixed form. A ticket can point at another catalog plugin several ways: an explicit `PAC-N`; a CP-number shorthand ("CP7," "C7" — every ticket title in this catalog follows the `CP<N>: <Display Name>` convention, so both forms show up); or just the plugin's name with no number at all ("mirrors the AI Intake Qualifier," "same pattern as the Contract Review skill"). Scan the ticket's description and comments for all three forms, other than the ticket's own PAC-N/CP-N and the parent epic already fetched in step 2, and read the surrounding sentence to judge whether it's pointing at shared design rationale worth pulling in, versus an incidental mention (a doc citation, a "replaces/supersedes X" note) that isn't. When it's genuinely not clear which, ask the user rather than guess or silently skip it.

   **Resolving to a PAC-N:**
   - Already a `PAC-N` → use it directly.
   - A CP-number or a bare plugin name → run a Nifty full-text search scoped to this project, e.g. `tasks_query {resource: "task", operation: "search", projectId: "FivxoeVG9E", query: "CP7"}` (or the name string). A single clear title match (`CP<N>: <Display Name>`) resolves to that ticket's PAC-N. If the search returns no result, or more than one plausible match, ask the user which ticket they mean rather than guessing.
   - Already supplied a URL/path inline in the invocation → treat it as an override, skip the Nifty lookup entirely for that one.

   **Pulling in the build**, once a PAC-N is resolved:
   - Read its `Plugin Repo/Marketplace URL` custom field.
   - If that field has a URL, fetch it — `WebFetch` for a couple of files (raw `CLAUDE.md`/`SKILL.md`/reference doc), or `gh repo clone`/`git clone --depth 1` into the scratchpad dir if you need to browse more. Read for the specific shared pattern being invoked (a compliance-boundary phrasing, a fact-grounding rationale, a naming convention) — never as code to copy wholesale, and never add it to this repo as a dependency, submodule, or committed file.
   - If the field is empty (not yet published) or the fetch fails, ask the user for the repo URL or a local path rather than guessing or silently skipping the reference — a stated design cross-ref that goes unread leaves a gap the ticket explicitly flagged.

   A ticket that doesn't reference any other plugin needs none of this — don't go looking for cross-refs that aren't there.
4. **Read the three fixed reference docs** if not already in context: `docs/NTC-A-1.md`, `docs/PAC-A-3.md`, `docs/how-to-add-a-new-lead-magnet-template.md`. These are stable across every ticket — re-reading them is what keeps a new build consistent with the rest of the catalog.
5. **Derive the build** from the ticket + parent epic + any resolved cross-ref, never invented: plugin slug, display name, skill slug, CP number, and the ticket-specific "assisted draft, always reviewed" boundary (PAC-A-3, "The rule that's different from the n8n track").
6. **Summarize the plan back to the user** in a short paragraph — ticket name, plugin slug, what's being rewritten, and which cross-ref (if any) informed it — before doing the full rewrite. This is a repo-wide change that deletes the previous plugin's files; worth a sanity check even though it's this repo's standing, authorized workflow.
7. **Rewrite the repo** per CLAUDE.md's "What gets rewritten vs. left alone" list. Before deleting the old `plugin/skills/<slug>/` and `tests/skills/<slug>.md` (+ fixtures) directories, run `git status` on those paths first — an untracked file there (e.g. a draft output left over from a prior manual test run) is in-progress work, not build scaffolding, and should be stashed or flagged to the user rather than deleted silently.
8. **Validate**: `npm run clean && npm run build`, then grep the tree for the previous plugin's name/slug to catch anything left behind.
9. **Commit only once the user confirms the build is ready** — don't commit unprompted, same standing rule as everywhere else in this project. When they do, commit as a single `Build PAC-<N> CP<M>: <Display Name> Skill plugin` commit (no `Co-Authored-By` line — see CLAUDE.md's Commit style). Do not push, tag, or run `npm run release` without the user's separate, explicit go-ahead.
10. **Never** describe this as a replacement, migration, or swap — anywhere: commit message, README, RELEASE.md, code comment. Every build reads as though authored fresh for its own ticket. This rule has no exceptions.

## Notes

- If the Nifty ticket can't be found, or its scope is ambiguous after reading the ticket + parent epic, stop and ask rather than guessing a CP number or plugin name.
- If the ticket describes something that crosses from "assisted draft" into the tool making a legal judgment (PAC-A-3, "The rule that's different from the n8n track," point 4), flag it to the user rather than building it.
- A cross-ref's repo is read-only research, not a build input to merge in. This repo's rewritten plugin must stay self-contained — no path, submodule, or copied file pointing back at another CP's repo.

## Improved prompt

With this skill in place, starting a ticket collapses to:

```
/work-pac-ticket PAC-52
```

Everything the old ad-hoc prompt spelled out by hand — which docs to read, fetching the parent epic and linked tasks, and the no-mention-of-"replace" rule — is now fixed, durable behavior instead of something to retype per ticket.

For a session where this skill isn't loaded (a fresh clone, another engineer's machine before they've pulled `.claude/skills/`), the long-form equivalent is:

```
Work on PAC-52. Fetch it from Nifty (project niceId PAC, id FivxoeVG9E), along with its
parent epic and any linked/dependency tasks, for full context. Read these three fixed
reference docs first: docs/NTC-A-1.md, docs/PAC-A-3.md, docs/how-to-add-a-new-lead-magnet-
template.md.

This repo is a reusable per-ticket template, not owned by PAC-52's predecessor: rewrite
plugin/, tests/skills/, README.md, RELEASE.md, package.json, and CLAUDE.md's ticket-specific
sections to match PAC-52, following the file-by-file recipe in CLAUDE.md's "Working a new
ticket" section. Do not describe this as a replacement, migration, or swap anywhere —
no commit message, doc, or comment should reference what was here before; this build should
read as though it were authored fresh for PAC-52.
```

The long form exists only as a fallback — once `.claude/skills/work-pac-ticket/` is present, use the one-liner.
