# Quota-first routing policy

Use this policy when `web-gpt-skill` runs in its default `ECONOMICAL` mode.

## Primary objective

Minimize Codex agentic reasoning while preserving correctness through repository-grounded execution and verification.

## Spend Codex effort on indispensable local work

Codex should spend effort on actions that require the real local environment:
- repository/file discovery
- narrow source inspection
- reading current diffs/configuration
- running commands and tests
- editing files
- checking actual runtime outputs
- Git operations
- validating that Web GPT assumptions match local reality

## Avoid spending Codex effort on portable reasoning

Delegate reasoning that does not require continued direct manipulation of the local environment:
- multi-hypothesis root-cause reasoning
- scientific interpretation
- algorithm/architecture design
- metric selection and statistical reasoning
- conceptual code review
- experiment/diagnostic design
- prose synthesis of technical conclusions

## Early-delegation heuristic

Delegate when any one of these is true and the answer is not obvious after a shallow inspection:
- two or more plausible conceptual causes
- a methodological or scientific choice materially affects the result
- the task spans multiple modules/stages
- the user asks "why", "which approach", "how should we design", or "is this interpretation valid"
- a first local fix failed
- a wrong decision would cause a large rewrite or invalid analysis

Do not wait for multiple failed Codex attempts before delegating in ECONOMICAL mode.

## No duplicate reasoning

Bad pattern:

```text
Codex deeply diagnoses -> Codex designs solution -> Web GPT reviews same solution -> Codex re-diagnoses
```

Preferred pattern:

```text
Codex shallow inspection -> Web GPT deep diagnosis/design -> Codex apply/test -> targeted Web GPT follow-up only if new evidence appears
```

## Context minimization targets

These are soft targets, not rigid limits:
- prefer 1-3 directly relevant files/functions
- prefer 30-100 lines of key logs instead of full logs
- include schemas/shapes/units/timestamps as compact facts
- include only the current diff when a regression is change-specific
- keep the consultation packet small enough that Web GPT can reason from it without unrelated repository noise

If more context is truly required, add it incrementally in the same thread instead of front-loading the entire repository.

## Surface invariant

ECONOMICAL reasoning delegation goes to regular Chat only.

Never substitute Work simply because Chat is temporarily unavailable. Work may be used only for a separate user-requested Work objective, not as a quota-saving fallback.

## Fallback

If Chat delegation cannot run:
- report the reason;
- complete locally when required and feasible;
- do not claim quota savings for that task.
