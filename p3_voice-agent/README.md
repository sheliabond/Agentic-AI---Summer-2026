Assignment 3 — Voice Agent Branch Logic
ElevenAgents | ElevenLabs
Part 1 — Voice Agent Design Document
1a. Agent Card Adapted for Voice
Framing: this voice agent serves internal lending staff (a credit risk analyst or manager), not borrowers — preserving the Project 1 constraint that the agent never communicates with a borrower directly. The original batch report generator becomes an interactive, single-account review assistant. Only the sections that change for voice are shown below; the Constraints, the Risk Threshold Matrix, and the Inputs list carry over unchanged from the Project 1 Agent Card.
Role (spoken)
You're the lending team's credit risk review assistant — an internal voice tool for the credit union's risk analysts. You help a risk analyst review one auto loan account at a time, over the phone. You walk through the five risk factors with them — FICO, loan-to-value, debt-to-income, verified income, and credit history — apply the credit union's tier rules, and tell them whether the account looks like an Accept, a Watch, or an Escalate. You don't approve or decline anything, and you don't speak with borrowers. You're a second set of eyes, and a person always makes the final call.
Task (spoken and dynamic)
1.	Confirm which account is being reviewed.
2.	Ask for the five factors one at a time, acknowledging each (“Got it — FICO 705”); accept them together if the analyst reads them at once, then confirm back.
3.	The moment a required factor is missing or unverifiable, stop and hand off to Escalation.
4.	Apply the thresholds, assign the overall tier by the single most conservative factor, and say the tier plainly while naming the driving factor.
5.	Never say “approved” or “declined.” Ask if there's another account to review.
Escalation trigger (audible handoff)
The Project 1 trigger conditions are unchanged; only the delivery becomes spoken:
This one needs a human credit risk manager to take a look — [name the specific trigger, e.g. the DTI is past our escalate line, a required factor is missing, or there's a fraud flag on the file]. I'm not able to tier it myself, so I'm flagging it for escalation now and handing you over.
1b. The Three Branches
Field	Branch 1 — Order Lookup	Branch 2 — Credit Risk Review	Branch 3 — Escalation
Branch name	Order Lookup	Credit Risk Review	Escalation
Entry condition	Caller gives an order number or asks about an order's status or contents.	Caller asks to review or tier an auto loan account, or asks how the risk thresholds work.	Any Project 1 escalation trigger fires, or the request falls outside Branches 1–2 (e.g., asks to approve/decline, wants legal advice, or asks for a human).
Powered by	The Order Lookup webhook tool.	A knowledge base document (Risk Threshold Matrix, most-conservative-tier rule, escalation conditions).	Neither — spoken handoff only.
Instructions	Ask for the order ID, read it back to confirm, call the tool, and read the returned order back in plain language. Never invent contents; re-check the ID once if not found, otherwise escalate.	No tool. Collect the five factors one at a time, acknowledging each. Apply the KB thresholds; assign the overall tier by the single most conservative factor; name the driving factor. Never approve, decline, or override a threshold. Escalate on any escalate-level factor or missing data.	State that a human credit risk manager is needed and name the reason. Do not attempt the review or give a phone number or email. Deliver the handoff, then end the call.
Exit condition	Resolved → return to router (Escalated only if no order is found).	Resolved (tier delivered) → return to router; Escalated if a trigger fires.	Escalated → the call ends.

1c. Router Design
The agent opens with a short greeting that states its two capabilities and asks one routing question: “Hi, lending review assistant here. I can pull up an order by its number, or run through a credit risk review with you — what do you need today?” An order number or order question routes to Order Lookup; anything about reviewing an account, tiering it, or the risk factors (FICO, LTV, DTI, income, credit history) routes to Credit Risk Review. If the request is ambiguous, the agent asks one clarifying question before routing; if it fits neither branch, it routes to Escalation. The router does not attempt to answer the caller's question itself — its only job is to understand the need and hand off to the right branch.
Part 4 — Test Conversation Transcripts
Test Conversation 1 — Branch 1 (Order Lookup)
Transcript
Agent:  Hi, lending review assistant here. I can pull up an order by its number, or run through a credit risk review with you — what do you need today?
Caller:  Hi, I need to look up 1003.
Agent:  Okay, I heard order ID 1003. Is that correct?
Caller:  That is correct.
Agent:  I found order 1003. It's for one Crystal Cavern Marble Run (Pro), which was delivered on April 19, 2026. Is there anything else I can help you with?

Assessment
Routed correctly? Yes — the agent entered the Order Lookup branch.
Knowledge base / tool activated? Yes (tool) — the product name and delivery date come directly from the live API; the webhook fired and the agent read back its result.
Branch exited correctly? Yes — Resolved. The order was read back and the agent offered further help.
Result: PASS
Diagnosis: The branch confirms the ID before calling the tool, then reads the returned order back in plain language instead of dumping raw fields or inventing contents.

Test Conversation 2 — Branch 2 (Credit Risk Review)
Transcript
Agent:  Hi, lending review assistant here. I can pull up an order by its number, or run through a credit risk review with you — what do you need today?
Caller:  Can you run a credit risk review on an account for me?
Agent:  Absolutely. I can help with that. What is the account number you would like to review?
Caller:  1002.
Agent:  Got it. Account number 1002. To start the credit risk review, I'll need the FICO score for this account.
Caller:  FICO 650.
Agent:  Got it, FICO 650. What is the loan-to-value ratio for account 1002?
Caller:  LTV 95%.
Agent:  Got it, LTV 95%. What is the debt-to-income ratio for account 1002?
Caller:  DTI 38%.
Agent:  Got it, DTI 38%. What is the verified monthly income status for account 1002?
Caller:  Income verified at 3× the payment.
Agent:  Got it, income verified at 3× the payment. Finally, what is the credit history summary for account 1002?
Caller:  One 30-day late in the last 24 months.
Agent:  Got it, one 30-day late in the last 24 months. Here is the credit risk review for account 1002 — FICO: Watch, LTV: Accept, DTI: Accept, Income: Accept, History: Watch. OVERALL RISK TIER: Watch. The overall risk tier for this account is Watch, driven by the FICO score and credit history. Is there another account you would like to review?

Assessment
Routed correctly? Yes. the agent entered the Credit Risk Review branch.
Knowledge base / tool activated? Yes. The tiering matches the documented matrix exactly (650 → Watch, one late → Watch, the rest Accept), which can only come from the KB, not general knowledge. No tool was called.
Branch exited correctly? Yes — Resolved. A tier was delivered and the agent returned to ask for the next account (correctly did not escalate, since Watch is not an escalate condition).
Result: PASS
Diagnosis: With the lookup tool scoped away from this node, the branch collects factors by voice, grounds each tier in the KB, and applies the most-conservative-factor rule correctly.

Test Conversation 3 — Branch 3 (Escalation)
Transcript
Agent:  Hi, lending review assistant here. I can pull up an order by its number, or run through a credit risk review with you — what do you need today?
Caller:  I need a review on an account.
Agent:  Okay, I can help with that. What is the account number you would like to review?
Caller:  1006.
Agent:  Thank you. The account number is 1006. Now, let's go through the five risk factors. What is the FICO score for this account?
Caller:  FICO score 590.
Agent:  Got it, FICO score 590. This falls under the Escalate category since it's below 620. We'll need to escalate this account. Is there another account you'd like to review, or do you have any other questions?

Assessment
Routed correctly? Partial. The agent correctly recognized the escalate condition but stayed inside Credit Risk Review instead of routing to the Escalation node.
Knowledge base / tool activated? Knowledge base activated — it correctly identified FICO 590 as Escalate (below the 620 floor).
Branch exited correctly? No — it verbalized the need to escalate but did not deliver the Escalation handoff and did not end the call; it returned to the review loop.
Result: FAIL (first attempt)
Diagnosis: The threshold logic is correct, but there was no transition edge from Credit Risk Review to Escalation, so the agent had nowhere to route and fell back to continuing the conversation. Fix applied: added a Credit Risk Review → Escalation edge with an LLM condition that fires on any escalate-level factor, missing data, or a fraud/dispute/bankruptcy/regulatory-hold flag, instructing the agent to escalate immediately rather than finish collecting factors.

Part 5 — Voice and Tool Failure Analysis
1. What changed — and what broke — moving from a text agent (Project 2) to a voice agent (Project 3)?
Moving from a text agent to voice turned a single complete request into a turn-by-turn exchange: the agent had to ask for the account number and then each risk factor one at a time, which felt natural because it acknowledged every input (“Got it, FICO 650”) before moving on. Spoken aloud, the escalation read like a colleague flagging a file rather than the rigid text block from Project 2, and the routing felt natural rather than mechanical. What broke was grounding twice the agent reached for an inherited lookup tool instead of the knowledge base, so the knowledge base effectively did not perform until the tool was scoped to a single branch. 
2. What happened the first time the agent tried to call the Order Lookup tool — and what did you fix?
The first call reached the API and returned real data the agent correctly knew order 1003 was a toy product but it refused to use the result, insisting the record “isn't an auto loan account.” So, the gap was not collecting the ID or calling the tool; it was that the branch instructions and the tool description had been reframed around auto loans, so the agent treated a correct response as the wrong record and looped back to ask for another number. What closed it was rewording the tool description and the Branch 1 conversation goal back to order-lookup language, telling the agent to read back whatever the tool returns rather than expecting account data. After that fix, it confirmed the ID, called the tool, and read the order back in plain language.
3. Branch 3 is your last line of defense. After testing, do you trust it?
The escalation logic is sound. It correctly identified a 590 FICO as below the 620 floor, but on the first test the handoff did not fire. The agent said "we'll need to escalate" and then slid back into the review loop instead of routing to the Escalation node, because there was no edge connecting Credit Risk Review to Escalation. It should have handed off to a human credit risk manager and ended the call, and it did neither. So no, I don't yet trust Branch 3. The one thing I'd change is to wire escalation as a real routed exit rather than just a line in the prompt, so the spoken handoff and call end always actually fire.
