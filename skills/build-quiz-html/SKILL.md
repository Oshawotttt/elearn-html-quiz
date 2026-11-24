---
name: build-quiz-html
description: Builds an HTML quiz page (no answers) from a Questions.md, quiz screenshots or quiz text. One self-contained file, or a folder with the page and its images when the quiz has images. Run only when the user invokes /build-quiz-html.
argument-hint: "<questions.md | screenshots | text> [output folder]"
disable-model-invocation: true
---

# Build quiz HTML

Arguments: $ARGUMENTS

Files used (relative to this file):
- `quiz-template.html`: the page. It has one placeholder, `{{QUESTIONS}}`.
- `../format-quiz-docs/questions-format.md`: the Questions.md format the page reads, including image lines (section **Images**).
- `../format-quiz-docs/quiz-rules.md`: file names (section **File names**), images (section **Images**), marks and eLearn screenshot rules.

## File names

Every file and folder you create has no spaces. Build `<name>` from the quiz title as in **File names** of `quiz-rules.md`: replace each space with `-`, drop every character other than letters, digits, `.`, `_` and `-`, collapse repeated `-`, trim `-` from both ends, keep the case. Example: `Finals Practice Quiz 2 (Short)` gives `Finals-Practice-Quiz-2-Short.html`.

## Output layout

- **Quiz with no images**: `<output folder>/<name>.html`. One self-contained file.
- **Quiz with images** (the Questions.md has at least one image line `![<alt>](images/<file>)`):
  - `<output folder>/<name>/<name>.html`
  - `<output folder>/<name>/images/<file>` for every image the Questions.md references.

  The `<name>/` folder is the thing the user zips and sends. The page loads its images from the `images/` folder beside it, so the two must stay together.
- The `-Questions.md` and its own `images/` folder stay where they are, outside the `<name>/` folder.
- `<name>/` already exists: overwrite files with the same names and leave every other file in it.

## Steps

1. **Find the input.**
   - A `.md` file given: use it.
   - Screenshots or text given: use them (step 2).
   - Nothing given: use the newest `*-Questions.md` in the working folder. None found: ask the user for the input. Stop until they answer.
   - Output folder: the one in the arguments. None given: the folder of the input file, else the working folder.
2. **Get a Questions.md in the format.** Read `../format-quiz-docs/questions-format.md`.
   - The input already follows it: use it as is.
   - Otherwise (screenshots, loose text, a `.md` in another layout): convert it. Extract every question and option exactly as written, in the original order. Read eLearn screenshots with the eLearn rules in `quiz-rules.md`, and never copy the student's ticks, crosses or scores. Include no answers, hints or explanations.
   - Screenshots with figures (graphs, diagrams, table images, code images): crop each figure into `<output folder>/images/` and add its image line, following **Images** in `quiz-rules.md`.
   - Write the result to `<output folder>/<name>-Questions.md` and use that file.
3. **Check the images.** List every image path in the Questions.md (`grep -o "!\[[^]]*\](images/[^)]*)"`). Each path is relative to the folder of the Questions.md.
   - No image lines: use the no-image layout. Skip to step 4.
   - Any image file missing: stop before writing the `.html`. Tell the user each question number and path that is missing.
   - All present: use the image layout.
4. **Copy the template.** Copy `quiz-template.html` with a shell copy (`cp` or `Copy-Item`) to:
   - no images: `<output folder>/<name>.html`
   - images: `<output folder>/<name>/<name>.html` (create `<name>/` first)

   Never write or rewrite the template by hand.
5. **Copy the images** (image layout only). Create `<output folder>/<name>/images/`. Copy (shell copy) each image the Questions.md references, and only those, from the `images/` folder next to the Questions.md into it. Keep each file name exactly as written in the image line, including its case. Leave the originals where they are.
6. **Prepare the question text.** Take the full text of the Questions.md, from the `# <title> - Questions` line to the end. Replace every `</script` (any case) with `<\/script`. The page reads it back as `</script`. Change nothing else. Image lines stay exactly as written.
7. **Fill the placeholder.** Use the Edit tool on the `.html` from step 4: replace `{{QUESTIONS}}` with the text from step 6. Make no other edit.
8. **Check.**
   - `{{QUESTIONS}}` no longer appears in the `.html`.
   - The number of `**Question N**` headers in the `.html` equals the number in the Questions.md.
   - The `.html` contains no answer key.
   - Image layout: every `images/...` path in the `.html` exists under `<name>/images/` (`grep -oiE "images/[A-Za-z0-9._-]+\.(png|jpe?g|gif|webp)"` on the `.html`, then `ls` each under `<name>/`).
9. **Report:**
   - the `.html` path, the `-Questions.md` path if you wrote one, and the question count
   - image layout: the `<name>/` folder path and the image count. Tell the user to zip and send this whole folder, so the images travel with the page.

   Tell the user: open the `.html` by double-clicking it; in Chrome or Edge the first answer opens a Save dialog for autosave to `<name>-attempt.md`.
