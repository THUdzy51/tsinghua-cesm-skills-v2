# Usage validation

The skill is intended to conserve Codex usage, but savings depend on task complexity, context size, model/configuration, and current product accounting.

## A/B protocol

Use two genuinely comparable tasks or two reproducible variants of one task.

### A: Codex-only
- run in `LOCAL` mode;
- record the available Codex usage/status before and after if the current client exposes it;
- record task scope and outcome.

### B: web-gpt-skill ECONOMICAL
- use the same repository/context scale;
- delegate the main conceptual reasoning to regular Chat;
- record Codex usage/status before and after;
- record number of Web GPT consultations and outcome quality.

Compare:
- Codex allowance consumed, if measurable;
- total task completion quality;
- number of repeated local reasoning/debug loops;
- time spent on rework.

## Interpretation

Do not use one easy task and one hard task as evidence of savings.

Do not promise a fixed percentage. The design goal is simply:

```text
less deep reasoning inside Codex + more execution-focused Codex work
```
