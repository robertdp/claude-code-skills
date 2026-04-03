---
name: interview
description: Structured interviews to gather requirements, surface unknowns, and explore tradeoffs before planning or decision-making. Use when user says "interview", "gather requirements", "surface unknowns", or invokes /interview or /robert:interview.
allowed-tools: Read, Glob, Grep, Agent, WebSearch, WebFetch
---

## Objective

Conduct adaptive interviews through structured questioning. Surface assumptions, unknowns, and tradeoffs before moving to action.

## Domain Detection

Detect from user's statement, apply domain-appropriate frameworks:
- **Technical**: architecture, APIs, implementation, performance, security
- **Product**: requirements, users, success metrics, priorities, constraints
- **Problem-solving**: root cause, debugging, decisions, risks, dependencies

## Question Strategy

Use AskUserQuestion with 2-4 questions per round:
- Each question has 2-4 specific options
- Use single-select (default) for decisions where exactly one path must be chosen. Use `multiSelect: true` when multiple options can apply simultaneously (e.g., "which concerns apply?", "which constraints exist?").
- Progress from broad to specific
- Batch related questions to reduce round-trips
- When the user picks "Other", ask one follow-up to clarify before continuing.
- Most interviews complete in 2-3 rounds. Stop when a round surfaces no new unknowns, constraints, or tradeoffs — proceed directly to Handoff.
- **Verify, don't assume**: When a user answer can be checked against the codebase or external sources, check it. Confident answers are not necessarily correct answers.
- **Backtracking**: If an answer contradicts or invalidates an assumption from an earlier phase — or conflicts with what you found in the codebase — revisit that phase before continuing. State what changed and re-confirm before moving forward.

## Context Gathering

Before and between question rounds, gather context autonomously when it would improve the next question. This is not a fixed step — use judgement based on the domain:

- **Code/technical topics**: Scan project structure, read key files (configs, entry points, schemas), check dependencies. Use Agent with subagent_type=Explore for broader searches. Populate question options from what you actually find rather than generic defaults.
- **Product/business topics**: Use WebSearch when the domain, market, or tooling is unfamiliar enough that it would sharpen your questions.
- **Problem-solving**: Read relevant code, logs, or error paths to verify user claims before building on them.

Only ask the user questions you cannot answer by reading code or searching. Intent, priorities, and business context come from the user. File structure, dependencies, existing patterns, and tech stack come from investigation.

## Interview Phases

1. **Context & Goals**: What and why.
2. **Constraints**: Limitations, non-negotiables, exclusions.
3. **Deep Dive**: Domain-specific details and choices.
4. **Assumptions**: Look for gaps, contradictions, and wrong information in what's been gathered — both from user answers and your own investigation. When you find a conflict between a user answer and codebase evidence, present both sides and ask which is current. When an assumption can't be verified, say so and ask the user to confirm. Surface unstated dependencies and risks the user hasn't raised.
5. **Summary**: Confirm understanding with a structured summary of what was gathered — goals, constraints, decisions, and open risks.
