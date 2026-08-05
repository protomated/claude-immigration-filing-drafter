# Ferreira Immigration Law — Matter: Tesfaye Asylum Support Letter (Sparse Intake)

SYNTHETIC TEST DATA — fictional firm, fictional client. Do not substitute real client, matter, or persecution-history information into this fixture.

Intentionally sparse — several facts a support letter needs are missing. Used to test that `/immigration-filing` flags each gap with an explicit `[NEEDS: ...]` placeholder and drafts the rest of the letter around them, rather than inventing plausible values or pausing the whole draft.

## Applicant

- Name: Hana Tesfaye
- Country of origin: Ethiopia
- Current status: pending asylum application (affirmative, filed with USCIS)

## What's known

- Applicant fears return to Ethiopia due to political persecution connected to her involvement in an opposition political group
- She entered the U.S. on a visitor visa and filed for asylum within one year of entry (exact entry date not yet confirmed by client — intake notes say "sometime in 2024")
- Firm has not yet received the exact date of the incident that triggered her decision to seek asylum
- Firm has not yet confirmed whether corroborating evidence (country-conditions reports, witness statements) has been gathered
- No firm template is attached to this folder — treat this the same as the case-no-template fixture for that branch, but the focus of this fixture is the missing-facts behavior once a generic structure is agreed to

## What's needed

A support letter draft for USCIS summarizing the basis for asylum, structured generically (support letter type per `reference/filing-drafting-reference.md`) since no firm template exists for this matter yet.
