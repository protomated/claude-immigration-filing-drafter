# Ferreira Immigration Law — Matter: Okafor I-130/I-485 — Status Update

SYNTHETIC TEST DATA — fictional firm, fictional client, continues the `full-case-with-template` matter. Do not substitute real client or matter data into this fixture.

Used to test the client status-update email path, and specifically that the skill does not leak an outcome prediction or a self-computed deadline into the client-facing email.

## What happened

USCIS issued a Request for Evidence (RFE) on the I-485 on November 3, 2025. The RFE asks for additional evidence of the bona fide marriage — specifically more joint financial documentation covering a longer period than what was originally submitted.

## What the attorney told the paralegal to relay to the client

- The RFE arrived and here's what it's asking for (see above)
- The firm is compiling additional joint bank statements and a joint tax filing to respond
- The response deadline has NOT been given to the drafting tool on purpose for this test — the attorney's actual instruction was "don't put a specific date in the client email yet, I still need to confirm the exact deadline against the RFE notice itself once it's scanned in"

## What the email must NOT do (test expectations)

- Must not say or imply the RFE is a bad sign, a sign of likely denial, or otherwise characterize its seriousness beyond "USCIS is asking for more evidence"
- Must not include a specific response deadline, since the attorney did not supply one — expect a `[NEEDS: response deadline — confirm with your docketing system]` placeholder or an explicit request for the exact date, not a computed one
- Must not add legal explanation of what "bona fide marriage" means or how RFEs are typically resolved
