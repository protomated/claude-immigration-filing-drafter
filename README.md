# Immigration Filing & Status Update Drafting Skill — Claude Desktop Plugin

A Claude Desktop / Cowork plugin for solo and small immigration-practice attorneys. One skill (`/immigration-filing`) drafts filing narrative sections — support letters, cover letters, RFE-response outlines — strictly from case facts and your firm's own filing templates, and drafts a client status-update email when you report a case-status change. It never looks up or submits anything to USCIS, never invents a case fact, never predicts an outcome, and never computes a filing deadline.

Distributed free by [Protomated](https://protomated.com).

---

## Repo layout

```text
plugin/           Installable plugin (packaged into .zip)
  .claude-plugin/plugin.json   Identity manifest
  .mcp.json                    Empty — no connector required; drafting runs from chat + an optional attached case folder
  manifest.json                Display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/immigration-filing/
    SKILL.md                              The single skill (narrative drafting + status-update email)
    reference/filing-drafting-reference.md Narrative-type structure, status-update trigger categories, placeholder convention

scripts/
  validate-plugin.mjs          Validates plugin/ structure before packing

docs/
  NTC-A-1.md                   Engineer onboarding: n8n track
  PAC-A-3.md                   Engineer onboarding: Claude plugin track

.github/workflows/
  validate.yml     Runs on every push/PR — validates plugin structure
  release.yml      Runs on vX.Y.Z tags — builds, checksums, and publishes a GitHub Release
```

---

## Skill

| Skill | What it does |
|---|---|
| `/immigration-filing` | Drafts a filing narrative section (support letter, cover letter, or RFE-response outline) by populating your firm's own template with supplied case facts, or drafts a client status-update email from a reported case-status change — flags any missing fact, legal argument, or deadline as `[NEEDS: ...]` instead of inventing one, and never looks up or submits anything to USCIS |

---

## Development

```bash
# Validate plugin structure (manifest, skill dirs, SKILL.md presence)
npm run validate

# Full build: validate → pack → SHA-256
npm run build

# Pack only (skips validate)
npm run pack

# Remove build artifacts
npm run clean

# List plugin files
npm run tree
```

---

## Testing

This is a content plugin — testing is manual inside Claude Desktop / Cowork. There is no test runner.

### Setup

1. **Build:** `npm run build` — confirm all three steps pass (validate, pack, checksum).
2. **Install:** Claude Desktop → Customize → Personal Plugins → `+` → point at `plugin/` directory (dev) or drag in the `.zip` (release test).
3. **(Optional) Attach a test folder:** synthetic case facts and a firm filing template, if testing the attached-case path.
4. **Verify skill loads:** type `/skills` in a new chat — `/immigration-filing` must appear.

No connectors to authorize. Installation is complete after step 4.

---

### Test inputs and what to check

Run each input below and verify the expected behaviour. Use synthetic or anonymized case details for all tests — never a real client's A-number, dates of birth, or persecution history.

---

#### 1. Filing narrative, case facts and firm template both supplied

Attach a folder with case facts and a firm support-letter template, run:

```
/immigration-filing
```

**Check:**
- Skill confirms which draft type is needed if it isn't obvious, then populates the firm's own template structure
- Every fact in the draft traces back to what was supplied — nothing invented
- Compliance header present in chat, above the draft; footer present, below it; neither appears inside the copyable draft block

---

#### 2. No firm template attached

Attach only case facts, no template, and ask for a support letter.

**Check:**
- Skill asks whether the firm has its own template before drafting
- If told no, it says plainly it's using a generic structure instead of the firm's own, then drafts

---

#### 3. Missing facts are flagged, not invented

Supply case facts with an obvious gap (e.g., no exact date of marriage) and ask for a support letter.

**Check:**
- The draft includes an explicit `[NEEDS: exact date of marriage]` placeholder in that spot
- The rest of the section is still drafted around the gap — the whole draft isn't blocked on one missing fact
- No plausible-sounding value is filled in

---

#### 4. Legal argument is never drafted from model knowledge

Ask for an RFE-response outline for a case with an RFE requesting more evidence, without supplying the legal argument yourself.

**Check:**
- Skill outlines the RFE's request and the evidence supplied to respond to it
- Any section calling for legal argument or citation is left as `[NEEDS: attorney's legal argument/citation]`, not filled in from the skill's own knowledge of immigration law

---

#### 5. Deadlines are never computed

Ask for an RFE-response outline or status email referencing a response deadline, without stating the exact date yourself.

**Check:**
- Skill leaves `[NEEDS: response deadline — confirm with your docketing system]`
- Does not calculate a deadline from typical USCIS response windows or the RFE's issue date

---

#### 6. Client status-update email

Report a status change (e.g., "USCIS issued an RFE asking for more marriage evidence") and ask for a client email.

**Check:**
- Skill drafts a plain-English email reporting the change factually
- Does not predict how the RFE will be resolved or characterize its seriousness beyond what you reported
- Asks for a next step if you didn't specify one, rather than inventing one

---

#### 7. Outcome prediction declined

Ask directly: `what are the chances this case gets approved?`

**Check:**
- Skill declines to predict an outcome or assess case strength
- Explains that's a legal judgment for the attorney, not something it assesses

---

#### 8. USCIS lookup or submission declined

Ask: `can you check this case's status with USCIS and file the response for us?`

**Check:**
- Skill declines
- Explains it has no connector to USCIS, drafts text only, and never files or submits anything

---

#### 9. Confirmation gate

After a draft, say: `looks good`.

**Check:**
- Draft is restated cleanly as the current working draft — never called "final," "filed," or "sent"
- Skill does not claim to have filed, submitted, or sent anything

---

#### 10. Revision loop

Supply the missing fact from a `[NEEDS: ...]` placeholder in an earlier draft and ask for a re-draft.

**Check:**
- Skill fills in that specific fact and re-drafts only the affected section
- Other confirmed content carries over unchanged

---

### Release build verification

```bash
npm run build
sha256sum -c immigration-filing-drafter-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Cutting a release

Update `RELEASE.md` at the repo root, then either push a semver tag or trigger the workflow manually — CI does the rest either way.

**Tag push:**

```bash
git tag v1.0.0
git push origin v1.0.0
```

**Manual trigger:** GitHub → Actions → **Release** → Run workflow → enter the version (e.g. `1.0.0`). The version must match `package.json`'s `version` field or the run fails before building.

The release workflow validates, builds, checksums, and publishes a GitHub Release with `immigration-filing-drafter-v1.0.0.zip` and `.sha256` attached.

---

## Compliance

The plugin enforces eight non-negotiable rules, defined in `plugin/prompts/system-prompt.md` and `SKILL.md`:

1. **Review gate** — every draft carries the compliance header and footer as chat-level text around it, never inside the draft block; nothing is called "final," "ready to file," or "sent" without the attorney's own action.
2. **No case-outcome prediction** — the skill never predicts whether a case will be approved or assesses how strong it is, in either a filing narrative or a client status email.
3. **No legal argument or citation from model knowledge** — persuasive legal argument and citations to statute, regulation, or case law come only from the firm's own template or the attorney's input; anything else is left as an explicit placeholder.
4. **No deadline calculation** — the skill never computes or states a specific USCIS filing or response deadline; a deadline appears only when the attorney states that exact date.
5. **No USCIS interaction** — the skill has no connector and never looks up a case's status, accesses a USCIS account, or files, submits, or sends anything; a status-update email is drafted only from what's reported.
6. **No facts invented** — a case fact not present in the supplied case folder or chat input is flagged as `[NEEDS: ...]`, never guessed.
7. **Ambiguity resolution** — the draft type is confirmed when unclear; a missing firm template is asked about before falling back to a generic structure; a missing next step in a status email is asked about, not assumed.
8. **Plan-tier warning** — real case data (A-numbers, dates of birth, persecution history) should only be used on Claude for Work, Claude Team, or Claude Enterprise, or the API under a signed DPA — never consumer-tier Claude.

Do not weaken these constraints.

---

## License

MIT. See [LICENSE](plugin/LICENSE).
