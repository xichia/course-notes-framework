# Sanitize for Public Release

## Prompt

Create a sanitized public-release candidate from **[path to private notes repository or directory]**.

Work entirely in proposal mode. Do not delete, rewrite, move, or stage any file without my explicit approval. Present every proposed change as a clear action with the affected file path and the reason it is needed. If a decision is unclear, ask before acting.

Read the full [publication policy](../docs/publication-policy.md) and [public release checklist](../docs/public-release-checklist.md) first. This prompt implements those policies; do not contradict them.

## Public-Safe Content

The following may remain in a public release when they contain no embedded course material:

- Course Notes system files: scripts, Makefile, templates, prompts, docs, tests, and validation logic
- Generated files rebuilt from public-safe sources (`manifest.json`, `REVIEW_QUEUE.md`)
- Synthetic/demo content
- Original notes with `visibility: public-original` and `source-risk: original`
- Open-licensed material only when its licence and attribution requirements are explicitly documented
- `visibility: public-framework` items (documentation, templates, prompts, scripts, or synthetic examples)

## Not Public-Safe by Default

The following must be removed or replaced with synthetic/demo content. Do not keep them in a public release:

- Real lecture-derived notes, raw lecture notes, slides, recordings, transcripts
- LMS material, restricted links, announcements, or copied handbook text
- Handouts, problem-sheet text, solution sheets, or derived examples
- Exam questions, assessment material, or course-derived exam maps
- Lecturer-specific hints, examples, anecdotes, and teaching sequences
- Copied diagrams and close paraphrases that preserve protected teaching expression
- Personal study evidence that should remain private
- Any file with `visibility: private`
- Any file with `source-risk` of `lecture-derived`, `problem-sheet-derived`, `exam-derived`, `lms-derived`, or `unknown`
- Files with missing `visibility` or `source-risk`
- Absolute local paths, home-directory paths, drive-letter paths, or file URI scheme references
- Sensitive terms, credentials, private links, student identifiers, or personal data

AI summarization, rewriting, translation, or paraphrasing does not automatically make course material safe to publish. Sanitization must remove protected expression, not merely change its wording.

## Metadata Rules

- Preserve `visibility` and `source-risk` exactly on every surviving note.
- Never upgrade `visibility` from `private` or change `source-risk` to `open-licensed` without explicit instruction and provenance evidence.
- Keep uncertain provenance as `source-risk: unknown` and do not publish it.
- If a note's `source` field contains suspicious terms (e.g., "lms", "lecture slide", "exam question", "problem sheet"), flag it for manual review even if visibility appears safe.

## Proposal Workflow

For each course in the repository:

1. List every file under `courses/<course>/`.
2. For each file, check `visibility`, `source-risk`, `source`, and whether the content reproduces protected teaching expression.
3. Categorize each file as keep, replace with synthetic/demo, or remove.
4. Present the full categorized list for approval before making any changes.
5. Only proceed to the next step after I approve the plan.

After the approved files are finalized:

6. Rebuild generated files from only the public-safe content: `make manifest && make review`.
7. Verify no tracked risky files remain: `.gitignore` is not enough for files already tracked by Git. If any unsafe file is tracked, propose `git rm --cached <path>` to stop tracking it while preserving the local working copy. Do not delete local files unless I explicitly approve deletion.
8. If risky files were already pushed to any remote, warn that Git history cleanup may be required before public release — do not attempt history rewriting without explicit instruction.

## Required Checks

After all changes are approved and applied, run:

```bash
make all
make validate-public
make pre-release
git status
```

Do not weaken `make validate-public` or `make pre-release`. If either fails, identify the specific files and metadata causing the failure and propose fixes — do not skip or downgrade checks. Passing the automated checks is necessary but not sufficient: complete the manual [public release checklist](../docs/public-release-checklist.md) and manually review every file before publishing.
