# Web GPT reasoning packet

Compile local evidence into the following structure. Keep it concise and factual.

```text
ROLE
Act as the deep-reasoning consultant. Codex will execute and test locally.
Do not assume access to files or state that are not included below.

OBJECTIVE
<one precise problem or decision>

CURRENT EVIDENCE
- <observed fact / error / metric / behavior>
- <observed fact>

RELEVANT IMPLEMENTATION
<minimal code snippets, equations, schemas, call path, config, or diff>

CONSTRAINTS / INVARIANTS
- <must remain true>
- <interface/unit/time/sample constraints>

ATTEMPTS / NEW EVIDENCE
- <only relevant attempts and outcomes>

QUESTION
<the exact reasoning task>

OUTPUT CONTRACT
Return a decision-ready answer with:
1. ranked diagnosis or recommendation;
2. reasoning tied to the supplied evidence;
3. assumptions and uncertainties;
4. exact implementation plan at file/function/algorithm level;
5. minimal patch logic or pseudocode where useful;
6. tests/diagnostics, including expected results if the hypothesis is right;
7. competing explanations/failure modes and what evidence would change the conclusion.

Be concise where the decision is clear. Spend detail on the hard reasoning, not on generic background.
```

## Quota-saving prompt principle

The Web GPT response should contain enough reasoning and implementation guidance that Codex can move directly to verification rather than reconstructing the analysis locally.

For a follow-up after a failed test, send only:
- the prior conclusion that was acted on;
- the new test result/error;
- the smallest changed code/diff needed to interpret it;
- a request to revise the diagnosis.
