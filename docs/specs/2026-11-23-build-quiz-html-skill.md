# Build quiz HTML skill

**Status:** approved
**Base commit:** `0243b3a1ab862dfd30bda3307b1f7ef30f8c71fc`
**Repo:** `d:\SMU\quizzer\elearn-html-quiz` (plugin `quizzer`)
**Context file to update:** none named

**Commands**
- Full test suite: none
- Single test file: none
- Typecheck: none
- Lint: none

No test files are added (user: "verfyers can just double check, don't need test cases"). Acceptance criteria are one-off commands and reads the verifier runs.

Repo docs: only `README.md` (one line). No CLAUDE.md, ADRs or glossary in the repo. Workspace rule in `d:\SMU\quizzer\CLAUDE.md` (outside repo): skills must name created files without spaces. Existing skills `format-quiz-docs`, `generate-quiz-docs` and `skills/format-quiz-docs/quiz-rules.md` are the de facto conventions. Reference implementation (hand-built, not in repo): `d:\SMU\quizzer\example\Finals Practice Quiz 2 (Short).html`.

## Dependencies and blockers

| Subtask | Blocked by |
|---|---|
| S1 | none |
| S2 | S1 |
| S3 | S2 |
| S4 | S3 |

Run order: S1, S2, S3, S4.

External blockers: none.

## Branch plan

Starting branch: `main`

| Order | Branch | Based on | Subtasks |
|---|---|---|---|
| 1 | `feat/build-quiz-html` | `main` | S1, S2, S3, S4 |

## Out of scope

- Photos/images in questions (user: "photos is out of scope. i added to TODO"; tracked in `docs/specs/TODO.md`). `transient`.
- Automated test files (user). `transient`.
- Answer key, auto-marking or instant feedback inside the HTML (user confirmed). Recorded by S4 in README.
- Student name field (user: "nope they'll save in their own laptop"). `transient`.
- Firefox/Safari autosave (no browser API). Recorded by S4 in README.
- Hosting the quiz online. Recorded by S4 in README.
- Any network request from the page: no CDN, web fonts, analytics, remote images or remote save. Recorded by S4 in README.

## Subtasks

### S1 — Extract question format reference; no-space filenames in existing skills

- **Goal:** The Questions.md format lives in one reference file used by all three skills. Existing skills name output files without spaces.
- **Blocked by:** none
- **Branch:** `feat/build-quiz-html`
- **Order on branch:** 1
- **Files:** `skills/format-quiz-docs/questions-format.md` (new), `skills/format-quiz-docs/quiz-rules.md`, `skills/format-quiz-docs/SKILL.md`, `skills/generate-quiz-docs/SKILL.md`
- **Seams:** the skill files' text.
- **Decisions:**
  - Extract the question format into a reference file instead of inside the skills (user: "should i extract the question format as a reference? instead of inside the skills?", agreed).
  - The format covers every quiz the format and generate skills produce (user: "the format is for quizes in general right? those supported by generated or formatted quiz?").
  - Format: `# <title> - Questions` heading; optional intro paragraph; per question `**Question N** (P points)` with `(P points)` optional and `(Select all that apply)` marking multi-select; stem in markdown (paragraphs, bold, italic, inline code, code blocks, lists, tables); options `- a. text` to `- e. text`; true/false = single choice with `- a. True` / `- b. False`; written = no options, optional `**Word limit**: Maximum N words.`, default 250.
  - "no spaces are used for file names, use '-'" (user). Rule: spaces become `-`; drop characters other than letters, digits, `.`, `_`, `-`; collapse repeated `-`; trim `-` at ends; keep case. `Finals Practice Quiz 2 (Short)` gives `Finals-Practice-Quiz-2-Short-Questions.md` and `Finals-Practice-Quiz-2-Short-Answers.md`. Applies to `.md` and `.docx`.
- **Acceptance criteria:**
  - [x] `grep -l questions-format.md skills/format-quiz-docs/quiz-rules.md skills/format-quiz-docs/SKILL.md skills/generate-quiz-docs/SKILL.md` lists all three files.
  - [x] `grep -rn "<topic> - Questions\|<topic> - Answers" skills/` prints nothing.
  - [x] `quiz-rules.md` states the no-space rule with the `Finals-Practice-Quiz-2-Short-Questions.md` example (read).
  - [x] `questions-format.md` covers every item in the Format decision above (read).
- **Doc deltas:** `questions-format.md` is the doc. README: filename rule.
  - `questions-format.md` adds rules the Format decision left open: exactly one `#` heading; `(1 point)` singular; options are the last block of a question, one line each, letters from `a` with no gaps, at least 2; a stem bullet must not start with `a. `–`e. ` so it is never read as an option; `&nbsp;` lines as answer space in `.md` (matches the example). Why: `/build-quiz-html` parses this file at runtime, so the parse must be unambiguous.
  - `quiz-rules.md` now says true/false questions get options `a. True` / `b. False` (was "mark as such"). Why: matches the Format decision.

### S2 — Single-file template

- **Goal:** `skills/build-quiz-html/quiz-template.html` is one self-contained file, generalized from the example. It has exactly one placeholder, `{{QUESTIONS}}`, inside `<script type="text/markdown" id="qsrc">`, which takes a whole Questions.md. It keeps the example's UI: nav grid, flagging, timer, progress bar, dark mode, word counter, Export, Copy Markdown, Reset.
- **Blocked by:** S1
- **Branch:** `feat/build-quiz-html`
- **Order on branch:** 2
- **Files:** `skills/build-quiz-html/quiz-template.html`
- **Seams:** pure functions in the template's `<script id="quiz-core">` block: `parseBank(md)`, `renderMarkdown(md)`, `safeName(title)`, `buildAttempt(quiz, state)`. The verifier loads them with `node:vm`.
- **Decisions:**
  - Template is one file, same as the output (user: "shouldn't the actual template just be one file since the actual thing is also one file?").
  - "i don't want to create from scratch each time, i want a format it can use" (user).
  - The page parses the pasted Questions.md at runtime; no per-question placeholders.
  - Title, storage key and filenames come from the `# <title> - Questions` heading at runtime. Storage key from `safeName(title)` so quizzes never share saved state (code: example uses one fixed `STORE` constant; all `file://` pages share one localStorage in Chrome).
  - Storage: `localStorage`; answers survive closing the tab and browser until Reset or clearing site data (user: "ok").
  - Autosave (user: "by default autosave to ./<name>-attempt.md and overwrite for autosave"; mechanics "ok"): Chrome/Edge, on first answer click a Save dialog opens prefilled `<safeName(title)>-attempt.md`; each change after that overwrites the whole file; handle kept in IndexedDB, one permission click to resume after reload. Firefox/Safari: visible notice "Autosave needs Chrome or Edge"; Export downloads the `.md`.
  - Export button filename uses `safeName`, no spaces.
  - Attempt `.md` content keeps the example's layout (header, marker instructions, every question with options and the student's answer); course-specific marker wording comes from the quiz intro, not hard-coded CS205 text.
  - `</script` inside question text must not end the script block: the skill writes it as `<\/script` (S3), and the page reads it back as `</script`.
- **Acceptance criteria:**
  - [x] `grep -c "{{QUESTIONS}}" skills/build-quiz-html/quiz-template.html` prints `1`; `grep -o "{{[A-Z_]*}}"` finds no other placeholder.
  - [x] `grep -nE "(src|href)=\"(https?:)?//|url\((https?:)?//|fetch\(|import\(" skills/build-quiz-html/quiz-template.html` prints nothing.
  - [x] `grep -n "CS205" skills/build-quiz-html/quiz-template.html` prints nothing.
  - [x] A `node -e` script that extracts `#quiz-core`, runs it in `node:vm`, and parses the body of `d:\SMU\quizzer\example\Finals Practice Quiz 2 (Short) - Questions.md` reports 17 questions, Q3 type `multi`, Q16 type `written`; `safeName('Finals Practice Quiz 2 (Short)')` returns `Finals-Practice-Quiz-2-Short`; `renderMarkdown('<img src=x onerror=1>')` output has no `<img`.
  - [x] Template calls `showSaveFilePicker` with `suggestedName` from `safeName(...) + '-attempt.md'` and writes with `createWritable()`, `write()`, `close()` (read).
- **Doc deltas:** README: storage and autosave per browser; "page makes no network requests".
  - `parseBank(md)` returns `{ title, intro, questions }`; each question is `{ n, points, type, stem, opts, wordLimit }` with `type` one of `single`, `multi`, `written` and `points` `null` when the header has none. Why: one object feeds the page, the attempt file and the verifier.
  - `safeName` keeps Unicode letters and digits (`\p{L}\p{N}`), turns any whitespace into `-`, and falls back to `quiz` when nothing is left. Why: the S1 rule says "letters" without limiting to ASCII, and an empty name would give `-attempt.md`.
  - Storage key `quizzer:<safeName>`, IndexedDB database `quizzer-autosave:<safeName>`. Why: per-quiz state, as the storage-key decision requires.
  - `&nbsp;` lines are dropped from intro and stems; `**Word limit**` is read anywhere in a written question outside code fences; option lines inside code fences stay in the stem. Why: `questions-format.md` puts `&nbsp;` answer space after the word limit.
  - Export filename is `<safeName>-attempt-YYYY-MM-DD-HHMM.md`. Why: each export is a separate download and must not clash with the autosave file `<safeName>-attempt.md`.
  - Autosave: if the student cancels the Save dialog, it is not shown again on that page visit; "Set autosave file" stays available. After a reload, the first answer (or "Resume autosave") asks for permission. Firefox/Safari hide the autosave button and show "Autosave needs Chrome or Edge". Why: no repeated dialogs while answering.
  - The quiz intro is shown above the questions and copied into the attempt under "Quiz instructions"; marker steps are listed only for question types present; total marks and a per-range breakdown are computed when every question has points. Why: replaces the example's hard-coded CS205 marker text.
  - The written-answer code fence is longer than any backtick run in the answer (min 4). Why: an answer containing ```` would otherwise break the attempt file.
  - `renderMarkdown` also handles `#` headings, `*` bullets and `1)` lists, and escapes all HTML (including `'`). An empty or unparsable bank shows "No questions found". Why: robustness for converted input.
  - Known limit, needs no doc: a bank containing `<!--` followed by `<script` could still confuse the HTML parser; only `</script` is escaped, as the spec decides.

### S3 — `/build-quiz-html` skill

- **Goal:** `/build-quiz-html <questions.md | screenshots | text> [output folder]` produces `<safeName(title)>.html` from the template.
- **Blocked by:** S2
- **Branch:** `feat/build-quiz-html`
- **Order on branch:** 3
- **Files:** `skills/build-quiz-html/SKILL.md`
- **Seams:** the skill file's text.
- **Decisions:**
  - It is a Claude skill with no build script (user: Q9 option A, "yes"). Steps: copy `quiz-template.html` to the output path with a shell copy; replace `{{QUESTIONS}}` with the Questions.md text using the Edit tool; never rewrite the template by hand.
  - Before pasting, replace every `</script` in the question text with `<\/script`.
  - Input not in `questions-format.md` format (screenshots, loose text) is first converted to that format and written as `<safeName>-Questions.md`, then filled in. No answers in the HTML.
  - No file given: newest `*-Questions.md` in the working folder, else ask.
  - `disable-model-invocation: true`, same as the other skills (code: `skills/*/SKILL.md`).
  - Created filenames have no spaces (docs: `d:\SMU\quizzer\CLAUDE.md`).
- **Acceptance criteria:**
  - [ ] `SKILL.md` frontmatter has `name: build-quiz-html` and `disable-model-invocation: true`; body references `../format-quiz-docs/questions-format.md` and `quiz-template.html` (`grep`).
  - [ ] Body states the copy-then-Edit steps, the `</script` rule and the no-space filename rule (read).
- **Doc deltas:** README: skill usage.

### S4 — README and version

- **Goal:** README documents the three skills, the format reference, filename rule, storage/autosave, and sharing.
- **Blocked by:** S3
- **Branch:** `feat/build-quiz-html`
- **Order on branch:** 4
- **Files:** `README.md`, `.claude-plugin/plugin.json` (version `0.2.0`)
- **Seams:** none.
- **Decisions:**
  - Telegram IP warning (user: "they open the file from tele, then it showed before it opened"): Telegram Desktop shows this for every `.html` file before opening, whatever its content. The page makes no network requests, so opening it does not reveal the IP. README says: safe to proceed; tick "don't ask again"; or send the file zipped.
  - Non-transient out-of-scope items above.
- **Acceptance criteria:**
  - [ ] `grep '"version": "0.2.0"' .claude-plugin/plugin.json` matches.
  - [ ] README covers all three skills, `questions-format.md`, the filename rule, storage/autosave per browser, no network requests, the Telegram note, and the out-of-scope items (read).
- **Doc deltas:** this subtask is the doc delta.

## Human checks

- [ ] S2: Fill the template with the example Questions.md, double-click in Chrome. Expected: looks and behaves like `example/Finals Practice Quiz 2 (Short).html`.
- [ ] S2: DevTools Network tab, reload. Expected: zero requests besides the file.
- [ ] S2: Chrome/Edge: answer a question. Expected: Save dialog prefilled `Finals-Practice-Quiz-2-Short-attempt.md`; each change overwrites it; reload restores answers, autosave resumes after one click.
- [ ] S2: Firefox: answer, close browser, reopen file. Expected: answers restored; autosave notice; Export downloads a no-space `.md`.
- [ ] S3: `/build-quiz-html` on the example Questions.md and on a screenshot. Expected: no-space `.html` (and `-Questions.md` for the screenshot) reported, opens correctly.
- [ ] S4: Send the `.html` via Telegram. Expected: Telegram's own warning still shows (cannot be removed); after proceeding, Network tab shows zero requests.
