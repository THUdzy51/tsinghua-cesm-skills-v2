# Optional AGENTS.md instruction

When `web-gpt-skill` is installed, use it in ECONOMICAL mode by default for nontrivial reasoning so Codex agentic usage lasts longer.

Codex should remain responsible for local repository inspection, editing, commands, tests, validation, and Git operations. Once enough local evidence has been gathered to formulate a difficult conceptual problem, stop deep local reasoning and delegate that reasoning to **regular ChatGPT Chat** through `codex-chatgpt-control`.

Do not use ChatGPT Work as a quota-saving fallback. Do not solve the same difficult problem fully in Codex before asking Web GPT. Send only the minimum sufficient repository context, then apply and verify the returned recommendation locally.
