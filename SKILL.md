---
name: review-design
description: Review PRs, commits, diffs, or local code changes for design quality and concrete defects. Use for requests such as "review this PR", "审查这个改动", or design review; compare simpler viable implementations. Does not apply to summarizing a PR or learning from past review comments.
---

# Review Design

Find changes the author should make, including design problems in code that works. The goal is the simplest sufficient implementation of the requested behavior, with existing safety and compatibility intact. Review only; applying or publishing changes needs authorization from the surrounding task.

## Apply this review request

A short request such as `/review-design PR <url>` or “review this change” invokes the complete review below. Resolve the requested diff and its base from repository context; ask only if materially ambiguous or inaccessible. Respect the surrounding task's scope and permissions. Default to Chinese unless the user requests another language.

先根据任务目标和基线中的调用方，判断最小充分实现，再与完整 diff 比较。按需读取明确提供的仓库规则和语义文档。重点找不必要的机制、错置职责、共享接口增长以及可证实的错误；对建议的修法检查时序与行为是否保持。不要将新增说明或测试对当前机制的描述自动当作不可改变的需求。除非用户明确要求其他语言，用中文给出具体发现、必要取舍和实际验证范围，保留重要的不确定性。

## Establish what has to work

Read the request and the complete diff. Separate the intended user or caller outcome from the mechanism chosen to implement it. Check the base version's contracts, consumers and analogous features before relying on new design notes. A new test demonstrates the implementation's chosen behavior; it does not by itself establish that an extra mechanism is necessary. Respect explicit user requirements and established contracts. If a proposed behavior's necessity is uncertain, identify the tradeoff rather than silently declaring it required or disposable.

Sketch a smaller path using what the base already provides. Trace the actual producer, owner, transport or projection, consumer and lifetime; omit stages that do not exist. Then locate what the submission adds beyond that path and the concrete guarantee each addition buys. An implementation can be correct while placing a capability at the wrong level or paying too much for an optional benefit.

Read the applicable repository instructions. If `docs/defensive-patterns/README.md` or an explicitly supplied reference copy exists, use its index to read relevant rules and their boundaries. Follow semantic links needed to settle a question, not every linked document. Match a rule's conditions to the code and current contract; a modern example neither creates a missing historical API nor proves every similar mechanism is needed.

## Challenge the change at its consequential boundaries

- **Ownership:** compare each new service, plugin, cache or coordinator with the operations and records its dependencies already own. State what could be derived or composed there, and what the smaller placement would lose. Do not replace duplication with another cache or move the same complexity under a different name.
- **Shared interfaces:** trace new fields, parameters, events and exported types to production consumers. Distinguish layers that interpret them from forwarding layers. Try binding feature-specific data and behavior at the caller or registration point. One consumer invites this comparison but does not prove the interface is wrong; external callers and trust boundaries need their own evidence.
- **Frontend:** follow initial load, live updates, reconnect and history through existing channels to rendering. For a presentation change that expands host state, wire payloads or generic control code, construct the feature-local alternative first. Explain any missing signal or guarantee that actually forces a shared change. Do not confuse a preferred transport rule with evidence that a new transported fact is required.
- **Hot paths and representations:** trace work per request, keypress, node or update, including physical I/O and allocations. Compare a cheaper existing read, a bounded representation or a documented fallback before adding memoization, precision machinery or recovery state. Establish valid input ranges, tolerances and intermediate operations; a hypothetical extreme or an unmeasured benefit is not a proof either way.
- **Scope:** distinguish what the requested outcome needs from independent policy changes, cleanup or optional optimization. Identify meaningful work that can be removed or separated without hiding its lost benefit.

Correctness issues discovered along these paths belong in the review. Investigate all substantive concerns; do not stop at a fixed number or pad the answer with style preferences.

## Check the proposed repair

For each finding, verify its premise and trace the proposed alternative through the same consumers. Try a concrete case that would make the recommendation wrong. In particular, two reads of one fact can protect different moments or boundaries: record the read time, any await, mutation, invalidation or trust transition between them. Same owner does not imply same snapshot. Cancellation, resource release and awaited teardown are also different guarantees.

A smaller fix must still implement the behavior it promises, including failure and recovery. Cross-item version equality does not alone establish each item's consistency. A rendering-boundary concern is not resolved by merely caching the same invalid composition. Keep a supported design concern even when the full repair remains uncertain, but label the proposed repair's missing proof and do not recommend an unsafe deletion as equivalent. Never infer that a security check is unnecessary merely because no internal caller or exploit was found.

## Deliver

Use Chinese by default; honor an explicit request for another language. Start with one or two sentences describing the change and where its essential mechanism lives. Then provide:

- Actionable findings, ordered by consequence: a clickable source location or symbol link, the causal issue and affected condition, supporting code, and a smaller or corrected path with its material tradeoff. Distinguish demonstrated defects, supported design concerns and unresolved product choices.
- A compact design assessment: actual files and additions/deletions/net change, separating production, tests/docs and generated/mechanical changes; counts of added or expanded shared APIs, and material new types, events and state with their consumers; and the strongest viable alternative. For alternatives give honest ranges or explicit unknowns, comparing complete alternatives against the same baseline, including affected consumers and testing obligations. Do not turn every local branch into a report table.
- For frontend changes, explain the complete relevant data path. Account for frontend-driven control-layer changes and new remote events, including their purpose and why existing paths suffice or fall short, even when the changes are justified.
- What was actually checked, tests actually run and consequential uncertainty. Source inspection is not a passing test. A justified no-finding result is acceptable.

Before sending, remove unsupported claims and contradictory recommendations, not inconvenient concerns. Ensure the final recommendation follows the analysis rather than listing a smaller alternative and then declaring the original necessary without evidence.
