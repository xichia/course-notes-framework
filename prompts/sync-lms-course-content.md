# Sync LMS Course Content

## Prompt

Perform a routine LMS content sync for **[course code, or "every active course" if
unspecified]**: check what the LMS is showing now, compare it against what the private
repository already stores, download what is genuinely new, and record what changed.

This is the authoritative instruction for that task. It covers both **announcements**
and **released or updated unit materials**.
[`sync-lms-announcements.md`](sync-lms-announcements.md) remains the detailed reference
for the announcement half — its schema, hash rule, and reconciliation conventions are
what the tooling implements — but start here, because announcements alone are only part
of a sync.

This is maintenance, not a course-content audit and not a study session. Do not review,
rewrite, or comment on `study/` modules, concept notes, or any note metadata as part of
it.

## Prerequisites

1. **A repository baseline.** Run `git status` in *both* the outer repository and the
   nested private repository before you start, and note what is already uncommitted.
   Pre-existing work must never be attributed to this run — or disturbed by it.
2. **A browser session.** The material crawl drives a persistent Chromium profile stored
   outside every tracked path. You will be prompted to complete the institutional login
   and MFA by hand; the tool pauses and waits. It never automates authentication.
3. **Playwright**, in the local environment beside the import tools. Only the crawling
   commands (`plan`, `sync`, `discover --browser`) need it. `discover`, `verify`, and
   `reconcile-announcements` are standard library only and run anywhere.

## The commands

All commands live in one script: `lms_sync.py`, under `tools/lms-import/` in the private
tree. Run them from the repository root.

```bash
# 1. Which courses exist, and what are their LMS IDs? (offline, read-only)
python3 private/tools/lms-import/lms_sync.py discover

# 2. Map each course's top-level areas once, or after the LMS is restructured.
#    Writes lms-import/course-map.json; needs a live session.
python3 private/tools/lms-import/lms_sync.py discover --browser --course <COURSE-CODE>

# 3. Dry run: crawl, compare, report proposed changes, write nothing.
python3 private/tools/lms-import/lms_sync.py plan --course <COURSE-CODE>

# 4. Apply: crawl, download, update the stored inventories.
python3 private/tools/lms-import/lms_sync.py sync --course <COURSE-CODE>

# 5. Announcements: reconcile a captured set against the stored inventory (offline).
python3 private/tools/lms-import/lms_sync.py reconcile-announcements \
    --course <COURSE-CODE> --candidates <captured>.json --apply

# 6. Validate what you just wrote (offline).
python3 private/tools/lms-import/lms_sync.py verify
```

Omit `--course` to select every active course. Pass it more than once for a subset.
`--check-date YYYY-MM-DD` overrides the recorded check date;
`--max-pages-per-area N` raises the per-area traversal cap; `--retry-known-missing`
re-attempts downloads previously confirmed to 404.

**Always run `plan` before `sync`.** `plan` writes nothing at all — no pages, no
attachments, no inventory — so it is the safe way to see what a sync would do.

## How courses and LMS IDs are discovered

`discover` reads the mapping out of the repository rather than from a hardcoded table.
For each course directory it takes `lms_course_id`, `unit_code`, `course`, and
`source_url` from `admin/announcements/inventory.json`, falling back to
`lms-import/import-summary.json` and `lms-import/course-map.json`. It reports, as
problems with a non-zero exit status:

- a course directory with no recorded LMS ID;
- two directories claiming the same LMS ID;
- a course whose announcement record and import summary disagree about its ID.

Fix those before syncing. When you open a course in the browser, **verify the on-page
unit code and title match the directory name** rather than trusting a stored mapping
blindly — a re-used Blackboard shell from a previous offering is the usual cause of a
mismatch, and it is a stop-and-ask condition.

## What is monitored

`discover --browser` enumerates the course's top-level areas and writes them to
`lms-import/course-map.json` with an `include` flag and a reason. Areas whose titles
look like course content — outline, materials, modules, topics, weeks, lectures,
tutorials, workshops, laboratories, practicals, assessments, resources, worksheets,
solutions, datasets, revision, exams — default to included. Anything unrecognised
defaults to `include: false` with `"unrecognised area — review and set include
manually"`, so a new area is surfaced for a decision instead of being crawled silently
or missed silently. **A human decision recorded in a previous run always wins**: re-running
discovery preserves your `include` flags and only adds newly appeared areas.

Within each included area the crawl follows nested folders, exhausts pagination and
"show more" / "load more" / older-history controls before treating a page as fully seen,
and collects every attachment linked from any page it reaches. Announcements are handled
separately (below), because their reconciliation is content-based rather than file-based.

### Never collected, never touched

The crawl refuses, by URL and by link text, anything matching grades, groups,
submissions, attempts, feedback, discussions, messages, analytics, tracking, assignment
upload, or test/quiz-taking. These are recorded as excluded links with their labels and
nothing more. Beyond that:

- No form is submitted, nothing is marked complete or reviewed, and no LMS state is
  changed. The whole run is read-only.
- Third-party services (lecture capture, reading lists, video platforms, collaboration
  tools) and every external host are **inventoried but never followed**. A link record is
  the deliverable, not the content behind it.
- Cookies, tokens, browser profile data, and other authenticated artefacts are never
  read into, printed by, or stored in the repository.

## How content is compared

Materials are matched on **stable LMS identity first**: the Blackboard file identifier
embedded in an attachment URL (`xid-<n>_<n>`), falling back to the canonical URL with the
fragment stripped when no identifier is exposed. Filenames are never used for matching —
lecturers reuse them freely across revisions. Each candidate is classified as:

| Status | Meaning |
| --- | --- |
| `NEW` | No stored record for this LMS object. |
| `UPDATED` | Stored record exists; the downloaded bytes hash differently. |
| `UNCHANGED` | Stored record exists and the hash is identical. |
| `MISSING_FROM_CURRENT_LMS_VIEW` | Stored but not rendered this run, or HTTP 404 upstream. |
| `UNCERTAIN` | Fetch failed, timed out, returned a login-shaped page, or produced bytes that contradict the filename. |
| `DUPLICATE_CONTENT` | Byte-identical to another object captured in the same run. |

**An absent item is never a deletion.** It may be hidden, unreleased, moved,
access-restricted, or simply missed by a partial page load. It is marked, given a
`missing_since` date, and its stored bytes stay exactly where they are.

**Content equality is evidence of equal content, not proof of identity.** Two objects
that hash the same are recorded as `DUPLICATE_CONTENT` with a `duplicate_of` pointer and
stored once; the tool never asserts they are the same LMS object. The same rule governs
announcements — see the "Content equality is not identity" section of
[`sync-lms-announcements.md`](sync-lms-announcements.md).

## Where content is stored

Everything lives under the Git-ignored private tree — see
[repository-layout.md](../docs/repository-layout.md). Per course:

```text
private/courses/<COURSE-CODE>-<term>/
├── admin/announcements/          announcement records (see sync-lms-announcements.md)
│   ├── ANNOUNCEMENTS.md          human-readable capture, hand-maintained
│   ├── inventory.json            machine-readable capture and check log
│   └── proposed-additions-<date>.md   generated: sections to merge by hand
└── lms-import/
    ├── course-map.json           discovered areas and their include decisions
    ├── sync-inventory.json       the material inventory: one record per LMS object
    ├── SYNC_REPORT.md            per-run summary by area and status
    ├── known-missing.json        attachments confirmed to 404 upstream
    └── areas/<area>/
        ├── manifest.json         pages, candidates, record-only and excluded links
        ├── pages/NN-<label>-<content-id>/{page.html,readable.txt,links.json}
        └── attachments/          the downloaded bytes, verbatim
```

`areas/`, `pages/`, `attachments/`, and `manifest.json` are the layout the earlier
per-course importers already established; a sync extends that tree rather than creating a
parallel one. Curating downloaded attachments into `materials/<category>/` for study use
is a **separate, later step** governed by the course's curation manifest — a sync never
reorganises `materials/`.

## Preserving earlier versions

- Downloaded bytes are stored exactly as received. Nothing is re-encoded, renamed for
  tidiness, or normalised.
- A changed version **never overwrites** the stored one. The previous file stays on disk;
  the new file is written beside it with a deterministic `-<first 8 hex of sha256>`
  suffix, and the superseded version is appended to the record's `versions` array with
  its hash, path, size, original filename, and `superseded_on` date.
- Filenames are sanitised deterministically: directory components and traversal
  sequences are stripped, reserved characters replaced, length bounded. The lecturer's
  original filename is preserved in the inventory as `lms_filename` regardless of what
  the file is called on disk.
- Announcement inventories are archived with the established
  `-<YYYY-MM-DD>-attempt` suffix before being replaced, with a counter so a second run on
  the same day cannot clobber the first archive.
- A failed or partial capture is evidence too. Keep it; never quietly replace it.

## Provenance recorded per item

`course`, `area`, `stable_lms_id`, `canonical_url`, `source_page`, `source_page_url`,
`title`, `lms_filename`, `stored_path`, `content_type`, `size_bytes`, `sha256`,
`visible_release_or_update_text` (the surrounding on-page text, which is where Blackboard
exposes release and size information), `rendered_occurrences` and `also_seen_on` (when
one object is linked from several pages), `first_seen_check`, `last_seen_check`,
`collected_at`, `status`, `versions`, and `notes`.

Material hashes are the SHA-256 of the exact downloaded bytes, with no normalisation.
Announcement hashes follow the rule already stored in every announcement inventory and
restated in [`sync-lms-announcements.md`](sync-lms-announcements.md); the only
normalisation is `strip()` with LF line endings, and it is recorded in `hash_basis`.

Each run appends one entry to `checks` — date, collection time, mode, areas covered,
pages captured, rendered link count, distinct item count, a sentence on whether
pagination controls existed and were exhausted, and counts by status. Existing check
history is never rewritten. Inventories are deterministic: the same inputs produce the
same bytes regardless of the order the LMS rendered its links.

## Announcements within a sync

Announcement bodies are rendered by the LMS as free text with no per-item identifier or
permalink, so the parsing half is **not reliably automated** and this workflow does not
pretend otherwise. The repeatable procedure is:

1. Open the course outline in the authenticated browser and read the announcements
   region, scrolling to the footer and exhausting any older-history control.
2. Transcribe each announcement **verbatim** — including the source's own typos — into a
   JSON list of `{title, posted_on, posted_by, posted_to, body, links, attachments}`.
   Never paraphrase, correct, or tidy the wording.
3. Run `reconcile-announcements --candidates <that file>`. It computes each hash with the
   documented rule, classifies every entry as NEW / UPDATED / UNCHANGED / MISSING, folds
   duplicate rendered blocks into one record while recording the occurrence count and the
   dedup basis, appends a check entry, and preserves every hand-written key in the stored
   inventory (`schema_note`, `duplicate_block_anomaly`, and similar).
4. It **refuses** a capture where any block lacks a title, posted-on, or body, rather than
   recording a partial capture as an authoritative check.
5. It writes `inventory.json` and a `proposed-additions-<date>.md`. `ANNOUNCEMENTS.md`
   stays hand-maintained, because prior syncs recorded conflicts and anomalies in `>`
   blockquotes under specific entries and regenerating the file would destroy them. Merge
   the proposed sections in yourself, keeping those annotations.

Run it without `--apply` first to see the diff and the proposal without writing.

## Validation

After a sync, run only these:

1. `lms_sync.py verify` — parses every touched inventory, recomputes each announcement
   hash from its own stored fields and diffs it against `content_sha256`, re-hashes every
   stored material file against its recorded `sha256`, reports records whose file is
   missing from disk, and scans the inventories for anything session-artefact shaped.
   Non-zero exit on any finding.
2. `python3 -m unittest test_lms_sync` from the tool directory — the offline suite. It
   needs no credentials, no network, and no browser.
3. A re-run of `plan` on an unchanged course: every item should come back `UNCHANGED`
   and nothing should be proposed. That is the idempotence check.
4. A read of each changed diff for obvious damage — truncated hashes, mismatched counts,
   broken Markdown structure.
5. `git status` in both repositories, restricted to the paths this run should have
   touched.

Do **not** run the full private-notes validator or the whole repository test suite as
part of a routine sync. They scan the entire course tree and surface pre-existing,
unrelated findings that have nothing to do with what changed. Run them only if asked for
a broader validation pass. If a validation command fails for an environmental reason —
a missing interpreter, an unusable virtual environment — report the exact command and the
actual cause. Do not describe it as a content regression.

## Security and Git boundaries

- Authenticated session state stays in the Git-ignored browser profile directory. Never
  copy it, print it, or commit it. Never write a cookie, token, authorization header, or
  password into any repository file.
- Everything a sync writes belongs under the ignored private tree. The outer public
  repository must show no diff from a sync.
- Leave every change **unstaged and uncommitted** in both repositories unless the user
  explicitly authorises otherwise in the same request. Never stage, commit, stash, reset,
  clean, pull, or push as part of a routine sync.
- Never write a machine-specific filesystem path into a note, prompt, script, or
  inventory.

## When to stop and ask

Stop, record what you found, and ask rather than deciding alone when you hit:

- an authentication prompt, MFA challenge, or CAPTCHA mid-run — pause for the user; never
  attempt to bypass one;
- a fetch returning a login-shaped page, which means the session expired mid-run and the
  results are partial (the tool flags this and exits non-zero);
- an on-page unit code or title that disagrees with the course directory name;
- an area that appeared, disappeared, or was restructured since the stored `course-map.json`;
- a large batch of items going `MISSING` at once, which usually means a partial page load
  or a permissions change rather than a genuine removal;
- an `UPDATED` unit outline, assessment instruction, rubric, or due date — those change
  what the student must do, so flag them explicitly rather than filing them silently;
- an announcement contradicting another announcement or the stored unit outline;
- anything the tool marked `UNCERTAIN`, which by definition means the evidence did not
  settle the question.

## Report

End with: courses checked and access time; counts by status per course and content type;
files created or modified; anything flagged for review; validation results including the
exact commands run; and the final `git status` for both repositories.
