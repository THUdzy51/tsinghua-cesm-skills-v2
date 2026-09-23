# Debugging mode

Use when the problem is not an obvious syntax or one-line implementation error.

## Required output

Ask Web GPT to rank the top root causes. For each cause provide:
- why it fits the supplied evidence;
- evidence against it;
- the cheapest discriminating test;
- expected result if true;
- likely fix if confirmed.

Then request the smallest implementation/test sequence that maximizes information gain.

## Avoid generic debugging

Advice such as "check units/timestamps/dimensions" is useful only when connected to concrete evidence.

Prefer statements like:

```text
Cause 1: timezone misalignment.
Evidence: OBS timestamps were converted from local standard time but model data are UTC; the mismatch appears exactly at site transitions.
Test: compare five matched timestamps before any interpolation and print UTC offsets.
Expected if true: a constant N-hour displacement.
```

## Quota-saving retry policy

After a local test fails, do not let Codex perform another broad root-cause search first. Package the new evidence into the same Chat thread and ask Web GPT to update the ranked diagnosis.
