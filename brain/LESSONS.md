# Lessons

> Validated, reusable insights only (not guesses). IDs: `L-###`.
> Format: problem -> cause -> solution, with evidence.

## Template
```
### L-### Title
- Context: where it applies
- Problem / Cause: what went wrong and why
- Solution: what works
- Evidence: test, commit, link
```

## Known Pitfalls
### L-001 Web retrieval limits in research tasks
- Context: Claude Code research using the fetch helper.
- Problem / Cause: long pages are truncated and the rest is only summarized; freedesktop/GNOME GitLab refused automated fetches.
- Solution: request verbatim quotes and mark summarized text separately; for source files try official GitHub mirrors or Apple's `tutorials/data/...json` documentation endpoints.
- Evidence: [T-005 report](../docs/research/T-005-global-capture.md) sections 1, 9 and 10 (2026-11-25).

## Reusable Solutions
- none yet
