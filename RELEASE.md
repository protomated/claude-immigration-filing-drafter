# Immigration Filing & Status Update Drafting Skill v1.0.1

Adds Legal Builder Hub freshness frontmatter (`freshness_category: regulatory`, 6-month window — immigration is a fast-moving domain, though this skill bundles no citable external law itself). No functional changes.

## What's included

### `/immigration-filing` — Immigration Filing & Status Update Drafting Skill

A filing narrative and status-update drafting assistant for solo and small immigration-practice attorneys:

- **Filing narrative sections:** support letters, cover letters, and RFE-response outlines, drafted by populating your firm's own template with case facts you supply — never a generic structure substituted silently.
- **Client status-update emails:** a plain-English email drafted from a case-status change you report — no outcome prediction, no added legal characterization.
- **No facts invented:** any missing fact — a date, a relationship detail, a piece of evidence — is flagged as an explicit `[NEEDS: ...]` placeholder in the draft, never guessed.
- **No legal argument from model knowledge:** persuasive legal argument and citations to statute, regulation, or case law come only from your firm's template or your own input; anything else is left as a placeholder for you to fill in.
- **No deadline calculation:** the skill never computes a filing or response deadline from general USCIS processing rules — a deadline appears only when you state the exact date; otherwise it's flagged for your docketing system to confirm.
- **No USCIS interaction:** no connector, no case-status lookups, no e-filing or submission. A status update is drafted only from what you report happened.

Handles: the narrative drafting time behind every immigration filing, and the client status-update emails that come with each case-status change — for firms juggling 50 to 200+ pending matters at once.

## Setup

Install time: about 5 minutes. Download the zip, drag it into Claude Desktop's Extensions panel. No connectors to authorize. Open a new chat, type `/skills`, and verify `/immigration-filing` appears. Optionally attach a workspace folder with case facts and your firm's own filing template before running it.

## Compliance

Requires Claude for Work, Claude Team, or Claude Enterprise (or the Claude API under a signed DPA) before attaching a real case file. Every draft carries an "ASSISTED IMMIGRATION FILING DRAFT — ATTORNEY REVIEW REQUIRED BEFORE FILING OR SENDING" header and footer as chat text around the draft, never inside the copyable draft block. The skill never predicts a case outcome, never writes legal argument or cites law from its own knowledge, never computes a filing deadline, never invents a case fact, and never looks up, files, or sends anything to USCIS or a client.
