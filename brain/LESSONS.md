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
- Problem / Cause: long pages are truncated and the rest is only summarized; freedesktop/GNOME GitLab and keychron.com refused automated fetches; this PC cannot render PDF pages (no `pdftoppm`), so diagrams in PDFs are unreadable.
- Solution: request verbatim quotes and mark summarized text separately; for source files try official GitHub mirrors or Apple's `tutorials/data/...json` documentation endpoints. For PDFs and firmware source, download with `curl` to the scratchpad and use the available `pdftotext`; pin repository files to a commit.
- Evidence: [T-005 report](../docs/research/T-005-global-capture.md) sections 1, 9 and 10; [T-006 report](../docs/research/T-006-event-model.md) sections 1 and 13 (2026-11-25).

## Reusable Solutions
- none yet
