# Course Notes

Course Notes is a Markdown-first system for building durable study notes, practising active recall, and directing review toward the topics that need it most.

The repository supports public and private work in one working copy:

- tracked `courses/` is the public course-content surface;
- ignored `private/` holds unsanitized source material, personal study work, curated source packs, and private study modules;
- dependency-free Python tools validate notes and generate retrieval and review views.

No database, server, or specific editor is required.

> **Private by default**
>
> Raw, personal, unsanitized, or uncleared course-derived material belongs under the Git-ignored `private/` directory. A derivative may enter tracked `courses/` only after a separate sanitization, classification, and human review process.
>
> Automated checks reduce accidental disclosure; they do not establish copyright clearance, institutional permission, or legal authority to publish.

The framework code, templates, prompts, and documentation are licensed under the [MIT License](LICENSE). That licence does not extend to private or third-party course content added by a user.

## Quick start

```bash
git clone <repository-url>
cd course-notes
make all
```

`make all`:

1. validates tracked notes;
2. rebuilds `manifest.json`;
3. rebuilds `REVIEW_QUEUE.md`;
4. runs the test suite.

The repository includes a fictional demo course, so the public workflow can be tried without adding private material.

Useful starting points:

- [`INDEX.md`](INDEX.md) — human navigation for the demo course
- [`REVIEW_QUEUE.md`](REVIEW_QUEUE.md) — generated study priorities
- [`LLM_GUIDE.md`](LLM_GUIDE.md) — repository-aware LLM protocol
- [`docs/course-onboarding.md`](docs/course-onboarding.md) — adding a real course
- [`docs/operating-checklist.md`](docs/operating-checklist.md) — daily commands and release checks

Before committing or pushing public changes, run:

```bash
make public-safety
```

Make is optional. The underlying tools use Python 3 and the standard library.

## Core workflows

### Public notes

Tracked `courses/` contains synthetic, original, openly licensed, or separately sanitized notes intended for the public repository.

Public notes:

- follow the [`TEMPLATE.md`](TEMPLATE.md) note contract;
- use an allowed `visibility` and `source-risk` classification;
- pass normal validation;
- pass the stricter public-release checks before publication.

The included `courses/demo-course/` tree is fictional and demonstrates the public format. It is not an onboarding destination for real course material.

### Private notes and source material

Unsanitized notes, raw source material, personal study evidence, and material whose publication status is unclear belong under ignored `private/` paths.

Private Markdown notes that follow the normal note schema may mirror the tracked course layout and use:

```bash
make study-validate
make study-manifest
make study-review
make study-all
```

These commands are for schema-based private notes. They are not general validators for every artifact under `private/`.

Curated source packs, LMS imports, raw files, and other workflow-specific material may use different private structures. They remain ignored and should be handled by the workflow that created them.

### Study-layer modules

The study-layer workflow turns a curated private source pack into a bounded topic module containing:

```text
STUDY_GUIDE.md
QUESTION_BANK.md
ANSWER_KEY.md
COVERAGE_MATRIX.md
source-map.json
```

Several validated modules can then be connected through a course-level study index, revision plans, and a source-issue register.

The study-layer workflow normally remains private. Mechanical validation checks structure and provenance; separate subject-matter review checks reasoning, calculations, and teaching quality.

Start with:

- [Study-layer workflow](docs/study-layer-workflow.md)
- [Study-module contract](docs/study-module-contract.md)
- [Study-layer validation](docs/study-layer-validation.md)
- [Generate one study module](prompts/generate-study-module.md)
- [Generate a course-level study index](prompts/generate-course-study-index.md)

### Promoting material to the public tree

Keeping a file under `private/` is not sanitization.

A derivative may be promoted into tracked `courses/` only when it has been:

1. rewritten or transformed into an appropriate public artifact;
2. stripped of private, identifying, restricted, or institution-specific material;
3. assigned an allowed publication classification;
4. manually reviewed against the publication policy and release checklist;
5. validated by the public safety gate.

Required references:

- [docs/publication-policy.md](docs/publication-policy.md)
- [docs/public-release-checklist.md](docs/public-release-checklist.md)
- [Sanitize for public release prompt](prompts/sanitize-for-public-release.md)

## What the framework provides

- **Structured Markdown notes** with stable IDs, relationships, review metadata, practice questions, and mistake evidence.
- **Validation** for metadata, links, IDs, dates, note structure, publication classifications, and local-path portability.
- **Generated views** through `manifest.json` and `REVIEW_QUEUE.md`.
- **Review tooling** for updating review dates after active recall.
- **Manifest-first LLM use** so an executor can select relevant notes before opening full files.
- **Reusable prompts** for importing, explaining, quizzing, revising, sanitizing, and generating private study modules.
- **Publication-safety tooling** shared by manual commands, optional local hooks, and CI.
- **A dependency-free test suite** using Python’s standard library.

## Repository map

```text
course-notes/
├── courses/          tracked public course notes and the fictional demo course
├── private/          ignored local workspace for private notes and source material
├── docs/             operating, onboarding, publication, and study-layer guides
├── prompts/          reusable human and repository-executor workflows
├── templates/        course, syllabus, lecture, and note templates
├── scripts/          optional managed pre-commit and pre-push hooks
├── tests/            regression tests
├── handoffs/         dated historical implementation snapshots
├── TEMPLATE.md       canonical concept-note structure
├── LLM_GUIDE.md      retrieval and repository-editing protocol
├── INDEX.md          maintained human navigation
├── PROJECT_STATE.md  current operational state
├── Makefile          command shortcuts
├── manifest.json     generated retrieval index
└── REVIEW_QUEUE.md   generated study-priority view
```

The root-level Python tools implement validation, generation, review updates, and publication checks. Shared note and validation logic lives in `studylib.py`.

`manifest.json` and `REVIEW_QUEUE.md` are generated files. Change source Markdown and rebuild them rather than editing them by hand.

## Common commands

| Command | Purpose |
|---|---|
| `make validate` | Validate tracked notes |
| `make manifest` | Validate and rebuild `manifest.json` |
| `make review` | Validate and rebuild `REVIEW_QUEUE.md` |
| `make test` | Run the complete regression suite |
| `make all` | Validate, regenerate both public views, and run tests |
| `make validate-public` | Apply publication classifications and public-release checks |
| `make study-all` | Validate and regenerate views for schema-based private notes |
| `make reviewed NOTE=<id>` | Update review dates after studying a public note |
| `make study-reviewed NOTE=<id>` | Update review dates for a private schema-based note |
| `make public-safety` | Run the canonical publication-safety gate |
| `make install-hooks` | Install the managed pre-commit and pre-push hooks |
| `make uninstall-hooks` | Remove only Course Notes-owned hooks |
| `make pre-release` | Run the safety gate and regenerate public views |

For daily sequences, direct Python equivalents, reproducible dates, and hook behaviour, see the [Operating checklist](docs/operating-checklist.md).

## Note contract

Every study document under `courses/`, except navigational `README.md` files, uses constrained frontmatter.

| Field | Purpose |
|---|---|
| `id` | Permanent repository-unique identifier |
| `title` | Human title matching the document heading |
| `course` | Exact course directory name |
| `type` | Document type such as concept, lecture, problem sheet, exam map, glossary, or reference |
| `topic` | Stable topic slug |
| `aliases` | Alternative retrieval terms |
| `prerequisites` | IDs that should be understood first |
| `related` | Other relevant note IDs |
| `exam-weight` | `none`, `low`, `medium`, or `high` |
| `status` | Recall state: `new`, `learning`, `shaky`, `solid`, `mastered`, `reference`, or `archived` |
| `last-reviewed` | Date of the latest active-recall review |
| `review-after` | Earliest suggested next-review date |
| `source` | Honest provenance description |
| `visibility` | Private or allowed public classification |
| `source-risk` | Provenance-risk classification |

Inline lists use the form:

```yaml
related: [first-id, second-id]
```

`TEMPLATE.md` is the canonical concept-note structure. `LLM_GUIDE.md` explains how executors should interpret and edit it.

## Review model

Status describes recall and application, not how polished a note looks.

A review should involve active recall or problem solving before rereading. After reviewing:

1. record genuine mistakes as evidence;
2. update `last-reviewed` and `review-after`;
3. propose status changes separately;
4. regenerate the manifest and review queue.

Use:

```bash
make reviewed NOTE=<id>
```

or, for a reproducible date:

```bash
make reviewed NOTE=<id> DATE=YYYY-MM-DD
```

A status upgrade should normally be supported by repeated evidence separated by time or task type. Confidence, one correct response, or improved prose alone is not sufficient.

Concept notes include:

- a worked example where appropriate;
- practice questions at easy, medium, and exam-style levels;
- a structured Mistake Log for errors actually observed during study.

The generated review queue combines status, exam weight, review timing, review age, recent mistakes, and missing practice into a prioritization score. The score directs attention; it is not a grade or an automatic status decision.

## Manifest-first LLM use

An executor should normally:

1. read `manifest.json`;
2. filter by course, topic, type, status, alias, or exam weight;
3. open only selected source Markdown files;
4. follow prerequisite or related IDs only when necessary;
5. treat source Markdown as authoritative and generated views as disposable indexes.

See [`LLM_GUIDE.md`](LLM_GUIDE.md) and the [Prompt index](prompts/README.md).

## Publication safety

`make public-safety` is the canonical local gate. It:

1. rejects tracked files under `private/`;
2. rejects staged changes under `private/`;
3. scans relevant working-tree and staged public text for configured leak patterns;
4. runs `validate_notes.py --public-release`;
5. runs the complete test suite.

The repository also provides optional managed hooks:

- `pre-commit` runs the gate before a commit;
- `pre-push` runs the gate before a push.

Install or remove them with:

```bash
make install-hooks
make uninstall-hooks
```

Installation refuses to overwrite unrelated hooks. Removal deletes only byte-identical hooks owned by this project.

Both hooks are local, optional, and bypassable:

```bash
git commit --no-verify
git push --no-verify
```

CI runs the same canonical gate after a push or on a pull request and separately checks that generated public files are current. A green local result is not a claim that remote CI has passed; CI is post-push ratification.

The checks are deliberately conservative but still limited. They do not understand copyright, confidentiality, institutional policy, or every possible form of sensitive content. Human review remains the publication authority.

An optional local `.public-release-blocklist` can add repository-specific smoke-test terms. It is ignored by Git and should not be committed.

## Documentation by task

| Goal | Start here |
|---|---|
| Understand the repository layout | [Repository layout](docs/repository-layout.md) |
| Add a real course | [Course onboarding](docs/course-onboarding.md) |
| Run daily and pre-release commands | [Operating checklist](docs/operating-checklist.md) |
| Decide what may be published | [Publication policy](docs/publication-policy.md) |
| Review a release candidate | [Public-release checklist](docs/public-release-checklist.md) |
| Generate a private study module | [Study-layer workflow](docs/study-layer-workflow.md) |
| Understand the module files and stable IDs | [Study-module contract](docs/study-module-contract.md) |
| Validate generated study material | [Study-layer validation](docs/study-layer-validation.md) |
| Give an executor repository instructions | [LLM guide](LLM_GUIDE.md) |
| Browse reusable prompts | [Prompt index](prompts/README.md) |
| Check the current operational state | [Project state](PROJECT_STATE.md) |
| Review historical implementation context | [Handoffs](handoffs/) |

## Development

Keep changes scoped and review the actual diff before committing.

For normal public changes:

```bash
make all
make public-safety
git diff --check
git status --short
```

Automated checks are necessary, not sufficient. Contributors remain responsible for verifying that a change is correct, appropriately sourced, and suitable for the public repository.

## Scope and limitations

Course Notes is not:

- a full LMS mirror;
- an automatic importer for every source system;
- an automatic publishing or clearance system;
- a backup system for ignored private files;
- proof that course-derived material may be redistributed;
- a guarantee that generated explanations, calculations, or study questions are correct;
- a replacement for human review.

It is a practical framework for keeping study knowledge inspectable, reviewable, source-aware, and separated from material that should remain private.
