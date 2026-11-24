
# Shared AI Agent Instructions

## 1. Core Principles

- Prioritize correctness, reliability, security and maintainability.
- Quality is more important than speed.
- Prefer simple solutions over unnecessary complexity.
- Work incrementally with small, verifiable changes.
- Never assume undocumented requirements.
- Communicate with the user in German.
- Write AGENTS.md and all files in brain/ in English (D-002).

## 2. Shared Project Memory

The brain/ directory is the persistent, shared project memory
for all coding agents, including Claude Code and Codex.

- PROJECT.md: Project goals, requirements and constraints.
- ARCHITECTURE.md: Technical architecture and conventions.
- STATUS.md: Current project state.
- TASKS.md: Planned, active and completed tasks.
- HANDOFF.md: Latest agent handoff and next action.
- DECISIONS.md: Important decisions and their rationale.
- LESSONS.md: Reusable lessons and known pitfalls.

Treat these files as shared knowledge, not as a conversation log.
Keep them concise, accurate and actionable.

## 3. Session Start

At the beginning of every new session:

1. Read brain/STATUS.md.
2. Read brain/HANDOFF.md.
3. Identify the current task and its scope.
4. Read additional brain files only when relevant.
5. Check the actual repository state when necessary.
6. Continue from the last verified project state.

If documentation conflicts with the code or test results,
investigate the discrepancy before proceeding.

## 4. Context and Token Efficiency

- Never load the entire brain or repository by default.
- Search for relevant files before reading large files.
- Read only the context required for the current task.
- Avoid repeatedly reading unchanged documentation.
- Avoid unnecessary tool calls and verbose explanations.
- Never store complete chat transcripts in the brain.
- Reference existing files instead of duplicating content.
- Keep handoffs short and focused on actionable information.
- Avoid unnecessary agent review cycles.
- Never activate paid extra usage without user approval.

## 5. Task Workflow

For each development task:

1. Understand the requirements.
2. Define the scope and acceptance criteria.
3. Inspect the relevant existing code.
4. Plan the implementation proportionally to its complexity.
5. Implement the smallest complete solution.
6. Run relevant tests and quality checks.
7. Update the relevant project memory files.
8. Prepare a handoff when appropriate.

Do not expand the task scope without user approval.

Task history:
- Completed tasks must retain their original acceptance criteria
  and relevant verification results.
- Do not discard task history when moving tasks to Done.
- Never reconstruct missing historical acceptance criteria
  as if they were original facts.

## 6. Agent Collaboration

Claude Code and Codex are equal collaborators.

Either agent may act as:
- Planner
- Implementer
- Reviewer
- Debugger

The shared brain and Git repository are the communication
channel between agents.

- Do not assume access to another agent's conversation.
- Do not overwrite another agent's unfinished work.
- Only one agent should modify the same working tree at a time.
- Preserve existing decisions unless evidence justifies a change.
- Document disagreements with evidence and alternatives.
- Escalate unresolved important decisions to the user.

## 7. Independent Review

For important or complex changes, prefer an independent review
by the agent that did not implement the change.

Review for:
- Correctness
- Security vulnerabilities
- Missing edge cases
- Missing or inadequate tests
- Regressions
- Maintainability
- Unnecessary complexity

During review, do not modify code unless explicitly requested.

Classify findings by severity and provide evidence.
Do not invent issues to justify a review.

## 8. Testing and Verification

- Never claim that tests passed unless they were executed.
- Report failed, skipped and unavailable checks honestly.
- Prefer automated tests over speculative validation.
- Add regression tests for significant bug fixes.
- Do not mark incomplete or unverified tasks as completed.
- Avoid unrelated refactoring.

## 9. Git and Safety

- Inspect Git status before significant changes.
- Preserve unrelated user changes.
- Keep changes small and logically grouped.
- Never commit secrets, credentials or API keys.
- Never execute destructive Git commands without approval.
- Do not push, deploy or publish without user approval.
- Create commits only when requested or explicitly authorized.

## 10. Memory Updates

Update shared memory after meaningful progress.

- STATUS.md: Current verified state and active blockers.
- TASKS.md: Task status and next priorities.
- DECISIONS.md: Significant decisions and reasoning.
- ARCHITECTURE.md: Relevant architectural changes.
- LESSONS.md: Validated, reusable insights.
- PROJECT.md: Approved changes to requirements.

Do not update every file unnecessarily.
Avoid duplication between files.
Do not read or modify brain/.obsidian/ unless requested.

## 11. Session End and Handoff

Before ending a meaningful work session, update HANDOFF.md with:

- Current task
- Agent and role
- Work completed
- Relevant changed files
- Test and verification results
- Open issues and blockers
- Next recommended action

Separate verified facts from assumptions.

The next agent must be able to continue without needing
the previous conversation.

## 12. Definition of Done

A task is complete only when:

- Its acceptance criteria are satisfied.
- Relevant checks have been performed.
- Known issues are documented.
- Relevant project memory is updated.
- The implementation is consistent with project requirements.

Never hide uncertainty, incomplete work or failed checks.

## 13. Shared Workflow Commands

When the user sends one of these keywords, treat it as an explicit
user instruction, not as a native CLI command. All other rules apply.

- start: Load required project context, identify the current state
  and next task. Report the next action without implementing it.
- plan: Analyze the selected task, define acceptance criteria and
  propose an implementation plan. Wait for approval before implementation.
- implement: Execute an approved task, run relevant checks and
  update shared memory.
- review: Independently review the specified changes. Report
  prioritized findings with evidence. Do not modify implementation code.
- handoff: Update shared memory and prepare a concise handoff
  for the next agent.
- checkpoint: Verify repository state, relevant checks and documentation
  consistency. Propose a Git commit without executing it.
- compact: Review shared memory for outdated or redundant information.
  Propose safe reductions without deleting important decisions,
  requirements or verification history.
