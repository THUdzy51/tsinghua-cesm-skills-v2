# codex-chatgpt-control -> web-gpt-skill redesign

## 1. What the upstream project should continue to own

`codex-chatgpt-control` is the transport/runtime layer. Its reusable strengths include:
- semantic Chat/Work operations instead of hand-written selector scripts;
- explicit `experience.open` behavior;
- visible capability discovery and configuration inspection;
- strict postcondition verification after configuration changes;
- visible signed-in session only;
- redacted reports by default;
- explicit upload/path permissions;
- structured blockers;
- submit-once / poll-the-same-thread behavior;
- thread/task continuity.

The upstream plugin also provides a `chatgpt-delegate` workflow specifically for visible Chat/Work delegation. `web-gpt-skill` should build on those capabilities rather than fork the browser runtime.

## 2. Our actual objective

The primary requirement is **Codex quota conservation**.

The old design treated Web GPT mainly as an optional second opinion after Codex had already reasoned substantially. That is quality-oriented but can duplicate expensive reasoning.

The redesigned policy is:

```text
Codex shallow grounding
        -> Web GPT regular Chat deep reasoning
        -> Codex local implementation/test
```

Codex should not deeply solve the portable reasoning problem before delegation.

## 3. Why the split matters

Tasks such as repository search, patching, command execution, and tests require local state and belong in Codex.

Tasks such as root-cause ranking, scientific interpretation, algorithm design, architecture tradeoffs, methodology critique, and code reasoning are portable once the relevant evidence has been extracted. Those are the preferred delegation targets.

## 4. Chat-only quota-saving invariant

`ECONOMICAL` mode explicitly uses regular Chat for reasoning delegation and forbids silent fallback to Work for quota-saving purposes.

This is intentionally a workflow policy rather than a claim that product accounting can never change. The skill must never promise guaranteed or fixed savings.

## 5. Context minimization

Large Codex context itself costs work. Therefore the router does not merely move the final reasoning answer; it also minimizes how much Codex must read before handoff.

The preferred pipeline is:
1. repository search;
2. narrow file/function/log extraction;
3. compact evidence packet;
4. Web GPT reasoning;
5. local apply/test.

## 6. Avoid duplicate reasoning

Old pattern:

```text
Codex diagnose -> Codex design -> GPT review -> Codex reconsider
```

New pattern:

```text
Codex gather evidence -> GPT diagnose/design -> Codex verify -> targeted GPT follow-up only when new evidence appears
```

## 7. Scientific specialization

For research workflows the skill asks Web GPT to distinguish observation, model output, assumption, hypothesis, inference, and causal claim. It also separates process skill from aggregate state-variable metrics such as RMSE so compensating errors are less likely to be misinterpreted.

## 8. Reliability boundary

Browser/UI maintenance stays upstream. Our skill owns only:
- routing policy;
- context minimization;
- prompt compilation;
- scientific/debugging contracts;
- post-consultation local verification;
- quota-conservation behavior.
