<required_reading>
No pre-reading — this workflow reads and updates the reference files themselves.
</required_reading>

<process>

## Self-Update: Research and Refresh Reference Material

<step name="research">
Search for the latest documentation on authoring-relevant topics:

1. **Skills**: WebSearch "Claude Code skills SKILL.md documentation" — new frontmatter fields, body structure, description conventions
2. **Subagents**: WebSearch "Claude Code custom subagents .claude/agents documentation" — new frontmatter fields, capabilities, delegation patterns
3. **Agent teams**: WebSearch "Claude Code agent teams documentation" — architecture, quality gates, communication, plan approval
4. **Plugins**: WebSearch "Claude Code plugins documentation" — manifest schema, component specs, hook/MCP/LSP configuration
5. **Prompting**: WebSearch "Claude Code prompt engineering best practices" — new guidance from Anthropic on writing skill and agent instructions
6. **Self-update coverage**: WebSearch "Claude Code extensibility features documentation" — identify new extension types, authoring concepts, or configuration areas not covered by items 1–5 above

For each area, WebFetch official documentation pages for current details.

**Scope filter**: Only capture information relevant to authoring skills, agents, plugins, and orchestration. Exclude:
- CLI commands and flags (how users run things)
- Installation and setup procedures
- Environment variables for runtime configuration
- Debugging and troubleshooting guides
- Marketplace distribution and publishing
- Keyboard shortcuts and UI features
- Enterprise administration
</step>

<step name="diff">
For each reference file, read the current content and compare against research findings:

Read these files:
- [reference/skill-format.md](../reference/skill-format.md)
- [reference/subagent-format.md](../reference/subagent-format.md)
- [reference/teams-reference.md](../reference/teams-reference.md)
- [reference/plugin-format.md](../reference/plugin-format.md)
- [reference/decision-frameworks.md](../reference/decision-frameworks.md)
- [reference/orchestration-patterns.md](../reference/orchestration-patterns.md)
- [reference/enforcement.md](../reference/enforcement.md)
- [reference/prompting.md](../reference/prompting.md)

For each file, identify:
- **Stale authoring information**: Fields, APIs, or patterns that have changed
- **Missing authoring features**: New fields, options, or configuration that bootstrap should know about
- **Deprecated authoring approaches**: Things that no longer work or are discouraged
- **Incorrect details**: Errors in the current reference

**Quality check** — for each proposed addition, ask: "Does bootstrap need this to generate better skills/agents/plugins?" If the answer is no (it's operational, runtime, or user-facing), exclude it.

Also check the SKILL.md router and workflow files for consistency with the reference material. In particular, check whether the research step in this workflow (self-update.md) covers all current extension types and authoring areas — add or remove search topics as needed.
</step>

<step name="present">
Present the findings to the user in a structured format:

### Changes Found

For each reference file, list:
- **File**: path
- **Updates needed**: specific changes with before/after
- **New additions**: authoring features to add
- **Removals**: deprecated or out-of-scope content to remove
- **No changes**: explicitly state if file is current

### Recommended Actions

Prioritised list of changes, grouped by:
1. **Incorrect/broken** — must fix (wrong field, deprecated API, invalid example)
2. **Missing authoring features** — should add (new field, new option, new pattern)
3. **Stale guidance** — update (outdated best practice)
4. **Scope cleanup** — remove operational content that doesn't serve authoring

**Wait for user approval before making any changes.**
</step>

<step name="update">
After user approval, edit the reference files:

1. Remove deprecated or out-of-scope content
2. Update changed fields, APIs, and patterns
3. Add new authoring features and capabilities
4. Update examples if APIs changed
5. Maintain consistent formatting and style across files
6. Check for content duplication — keep the authoritative version, replace duplicates with cross-references

Use the Edit tool for targeted changes. Only rewrite a file completely (Write) if the majority of content needs changing.
</step>

<step name="verify">
After updates:

1. Re-read each modified file to confirm changes are correct
2. Check for contradictions between reference files
3. Check for content duplication across files — each concept should have one authoritative location
4. Verify that workflow files are still consistent with updated references
5. Confirm SKILL.md router is still accurate

Present a summary of all changes made.
</step>

</process>

<success_criteria>
- All reference files checked against current documentation
- Only authoring-relevant content added (no operational, runtime, or user-facing content)
- Stale information identified and presented to user
- Changes made only with user approval
- No contradictions or unnecessary duplication between reference files after update
- Workflow files still consistent with updated references
</success_criteria>
