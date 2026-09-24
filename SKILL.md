---
name: review-code
description: Review code, diffs, commits, and pull requests for unnecessary complexity, public API growth, duplicated authority, and simpler viable designs. Include change counts and material tradeoffs; scrutinize frontend data flow, shared control changes, and new remote events.
---

# Review Code

Decide which design costs are justified. Treat the submitted change as one candidate design, not as the specification of what is needed. Establish what is actually required from evidence outside the change, sketch the smallest implementation that meets it with existing mechanisms, and then judge the submission against that sketch. Preserve real behavior and safety throughout. Report correctness and security defects you encounter without claiming a complete audit.

## Working stance

The most common failure in this kind of review is reading the mechanism correctly and then explaining why the submitted design is fine. Guard against it deliberately:

- A PR description, commit message, design note, new test, new interface, or new abstraction is a claim made by the change. It is evidence of intent, not evidence of a requirement. Only cite it as a requirement when an independent source (production caller, existing contract, persisted data, security or compatibility constraint, measured need) confirms it.
- The absence of visible consumers is missing evidence, not proof of disuse. Say what you could not find and how confident you are.
- The opposite error is treating every current behavior as mandatory. Some behavior is incidental to how the code happened to be written. Distinguish behavior something depends on from behavior nothing is known to depend on, mark the latter unverified, and say what evidence would settle it. Do not silently assume either way.
- A local defect or optimization is never the end of the review. After noting it, return to the flow-level question: who owns this fact, who decides it, and does this mechanism need to exist.

Keep the review proportionate. Small or mechanical changes need a short review with statistics; a well-supported no-finding result is a legitimate outcome. Do not aim at a number of findings.

## Establish the requirement and the existing path

Name the review baseline (the branch or commit you compare against) and the outcome the change affects for users, callers, or operators.

Build a requirement ledger before evaluating design. For each behavior the change adds, alters, or preserves, record:

| Column | Content |
| --- | --- |
| Behavior | What must happen, stated in terms of observable effect, not mechanism. |
| Source | Production caller, existing contract, persisted or wire format, security or compatibility constraint, measured performance need, or explicit user requirement. Cite location. |
| Status | **Required** (source independent of the change), **Chosen** (an implementation decision, including flexibility or optimization no current caller needs), or **Unverified** (plausible but no independent source found). |

Separate required behavior, compatibility, security, and performance needs from optional optimization and hypothetical flexibility. A requirement whose only source is the change itself is Chosen or Unverified, not Required.

Trace the existing path beyond the diff: producers, carriers, stores, consumers, and owners of each affected fact, including production callers and consumers in related changes when the PR modifies foundation code. Read the existing mechanisms the change could have reused. Name what already supplies part of the need.

For each added or expanded mechanism, state the demonstrated gap it fills and what existing services or direct composition already supply. A partial gap justifies only the smallest missing operation, not a new layer around it.

## Sketch the smallest viable design

Before assessing the submitted design in detail, write down, from the ledger alone, the smallest implementation you would expect: which existing operations, factories, closures, adapters, projections, or stores it would compose, and where each Required behavior would execute. Keep it to the changed flow; do not redesign unrelated code.

Then map the submitted design onto the sketch. Every place the submission adds a mechanism, layer, decision point, parameter, interface, or unit of state that the sketch does not need is a design choice. For each such choice, ask what Required behavior it serves that the sketch cannot, and record the answer. If the answer relies only on Chosen or Unverified items, say so.

If the sketch and the submission coincide, or the sketch fails a Required item you cannot satisfy otherwise, record that and move on. The sketch exists to prevent anchoring, not to manufacture alternatives.

## Ownership and authority across the flow

For every fact the change reads, derives, or decides, identify its owner: the component that establishes it and guarantees it. Then check, across the whole changed flow rather than file by file:

- **Duplicated authority:** the same fact decided, validated, or cached in more than one place, or a derived value recomputed where the owner already provides it.
- **Misplaced responsibility:** feature-specific logic in shared, generic, or control code; mechanical wiring where a projection or existing subscription would do; decisions made by a component that must infer state the owner already knows.
- **Unnecessary mechanism:** a service, layer, event, cache, retry, or coordination step whose gap is not demonstrated or is already covered.

Name locations and the smaller viable placement. Preserve independent admission, trust, security, wire and durable validation, and atomicity decisions; these are legitimate separate authorities even when they look like rechecks.

## Shared interface changes

Judge the need for a feature separately from the need to expand its shared interfaces. A change spanning packages or abstraction layers, or extending a generic execution protocol, is a material design choice even when it adds only optional fields or forwarding lines.

Trace each added operation, parameter, option, or result field from its producer to its actual consumers. Name the affected files and symbols, including exported types, forwarding wrappers, generators, and tests. Distinguish layers that interpret the value from layers that only pass it through, and explain why each shared interface must expose it. Tests exercising an option do not establish production demand. A single consumer prompts scrutiny but does not by itself make an interface unjustified.

Compare the expansion with binding the required data or behavior at the owning caller or assembly point through existing factories, closures, or adapters. Trace the alternative through the same consumers and confirm it preserves per-operation isolation, validation, lifetime, and any process, wire, or persistence requirement. Include added assembly work and runtime cost; moving parameters or leaving a shared package unchanged is not itself an improvement. Preserve independent choices actual callers need; a long type expression or naming alias alone establishes neither redundancy nor simplification.

Report separate conclusions for required capability and shared-interface necessity, using the material-choice verdicts below. Cite the affected interface chain and explain why the narrower alternative works or fails. If callers or feasibility remain unverified, report that uncertainty rather than accepting the expansion because the feature needs its data or because checks pass.

## Frontend flow and control changes

Apply this section to frontend changes and to any added remote event, including cases with no defects.

For frontend changes, make data flow the primary design review. Trace and cite each affected value's actual path from backend producers through projections, queries or streams, client stores, selectors or hooks, and rendering. Read existing paths outside the diff; omit stages that do not exist. Identify the owner of each fact and each derived value. Check initial load, live updates, reconnect, and history separately.

Compare existing projection stores and selectors against feature queries, remote events, duplicate caches, client-side reconstruction, and reload coordination. For a missing read, compare a minimal projection operation with a feature-service API. Verify fields, payload cost, sequencing, freshness, and lifecycle. An initial-read gap alone does not justify a second update channel.

Inventory every frontend-driven change to `session-control` or equivalent controllers, control streams, and client adapters, plus every added remote event. Report each affected file and symbol even when justified. Group symbols sharing one path only if their individual roles stay visible. Distinguish a new event from changes to an existing event's payload or emission, and scrutinize those changes through the affected control path.

For each inventory item, report:

| Concern | Required explanation |
| --- | --- |
| Flow | Backend producer, data or payload, carrier, and frontend consumer; name another consumer when no frontend applies. |
| Responsibility | Mechanical wiring, reusable projection infrastructure, or feature-specific behavior in shared control code. |
| Necessity | Demonstrated gap, existing read/projection/subscription alternatives, and added state, interfaces, or coordination. |
| Continuity | Baseline/update ordering, reconnect, disposal, and duplicate or stale delivery; mark inapplicable cases with a reason. |
| Verdict | Necessary with evidence, simplifiable with a concrete alternative, or unproven with the missing evidence named. |

These triggers do not ban backend changes or events. Explain justified changes even when there are no findings. Report duplication or misplaced ownership with locations and a smaller viable path. State explicitly when the inventory is empty.

## Decisions and parameters

Complete both inventories across the changed flow, including fixes made during review:

| Inventory | Evidence and classification |
| --- | --- |
| Decisions added or newly relied on: guards, branches, scans, validation, fallback probes, rollback, post-success inspection, and duplicated derived state | Identify the fact, its owner, the owner's guarantee, and the independent failure the decision prevents. Classify each as required, redundant, move-to-owner, or unproven. |
| Added or expanded parameters and options | Trace actual production arguments and the caller's genuinely independent choice. Classify each as required, removable, narrowable, or unproven. Check whether existing inputs or the owning operation already determine it. |

Presume redundant: rechecks of type, provider, data-structure, API, or lifecycle guarantees; raw scans that bypass the owner; inference of private state after a successful operation; checks that turn a documented successful fallback into failure; and handling of impossible internal states introduced solely to justify a check. Require a reachable failure before accepting a guard; do not invent one. Preserve independent admission, trust, security, wire and durable validation, and atomicity decisions.

Inspect every new or expanded public API and service for a smaller or private interface, and apply the shared-interface review wherever it crosses packages or abstraction layers.

## Compare material choices

For each material design choice identified above, compare the submitted design with the smallest viable reuse of existing mechanisms from your sketch. Add other credible alternatives only when they expose a meaningful choice. For each option, mark required behavior, correctness, security, compatibility, and required performance as satisfied, violated, or unproven. Reject violated options; make recommendations conditional where evidence is missing.

Before recommending removal or replacement, trace where every Required ledger item executes in the alternative, including initialization, updates, recovery, consumer changes, and testing obligations. A promise to preserve behavior or to add tests is not a trace. State the extra runtime or transport work reuse introduces, any capability, optimization, isolation, or future flexibility sacrificed, and whether any current consumer needs what is sacrificed.

Tie each claimed benefit of the submitted design to current callers and execution paths using measurements, code-established guarantees, or explicitly labeled hypotheses. State whether the benefits justify the added ownership and complexity, name concrete deletions or narrower signatures, and name the evidence that would change the choice. Consider separating optional shared-service optimization when it expands feature scope. Do not assign numerical scores, grades, or cost/benefit ratios.

## Change accounting

Report actual implementation statistics for the submitted change. Compare complete material alternatives against the same named baseline, including consumers and tests; never compare total PR size with the incremental edits of switching to an alternative.

Use actual diffs for implemented options. For unimplemented options, estimate ranges from named files, symbols, and steps; state high, medium, or low confidence and the main assumption. Unknown is not zero. Do not implement alternatives merely to count them.

### Counting rules

| Measure | Rule |
| --- | --- |
| LOC | Additions A, deletions D, changed lines A+D, signed net A−D. Separate production, tests/docs, generated output, and mechanical churn without double counting. Identify removals and moves; a deletion-heavy diff need not add machinery. Estimate additions and deletions consistently for alternative totals and net growth. |
| API | Count new caller-visible operations and independently usable parameters, options, and response fields, including expansions of existing ones. Name new or expanded owners and public versus internal exposure. Count a service method once, not again for its service; count independent options separately. List removals separately. |
| Types | Count new named interfaces, aliases, enums, and classes used as types. Distinguish domain concepts from naming-only aliases; list removals. Moves and re-exports are not new declarations. A declaration may appear in both API and type counts because the measures differ. |
| Files | Count distinct added, modified, deleted, and renamed files, a rename once. Separate the LOC categories, name affected packages, and include required follow-through for alternatives. |

Keep categories consistent across options. Alongside counts, describe services, state, interacting branches, lifecycle ownership, dependencies, and maintenance and testing obligations.

### Presenting a material choice

Use compact tables or parallel prose: option, required-behavior status, counts by category, extra benefit and its evidence, sacrifices, estimate confidence, and verdict. Separate the common required capability from optional benefit. Missing measurements do not prove zero benefit.

Verdicts: **recommended**, **viable tradeoff**, **defer pending evidence**, or **not viable**. For rejected alternatives, name the requirement lost. For deferred ones, name the evidence or workload threshold that would decide.

Before delivering comparisons, confirm every unimplemented option has LOC ranges rather than point estimates and every option reports file counts by production and supporting category. Naming affected files alone does not supply category counts. Correct omissions before sending.

## Anchoring check before delivery

Reread your draft against these questions and revise where the answer is unfavorable:

1. Is any item marked Required whose only source is the PR's own code, tests, description, or design notes?
2. Did any conclusion accept the submitted design because it works, checks pass, or the feature "needs its data," rather than because the alternative was traced and found lacking?
3. For each local defect or optimization reported, did you also state the flow-level ownership or necessity conclusion for that mechanism?
4. Did you declare any component unused because you found no callers, without saying where you looked and how confident you are?
5. Did you declare any current behavior mandatory without a consumer or contract, when it should be Unverified?
6. Does every recommended alternative trace where each Required behavior executes?

## Deliver the review

Give the complete review in the final response, not as an attachment or link. Use source links or file:line references as evidence. Organize it as:

- **Findings:** ordered by consequence, each with affected locations, evidence, impact, and a viable fix. Simplification findings must supply a smaller implementation that preserves Required behavior, with the trace. Parameter findings must show actual caller needs and the smaller signature. Report all redundant and move-to-owner decisions and all removable or narrowable parameters; group related findings without hiding locations.
- **Design assessment:** change statistics; the requirement ledger summary (Required, Chosen, Unverified counts with the notable items); the sketch and where the submission departs from it; material alternatives with verdicts; and every applicable frontend, control, and event callout. Distinguish actionable defects from justified tradeoffs. For trivial changes or when no credible alternative exists, say why no comparison is needed but still report statistics.
- **Coverage and uncertainty:** grouped conclusions on necessity, API growth, overdesign, services, decision authority, and parameters, without repeating six separate answers. Name missing evidence and confidence for Unverified and unproven items, tests actually run, callers and authorities checked, and material scope limits.

A no-finding review must still include statistics, applicable callouts and material tradeoffs, the ledger summary, and a statement of whether both inventories were fully classified, naming the authorities and callers checked. If coverage is incomplete, disclose the gap rather than claiming completion. Stop after delivering the review; applying fixes or publishing the review requires authorization from the surrounding task.
