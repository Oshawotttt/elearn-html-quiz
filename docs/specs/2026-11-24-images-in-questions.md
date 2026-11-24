# Images in questions

**Status:** approved
**Base commit:** `99d7bf1e78a4c227cd1192ec50257fcac0c0681f`
**Repo:** `d:\SMU\quizzer\elearn-html-quiz` (plugin `create-quiz`)
**Context file to update:** none named

**Commands**
- Full test suite: none
- Single test file: none
- Typecheck: none
- Lint: none

No test files in the repo (previous spec, user: "verfyers can just double check, don't need test cases"). Acceptance criteria are one-off `node -e` / `grep` commands and reads the verifier runs. `node` v24 and Pillow 12.3 are installed.

Repo docs: `README.md`, `docs/specs/TODO.md` (item "Add support for photos"), prior spec `docs/specs/2026-11-23-build-quiz-html-skill.md`. No CLAUDE.md, ADRs or glossary in the repo. Workspace rules in `d:\SMU\quizzer\CLAUDE.md` (outside repo): skills name created files without spaces; skills are tested by a fresh subagent that reads the `SKILL.md`.

## Dependencies and blockers

| Subtask | Blocked by |
|---|---|
| S1 | none |
| S2 | S1 |
| S3 | S2 |
| S4 | S1 |
| S5 | S3, S4 |
| S6 | S5 |

Run order: S1, S2, S3, S4, S5, S6.

External blockers: none.

## Branch plan

Starting branch: `main`

| Order | Branch | Based on | Subtasks |
|---|---|---|---|
| 1 | `feat/question-images` | `main` | S1, S2, S3, S4, S5, S6 |

## Out of scope

- Embedding images inside the `.html` (base64). The page references image files; the `.html` and its `images/` folder ship together in one folder (user: "just reference"). Recorded by S5 in README.
- Generating new diagrams or pictures in `/generate-quiz-docs`. It only reuses existing images. Recorded by S4 in `generate-quiz-docs/SKILL.md`.
- Remote images (`http://`, `https://`, `//`). The page still makes no network requests. Recorded by S1 in `questions-format.md` and S5 in README.
- Zipping the output. The user zips the folder. `transient`.
- Automated test files. S6 adds a visual test only, for a human to look at. `transient`.

## Interfaces

**Image line in a Questions.md** (S1 defines, S2 renders, S3 copies, S4 writes):

```markdown
![<alt text>](images/<file>)
```

- `<file>` is `<name>-Q<N>.<ext>` or `<name>-Q<N>-<k>.<ext>` (k = 2, 3, ...), where `<name>` is the safe quiz name.
- The path always starts with `images/`, has no subfolders, no spaces, no `..`, no backslash, no scheme. It is relative to the doc (`.md`) or the `.html`.
- `<ext>`: `png`, `jpg`, `jpeg`, `gif`, `webp`.
- The page accepts exactly the paths matching `^images/[A-Za-z0-9._-]+\.(png|jpe?g|gif|webp)$` (case-insensitive) and renders any other `![...](...)` as plain text.

**HTML output layout** (S3 produces, S2 relies on):

- Quiz with no images: `<output folder>/<name>.html`, as before.
- Quiz with images: `<output folder>/<name>/<name>.html` and `<output folder>/<name>/images/<file>` for every image the quiz references. The `<name>/` folder is the thing the user zips and sends.

## Subtasks

### S1 — Image rules in the format and quiz rules

- **Goal:** `questions-format.md` defines the image line and where it may appear. `quiz-rules.md` defines the `images/` folder, image file names, cropping, and how images reach `.md` and `.docx` output.
- **Blocked by:** none
- **Branch:** `feat/question-images`
- **Order on branch:** 1
- **Files:** `skills/format-quiz-docs/questions-format.md`, `skills/format-quiz-docs/quiz-rules.md`
- **Seams:** the reference files' text.
- **Decisions:**
  - md: "just screenshot and name the photos, then add in a reference folder and have the md refernce it" (user).
  - word: "ss and paste it inside" (user). Claude inserts the images from the `images/` folder into the `.docx` (docx skill), at the same spot as the Markdown image line; the user does not paste by hand (user: "ok to assumed").
  - Image files follow the no-space name rule (docs: `d:\SMU\quizzer\CLAUDE.md`).
  - Folder and names: `images/` next to the docs; `images/<name>-Q<N>.png`, second image for the same question `images/<name>-Q<N>-2.png` (user: "ok to assumed"). Full rule in **Interfaces**.
  - Cropping is in scope: "i don't want the whole picture even maybe with the answer in the question. i just want the graph or wtv relevant" (user). Claude crops each figure out of the source screenshot itself:
    1. Get the screenshot's pixel size with Pillow (`python -c "from PIL import Image; print(Image.open(r'<src>').size)"`).
    2. View the screenshot, pick the figure's box as fractions of width and height, add a small margin.
    3. Crop with an inline Pillow command (`Image.open(src).crop((l*w, t*h, r*w, b*h)).save(out)`); no script file.
    4. View the crop. It must hold the whole figure and none of: question text, option text, ticks, crosses, highlights, scores, "Correct answer" feedback or anything else that gives the answer away. Otherwise adjust the box and crop again.
    5. After 3 failed tries: ask the user for that image, naming the question number.
  - The user may also supply ready images (`Q<N>.png`, `Q<N>-2.png`); the skill copies them to the names above instead of cropping (user: "ok to assumed").
  - Image line placement: in the stem on its own line, after the stem text and before the options; also inside option text, `- a. ![alt](images/x.png)` (user: "ok to assumed").
  - No remote images (docs: README "No network requests").
- **Acceptance criteria:**
  - [x] `questions-format.md` has an **Images** section covering: line format, the path rule from **Interfaces**, allowed extensions, placement in stem, option images, no remote URLs, Answers doc repeats the same image lines (read).
  - [x] `quiz-rules.md` has an **Images** section covering: `images/` folder next to the docs, naming, the 5 crop steps with the must-not-contain list, user-supplied images, `.docx` insertion (read).
  - [x] `grep -n "PIL" skills/format-quiz-docs/quiz-rules.md` matches.
  - [x] `grep -n "images/" skills/format-quiz-docs/questions-format.md skills/format-quiz-docs/quiz-rules.md` matches in both files.
- **Doc deltas:** these files are the doc.
  - Crops are saved as `.png`; user-supplied images keep their extension. Why: Pillow writes PNG losslessly; no doc beyond `quiz-rules.md`.
  - Images in one question are numbered stem first, then options a, b, c. Why: gives a fixed order for `-2`, `-3` suffixes; recorded in `quiz-rules.md`.
  - Image lines may not go in the title or question header. Why: the header is one parsed line; recorded in `questions-format.md`.

### S2 — Template renders images

- **Goal:** `quiz-template.html` renders image lines in intro, stems and options as `<img>` with the relative path. Anything else in `![...](...)` form stays plain text.
- **Blocked by:** S1
- **Branch:** `feat/question-images`
- **Order on branch:** 2
- **Files:** `skills/build-quiz-html/quiz-template.html`
- **Seams:** `#quiz-core` functions loaded with `node:vm`: `renderMarkdown(md)`, `renderInline(s)`, `parseBank(md)`, `buildAttempt(quiz, state)`; grep on the template.
- **Decisions:**
  - "just reference" (user): `src` is the path as written. No embedding, no second placeholder; `{{QUESTIONS}}` stays the only placeholder.
  - Only paths matching the **Interfaces** regex become `<img>`; all others render as escaped plain text.
  - Alt text and path are HTML-escaped. No other HTML is let through (existing rule: escapes all HTML first).
  - Images scale to fit: `max-width:100%; height:auto`.
  - A broken image (file missing) shows its alt text and path, so the student sees something is missing.
  - The attempt file keeps the original `![alt](path)` text.
  - The page still makes no network requests.
- **Acceptance criteria:**
  - [x] `grep -o "{{[A-Z_]*}}" skills/build-quiz-html/quiz-template.html` prints only `{{QUESTIONS}}`, once.
  - [x] `node -e` vm script: `renderMarkdown('![Graph](images/a.png)')` contains `<img` with `src="images/a.png"` and `alt="Graph"`.
  - [x] Same script: each of `![x](https://e.com/a.png)`, `![x](//e.com/a.png)`, `![x](../a.png)`, `![x](images/a.svg)`, `![x](javascript:alert(1))` renders with no `<img`.
  - [x] Same script: `renderMarkdown('![x" onerror="1](images/a.png)')` output has no `onerror=` attribute (the `"` is escaped).
  - [x] Same script: `parseBank` on a bank with `- a. ![A](images/q1-a.png)` keeps it as option `a`; `renderInline` of its text gives an `<img`.
  - [x] Same script: `buildAttempt` output for a stem with an image line contains `![` and `images/`.
  - [x] Existing checks from the previous spec still pass: 17 questions from the example Questions.md, Q3 `multi`, Q16 `written`; `renderMarkdown('<img src=x onerror=1>')` has no `<img`; no `(src|href)="(https?:)?//`, `fetch(` or `import(` in the template.
- **Doc deltas:** README: page shows images from the `images/` folder beside it; still no network requests. `questions-format.md` covers the syntax (S1).
  - Broken images are swapped for an "Image missing: <alt> (<path>)" note by one capturing `error` listener on `document`, not an `onerror` attribute. Why: keeps every `<img>` free of inline handlers; README line covers the behaviour.
  - Bold/italic are applied to the text between images only, never across an image. Why: keeps `_` and `*` in alt text and file names from turning into `<em>` inside attributes; no doc needed.
  - Images get a white background behind them. Why: transparent PNG figures stay readable in dark mode; no doc needed.

### S3 — `/build-quiz-html` outputs a folder when the quiz has images

- **Goal:** For a quiz with images, `/build-quiz-html` writes `<output folder>/<name>/` holding `<name>.html` and `images/` with every referenced image, ready to zip. For a quiz without images, output is unchanged.
- **Blocked by:** S2
- **Branch:** `feat/question-images`
- **Order on branch:** 3
- **Files:** `skills/build-quiz-html/SKILL.md`
- **Seams:** the skill file's text.
- **Decisions:**
  - "output should be in a folder if got images, then easy for me to zip" (user).
  - Layout per **Interfaces**. Images are copied (shell copy) from the `images/` folder next to the Questions.md; the Questions.md keeps its images where they are.
  - Only images the Questions.md references are copied.
  - Screenshot input: crop images per the S1 rules into `<output folder>/images/` and write `<output folder>/<name>-Questions.md` before building. The `-Questions.md` and its `images/` stay outside the `<name>/` folder.
  - Missing image file: stop before writing the `.html`, and tell the user each question number and path.
  - Folder `<name>/` already exists: overwrite files with the same names, leave other files.
  - Step 6 check adds: every image path in the `.html` exists under `<name>/images/`.
  - Report: the folder path and that it is the thing to zip and send.
- **Acceptance criteria:**
  - [x] `SKILL.md` describes both layouts, the copy step, the missing-image stop, the overwrite rule and the extra check (read).
  - [x] `grep -n "images/" skills/build-quiz-html/SKILL.md` matches.
- **Doc deltas:** README: build-quiz-html output layout with images; zip the `<name>/` folder to share.
  - Image paths are relative to the folder of the Questions.md, and copied files keep the name and case written in the image line. Why: the page loads the path as written, and some hosts are case-sensitive; recorded in `build-quiz-html/SKILL.md`.
  - The skill description now says the output is a folder when the quiz has images. Why: the old text said "single self-contained" page; no other doc.

### S4 — Format and generate skills handle images

- **Goal:** `/format-quiz-docs` and `/generate-quiz-docs` put images in both docs, in `.md` and `.docx`.
- **Blocked by:** S1
- **Branch:** `feat/question-images`
- **Order on branch:** 4
- **Files:** `skills/format-quiz-docs/SKILL.md`, `skills/generate-quiz-docs/SKILL.md`
- **Seams:** the skill files' text.
- **Decisions:**
  - Follow the S1 **Images** rules in `quiz-rules.md`.
  - `/format-quiz-docs`: when a source question has a figure (graph, diagram, table image, code image), use the image the user supplied for that question, else crop it from the screenshot per S1. Ask the user only for figures it could not crop, listing each question number.
  - `/generate-quiz-docs`: may reuse an image from an example quiz or course content when a new question needs it, copied to `images/` under the new quiz's name. Never invent or draw images.
  - Answers doc repeats each image of the Questions doc.
- **Acceptance criteria:**
  - [x] Both `SKILL.md` files reference the **Images** section of `quiz-rules.md` (`grep -n "Images" skills/format-quiz-docs/SKILL.md skills/generate-quiz-docs/SKILL.md`).
  - [x] `format-quiz-docs/SKILL.md` says to crop figures from screenshots and ask only for ones it could not crop, by question number (read).
  - [x] `generate-quiz-docs/SKILL.md` says reuse only, never create images (read).
- **Doc deltas:** README: how to supply images; Claude crops figures from screenshots.
  - Answers doc marks a correct image option as `- **a.** ![..](images/..) ✔`. Why: "bold plus ✔" had no form for an option that is only an image; recorded in both `SKILL.md` files.
  - `/generate-quiz-docs` writes a different question when one would need an image that does not exist. Why: follows from reuse-only; recorded in `generate-quiz-docs/SKILL.md`.
  - Both skills report the `images/` folder and image count; `/format-quiz-docs` also reports the questions whose image it had to ask for. Why: the user must know which files to keep with the docs; no other doc.

### S5 — README, version, TODO

- **Goal:** README documents images; version bumped; TODO item done.
- **Blocked by:** S3, S4
- **Branch:** `feat/question-images`
- **Order on branch:** 5
- **Files:** `README.md`, `.claude-plugin/plugin.json`, `docs/specs/TODO.md`
- **Seams:** none.
- **Decisions:** none beyond the doc deltas of S1–S4.
- **Acceptance criteria:**
  - [ ] `grep '"version": "0.3.0"' .claude-plugin/plugin.json` matches.
  - [ ] `grep -n "Images in questions" README.md` finds no line under **Out of scope**; README has an images section covering cropping, user-supplied images, the `images/` folder, naming, the HTML folder layout and zipping, and the out-of-scope items above (read).
  - [ ] README no longer says the page is always one self-contained file without qualifying it for quizzes with images (read).
  - [ ] `docs/specs/TODO.md` has `- [x] Add support for photos` (`grep`).
- **Doc deltas:** this subtask is the doc delta.

### S6 — Visual test with an example HTML

- **Goal:** `tests/visual/` holds fixture screenshots and, at the end of the run, an example quiz folder built from them by the skills, for a human to look at. The folder is gitignored.
- **Blocked by:** S5
- **Branch:** `feat/question-images`
- **Order on branch:** 6
- **Files:** `.gitignore` (new), `tests/visual/` (ignored: `make-fixture.py`, `fixture/`, `out/`)
- **Seams:** the skills as a user runs them, through a fresh subagent.
- **Decisions:**
  - "create a test file for visual test only. for this, at the end create an example html inside. gitigonore pls" (user).
  - `.gitignore` has the line `tests/visual/`. Nothing under it is committed.
  - Fixture: `tests/visual/make-fixture.py` draws, with Pillow, `fixture/Visual-Test-Q1.png`: an eLearn-style review screenshot with the question text, a line graph, options with one ticked box and a tick and cross on the left, and a "Correct answer" line. Plus `fixture/Visual-Test-Q2.png` with a multi-select question whose options a and b are small bar charts. Quiz title in the screenshots: `Visual Test`.
  - The skills are run by a fresh subagent that reads each `SKILL.md` and follows its steps, since `disable-model-invocation: true` blocks invoking them (docs: `d:\SMU\quizzer\CLAUDE.md`, **Testing skills**). One subagent per skill run.
  - Run 1: `format-quiz-docs` with arguments `md tests/visual/out`, source `tests/visual/fixture/`. Run 2: `build-quiz-html` with arguments `tests/visual/out/Visual-Test-Questions.md tests/visual/out`.
- **Acceptance criteria:**
  - [ ] `git check-ignore tests/visual/out/Visual-Test/Visual-Test.html` prints the path.
  - [ ] `git status --porcelain` lists no file under `tests/visual/`.
  - [ ] `tests/visual/out/` contains `Visual-Test-Questions.md`, `Visual-Test-Answers.md`, `images/Visual-Test-Q1.png`, `Visual-Test/Visual-Test.html` and `Visual-Test/images/Visual-Test-Q1.png` (`ls -R`).
  - [ ] `Visual-Test-Questions.md` has an image line in Q1's stem and image lines in Q2's options a and b (`grep -n "!\["`).
  - [ ] Every `images/...` path in `Visual-Test/Visual-Test.html` exists under `Visual-Test/images/` (`grep -o` plus `ls`).
  - [ ] `Visual-Test/Visual-Test.html` contains no `{{` placeholder (`grep -c "{{"` prints `0`).
- **Doc deltas:** none. The **Testing skills** rule is already in `d:\SMU\quizzer\CLAUDE.md`.

## Human checks

- [ ] S2/S3: Build a quiz with one image in a stem and one in an option, open `<name>/<name>.html` in Chrome. Expected: both images show, scaled to fit, in light and dark mode.
- [ ] S3: Zip the `<name>/` folder, unzip it somewhere else, open the `.html`. Expected: images still show.
- [ ] S2: Open the `.html` with its `images/` folder renamed. Expected: each image shows its alt text and path instead.
- [ ] S2: DevTools Network tab, reload. Expected: only the file and its local images load; nothing from the internet.
- [ ] S4: `/format-quiz-docs word md` on an eLearn review screenshot of a question with a graph. Expected: `images/<name>-Q<N>.png` holds only the graph (no question text, options, ticks or answer feedback); it appears in both `.md` docs (VS Code preview) and both `.docx` docs.
- [ ] S4: `/format-quiz-docs md` with a ready `Q<N>.png` supplied. Expected: copied to `images/<name>-Q<N>.png`, not cropped.
- [ ] S6: Open `tests/visual/out/images/Visual-Test-Q1.png`. Expected: only the line graph; no question text, options, tick, cross or "Correct answer" line.
- [ ] S6: Open `tests/visual/out/Visual-Test/Visual-Test.html` in Chrome, light and dark mode. Expected: Q1 graph under the stem, Q2 bar charts inside options a and b, all scaled to fit.
