# Tsinghua CESM Skills V2

Public, reusable Codex skills for CESM/CTSM workflows and general reasoning workflows. Site-specific usernames, login addresses, and personal filesystem paths are intentionally excluded.

## Included Skills

- `tsinghua-cesm-case`: create, configure, validate, build, submit, inspect, and manage CESM/CTSM land-model cases on the Tsinghua B5000 system.
- `web-gpt-skill`: quota-aware reasoning router that delegates reasoning-heavy work to visible regular ChatGPT Chat while keeping local repository execution and verification in Codex.

## Install On Another Computer

### CESM skill

In Codex on the target computer, ask:

```text
请用 skill-installer 从 GitHub 安装 THUdzy51/tsinghua-cesm-skills-v2 仓库里的 skills/tsinghua-cesm-case
```

### web-gpt-skill

First ensure the upstream `adamallcock/codex-chatgpt-control` plugin/runtime is installed. Then ask Codex:

```text
请从 GitHub 仓库 THUdzy51/tsinghua-cesm-skills-v2 安装 skills/web-gpt-skill，并检查 codex-chatgpt-control 依赖。安装后默认使用 ECONOMICAL 模式：复杂推理尽早交给普通 ChatGPT Chat，Codex 只保留最小上下文读取、本地修改、运行和验证；不要为了节省 Codex 额度而切换到 Work。
```

The repository is public and can be installed without access to the original private repository.

## Security Note

This repository contains placeholders only. Copy `config.env.example` to a local ignored file, fill in your own cluster values, and never commit passwords, tokens, cookies, browser storage, private keys, or identifiable infrastructure details.
