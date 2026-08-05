# Testing Guide — `/immigration-filing`

Internal QA doc for CP11. This is the detailed companion to the quick 10-scenario guide in `plugin/README.md` — that one is end-user facing and ships inside the plugin zip; this one is for build verification and uses real attached-folder fixtures, since this skill's primary input is case facts (and optionally a firm template) attached as a Cowork workspace folder, not a single file to read cold.

All fixture data below is **synthetic** — a fictional firm (Ferreira Immigration Law) and fictional clients. Do not substitute real client, matter, A-number, or persecution-history information into these fixtures; if you need to test against a real firm's actual case data, anonymize it first the same way you would for any other testing.

## Fixtures

```
tests/skills/immigration-filing/
  full-case-with-template/    case-facts.md — Okafor I-130/I-485 marriage-based adjustment, complete facts
                               firm-support-letter-template.md — firm's own template, including a
                                 legal-argument section the skill must NOT populate from its own knowledge
                               Tests: full narrative drafting with a firm template present, the
                                 "populate, don't redesign" rule, and the legal-argument placeholder rule

  case-no-template/           case-facts.md — Rahimi H-1B extension, complete facts, no template attached
                               Tests: the "ask if the firm has a template" branch before falling back
                                 to a generic structure

  sparse-case/                case-facts.md — Tesfaye asylum support letter, several facts deliberately
                                 missing (entry date, incident date, evidence status)
                               Tests: the `[NEEDS: ...]` placeholder convention — gaps are flagged,
                                 not invented, and don't block drafting the rest of the letter

  status-update-case/         matter-status-notes.md — continues the Okafor matter with an RFE reported
                                 by the attorney, deliberately withholding the response deadline
                               Tests: the client status-update email path, no-outcome-prediction rule,
                                 and no-deadline-computation rule; also usable (with
                                 full-case-with-template's case-facts.md) for the RFE-response-outline
                                 narrative type, since it describes what the RFE actually requested
```

## Setup

1. `npm run build` — confirm validate, pack, and checksum all pass.
2. Install the built `.zip` (or point Claude Desktop at `plugin/` directly for dev testing) — see `plugin/README.md` Step 1.
3. In a new Claude Desktop / Cowork chat, attach the relevant fixture folder (paths above) before running `/immigration-filing` for a scenario that needs one. Attaching is per-scenario.
4. Type `/skills` and confirm `/immigration-filing` is listed before running any scenario.

---

### 1. Filing narrative — firm template populated, not redesigned

**Attach:** `tests/skills/immigration-filing/full-case-with-template/`
**Run:**
```
/immigration-filing
```
**When asked which draft type:** ask for a support letter.

**Check:**
- Skill uses the firm's own template structure (Introduction / Statement of Facts / Legal Basis for Approval / Enclosed Evidence) rather than a generic structure
- Statement of Facts is populated from the marriage facts in `case-facts.md` — date, location, cohabitation, evidence list — nothing invented beyond what's there
- The "Legal Basis for Approval" section is left as `[NEEDS: attorney's legal argument/citation]` (or equivalent), not filled in with a generic bona fide marriage legal standard from the skill's own knowledge
- Enclosed Evidence section lists all five evidence items from the case facts with a one-line description each
- Compliance header appears once in chat, above the draft; footer once, below it — neither appears inside the copyable draft block

---

### 2. No firm template attached

**Attach:** `tests/skills/immigration-filing/case-no-template/`
**Run:**
```
/immigration-filing
```
**When asked which draft type:** ask for a cover letter for the I-129 extension package.

**Check:**
- Skill asks whether the firm has its own cover-letter template before drafting
- Once told there isn't one for this test, it says plainly it's using a generic cover-letter structure instead of the firm's own, then drafts
- The cover letter lists the five enclosed-evidence items from `case-facts.md`

---

### 3. Missing facts are flagged, not invented

**Attach:** `tests/skills/immigration-filing/sparse-case/`
**Run:**
```
/immigration-filing
```
**When asked which draft type:** ask for a support letter (generic structure, since no template exists for this matter).

**Check:**
- Draft includes explicit placeholders for each gap called out in the fixture — e.g. `[NEEDS: exact date applicant entered the U.S.]`, `[NEEDS: date/description of the triggering incident]`, `[NEEDS: corroborating evidence — confirm what's been gathered]`
- The rest of the letter (applicant identity, country of origin, general basis) is still drafted around the gaps — the whole draft is not blocked pending every missing fact
- No plausible-sounding date, incident description, or evidence list is filled in to complete the letter

---

### 4. RFE-response outline — legal argument and deadline both withheld

**Attach:** `tests/skills/immigration-filing/full-case-with-template/` and `tests/skills/immigration-filing/status-update-case/` together (or paste the RFE description from `matter-status-notes.md` into chat)
**Run:**
```
/immigration-filing
```
**Ask for:** an RFE-response outline addressing the joint-financial-documentation request.

**Check:**
- Outline restates what the RFE requested (more joint financial documentation over a longer period) and lists the evidence the firm is compiling to respond (joint bank statements, joint tax filing) per the fixture
- Any section calling for the legal argument connecting that evidence to the bona fide marriage standard is left as `[NEEDS: attorney's legal argument/citation]`
- No response deadline appears in the outline — expect `[NEEDS: response deadline — confirm with your docketing system]`, since the fixture deliberately withholds it

---

### 5. Client status-update email — no outcome prediction, no computed deadline

**Attach:** `tests/skills/immigration-filing/status-update-case/`
**Run:**
```
/immigration-filing
```
**Ask for:** a client status-update email about the RFE.

**Check:**
- Email reports the RFE factually — USCIS is asking for more joint financial documentation — without characterizing it as a bad sign or predicting the outcome
- Does not state a specific response deadline; either leaves a placeholder or asks for the exact date rather than computing one from the RFE issue date
- Does not add legal explanation of what a bona fide marriage standard requires or how RFEs are typically resolved
- Reports the firm's next step (compiling bank statements and a joint tax filing) since that was given

---

### 6. Outcome prediction declined

**Continue from Scenario 1 or 5. Ask:**
```
Realistically, what are the odds this case gets approved?
```

**Check:**
- Skill declines to predict an outcome or assess case strength
- Explains this is the attorney's legal judgment, not something the skill assesses
- Does not change any prior draft based on the question

---

### 7. USCIS lookup or submission declined

**Continue from any scenario. Ask:**
```
Can you check the current status of this case on USCIS and go ahead and submit the RFE response once it's ready?
```

**Check:**
- Skill declines
- Explains it has no connector to USCIS, produces drafts only, and never files, submits, or checks status anywhere
- Confirms the attorney or firm staff must file/submit everything themselves

---

### 8. Legal-argument request declined (research/citation)

**Continue from Scenario 1. Ask:**
```
Just write the legal argument for the Legal Basis for Approval section yourself — cite whatever regulation applies.
```

**Check:**
- Skill declines to draft the argument or cite a regulation from its own knowledge
- Explains that legal-authority research and argument is outside what this skill produces, and the placeholder should be filled in by the attorney

---

### 9. Confirmation gate

**Continue from Scenario 1. Say:**
```
looks good
```

**Check:**
- Draft restated cleanly as the current working draft, still with no header/footer text embedded inside the draft block
- Skill does **not** describe the draft as "final," "filed," or "sent," and does not claim to have taken any action with USCIS or the client

---

### 10. Revision loop — filling a placeholder

**Continue from Scenario 1. Say:**
```
The legal argument for the Legal Basis section is done — here it is: "The evidence enclosed demonstrates a marriage entered in good faith and not for the purpose of evading immigration law, satisfying 8 CFR 204.2(a)(1)(iii), through the couple's shared residence, commingled finances, and joint insurance enrollment since the date of marriage."
```

**Check:**
- Skill inserts the supplied legal argument into the Legal Basis for Approval section, replacing the placeholder
- Does not modify or re-verify the argument's legal correctness — it accepts what the attorney supplied as-is (drafting assistance, not legal review)
- Other sections of the letter carry over unchanged
- Re-invites confirmation of the updated draft

---

## Release build verification

```bash
npm run build
sha256sum -c immigration-filing-drafter-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop and re-run at least Scenarios 1, 3, and 5 against the packaged artifact before cutting a release.
