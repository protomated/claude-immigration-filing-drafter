# Connectors

This plugin requires no MCP connector. `/immigration-filing` drafts from case facts and your firm's own filing template — typed into chat, or attached as a workspace folder in Claude Desktop / Cowork — no separate authorization step, no credentials.

## How the plugin reads your files

Cowork's filesystem access is attach-only: the plugin can only see files inside a folder you've explicitly attached to the conversation. It does not browse your computer, does not search beyond that folder, and does not retain access after the conversation ends. It never connects to USCIS, your case-management system, or any other outside service — there is nothing for this plugin to authorize.

To use `/immigration-filing`, attach a folder containing:

- **Case facts** — petitioner/applicant and beneficiary details, filing type, procedural history, and the specific facts a filing narrative needs. Anything not included is flagged with a `[NEEDS: ...]` placeholder in the draft rather than guessed.
- **Your firm's own filing template**, for whichever narrative type you need (support letter, cover letter, RFE-response outline) — the skill populates it, it does not redesign it. If you don't attach one, the skill asks whether your firm has one before falling back to a generic structure.
- For a client status-update email, you don't need to attach anything — just tell the skill what changed in the case, in chat.

The plugin drafts the requested narrative section or status-update email for your review. It never looks up a case's status, logs into a USCIS account, or files or sends anything itself — you review, finalize, and send every output yourself.

## Privacy note

The plugin processes case facts and any attached template files within your Claude Desktop / Cowork conversation under your Claude plan's data handling terms. No case fact, draft, or status update is transmitted to Protomated or any third party.

Before attaching a real case folder — case facts, A-numbers, dates of birth, immigration or persecution history — confirm you are on Claude for Work, Claude Team, or Claude Enterprise, or using the Claude API under a signed Data Processing Agreement (DPA). See the main README for plan requirements.

## Using this in ChatGPT Desktop

This skill also works in ChatGPT Desktop. There's no Filesystem connector to attach there — instead, attach your case facts and firm template directly to the conversation before running the skill.
