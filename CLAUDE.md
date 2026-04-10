# Claude + Codex repo workflow

You are operating in a repo that uses:
- Claude Code for planning and review
- Codex MCP for implementation and command execution

## Required sequence

For implementation tasks, always follow this order:

1. Use the `planner` subagent first.
2. Show the plan to the user unless they explicitly asked for silent execution.
3. After plan approval, use the Codex MCP tool to implement the plan.
4. After Codex completes, use the `reviewer` subagent.
5. If review fails, do at most one additional Codex fix round unless the user asks for more.

## Planning requirements

The planner must produce:
- Goal
- Acceptance Criteria
- Constraints
- Target Files
- Test Plan
- Risks
- Execution Prompt for Codex

## Codex execution requirements

When calling Codex:
- Send the approved plan, not a vague task
- Prefer the smallest patch that satisfies the plan
- Stay within target files unless necessary
- Run the listed test commands when possible
- Return:
  - summary of changes
  - files changed
  - commands run
  - test results
  - unresolved concerns

## Review requirements

The reviewer must:
- Compare implementation against the approved plan
- Check every acceptance criterion
- Identify unexpected file changes
- Identify missing tests or weak edge cases
- Return PASS or FAIL
- If FAIL, provide a single fix prompt for Codex

## General repo rules

- Prefer minimal diffs
- Do not add dependencies unless the task explicitly requires it
- Do not change public APIs unless required
- Do not self-approve code without review
- Be explicit about uncertainty

## Codex MCP call configuration

When calling the Codex MCP tool, always use:

- cwd: "."
- approval-policy: "on-request"
- sandbox: "workspace-write"
- include-plan-tool: false

## Artifact persistence

The planner subagent must always persist its final output to:
- .claude/artifacts/latest-plan.md

The reviewer subagent must always persist its final output to:
- .claude/artifacts/latest-review.md

These files should be overwritten on each run and contain only the finalized artifact.