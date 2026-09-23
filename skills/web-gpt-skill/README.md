# web-gpt-skill

A quota-aware Codex skill that makes Codex usage last longer by delegating reasoning-heavy subtasks to the user's visible **regular ChatGPT Chat**, while keeping local repository execution and verification in Codex.

## Goal

The goal is not to increase or bypass Codex limits. The skill conserves Codex usage by changing the division of labor:

```text
Codex: inspect minimally -> Web GPT Chat: reason deeply -> Codex: edit/run/test
```

The default mode is `ECONOMICAL`.

## Key behavior

- delegates reasoning early instead of after multiple long Codex attempts;
- prevents "Codex solves it first, Web GPT solves it again" duplication;
- uses regular Chat for quota-saving delegation;
- never silently substitutes ChatGPT Work for this purpose;
- minimizes repository/log context before handoff;
- asks Web GPT for implementation-ready reasoning and validation tests;
- keeps Codex as the authority for actual local files and runtime results;
- follows upstream submit-once, same-thread polling, verified configuration, and structured-blocker behavior.

## Dependency

Install the upstream `adamallcock/codex-chatgpt-control` Codex plugin/runtime first. The upstream project currently documents Codex Desktop setup through its plugin marketplace:

```bash
codex plugin marketplace add adamallcock/codex-chatgpt-control --ref main
codex plugin add codex-chatgpt-control@codex-chatgpt-control
```

A compatible browser bridge and a visible signed-in ChatGPT session are still required for real Chat browser runs.

`web-gpt-skill` does not copy or fork the upstream browser runtime. This keeps ChatGPT UI compatibility fixes upstream.

## Install this skill from the user's private skills repository

Ask Codex:

```text
请从 GitHub 仓库 THUdzy51/tsinghua-cesm-skills-v2 安装 skills/web-gpt-skill，并同时检查 codex-chatgpt-control 依赖是否已经安装。安装后默认使用 ECONOMICAL 模式：复杂推理尽早交给普通 ChatGPT Chat，Codex 只保留最小上下文读取、本地修改、运行和验证；不要为了省 Codex 额度而切换到 Work。
```

The repository is public and can be installed without access to the original private repository.

## Modes

- `ECONOMICAL` (default): aggressively offload portable reasoning to regular Chat.
- `BALANCED`: delegate only materially difficult reasoning.
- `LOCAL`: Codex-only execution and reasoning.

Example:

```text
$web-gpt-skill ECONOMICAL: inspect this failing pipeline only enough to formulate the problem, delegate the root-cause reasoning to regular Chat, then implement and test the recommendation locally.
```

## Files

- `SKILL.md` — executable agent policy.
- `references/quota-first-routing.md` — early delegation and no-duplicate-reasoning rules.
- `references/prompt-contract.md` — minimal consultation packet.
- `references/scientific-reasoning.md` — process-vs-metric and causal/scientific reasoning.
- `references/debugging.md` — ranked root causes and discriminating tests.
- `references/usage-validation.md` — A/B method for checking whether the workflow actually reduces Codex usage.
- `examples/AGENTS.md.snippet.md` — optional repository-level instruction.

## Important limitation

Product usage accounting can change. This skill therefore must not promise a fixed amount of savings or rely on hidden/private interfaces. Its durable principle is simply to keep deep, portable reasoning out of Codex when a visible regular Chat consultation can do that reasoning instead.
