# Sync LMS Announcements

> **Part of a larger workflow.**
> [`sync-lms-course-content.md`](sync-lms-course-content.md) is the authoritative
> instruction for any routine LMS sync, and covers both announcements and newly released
> or updated unit materials. Start there. This document is the announcement layer in
> detail — the schema, hash rule, comparison semantics, and preservation rules that the
> tooling implements and that the broader workflow refers back to.
>
> The mechanical half of the procedure below is now scripted:
> `lms_sync.py reconcile-announcements` (in `tools/lms-import/` in the private tree)
> computes the hashes, classifies each entry, folds duplicate rendered blocks while
> recording the occurrence count, appends the check entry, and archives the prior
> `inventory.json`. What stays manual, deliberately, is transcribing the announcements
> verbatim from the browser and merging the proposed sections into `ANNOUNCEMENTS.md` by
> hand so the `>` blockquote annotations from earlier syncs survive. The judgement
> sections below — what to flag, what never to resolve alone — apply either way.

## Prompt

Perform a routine announcement sync between the Curtin LMS (Blackboard Ultra) and the private
course records for **[course code, or "every active course in `private/courses/`" if unspecified]**.

This is a narrow, repeatable maintenance task, not a course-content audit. Touch only
`admin/announcements/` for the courses in scope. Do not review, edit, or comment on `materials/`,
`study/`, `lms-import/`, or any other course content unless an announcement directly conflicts
with something already recorded there (see "When to flag instead of resolve" below).

## Where records belong

Everything lives under the Git-ignored `private/` tree — see
[repository-layout.md](../docs/repository-layout.md). For each course:

```text
private/courses/<COURSE-CODE>-<term>/admin/announcements/
├── ANNOUNCEMENTS.md   human-readable capture: one section per distinct announcement,
│                      plus a short "Derived notes" section at the end
├── inventory.json     machine-readable capture: one object per distinct announcement
├── raw.txt            optional: raw transcription, used when a capture attempt partially
│                      failed or needs its own provenance trail separate from the summary
└── README.md          optional: one-paragraph pointer, used when the other files need
                        context a first-time reader wouldn't have
```

If a course has no `admin/announcements/` directory yet, create it — do not put announcement
records anywhere else (not in `materials/`, not in the course root). If the course directory
itself does not exist yet, only create the minimal scaffold this task needs (a short `README.md`
plus `admin/announcements/`); do not fabricate `materials/`, `study/`, or `lms-import/` content
you have no source for. Follow [course-onboarding.md](../docs/course-onboarding.md) if the user
separately asks for a full course setup.

## Conventions already in use

Match these exactly; do not invent a new schema per run.

- **`inventory.json` fields per announcement:** `title`, `posted_on`, `body`, `posted_by`,
  `posted_to`, `stable_lms_id`, `canonical_url`, `links` (array), `attachments` (array),
  `pinned_or_priority`, `content_sha256`, `reconciliation_status`
  (`NEW`/`UPDATED`/`UNCHANGED`/`MISSING_FROM_CURRENT_LMS_VIEW`), `first_seen_check` (date the
  record was first captured).
- **File-level `inventory.json` fields:** `course`, `lms_course_id`, `unit_code`, `source_url`,
  `hash_basis` (the exact normalization string used), `history_controls_checked` (a sentence
  confirming whether pagination/load-more/older-history controls exist and were exhausted),
  `checks` (an array, one entry per sync date, each with `checked_date`, `collected_at`
  ISO-8601 UTC, `rendered_block_count`, `distinct_record_count`, and a `reconciliation` count
  object), `announcement_count`.
- **Hash rule:** `sha256` of the UTF-8 string
  `f"{title.strip()}\n{posted_by.strip()}\n{posted_on.strip()}\n{body.strip()}"`, LF line
  endings, no other normalization. Recompute with a short Python one-liner or script — do not
  hand-transcribe hex digests; verify every stored hash against its own record before finishing
  (a self-consistency check: recompute from the stored fields and diff against the stored
  `content_sha256`).
- **`ANNOUNCEMENTS.md` structure:** a one-line status summary at the top (counts by
  reconciliation category), then one `##` section per distinct announcement with a `Status:` line,
  the `Posted on:` line, the body verbatim, and a `Posted by:` line, in the order the LMS renders
  them. A `> **...**` blockquote directly under an entry is how prior syncs recorded a conflict,
  anomaly, or cross-check note — keep using that pattern rather than a separate free-floating
  commentary section for entry-specific notes. End the file with a "Derived notes" section
  covering deadlines, actions, Gen-AI policy, and pass conditions found in that check.

## How to compare

1. Read the existing `inventory.json` (and `ANNOUNCEMENTS.md`) for the course first. This is the
   baseline — do not treat the live LMS page as a blank slate.
2. Open the course's LMS outline/announcements page in an authenticated browser session. Verify
   the on-page course code and title rather than trusting a previously recorded mapping blindly.
3. Scroll to the end (footer) to confirm every rendered announcement was seen, and note explicitly
   whether a pagination / "show more" / "load more" / older-history control exists. Routine syncs
   still need this confirmed each time — Ultra course views vary in whether one exists.
4. Match live announcements to stored records by content, not position: same `title` +
   `posted_by` + `posted_on` + body. Classify each live announcement as:
   - **NEW** — no matching stored record.
   - **UPDATED** — same title/author/posted_on but a changed body (rare; LMS announcements are
     normally immutable once posted — treat an apparent update as worth a closer look, not a
     routine occurrence).
   - **UNCHANGED** — matches a stored record exactly (hash-identical).
   - **MISSING FROM CURRENT LMS VIEW** — a stored record has no live match. Do not delete it;
     mark it and leave it in place. Announcements can scroll out of a course's retention window
     without meaning anything was wrong.

## Content equality is not identity

Blackboard Ultra's rendered outline view exposes no per-announcement ID, permalink, or
second-precision timestamp. Two records with identical `content_sha256` are only known to have
*identical content* — never assert they are "the same announcement" in an identity sense, and
never use hash equality to justify silently discarding a rendered block without recording that a
duplicate-content decision was made. This matters most for boilerplate posts (e.g. "Acknowledgement
of Country", "Timetable changes") that legitimately repeat verbatim across different courses and,
occasionally, multiple times within the same course's render — see the `<COURSE-CODE>-<term>`
`inventory.json` for a worked example of documenting a duplicate-block anomaly instead of quietly
deduplicating it.

## Preserving earlier captures

Never overwrite a prior capture's evidence, even a failed one. If a previous attempt recorded a
tool error, an empty result, or wording that needs correcting, archive the old file next to the
new one (e.g. append `-<YYYY-MM-DD>-attempt` to the filename) before writing the replacement.
Announcement *wording* is source evidence — do not paraphrase, correct typos in, or "clean up" a
captured body; transcribe it exactly as rendered, including the source's own typos.

## Targeted validation (not a full repository audit)

After writing changes, run only:

1. JSON validity + hash self-consistency for every `inventory.json` file touched this run (parse,
   then recompute each `content_sha256` from its own stored fields and diff).
2. A quick read-through of each changed Markdown/JSON diff for obvious errors (truncated hashes,
   mismatched counts, broken `##` structure).
3. `git status` in the affected repo restricted to the touched paths — confirm only
   `private/courses/<course>/admin/announcements/...` changed.
4. A secret/session scan of the diff (`grep -i` for cookie/token/authorization/bearer/session/
   password/api-key/secret) — should find nothing; investigate before proceeding if it does.
5. Confirm the outer public repository (`course-notes`, not `private/`) shows no diff at all.

Do **not** run the full private-notes validator (`make study-validate`) or the full test suite
(`make public-safety`) as a routine part of this task — those check the entire `private/courses/`
tree (concept-note frontmatter, unrelated test fixtures, etc.) and will surface pre-existing,
unrelated findings that have nothing to do with this sync. Only run them if the user separately
asks for a broader validation pass.

## When to flag instead of resolve

Stop and record — in a blockquote under the relevant `ANNOUNCEMENTS.md` entry, plus a mention in
your final report — rather than silently deciding, when you find:

- Two announcements (old and new) that contradict each other (e.g. differing start weeks, changed
  weightings).
- An announcement that conflicts with a stored unit outline (assessment dates, venues, pass
  conditions, Gen-AI permission) — cross-check against `materials/unit-outline/` when it exists
  for that course.
- A rendered duplicate block with no stable ID to confirm whether it is one announcement or two.
- A course whose on-page code/title doesn't match what the repository's directory name implies.
- Any embedded form or attachment link — record it, but do not follow, download, or submit it as
  part of a routine sync.
- The LMS returning an auth prompt, MFA challenge, or CAPTCHA mid-sync — pause and ask the user to
  complete it; never attempt to bypass it.

## Git boundaries

Leave every change unstaged and uncommitted in both the outer repository and the private nested
repository, unless the user separately and explicitly authorizes staging or committing in that
same request. Never stage, commit, stash, reset, clean, pull, or push as part of a routine sync.
Check `git status` in both repositories before starting so pre-existing uncommitted work is never
mistaken for something this run produced — report it as a distinct baseline instead.

## Report

End with: courses checked and LMS access time; new / updated / unchanged / missing-from-view
counts per course; any conflicts or anomalies flagged; the exact files changed; validation
results; and final `git status` for both repositories.
