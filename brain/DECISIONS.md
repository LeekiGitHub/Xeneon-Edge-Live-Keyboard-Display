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
### D-008 Keyboard profile priorities
- Date: 2026-11-25 | Status: accepted | By: user
- Context: The owned keyboard is a Lemokey P1 Pro (75 %, ISO-DE, knob), not a TKL; no Moonlander is available.
- Decision: M1: Lemokey P1 Pro, Advantage360 SmartSet. M2 adds Voyager, Moonlander, generic TKL. QWERTZ-DE and QWERTY-US remain; physical ISO/ANSI geometry is separate from logical layout.
- Alternatives: TKL (ISO-DE, ANSI-US) as M1 profile (replaced).
- Consequences: M1 no longer requires a TKL or ANSI geometry profile; generic TKL variants are open.

### D-007 Evaluation frame (Gate G1)
- Date: 2026-11-25 | Status: accepted | By: user
- Context: Technology options need criteria fixed before they are assessed.
- Decision: [T-004 report](../docs/research/T-004-evaluation-frame.md): knock-outs, weights (C1-C9: 18/17/17/13/12/8/10/3/2), floors, evidence rules, effort criteria E1-E8, license rules. macOS prototype variants in T-007/T-009 if research justifies them, with separate approvals. 60 % evidence coverage permits only a preliminary ranking; the final architecture decision also requires all decision-relevant knock-outs to be sufficiently clarified.
- Alternatives: initially proposed weights and hard effort limits (adjusted, report section 7).
- Consequences: Weight or threshold changes need approval.

### D-006 T-003 work plan, constraints and model strategy
- Date: 2026-11-25 | Status: accepted | By: user
- Context: T-003 needs an evidence-based, risk-first process within limited usage quotas.
- Decision: Plan with T-004..T-013 and gates G1-G4 approved ([TASKS.md](TASKS.md)). Reports in `docs/research/`; prototypes only after approval, in `prototypes/`. No additional recurring costs in M1/M2 (PROJECT.md NF-Cost). Standard model tier by default; Advanced only with a stated reason, recommended to the user.
- Alternatives: Advanced tier for several research tasks (reduced to save quota).
- Consequences: No installs, prototypes, commits or pushes without approval.

### D-005 Requirements baseline approved; first-launch language
- Date: 2026-11-24 | Status: accepted | By: user
- Context: Closing T-002; first-launch UI language was open (R-07).
- Decision: PROJECT.md approved as baseline for T-003. UI language on first launch follows a supported OS language, else English; manually changeable.
- Alternatives: fixed English default (not chosen).
- Consequences: Remaining open items stay assigned to their phases (PROJECT.md section 8).

### D-004 Roles and requirement conventions
- Date: 2026-11-24 | Status: accepted | By: user
- Context: Review R-05/R-06 found unclear markers, milestones and responsibilities.
- Decision: Product Owner owns requirements, final approvals and license; agents research and propose; major architecture needs recommendation, independent review and approval; details within an approved frame are agent-decided. Markers `[C]`/`[P]`/`[O]` only. M2 is cumulative on M1. 30 ms/60 FPS are preliminary targets; reliable, fluid visualization is binding.
- Alternatives: `[A]` marker for assumptions (dropped, unused).
- Consequences: T-003 proposes performance measurement and limits for approval.

### D-003 Separation of live visualization and training data
- Date: 2026-11-24 | Status: accepted | By: user
- Context: Review R-01 found the capture and data lifecycle underspecified.
- Decision: System-wide input is transient, visualization only. Training data only during active, focused exercises; focus loss pauses. Persist aggregates; no retention of individual events/key timings; exceptions need a data-minimal proposal and explicit approval.
- Alternatives: persisting per-key timings after exercises (rejected unless T-003 proves necessity).
- Consequences: Scenarios PS-1..PS-9 in PROJECT.md; T-003 checks aggregate sufficiency.
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
