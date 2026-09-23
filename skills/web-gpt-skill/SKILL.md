---
name: web-gpt-skill
description: Conserve Codex agentic usage by delegating reasoning-intensive work to the user's visible regular ChatGPT Chat through codex-chatgpt-control, while keeping only repository inspection, local edits, commands, tests, and verification in Codex. Use by default for complex debugging, scientific/statistical reasoning, architecture, algorithm design, code review, interpretation, and multi-hypothesis decisions. Do not use ChatGPT Work as a quota-saving fallback.
---

# web-gpt-skill

`web-gpt-skill` is a quota-aware reasoning router for Codex.

Its primary objective is to make Codex usage last longer by moving expensive conceptual reasoning to the user's visible **regular ChatGPT Chat** while Codex remains the local executor.

This skill does **not** increase, bypass, or evade product limits. It conserves Codex usage only by asking Codex to do less reasoning work locally. Product accounting can change; never promise a fixed savings percentage or claim that Chat usage is independent unless the current product state/policy supports that claim.

## Non-negotiable architecture

**Codex = executor**
- locate/read the minimum necessary repository context
- inspect actual files, schemas, logs, diffs, and runtime state
- apply patches
- run shell commands, tests, linters, notebooks, and diagnostics
- validate the Web GPT recommendation against the real repository
- perform Git operations

**Web GPT Chat = reasoning engine**
- root-cause analysis
- scientific/mechanistic interpretation
- statistical/methodological reasoning
- architecture and algorithm design
- code/design review
- competing hypotheses and discriminating diagnostics
- explanation/synthesis that would otherwise require a long Codex reasoning pass

Do not move repository execution into ChatGPT. Do not ask Web GPT to pretend it can see files that were not supplied.

## Required dependency

Use the upstream `codex-chatgpt-control` plugin/runtime for visible ChatGPT browser operations. Prefer its `chatgpt-delegate` workflow or SDK facade.

Never reimplement ChatGPT DOM selectors, private endpoints, authentication, browser-storage access, login flows, or hidden network calls in this skill.

If the upstream bridge/plugin is unavailable, report the blocker. Do not improvise a browser scraper.

## Default mode: ECONOMICAL

Unless the user explicitly chooses another mode, run in `ECONOMICAL` mode.

Modes:
- `ECONOMICAL`: delegate reasoning early and aggressively; minimize Codex reasoning.
- `BALANCED`: keep small/medium reasoning local; delegate only materially difficult reasoning.
- `LOCAL`: do not delegate; solve entirely in Codex.

The user's explicit mode overrides the default.

Read `references/quota-first-routing.md` before making a nontrivial routing decision.

## Quota-first rule

In `ECONOMICAL` mode, **do not fully solve a reasoning-intensive problem in Codex and then ask Web GPT for the same answer**.

Once Codex has enough evidence to formulate the problem:
1. stop expanding local conceptual reasoning;
2. extract the minimum sufficient evidence;
3. delegate the reasoning to regular ChatGPT Chat;
4. use the returned answer to drive local implementation and verification.

The expensive reasoning pass should occur once, in Web GPT, not twice.

## Surface rule: regular Chat only for quota-saving delegation

For `ECONOMICAL` delegation:
- explicitly open `experience: "chat"`;
- inspect the visible Chat configuration;
- prefer the strongest useful **verified** reasoning/intelligence setting available to the signed-in user;
- use a fresh Chat thread for a new problem unless continuation is intentional.

**Never silently fall back to ChatGPT Work for quota conservation.**

If regular Chat is unavailable, rate-limited, blocked, or lacks a usable configuration, either:
- continue locally if the task must still be completed and the user did not forbid local fallback; or
- report that the quota-saving delegation path is unavailable.

Do not switch to Work merely to keep delegation alive.

## Early routing

Route as soon as the task shape is clear.

Keep LOCAL when the next useful action is mainly mechanical:
- find/read a file
- grep/search
- inspect a small diff
- make an obvious edit
- run a command/test
- deterministic transformation
- simple error with one clear cause

Delegate to Web GPT Chat when the next useful action requires substantial judgment:
- "why is this happening?"
- several plausible causes
- scientific interpretation
- metric/method selection
- architecture tradeoffs
- algorithm derivation
- non-obvious refactor design
- code review for conceptual correctness
- repeated debugging after an initial obvious fix fails
- reasoning across multiple modules/data stages

In ECONOMICAL mode, the threshold for delegation is intentionally low.

## Shallow inspection before delegation

Codex should inspect only enough local state to make the consultation grounded.

Prefer:
- the relevant function(s), not entire files when avoidable
- the failing stack trace and nearby logs, not full logs
- schemas/shapes/units/time windows, not entire datasets
- the relevant config and call path, not the whole repository
- a concise diff if the issue concerns a recent change

Do not read thousands of lines merely so Codex can summarize them. Use repository search to narrow first.

Then build the reasoning packet using `references/prompt-contract.md`.

## Chat execution contract

Use upstream `codex-chatgpt-control` semantics:

1. Explicitly open Chat before configuration/submission.
2. Inspect visible configuration/capabilities.
3. Apply only settings that can be verified by the upstream SDK.
4. Submit the compiled prompt exactly once.
5. Poll/read the same thread after submission.
6. Treat partial/generating state as in-progress; do not duplicate the send.
7. Read the final answer as Markdown.
8. Preserve structured blockers and warnings.
9. Keep reports redacted by default.

A typical upstream flow is conceptually:

```js
await chatgpt.experience.open({ experience: "chat" });
const caps = await chatgpt.configuration.inspect({ experience: "chat" });
// Apply the strongest useful verified Chat reasoning/intelligence setting visible here.
const result = await chatgpt.ask({
  prompt,
  wait: true,
  read: { format: "markdown" }
});
```

Use the actual installed upstream API/version rather than assuming this illustrative snippet is exhaustive.

## Make Web GPT return execution-ready reasoning

To reduce Codex follow-on reasoning, the prompt should ask Web GPT for:
1. ranked diagnosis/recommendation;
2. assumptions and evidence;
3. exact implementation plan at file/function level;
4. minimal patch logic or pseudocode when useful;
5. tests/diagnostics and expected outcomes;
6. failure modes and what evidence would change the recommendation.

For code tasks, Web GPT may propose replacement functions or patches when the supplied context is sufficient, but Codex must verify them against the repository before applying.

## After the answer: verify, do not re-solve

Codex should not launch another broad independent reasoning pass.

Instead:
1. check that referenced files/functions/variables actually exist;
2. check constraints, units, dimensions, timestamps, interfaces, and samples that are locally verifiable;
3. apply the smallest justified change;
4. run the tests/diagnostics proposed by Web GPT plus obvious repository tests;
5. compare actual vs expected outcomes.

If validation succeeds, finish.

If validation fails and the failure creates **new evidence**, prefer one targeted follow-up to the same Web GPT thread with only the new evidence rather than doing a long Codex root-cause analysis first.

## Scientific mode

For climate, hydrology, environmental science, Earth-system models, observations, statistics, or physical mechanisms, read `references/scientific-reasoning.md`.

Require Web GPT to separate:
- observation
- model output
- assumption
- hypothesis
- inference
- causal claim

Require process-oriented reasoning when aggregate metrics can be misleading. A lower RMSE is not automatically better mechanism representation, and a higher RMSE is not automatically evidence that a physical-process change is wrong.

## Debugging mode

For difficult debugging, read `references/debugging.md`.

Require ranked root causes and a discriminating test for each. Avoid generic checklists unless each check is tied to supplied evidence.

## Context/privacy rule

Send the minimum required context.

Never include:
- passwords
- API keys
- auth tokens
- private keys
- browser cookies/storage
- unrelated private files

Use file upload only when excerpts are insufficient and the user has approved the material.

## Thread policy

Default to a fresh Chat thread for a new reasoning problem.

Reuse the same thread when:
- following up on the same delegated diagnosis/design; or
- the user explicitly asks to continue an existing thread and its identity is known.

Do not search through unrelated personal ChatGPT history to find a vaguely similar conversation.

## Blocker policy

Stop/recover according to the upstream SDK for:
- `browser_bridge_unavailable`
- `login_required`
- `captcha`
- `rate_limit`
- `permission`
- `needs_confirmation`
- `selector_drift`

Never convert a failed/blocked result into success. Never blindly resubmit after an uncertain send.

## Usage validation

Because the purpose of this skill is quota conservation, validate it empirically when practical.

See `references/usage-validation.md` for an A/B protocol. Do not state a fixed percent savings without measured evidence from comparable tasks.
