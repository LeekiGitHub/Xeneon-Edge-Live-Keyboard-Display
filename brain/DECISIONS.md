# Decisions

> Significant decisions only, newest first. IDs: `D-###`, never reused.
> Status: `proposed` | `accepted` | `superseded by D-###`.

## Template
```
### D-### Title
- Date: YYYY-MM-DD | Status: proposed | By: <user/agent>
- Context: why a decision was needed
- Decision: what was chosen
- Alternatives: considered options and why rejected
- Consequences: impact, follow-ups
```

## Log
### D-002 English as documentation language
- Date: 2026-11-24 | Status: accepted | By: user
- Context: Brain files contained mixed English and German content.
- Decision: `AGENTS.md` and all files in `brain/` are written in English. Agents communicate with the user in German.
- Alternatives: German documentation; not chosen by user.
- Consequences: Existing German content translated to English without loss of information.

### D-001 Shared file-based memory in `brain/`
- Date: 2026-11-24 | Status: accepted | By: user
- Context: Claude Code and Codex need shared, persistent project context.
- Decision: Markdown files in `brain/`, rules in `AGENTS.md`, `CLAUDE.md` imports `AGENTS.md`; Obsidian as UI.
- Approval: User explicitly approved Obsidian as the UI for the shared memory on 2026-11-24 (session instruction to Claude Code). Obsidian is a viewing/editing tool only; brain files stay plain Markdown usable without it.
- Alternatives: not documented.
- Consequences: Brain files are part of the repo; `brain/.obsidian/` stays local (git-ignored).
