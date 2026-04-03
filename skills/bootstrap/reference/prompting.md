# Prompt Engineering for Agent Instructions

Techniques for writing effective skill and agent prompts. Condensed from Anthropic guidance and proven patterns.

## Be Explicit

Specific instructions outperform vague guidance. Claude follows instructions precisely — say what you mean.

```
# Vague
Create a dashboard

# Explicit
Create an analytics dashboard with date range filters,
sortable data tables, and export to CSV.
```

Explain *why* a behaviour matters. Claude generalises from motivation better than from bare rules.

```
# Rule without reason
Never use ellipses

# Rule with reason
Your response will be read aloud by text-to-speech,
so never use ellipses since TTS engines can't pronounce them.
```

## Right Altitude

Find the balance between over-specified (brittle) and under-specified (improvisation).

```
# Too specific (brittle if-else)
If the user asks about files, use Read.
If the user asks about search, use Grep.
If the user asks about structure, use Glob.

# Right altitude (heuristic)
Navigate the codebase using available tools:
- Read files to understand content
- Grep to search for patterns
- Glob to find files by name
Choose tools based on what information you need.
```

**Fragile tasks** need lower altitude (more specific instructions).
**Creative tasks** need higher altitude (principles and heuristics).

## Examples Drive Behaviour

Claude pays close attention to examples and will replicate patterns from them.

- Show exactly the behaviour you want
- Never include examples of undesired behaviour (even as "don't do this")
- One good example is worth several paragraphs of explanation
- Ensure examples demonstrate edge cases, not just the happy path

## XML Structure

Prefer XML tags over markdown headings in skill bodies:
- Clearer semantic boundaries
- Better token efficiency
- Unambiguous section markers

```xml
<objective>Purpose.</objective>
<workflow>
<step name="first">Instructions.</step>
</workflow>
<success_criteria>Verification.</success_criteria>
```

Reserve markdown for reference files and documentation (like this one).

## Positive Instructions

Tell Claude what TO do, not what NOT to do.

```
# Less effective
Do not use markdown in your response

# More effective
Write smoothly flowing prose paragraphs without formatting.
```

Negations work for critical safety constraints. For behaviour shaping, positive framing is more reliable.

## Progressive Disclosure

Keep the main prompt focused on workflow. Detailed docs go in reference files loaded on demand.

```
# Clutters the skill (all inline)
## API Reference
[500 lines of documentation...]

# Clean (progressive disclosure)
For API details, see [api-reference.md](api-reference.md).
```

Rules:
- SKILL.md under 500 lines
- Reference files for detailed documentation
- One level of nesting (SKILL.md → reference file, not reference → reference)
- Claude reads reference files when linked from SKILL.md

## Context Engineering

**Core principle**: Find the clearest set of instructions that produce reliable, predictable behaviour.

- Every line should change behaviour. Background text that doesn't affect output is noise.
- Long contexts can reduce recall accuracy. Keep instructions focused.
- Front-load the most important instructions — primacy bias exists in LLMs.
- Use consistent terminology throughout. Don't mix "API endpoint" / "URL" / "path."

## Model-Aware Design

Different models need different levels of detail:

**Haiku** (needs more guidance):
- Complete, step-by-step instructions
- Full working examples
- Explicit edge case handling
- Lower freedom — more structure

**Sonnet** (balanced):
- Clear structure with XML tags
- Examples for non-obvious cases
- Moderate freedom with guardrails

**Opus** (can infer):
- Concise instructions — can infer obvious steps
- High-level heuristics over step-by-step
- Higher freedom — benefits from principles over procedures

**Balancing for portability** (skill runs on multiple models):
- Include a complete example (Haiku needs it)
- State the key principle (Opus benefits)
- Add an escape hatch for edge cases (Sonnet uses it)

## Triggering Sensitivity

Claude is responsive to emphasis in system prompts. Dial back aggressive language.

```
# Over-triggers (Claude applies too eagerly)
CRITICAL: You MUST ALWAYS use this tool when...

# Appropriate
Use this tool when...
```

Reserve CAPS and emphasis for genuine safety constraints, not preference shaping.

## Depth Over Mechanics

A skill's value comes from the judgment it exercises, not the patterns it matches. Before writing any agent prompt, research what a senior practitioner in the domain would actually do. The agent's scope should match that — not a simplified subset scoped to what's easy to automate.

If an agent's work could be fully replicated by existing tooling, the prompt is too shallow. Push agents to reason about context, relationships, and trade-offs.

## Script Bundling

Complex logic is more reliable in a script than in inline instructions.

```
# Inline (fragile, hard to maintain)
## Validation Rules
1. Check field X exists
2. Verify field Y format
[20 more rules...]

# Script (testable, maintainable)
Run: python scripts/validate.py input.json
```

Scripts are:
- Testable independently
- Deterministic (no LLM interpretation)
- Maintainable (edit the script, not the prompt)
- Token-efficient (one line instead of twenty)

Make explicit whether Claude should execute or read a script.

## Verifiable Intermediate Outputs

For multi-step operations, create checkpoints that can be validated:

1. Generate a plan file (e.g., `changes.json`)
2. Validate with a script or human review
3. Execute the validated plan

This catches errors before they propagate. Cheaper than fixing mistakes after full execution.

