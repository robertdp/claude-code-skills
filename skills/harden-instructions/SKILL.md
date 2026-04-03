---
name: harden-instructions
description: Review and harden instruction files (CLAUDE.md, rules, specs) for clarity and compliance. Use when user says "harden instructions", "harden my rules", "tighten CLAUDE.md", "make rules stricter", "review my instructions", or invokes /harden-instructions or /robert:harden-instructions.
argument-hint: "[file-path]"
allowed-tools: Read, Write, Edit, Grep, Glob
---

<objective>
Analyse an instruction file against research-backed compliance principles and rewrite it for maximum clarity and likelihood of being followed. Works on any structured instruction document — CLAUDE.md, project rules, agent specs, skill prompts.
</objective>

<workflow>
<step name="locate">
<arguments>$ARGUMENTS</arguments>

If arguments contains a file path, use it directly. Otherwise, search for instruction files:
- `~/.claude/CLAUDE.md` (global)
- `.claude/CLAUDE.md` or `./CLAUDE.md` (project)

If multiple files found, use AskUserQuestion to ask which one. Note to the user that only the selected file will be audited — cross-file interactions between global and project CLAUDE.md files are out of scope for this run.
</step>

<step name="budget">
Before auditing, establish the instruction budget:

1. Count discrete instructions in the file (each bullet, imperative sentence, or standalone rule = 1 instruction).
2. Measure total line count.
3. Record both numbers — they constrain every subsequent recommendation.

**Thresholds:**
- Under 80 lines / ~40 instructions: healthy range. Audit normally.
- 80–150 lines / 40–80 instructions: caution zone. Every addition must justify its cost against the budget. Prioritise deletions and consolidation over additions.
- Over 150 lines / 80+ instructions: the file is likely past the point where individual rule quality matters. Length reduction is the highest-priority finding regardless of other checklist results. Research shows compliance degrades non-linearly with instruction density — a shorter file with imperfect rules outperforms a longer file with perfect ones.
</step>

<step name="audit">
Read the file and evaluate against each checklist item below. For each issue found, record:
- The checklist item violated
- The specific line(s) affected
- A concrete fix
- The estimated net change in instruction count / line count (positive = addition, negative = deletion)

**The checklist, in priority order:**

**1. Positioning**
LLMs have a U-shaped attention curve — instructions at the beginning and end of a document receive the most attention; the middle is where rules get lost.

- Are the highest-impact rules near the top? Context, background, and "about me" sections must not precede behavioural rules.
- In files over ~80 lines: are any high-priority rules buried in the middle third (roughly lines 30–70% of the document)? If so, flag them for repositioning to the top or end.
- For short files (under ~50 instructions): positioning effects are modest. Clarity of phrasing matters more than ordering — do not recommend restructuring a well-functioning short file solely for positioning.

**2. Atomicity**
Flag instructions that are compound within a sentence OR buried in prose paragraphs. Multiple instructions in one sentence — the later ones get dropped. Rules buried in paragraphs get lost.

Each instruction must be a standalone, independently actionable bullet — not compound within a sentence and not buried in prose.

**Cost gate:** Before recommending a split, assess whether both sub-rules are independently worth tracking at the cost of an additional instruction slot. If the lower-priority sub-rule is marginal, recommend deletion over splitting. Each split adds to total instruction count, which degrades compliance for every other rule in the document.

Not: "Prefer deletion over addition, simpler over clever. When choosing between approaches, pick the one that minimises long-term maintenance cost."
Instead: Three separate bullets — but only if all three are independently high-priority. Otherwise, keep the most important and delete the rest.

**3. Soft language**
Flag soft language only when a rule is **intended to be mandatory but is phrased as optional**:
- "prefer", "try to", "when possible", "ideally", "consider" — treated as optional by the model
- "should" — weaker than "must" or imperative mood

Replace with imperative ("Do X") or hard constraints ("Never Y", "Always Z").

Do not flag intentionally flexible guidance that correctly uses soft language. The distinction is intent: a rule meant to be mandatory but phrased as "try to X" is a genuine problem. A rule meant to be guidance phrased as "consider X" is working as designed.

Note: imperative language improves compliance most in short, lean files. In files over ~100 lines, length reduction delivers larger compliance gains than language hardening.

**4. Missing examples**
Flag missing examples only when a rule is genuinely ambiguous — when a competent reader could reasonably interpret it two or more ways.

Do not use a mechanical threshold (e.g., "3+ rules without an example"). Instead, assess whether the rule's intent is clear from the text alone. Attempt to rewrite the rule for self-evident clarity first. Recommend an example only when rewriting fails and the section is short enough that the example won't push critical rules into the attention-degraded middle of the document.

Each before/after example adds 6–15 lines. Account for this against the instruction budget.

**5. Conflict and priority**
Can any two rules contradict? Silent conflicts cause arbitrary behaviour — Claude picks one without telling the user.

**Primary fix:** Eliminate the conflict by tightening the scope of one rule. Structural scoping ("this rule applies only to conversation" / "this rule applies only to code output") removes the conflict rather than managing it.

**Fallback:** If scope tightening is not possible, add explicit priority notation ("When X and Y conflict, X takes precedence"). Note: priority notation is better than silence but research shows it is inconsistently enforced by the model. Scope tightening is the stronger fix.

Common conflicts:
- "Read broadly" vs "don't volunteer things not asked for"
- "Act directly" vs "confirm before writing code"
- Style rules vs artefact conventions

**6. Specificity**
Flag vague rules that rely on subjective judgement without anchoring. "Keep functions short" → how short? "Write clean code" → by what standard? Measurable beats subjective.

**7. Scope ambiguity**
Is it clear what each rule applies to? (conversation, code, commits, external communication, all). Rules without scope markers get applied inconsistently.

**8. Empty or stub sections**
Sections with no content or placeholder text dilute the document. Remove or populate.

**9. Negative examples**
For rules that are frequently violated, a "Not: X / Instead: Y" example prevents the most common failure mode. Flag high-stakes rules that lack one. Account for the line-count cost against the instruction budget.

**10. Hook candidates**
For any rule phrased as "always" or "never" that involves a file operation or shell command (e.g., "never write to the migrations folder", "always run tests before committing"), flag it as a hook candidate.

No amount of instruction hardening matches the reliability of hooks. Hooks are deterministic — they guarantee the action happens every time. Instruction rules are probabilistic — they increase the likelihood but never reach 100%. Recommend the user implement these as hooks in `.claude/settings.json` and remove or downgrade the corresponding rule in the instruction file.
</step>

<step name="filter">
Review every finding from the audit. For each one, ask: **would Claude actually misinterpret, partially comply with, or silently drop this rule as currently written?**

**Keep the finding** if:
- The current phrasing causes a real compliance risk — Claude is likely to miss, misread, or inconsistently apply the rule.
- Two rules genuinely conflict with no resolution, and Claude must pick one arbitrarily.
- An instruction is buried or compounded in a way that the later part gets dropped in practice.

**Discard the finding** if:
- The rule works well enough in practice despite technically violating a checklist item.
- The fix is a structural or stylistic preference, not a compliance improvement.
- The finding adds precision to something that is intentionally flexible.
- The current phrasing is clear to a competent reader and the rewrite would not change behaviour.

Err on the side of discarding. A short list of high-value findings is better than an exhaustive list that buries the signal.

**Aggregate check:** After assembling surviving findings, estimate the net change in instruction count and line count. If proposed changes would increase the file by more than ~15%, go back and pair each addition with a deletion candidate of equal or greater size. A batch of individually-correct findings can collectively degrade compliance by increasing total instruction count. Each addition must justify its cost against the budget established in the budget step.
</step>

<step name="report">
Present only the findings that survived filtering.

- **High** — Rules that are likely to be ignored or misinterpreted.
- **Medium** — Rules with a meaningful ambiguity that affects compliance.

Do not include a Low category. If a finding isn't worth acting on, discard it in the filter step.

Include the total count of issues, the estimated net line-count change, and a 1-sentence overall assessment.

Add this disclaimer: "These findings are predictions based on documented LLM failure patterns, not empirical measurements. A/B test the revised file against specific tasks to verify actual compliance improvement."

If all findings were filtered out, say so: the file is solid and doesn't need changes.
</step>

<step name="rewrite">
Use AskUserQuestion: "Want me to draft a revised version?"

If yes:
- Apply only the fixes that survived filtering
- Preserve the original intent and voice
- Do not add new rules the user didn't have
- Do not remove rules without flagging them for user review. If rule count exceeds the recommended threshold, identify the lowest-priority rules and surface them as deletion candidates with rationale — do not delete silently
- Write the revision to a `.draft` file next to the original (e.g., `CLAUDE.md.draft`)
- Summarise the structural changes made, including net instruction count and line count change

If no, stop after the report.
</step>
</workflow>

<edge_cases>
- **File is already strong**: Say so. Do not invent problems or pad the report with low-impact findings.
- **File is very short** (under 10 lines): Some checklist items won't apply. Skip them rather than forcing findings.
- **File is not an instruction document**: If the content doesn't contain rules or instructions, say so and stop.
- **Mixed content** (instructions + reference material): Only audit the instruction portions. Note that reference material is outside scope.
- **File exceeds 150 lines**: Length reduction is the primary recommendation. Individual rule improvements are secondary to getting the file shorter.
</edge_cases>

<success_criteria>
- Every finding references specific line(s) in the original file
- Every finding includes a concrete fix, not just a diagnosis
- Every finding includes an estimated net change in instruction count
- The rewritten version preserves original user intent — no silent additions or deletions
- The rewritten version does not exceed the original instruction count unless each addition is explicitly justified
- The rewritten version is measurably sharper: fewer compound rules, less ambiguous soft language, conflicts resolved by scope tightening
</success_criteria>
