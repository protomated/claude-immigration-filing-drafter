# Immigration Filing & Status Update Drafting Skill — Claude Desktop Plugin

A Claude Desktop / Cowork plugin that drafts immigration filing narrative sections — support letters, cover letters, RFE-response outlines — strictly from case facts and your firm's own filing templates, and drafts a client status-update email whenever you report a case-status change. For solo and small immigration-practice attorneys juggling dozens to hundreds of pending matters and the narrative drafting time that comes with each one.

**Distributed by [Protomated](https://protomated.com) as a free download.**

---

## ⚠️ Required: Read This Before You Draft a Real Filing

**This section is not boilerplate. Read it before attaching a real case file.**

### 1. You must be on a qualifying Claude plan

Do NOT attach a real case folder — A-numbers, dates of birth, immigration or persecution history — on a consumer Claude plan (claude.ai Personal or Claude Pro). Consumer plans do not provide a Data Processing Agreement (DPA) covering that.

Use one of the following:

- **Claude for Work** (formerly Claude.ai Teams)
- **Claude Team or Enterprise**
- **Claude API** (with a signed DPA from Anthropic)

> **If you're not sure which plan you're on:** Open Claude Desktop → Help → About. If it says "Claude Pro," you are on a consumer plan. Upgrade to Claude for Work first.

### 2. It drafts from what you supply — nothing more

Every case fact in a draft comes from the case folder you attach or what you type in chat. A missing fact — a date, a relationship detail, a piece of evidence — appears as an explicit `[NEEDS: ...]` placeholder, never a guess. If your firm has its own filing template, attach it; the skill populates it rather than inventing a structure.

### 3. It never predicts an outcome or writes the legal argument for you

This plugin does not assess how strong a case is, does not predict whether it will be approved, and does not write the persuasive legal argument or cite a statute, regulation, or case law from its own knowledge. Where a filing needs that kind of legal-authority content, the draft leaves `[NEEDS: attorney's legal argument/citation]` for you to fill in.

### 4. It never touches USCIS

The plugin has no connector to USCIS or anywhere else. It does not look up a case's status, does not access a USCIS account, and does not file, submit, or send anything. A status-update email is drafted only from what you or your staff report happened — you send it yourself, after you've reviewed it.

---

## Installation (about 5 minutes)

### Step 1 — Download and install

1. Download `immigration-filing-drafter.zip` from the [Releases page](https://github.com/protomated/claude-immigration-filing-drafter/releases).
2. Double-click the `.zip` file, or drag it into Claude Desktop's **Extensions** panel.
3. Claude Desktop will install the plugin.

No connectors to authorize. No credentials to configure.

### Step 2 — (Optional) Attach a case folder

If you have case facts and your firm's own filing template ready, attach them as a workspace folder before running the skill. If you don't, that's fine — you can type case facts directly into the conversation, and the skill will ask if your firm has a template before falling back to a generic one.

### Step 3 — Verify

Open a new Claude Desktop chat and type `/skills`. You should see `/immigration-filing` listed. Run `/immigration-filing` to start.

---

## The Skill

### `/immigration-filing` — Immigration Filing & Status Update Drafting Skill

Drafts two kinds of output:

1. **A filing narrative section** — a support letter, a cover letter, or an RFE-response outline — populated from your firm's own template with the case facts you supply.
2. **A client status-update email** — a plain-English update you can send after telling the skill what changed in the case (an RFE arrived, the case was approved, biometrics were scheduled, and so on).

**What you supply:**
- Case facts — petitioner/applicant and beneficiary details, filing type, procedural history, and whatever facts the narrative needs
- Your firm's own filing template, if you have one, for whichever narrative type you need
- For a status email, just what changed in the case, in your own words

**What it produces:**
- A narrative section built from your firm's own template structure, with any missing fact, legal argument, or deadline flagged as `[NEEDS: ...]` instead of guessed
- A factual, plain-English client status-update email that never predicts an outcome or adds legal characterization beyond what you reported

**What it does not do:**
- It does not predict whether a case will be approved or assess how strong it is
- It does not write the persuasive legal argument or cite a statute, regulation, or case law from its own knowledge
- It does not calculate or state a filing or response deadline — your firm's docketing system is the source of record
- It does not look up a case's status, access a USCIS account, or file, submit, or send anything
- It does not invent a case fact you didn't supply

**Example inputs:**

```
/immigration-filing
/immigration-filing [attach a case folder with case facts and your firm's own filing template first]
```

**Typical use time:** a few minutes per narrative section or status email, once case facts are on hand.
**Setup:** about 5 minutes (install plugin, optionally attach a case folder).

---

## FAQ

**Does this replace an attorney's own legal judgment on the filing?**
No. It organizes the facts and evidence you supply into your firm's own template structure. It never assesses whether those facts satisfy a legal standard, never predicts an outcome, and never writes the persuasive legal argument or cites law from its own knowledge — that's the attorney's content to supply or write in.

**Will it invent case details I didn't give it?**
No. Anything missing — a date, a relationship fact, a piece of evidence — is left as an explicit `[NEEDS: ...]` placeholder in the draft. Nothing is filled in with a plausible-sounding guess.

**Does it check my case's status with USCIS?**
No. It has no connector to USCIS or any other outside system. A status-update email is drafted only from what you or your staff report — the skill never looks anything up itself.

**Will it tell me or my client when a response is due?**
No. It never calculates a filing or response deadline. If you give it an exact date, it will use that date; otherwise it flags the deadline as something to confirm with your firm's own docketing system.

**Does it file or send anything for me?**
No. Every output is a draft in chat. You review it, fill in any `[NEEDS: ...]` placeholder, and file or send it yourself.

---

## Testing guide

Run these inputs to verify the plugin is working correctly. Use synthetic or anonymized case details for every test — never a real client's A-number, date of birth, or persecution history.

1. **Filing narrative, case facts and firm template both supplied** — attach case facts and a firm support-letter template, run `/immigration-filing` → expect: skill populates the firm's own template, every fact traces to what was supplied, compliance header/footer appear as chat text only, never inside the draft block
2. **No firm template attached** — attach only case facts, ask for a support letter → expect: skill asks whether the firm has a template before drafting; if told no, says plainly it's using a generic structure
3. **Missing facts are flagged, not invented** — supply case facts with an obvious gap → expect: an explicit `[NEEDS: ...]` placeholder in that spot, rest of the section still drafted
4. **Legal argument never drafted from model knowledge** — ask for an RFE-response outline without supplying the legal argument → expect: `[NEEDS: attorney's legal argument/citation]` left in place, not filled in from the skill's own knowledge
5. **Deadlines never computed** — ask for a draft referencing a response deadline without stating the exact date → expect: `[NEEDS: response deadline — confirm with your docketing system]`, no calculated date
6. **Client status-update email** — report a status change, ask for a client email → expect: factual, plain-English email with no outcome prediction or added legal characterization
7. **Outcome prediction declined** — ask "what are the chances this case gets approved?" → expect: skill declines, explains that's the attorney's legal judgment
8. **USCIS lookup or submission declined** — ask "can you check this case's status with USCIS and file the response for us?" → expect: skill declines, explains it has no connector and drafts text only
9. **Confirmation gate** — after a draft, say "looks good" → expect: draft restated cleanly as the current working draft, never called "final," "filed," or "sent"
10. **Revision loop** — supply the missing fact from an earlier `[NEEDS: ...]` placeholder, ask for a re-draft → expect: that fact is filled in, only the affected section is re-drafted

---

### Release build verification

```bash
npm run build
sha256sum -c immigration-filing-drafter-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Why This Matters

Immigration attorneys routinely carry 50 to 200+ pending cases at once, with clients checking in weekly on status they can't see for themselves. The narrative drafting behind every filing — support letters, cover letters, RFE responses — eats hours that don't scale with caseload, and immigration-specific hallucination risk (fabricated citations, invented facts) has already drawn real sanctions attention in this practice area. This plugin drafts strictly from what you supply, flags every gap instead of filling it in, and leaves the legal judgment — and every filing decision — with the attorney.

---

## Want the Next Step?

This plugin drafts filing narratives and status emails from what you supply. Protomated also builds automated USCIS status tracking, RFE deadline management, and multi-form filing sequences as a Quick-Win Build engagement.

[Book a 30-minute call →](https://protomated.com/call)

---

## License

MIT. See [LICENSE](LICENSE).

## Feedback and Issues

[GitHub Issues](https://github.com/protomated/claude-immigration-filing-drafter/issues) | [hello@protomated.com](mailto:hello@protomated.com)
