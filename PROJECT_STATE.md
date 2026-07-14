# Project State: Course Notes

## Current operational state

Course Notes is a Markdown-first study system with a public release surface.
Real or unsanitized material belongs in the Git-ignored `private/` workspace;
tracked `courses/` contains only publication candidates or already-approved
public material. `manifest.json` and `REVIEW_QUEUE.md` are generated views of
the tracked public tree.

`make public-safety` is the canonical publication-safety gate. It checks the
Git boundary, scans tracked and staged non-private files for defined leak
patterns, runs public-release validation, and runs the test suite. This is
moderate accidental-publication protection for a trusted workflow, not a
semantic, legal, or default-deny approval system. The optional local hooks
run that gate when installed; CI runs the same gate and separately verifies
that generated public files are current.

`.public-release-blocklist` is an optional local extension to public-release
validation. It is ignored by Git by default and must not be committed.

## Known risks and deferred decisions

- Passing automated checks is necessary but does not establish clearance to
  publish course-derived material. Follow `docs/public-release-checklist.md`
  before promoting anything from `private/` to `courses/`.
- Local hooks can be bypassed and are not installed automatically. Remote CI
  is only a post-push ratification; its green status cannot be claimed until a
  push occurs.
- Private material is deliberately untracked and requires independent backup.

## Historical context

The initial safety-infrastructure rollout is recorded in
`handoffs/2026-07-safety-infrastructure-completion.md`. It preserves the
implementation snapshot and the limits that required later reconciliation.
