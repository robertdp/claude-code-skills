<required_reading>
Read before starting:
- [reference/plugin-format.md](../reference/plugin-format.md) — Complete plugin spec
- [reference/skill-format.md](../reference/skill-format.md) — Skill format (plugins contain skills)
- [reference/subagent-format.md](../reference/subagent-format.md) — Subagent format (plugins contain agents)
- [reference/enforcement.md](../reference/enforcement.md) — Enforcement mechanisms
</required_reading>

<process>

## Build a Plugin

<step name="interview">
If the interview hasn't been completed, gather requirements now.

Extract via AskUserQuestion (2-4 questions per round):

**Purpose**:
- What does this plugin do? (single sentence)
- What components does it include? (skills, agents, hooks, MCP servers, LSP servers)
- Who is the audience? (personal use, team, community)

**Components**:
- Skills: What slash commands should it expose?
- Agents: What specialised subagents does it need?
- Hooks: What events should trigger automated actions?
- MCP: Does it need external tool integrations?
- LSP: Does it provide language server support?

**Configuration**:
- Does it need default settings? (currently only `agent` key supported)
- Does it need custom scripts? (for hooks, validation)
- Environment variable requirements?

Between rounds: explore the codebase for existing `.claude/` configurations that could be migrated to plugin format.
</step>

<step name="justify">
Confirm a plugin is the right choice over standalone configuration:

**Plugin is right when** (check at least one):
- [ ] Will be shared with team or community
- [ ] Needed across multiple projects
- [ ] Benefits from versioned releases and updates
- [ ] Has multiple components that belong together (skills + hooks + agents)

**Standalone is better when**:
- Personal workflow for one project
- Quick experiment before packaging
- Short skill names preferred over namespaced names

If standalone is sufficient, suggest that approach and confirm with the user. Offer to build standalone first with a migration path to plugin later.
</step>

<step name="design">
Design the plugin structure:

**Manifest**:
- name: kebab-case, unique, becomes namespace prefix
- version: start at 1.0.0
- description: brief purpose statement
- author, homepage, repository, license as needed

**Component placement**:
- Skills → `skills/<name>/SKILL.md`
- Agents → `agents/<name>.md`
- Hooks → `hooks/hooks.json`
- MCP servers → `.mcp.json`
- LSP servers → `.lsp.json`
- Settings → `settings.json` (only `agent` key)
- Scripts → `scripts/`

**For each skill**: Design following the skill workflow (skill.md). Apply enforcement assessment per skill.

**For each agent**: Design following the subagent workflow (subagent.md). Set appropriate tool restrictions and model.

**For hooks**: Identify events, write matchers, plan validation scripts. Use `${CLAUDE_PLUGIN_ROOT}` for all script paths.

</step>

<step name="write">
Create the plugin files:

1. Create directory structure
2. Write `.claude-plugin/plugin.json` manifest
3. Write each skill's `SKILL.md` (follow skill-format.md conventions)
4. Write each agent's `.md` file (follow subagent-format.md conventions)
5. Write `hooks/hooks.json` if hooks are needed
6. Write `.mcp.json` if MCP servers are needed
7. Write `.lsp.json` if LSP servers are needed
8. Write `settings.json` if default settings are needed
9. Write hook/utility scripts in `scripts/`
### Writing principles
- All script paths use `${CLAUDE_PLUGIN_ROOT}`
- All custom paths in manifest are relative, starting with `./`
- Components at plugin root, NOT inside `.claude-plugin/`
- Each skill follows skill-format.md conventions (XML tags, third-person description)
- Each agent follows subagent-format.md conventions (focused system prompt, output format)
- Hook scripts are executable (`chmod +x`)
- Version set in plugin.json
</step>

<step name="critical-review">
Launch a subagent (Agent, subagent_type=general-purpose) to review the produced files.

The reviewer checks:
- **Structure**: Components at root, manifest in `.claude-plugin/`, no misplaced files
- **Namespacing**: Plugin name is valid kebab-case, skills/agents will namespace correctly
- **Path references**: All paths use `${CLAUDE_PLUGIN_ROOT}` where needed, all relative with `./`
- **Component quality**: Each skill and agent meets its respective format standards
- **Hook validity**: Events are correct case, matchers are valid regex, scripts exist
- **Enforcement fit**: Restrictions match assessed risk for each component
- **Portability**: No absolute paths, no references outside plugin directory
- **Version**: Set in plugin.json

Report specific issues with file:line references.
</step>

<step name="review">
Walk through with the user:

1. Present directory structure
2. Present manifest content
3. Walk through each component (skills, agents, hooks, MCP, LSP)
4. Present critical review findings
5. Explain design choices (enforcement, tool restrictions, distribution)
6. Demonstrate how to test: `claude --plugin-dir ./plugin-name`

Iterate on feedback.
</step>

<step name="validate">
Checklist:

- [ ] Manifest has valid `name` (kebab-case)
- [ ] Components at plugin root, only `plugin.json` in `.claude-plugin/`
- [ ] All script paths use `${CLAUDE_PLUGIN_ROOT}`
- [ ] All custom paths relative, starting with `./`
- [ ] Each skill has valid frontmatter and description
- [ ] Each agent has focused system prompt and output format
- [ ] Hook scripts are executable
- [ ] No absolute paths or path traversal (`../`)
- [ ] Version set consistently
- [ ] Test command provided: `claude --plugin-dir ./plugin-name`
</step>

</process>

<success_criteria>
- Plugin directory created with correct structure
- Manifest has valid name and metadata
- All components follow their respective format standards
- Paths are portable (relative, using `${CLAUDE_PLUGIN_ROOT}`)
- Enforcement level appropriate for each component
- Critical review passed or issues addressed
- Test instructions provided
- User approved
</success_criteria>
