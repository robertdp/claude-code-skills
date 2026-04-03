---
name: improve
description: Interactive guided critique with multi-frame adversarial review, issue-by-issue walkthrough, solution tradeoffs, and fix implementation. Use when user says "improve", "guided critique", "walk me through issues", "critique and fix", or invokes /improve or /robert:improve.
argument-hint: "<topic or goal to improve>"
allowed-tools: Read, Write, Edit, Grep, Glob, Agent, WebFetch, WebSearch
---

<objective>
Multi-frame adversarial critique with guided issue resolution. Investigates a target, runs a single adversarial reviewer through multiple frames, then walks through each issue one at a time — explaining it, proposing solutions with tradeoffs, and implementing approved fixes. The user retains control at every step.
</objective>

<quick_start>
1. Clarify the target and goal (brief interview if needed)
2. Investigate — gather evidence, determine target scope
3. Run multi-frame adversarial review (single reviewer, all frames)
4. Walk through issues one by one — explain, propose solutions, get decision
5. Implement approved fixes (defer interconnected issues until all decided)
6. Summarise
</quick_start>

<workflow>
<step name="clarify">
**User input:**

<user-input>
$ARGUMENTS
</user-input>

Establish the target and goal. If the target is clear from the input, state it and proceed. If ambiguous, use AskUserQuestion to clarify:

- What specifically is being critiqued? (code, plan, architecture, document, idea)
- What is the goal or purpose of the target?
- Any specific focus areas or concerns?

Keep this brief — 1-2 questions max. Don't interview when the input is already specific enough.

State the goal explicitly in one sentence before proceeding. If the user corrects the stated goal, re-anchor to the corrected goal. If the correction happens mid-walkthrough, keep decisions already made, re-evaluate remaining findings against the new goal — drop any that are no longer relevant and continue with those that still apply.
</step>

<step name="investigate">
Gather evidence and determine target scope:

- Read all file-based targets (via @-references, paths, or glob patterns).
- For multi-file or codebase-scale targets, use Agent with subagent_type=Explore to map the relevant code — trace call paths, find related modules, check assumptions, identify architecture and conventions.
- For ideas, plans, or documents, identify unstated assumptions and check for internal contradictions.
- For verbal ideas or proposals, clarify the assumptions behind them and identify the constraints in play.
- Verify claims — don't critique by assertion. If a claim is verifiable against source code, read the code. If verifiable against documentation or known behaviour, use WebSearch or WebFetch.
</step>

<step name="review">
Read [reference/frames.md](reference/frames.md) and select frames based on target type. The Pessimist and Simplifier are strong defaults — keep both unless the target type makes one irrelevant. For code targets, also include The Engineer.

Notify the user of the selected frames before proceeding (e.g., "Running 4-frame review: Pessimist, Simplifier, Engineer, Challenger."). This gives them a chance to adjust scope or abort.

Always launch one Agent subagent — even when the issue seems obvious or the target is narrowly scoped. The subagent applies structured adversarial frames that surface findings the orchestrator's intuition misses. Do not substitute your own analysis for the reviewer. Do NOT set `run_in_background: true` — the returned findings are needed for the walkthrough.

**Model selection** — choose based on the target's risk profile:

Use **Opus** when any of these apply:
- The Adversary frame is selected (security-sensitive target)
- The target affects correctness-critical or high-consequence areas (payments, data integrity, PII)
- The target introduces architectural patterns or novel approaches
- 5+ frames selected
- Large targets (10+ files, or documents with significant cross-referencing)
- Absence detection is the primary value (missing validation, unhandled states, implicit assumptions)

Use **Sonnet** when:
- Small/medium targets (< 10 files, < 4 frames)
- Standard feature work following established patterns
- The review is focused on a narrow, well-understood concern

When in doubt, use Opus — the review is the quality gate.

```
subagent_type: general-purpose
model: [selected per criteria above]
```

Construct the reviewer prompt by filling in the template below. Copy frame focus descriptions verbatim from frames.md. For code targets, list file paths for the reviewer to read. For non-code targets, include the full target content directly.

```
You are reviewing [target description] through multiple adversarial frames.

**Goal:** [stated goal from clarify step]

**Frames to apply:**

[For each selected frame, copy verbatim from frames.md:]
- **[Frame Name]:** [Frame focus description]

**Target files to read:**
[File paths — for code targets]

**Target content:**
[Full text — for non-code targets]

**Investigation context:**
[Key findings from investigate step — dependencies, architecture, constraints, conventions]

**Instructions:**

1. For code targets: read all listed target files. For non-code targets:
   analyse the provided target content.
2. Review the target through each frame's lens. For each frame, actively
   look for issues within that frame's focus area. Take each frame seriously
   — don't rush through frames to get to the next one.
3. Actively look for what's missing — omissions, unhandled cases, unstated
   assumptions, absent validation, missing error paths. Problems of absence
   are the hardest to spot and the most valuable to surface.
4. Challenge mitigations: when the target identifies a risk and states a
   mitigation, treat the mitigation as a claim that requires verification —
   not as a resolved concern. Ask: does the mitigation actually work as
   described? Does the framework, runtime, or external system behave the
   way the mitigation assumes? A stated mitigation that doesn't work is
   worse than an unmitigated risk because it creates false confidence.
5. Use WebSearch and WebFetch to verify claims, check documented behaviour,
   find known issues, or research prior art. Don't guess when you can
   look it up.
6. Reference specifics: each finding must describe exactly one issue. If
   multiple locations are cited, each must independently exhibit the stated
   problem. Do not group items under a finding based on proximity, naming
   similarity, or thematic relatedness — group only when they share the
   same root cause and the same fix applies to all.
7. As you review across frames, cross-reference findings naturally. Note
   when multiple frames converge on the same issue — this is higher signal.
   Track interconnections between findings (where fixing one affects another
   or issues share a root cause).
8. After reviewing through all frames, produce a single synthesised list of
   findings. Deduplicate where multiple frames surface the same underlying
   issue — note which frames flagged each. Only merge when the same fix
   resolves both findings. If two findings look similar but require
   different fixes or affect different paths, keep them separate.

**Fact-checking:** For code targets, use Read on the specific files/lines
you reference. Verify each finding against actual code. For findings that
cite multiple locations, verify each location independently — if one cited
location does not exhibit the stated problem, split the finding or remove
the incorrect citation. For non-code targets, verify each finding against
the target material — check that quoted passages exist, citations are
accurate, and logical claims follow from the content. Use WebSearch or
WebFetch to verify claims about external libraries, APIs, documented
behaviour, or prior art. Drop findings based on incorrect evidence — note
why they were dropped.

**Cost/benefit:** For each surviving finding:
- What's the cost of fixing it? (effort, complexity introduced, risk of regression)
- What's the cost of NOT fixing it? (likelihood x impact of the problem)
- Is fixing this worth the effort?

**Ranking:** Order findings by:
- Cost/benefit ratio (highest net value first)
- Severity (blocking > warning > note)
- Frame agreement (flagged by 3 frames > 1 frame)
- Impact on stated goal

**Output format:** Start with a 2-3 sentence prose assessment of the
target's overall state against the goal. Then a numbered list of findings.
Always use a list, never a table. Each item:

**#N: Title** (severity) — Frames: [list]
Evidence: All file:line references, quotes, and citations
Issue: Full explanation of what's wrong and why it matters
Cost/benefit: Cost of fixing vs. cost of not fixing
Recommendations: Concrete recommendations — these inform solution options during the walkthrough
Interconnected with: #M, #K (if applicable)

**Relevance gate:** Before including a finding, apply this test: does fixing
this change the target's behaviour, reliability, or fitness for purpose? If
no, cut it. Drop stylistic preferences, theoretical concerns, organisational
nitpicks, and "could be slightly better" suggestions. When tests are passing,
don't re-verify facts the test suite already proves — focus on what isn't
already verified. Fewer high-impact findings beat many mixed ones — there
is no minimum count. If you're unsure whether a finding is real, verify it
against the code or external sources before including it — do not include
unverified suspicions.

**Allegiance check:** Your job is to report what you actually find, not to
justify every frame. If a frame finds no real issues, report nothing for
that frame — do not manufacture findings to fill the gap. If evidence is
ambiguous, say so rather than claiming it as a definitive finding.

**Tone:** Frame issues as direct, constructive observations. State what's
wrong and why it matters factually. Preserve all evidence and specificity.

Return the full findings as your final message.

Be specific. Cite evidence. No hedging.
```

**When The Historian is among the selected frames**, append this to the prompt:

```
**Historian note:** For The Historian frame, web research is a primary
activity, not just verification. Search for real precedents — post-mortems,
case studies, documented failure patterns. Do not invent plausible-sounding
prior art. If you cannot find genuine precedents, say so explicitly rather
than fabricating examples.
```

If the reviewer returns an error or empty findings on a non-trivial target, re-run up to 3 times. If all attempts fail, report the failure and ask the user whether to retry with fewer frames, a smaller scope, or a different model.

After the review completes, state the goal as a labelled sentence (e.g., `**Goal:** This API validates and routes incoming webhooks.`). Present the findings list to the user before starting the walkthrough. Flag interconnected groups.
</step>

<step name="walkthrough">
Go through issues one at a time. For each issue:

1. **Explain** — What's wrong and why it matters. Cite evidence: file:line references, quotes, concrete examples. Use the reviewer recommendations to inform your explanation and solution options.
2. **Clarify** (conditional) — If the issue is ambiguous, has multiple valid interpretations, or the right solution depends on context the reviewer didn't have (user intent, business constraints, future plans):
   - Use AskUserQuestion to clarify scope, intent, or constraints (1 round, 1-2 questions)
   - Investigate based on answers if needed (read code, check dependencies, search)
   - Skip this step when the issue and its solutions are clear-cut
3. **Solutions** — 2-4 concrete options. Always use a list, never a table. For each option:
   - **Option name**: one-line summary of the approach
   - **Tradeoffs**: pros and cons in a single bullet list (prefix + or -)
   - **Best when**: one sentence on when this is the right choice
4. **Decision** — Use AskUserQuestion to let the user choose. Always include these disposition options alongside the solution options:
   - **Skip** — Acknowledge but don't act on this issue now
   - **Defer** — Flag for later / add to backlog
   The user can also propose their own approach via "Other".

**Pacing:** Present one issue at a time. Wait for the user's decision before moving to the next. Exception: interconnected issues — present and explain the full group, explain how they interact, then collect decisions as a batch before implementing. If the user wants to skip ahead or stop early, respect that.
</step>

<step name="implement">
After each decision (or batch of interconnected decisions), implement the approved fix:

- Make the change
- Show what was changed (file:line references)
- Verify the fix doesn't break related code

If the user chose skip or defer, move on without changes.

If implementation reveals a new issue or a fix doesn't work as expected, flag it immediately and add it to the remaining issues. If there are no remaining issues, re-enter the walkthrough for the discovered issue before proceeding to the summary.

For non-code targets (ideas, plans, documents): "implementation" means writing up the improved version or updating the document as appropriate.
</step>

<step name="complete">
After all issues are addressed (including any discovered during implementation), provide a summary grouped by disposition. Omit empty groups.

**Fixed:**
- #N Title — what was changed

**Deferred:**
- #N Title — why

**Skipped:**
- #N Title — why

**Discovered:**
- #N Title — new issue found during implementation

</step>
</workflow>

<edge_cases>
- **No issues found**: State that the target looks sound against the stated goal. Skip the walkthrough.
- **Target not found**: If file paths are invalid or target content is empty/inaccessible, report the specific path or target that couldn't be read. Ask the user for correction before proceeding.
- **Non-code targets**: Reviewer receives target content directly instead of file paths. Solutions are recommendations, not file edits. "Implementation" means writing up the improved version.
- **User stops early**: Provide partial summary of what was decided so far and flag remaining issues.
- **Fix breaks something**: Flag immediately, revert if needed, add the breakage as a new issue in the walkthrough.
</edge_cases>

<style>
- One sentence beats three. Bullets beat paragraphs.
- No hedging ("might", "could", "seems like"). State problems as facts.
- Quantify impact where possible — numbers over adjectives.
- No rhetorical questions. Declarative statements only.
- Be direct in critique, constructive in framing. State problems as observations with evidence, not attacks.
- Cite lines, quote text, give concrete examples.
</style>

<success_criteria>
- Target and goal clearly established before critiquing
- Multiple adversarial frames applied by reviewer (minimum 3 for code targets)
- Findings deduplicated, fact-checked, cost/benefit assessed, and ranked
- Each issue explained with concrete solution options and tradeoffs
- User offered skip/defer/implement for every issue
- Interconnected issues grouped and decided together
- Approved fixes implemented correctly
- User retains control at every step
- Summary provided after all issues addressed
</success_criteria>
