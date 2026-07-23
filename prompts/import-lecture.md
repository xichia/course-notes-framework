# Import a Lecture

## Prompt

Process the rough lecture note at **[relative lecture path]** for **[course code]**. Work in proposal mode unless I explicitly tell you to create or edit files.

Read `manifest.json` first, then the course's `course.md`, `syllabus.md`, the raw lecture note, and only the existing concept notes needed for comparison. Treat the raw lecture as source evidence and leave it intact.

## Rules

- Preserve the lecture's original meaning, uncertainty, notation, and source references.
- Do not invent missing definitions, derivations, examples, learning outcomes, assessment coverage, or lecturer claims.
- Mark unclear material as `[TODO: verify from lecture/source]` and say what is missing.
- Keep small examples, announcements, and one-off details in the lecture note. Propose a concept note only when the idea is substantial and reusable.
- Match against existing notes by ID, title, aliases, topic, and meaning before proposing a new note.
- Preserve every existing filename and ID. For a proposed new note, suggest a stable `<course-code>-<concept-slug>` ID and check that it is unused.
- Add `prerequisites`, `related`, and Markdown links only to notes that already exist. If a desired target does not exist, list it as a proposed dependency instead of creating a broken link.
- Preserve provenance in `source` using a repository-relative lecture path plus any available slide, page, timestamp, or board-reference detail.
- Preserve `visibility: private` and the existing `source-risk`; lecture extraction does not make the result public or original.
- Do not change note status; lecture import is not evidence of recall.

## If the Source Is a Recording Transcript

See `docs/lecture-transcripts.md` for the full contract. In addition to the rules above:

- Treat the transcript as source evidence. Leave the raw file byte-for-byte intact, and store it in an ignored source-material directory rather than beside the notes.
- Normalize it to a timestamp-anchored `.txt` before reading it for content. Do not write a normalized transcript as `.md`; every `.md` under the courses directory except `README.md` is validated as a note and will fail.
- Cite stable timestamp anchors. Never bulk-copy the transcript into a note, and never quote a passage where a citation would do.
- Prefer the written course materials for code, commands, filenames, and numeric values. Automatic speech recognition renders these unreliably, so a transcript is evidence of *which* example was shown, not of how it is spelled. Say so in the note when you rely on the written material instead.
- Distinguish what the lecture stated from your inference and from study heuristics, and label each claim accordingly.
- Record acquisition, transformation, retained timestamps and speaker labels, the authoritative file to cite, and known transcript defects in a provenance record beside the transcript.
- Pass speaker labels through unchanged. Never resolve or infer a name the export did not contain.
- Where a transcript claim can be checked against material already committed to the repository, check it and report agreement or conflict explicitly.

## First Response: Proposal Only

Return:

1. a brief inventory of the lecture's substantial concepts;
2. a table classifying each as `update existing`, `propose new concept`, `keep in lecture`, or `unclear`;
3. existing note IDs that would be touched;
4. proposed filenames and IDs for new concepts;
5. missing information that needs user verification;
6. the exact source reference each concept would retain.

Stop and ask for approval before creating new concept notes or editing existing ones, unless I explicitly requested implementation.

## If Creation Is Approved

Use `TEMPLATE.md`, write only supported material, retain visible TODOs, and add links only after their target files exist. Leave the raw lecture unchanged. Run `make all` and report validation results plus every file created or updated.

Do not create public-facing concept notes from the lecture unless I explicitly request a separate sanitized candidate. Sanitization must remove protected lecture expression and course-specific examples; prefer synthetic replacements and never upgrade visibility without evidence and instruction.
