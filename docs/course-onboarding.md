# Course Onboarding

Use this checklist from the repository root. Replace `course-code` with a short lowercase identifier used consistently in paths and frontmatter. Once a note receives an ID, keep that ID stable.

Before starting, gather the official unit outline or syllabus, assessment information, course-resource locations, and any lecture material you already have. Missing information should remain visibly unknown.

Real, unsanitized course material belongs under the Git-ignored `private/courses/` workspace, not tracked `courses/`. It defaults to `visibility: private`. Keep `source-risk: unknown` unless provenance is known, and use a course-derived risk such as `lecture-derived` where applicable. Do not classify material as open-licensed without explicit licence evidence. The tracked `courses/` tree is only for material already approved for public inclusion.

## 1. Create the Course Structure

```bash
mkdir -p private/courses/course-code/concepts
mkdir -p private/courses/course-code/lectures
mkdir -p private/courses/course-code/problem-sheets
mkdir -p private/courses/course-code/exam
```

Optionally add a short `private/courses/course-code/README.md` for human navigation and `private/courses/course-code/glossary.md` when terminology begins to accumulate.

## 2. Add `course.md`

```bash
cp templates/course.md private/courses/course-code/course.md
```

- Replace `course-code` in the path, `id`, `course`, and body.
- Record the real title, term, lecturer, assessment structure, exam date when known, and official resources.
- Leave unknown values explicit; do not ask an LLM to fill administrative facts from guesswork.
- Keep weak areas and priorities evidence-based. They can start as not yet recorded.

## 3. Add `syllabus.md`

```bash
cp templates/syllabus.md private/courses/course-code/syllabus.md
```

- Copy or faithfully summarize official learning outcomes.
- Map weekly and assessment topics only from authoritative material.
- Record exam coverage, exclusions, and lecturer hints with source context.
- Put uncertain claims under `Unclear or Unconfirmed` instead of silently resolving them.

## 4. Add Raw Lecture Notes

For each lecture:

```bash
cp templates/lecture.md private/courses/course-code/lectures/yyyy-mm-dd-topic.md
```

- Replace all placeholders and give the lecture a stable, unique ID.
- Preserve chronology, rough wording, notation, and source references.
- Use `status: reference` when the lecture is a source record rather than material to schedule directly.
- Mark incomplete or uncertain material under `Unclear or Verify`.
- Never rewrite the raw lecture file merely to make a concept note cleaner.

## 5. Extract Durable Concept Notes

Use `prompts/import-lecture.md` with the raw lecture path. By default, the LLM should propose an extraction plan and stop before creating files.

- Update an existing concept when `manifest.json` shows a genuine match.
- Create a new concept only for a substantial, reusable idea.
- Copy `TEMPLATE.md` for an approved new concept.
- Preserve the lecture path or precise slide/timestamp reference in `source`.
- Keep unclear definitions, examples, or relationships visibly marked for verification.

## 6. Add Problem-Sheet Notes

Put attempts and corrections under `private/courses/course-code/problem-sheets/` with `type: problem-sheet`.

- Preserve the original question reference and your attempted method.
- Separate the attempt, correction, and reflection.
- Record a structured Mistake Log entry only when an actual error occurred.
- Link to concept notes only after their IDs exist.

## 7. Add Exam Information

Put exam maps, confirmed format information, and revision summaries under `private/courses/course-code/exam/`.

- Map official outcomes or assessment topics to existing note IDs.
- Record the source and date for exam-format claims.
- Keep unconfirmed rumours or assumptions out of the exam map.

## 8. Validate

```bash
make study-validate
```

Fix missing metadata, duplicate or unstable IDs, broken links, malformed mistakes, and course-directory mismatches before continuing.

## 9. Rebuild the Study Views

```bash
make study-manifest
make study-review
make test
```

Or run the complete private-study workflow with `make study-all`, then `make test`.

To propose a public candidate later, sanitize a copy into `courses/course-code/`, assign an allowed public `visibility` and `source-risk`, and follow the [public release checklist](public-release-checklist.md). Do not put a real course's first or working copy in `courses/`.

After the first import, work through the [course import friction test](friction-test.md). If retrieval or updating feels slow, improve navigation and source references before adding more tooling.
