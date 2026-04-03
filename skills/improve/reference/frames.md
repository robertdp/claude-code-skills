# Adversarial Frames

## Frame Catalogue

The Pessimist and Simplifier are strong defaults — keep them unless there's a clear reason to replace one. For code targets, also include The Engineer. Fill remaining slots with the frames most relevant to the target's risk profile. Each frame must cover a distinct concern — don't add frames that overlap with one already selected.

### Default Frames

| Frame | Focus |
|-------|-------|
| **The Pessimist** | Failure modes, edge cases in the code's own logic, implicit assumptions about external state, race conditions, data loss, unhandled states |
| **The Simplifier** | Over-engineering, unnecessary abstraction, accidental complexity, simpler alternatives that achieve the same goal |

### Technical Frames

| Frame | Focus | Best for |
|-------|-------|----------|
| **The Engineer** | Resource lifecycle and production fitness. Object instantiation patterns (per-request vs. shared vs. singleton) and whether expensive resources (tokens, connections, computed state) are correctly scoped. Hot-path analysis: unnecessary network calls, I/O, or computation that could be cached or amortised. Cross-implementation consistency: when the code adds a new variant of an existing pattern, compare resource models against existing ones — differences may require different strategies even when the interface is identical. | Code targets (include by default) |
| **The Adversary** | Injection, auth gaps, data exposure, abuse scenarios, resource exhaustion, privilege escalation | Code with external inputs, APIs, user-facing systems |
| **The Integrator** | Contract mismatches, version incompatibilities, upstream/downstream assumptions, breaking changes | APIs, integrations, multi-service systems |
| **The Operator** | Observability gaps, failure recovery, deployment risks, scaling bottlenecks, operational burden | Infrastructure, services, production systems |
| **The User** | UI state completeness and UX correctness. Variant coverage: when a type is widened (new union variant, new state shape), do all renderers handle every variant or does anything fall through to a default/empty state? Post-action feedback: after every user action, is there visible confirmation? Data flow through UI layers: does every component that receives changed data display something meaningful? Loading, empty, and error states for every data-dependent view. | Frontend targets, features modifying existing UI data flow |

### General Frames

| Frame | Focus | Best for |
|-------|-------|----------|
| **The Realist** | Feasibility, real-world constraints, resource requirements, timeline risks, unstated dependencies, cost/benefit ratio, opportunity cost, ROI, hidden costs, sunk cost traps | Plans, proposals, strategies, designs, decisions involving significant effort or budget |
| **The Challenger** | Challenges the core premise — should this exist at all? Questions whether the problem is real, the approach is warranted, or the goal is correct. Also checks internal contradictions, unsupported claims, logical gaps, missing evidence, ambiguous requirements | Any target where the direction or reasoning hasn't been stress-tested |
| **The Stakeholder** | Missing perspectives, affected parties not considered, second-order effects, unintended consequences for users/teams/customers | Anything with human impact — products, policies, org changes, UX decisions |
| **The Historian** | Prior art, lessons from similar attempts, why past approaches succeeded or failed, patterns that recur, known pitfalls in this problem space | Novel approaches, rewrites, migrations, strategy shifts — anything where "this has been tried before" is likely |

**Caveat — The Historian:** Most valuable when public post-mortems, case studies, or documented failure patterns exist (framework migrations, architectural shifts, technology adoption). Use WebSearch to find and cite real precedents rather than relying on training data alone. Weak on novel problems with no precedent, proprietary/internal history, and domains where the user is already the expert. Fabrication risk is real — if the Historian can't find genuine prior art, it may invent plausible-sounding precedents. Prefer concrete, cited evidence over generic wisdom.

### Custom Frames

Custom frames are fine — state the focus areas and what kind of target it suits. The goal is distinct, relevant perspectives — not padding to a number.

## Selection Rules

**The Pessimist and Simplifier are strong defaults.** Keep both unless the target type makes one irrelevant — e.g., a business strategy review may benefit more from The Realist than The Pessimist. For code targets, also include The Engineer.

Add 1-3 domain-specific frames based on target type. Select frames that cover **distinct concerns** — overlapping perspectives waste reviewer slots and increase synthesis noise.

Suggested combinations by target type:

- **Code with external interfaces** → Adversary, Integrator
- **Frontend code** → User, Adversary
- **Infrastructure/services** → Operator, Adversary
- **Plans/proposals/strategies** → Realist, Challenger
- **Documents/specs/requirements** → Challenger, Stakeholder
- **Architecture/design decisions** → Realist, Historian, Integrator
- **Products/features** → Stakeholder, Realist
- **Rewrites/migrations** → Historian, Operator, Realist

These are suggestions, not rules. Pick the frames that match the actual target. A security-focused code review doesn't need the Realist; a business strategy doesn't need the Operator.

**Maximum 6 frames** (defaults + up to 3-4 domain-specific). Beyond this, synthesis quality degrades — too many overlapping findings, too much noise to deduplicate.

If the user stated a specific focus (e.g., "for security"), ensure the matching frame is included and dominates the review. Other frames still run but are secondary.

## Non-Code Adaptation

For non-code targets, frames analyse the idea/plan/document itself. The Engineer and User frames are not applicable — exclude them. Reviewers receive the target content directly in their prompt rather than file paths. The same structured output format applies — evidence becomes quotes, references, or logical citations rather than file:line.
