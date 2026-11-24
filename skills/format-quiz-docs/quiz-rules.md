# Quiz doc rules (shared by /format-quiz-docs and /generate-quiz-docs)

## Arguments

`$ARGUMENTS` may give:
- Output format: `word`/`docx` or `md`. Both named: make both. None named: default to `.docx`.
- Output location: a folder or file path. None given: write next to the source files, or to the current working directory.

For `.docx`, use the `anthropic-skills:docx` skill.

## Output: always two docs

Take the quiz title from the source, or the source file/folder name. Name the docs `<name>-Questions` and `<name>-Answers`, where `<name>` is the title made safe for file names (see **File names**).

Write both docs in the format in `questions-format.md` (next to this file): title heading, question headers, stem, options, written questions. In `.docx`, keep the same text and order.

1. **Questions doc**: for attempting the quiz.
   - Questions and options only.
   - No answers, hints, explanations, ticks or highlights. Nothing that gives the answer away.
   - Number every question. Label options a, b, c, d...
   - Mark multi-select questions "(Select all that apply)". Give true/false questions the options `a. True` and `b. False`.
   - Leave answer space (blank lines) under short-answer and open questions.
2. **Answers doc**: for checking answers.
   - Same questions, same numbering and option order.
   - Mark each correct option clearly, e.g. bold plus ✔. For multi-select, list every correct option.
   - Add a 1-2 line explanation per question where it helps.
   - Tag each answer with its source:
     - `(given)`: the quiz showed the answer.
     - `(from course: <file>)`: you found it in the course content.
     - `(own answer)`: general knowledge, not confirmed by course content. Flag these so the user can double-check.

## File names

No spaces in file names. Build `<name>` from the title:

1. Replace each space with `-`.
2. Drop every character other than letters, digits, `.`, `_` and `-`.
3. Collapse repeated `-` into one.
4. Trim `-` from both ends.
5. Keep the case.

Example: title `Finals Practice Quiz 2 (Short)` gives `Finals-Practice-Quiz-2-Short-Questions.md` and `Finals-Practice-Quiz-2-Short-Answers.md`.

The rule applies to `.md` and `.docx` files. The title inside the doc keeps its spaces.

## Images

Use this section when a question has a figure: a graph, diagram, table image or code image. The image line format and where it goes are in **Images** in `questions-format.md`.

### Folder and names

- Put image files in an `images/` folder next to the docs, e.g. `<output folder>/images/`.
- Name them `<name>-Q<N>.<ext>` for the first image of question N, then `<name>-Q<N>-2.<ext>`, `<name>-Q<N>-3.<ext>`, ... `<name>` is the safe quiz name (see **File names**). Example: `images/Finals-Practice-Quiz-2-Short-Q4.png`.
- File names have no spaces. Allowed extensions: `png`, `jpg`, `jpeg`, `gif`, `webp`. Crops are saved as `.png`.
- Number images in the order they appear in the question: stem first, then options a, b, c, ...

### User-supplied images

The user may supply ready images named `Q<N>.png`, `Q<N>-2.png`, ... Copy each one (shell copy) to `images/<name>-Q<N>.png`, `images/<name>-Q<N>-2.png`, ... keeping its extension. Do not crop them.

### Cropping figures from screenshots

For every other figure, crop it out of the source screenshot yourself. The crop holds only the figure, never the whole screenshot.

1. Get the screenshot's pixel size with Pillow:
   `python -c "from PIL import Image; print(Image.open(r'<src>').size)"`
2. View the screenshot. Pick the figure's box as fractions of width and height (`l`, `t`, `r`, `b`). Add a small margin.
3. Crop with an inline Pillow command; do not write a script file:
   `python -c "from PIL import Image; im=Image.open(r'<src>'); w,h=im.size; im.crop((int(<l>*w), int(<t>*h), int(<r>*w), int(<b>*h))).save(r'<out>')"`
4. View the crop. It must hold the whole figure and none of:
   - question text
   - option text
   - ticks, crosses or ticked boxes
   - highlights
   - scores or marks
   - "Correct answer" feedback
   - anything else that gives the answer away

   Otherwise adjust the box and crop again.
5. After 3 failed tries: ask the user for that image, naming the question number.

### Images in the docs

- `.md`: add the image line (`![<alt text>](images/<file>)`) where the figure appears, per `questions-format.md`. The doc references the file; it does not embed it.
- `.docx`: insert the image file from `images/` into the document (`anthropic-skills:docx` skill), at the same spot as the Markdown image line: on its own paragraph in the stem, or inside the option. Scale it to fit the page width. The user does not paste images by hand.
- Both the Questions doc and the Answers doc show every image.
- No remote images. Every image is a local file in `images/`.

## Marks

Include marks only when shown. Missing or partial marks: ignore them. Do not invent marks.

## Finding answers when not provided

1. Search the working folder and the folder of the source files for course content: lecture notes, slides, PDFs, `.md`/`.txt` files. Use Grep/Glob on the question's key terms.
2. Answer from that content first, and cite the file.
3. No match: answer from your own knowledge and tag it `(own answer)`.

## eLearn screenshot rules

eLearn may show the correct answers or may not. If it does not, find the answers yourself (see above).

### Multi-select (checkbox) answers

Each option has a box (ticked or empty, meaning the student chose it or not). On the left of the box there is a mark: a tick or a cross.

| Left mark | Box | Meaning | Correct answer |
|---|---|---|---|
| ✔ tick | ticked | choosing it was correct | option IS correct |
| ✔ tick | empty | not choosing it was correct | option is NOT correct |
| ✘ cross | ticked | choosing it was wrong | option is NOT correct |
| ✘ cross | empty | not choosing it was wrong | option IS correct |

Short form: the option is correct when (left tick AND box ticked) OR (left cross AND box empty).

Options with no left mark: eLearn gave no feedback on them. Work them out yourself and tag `(own answer)`.

### Single-choice and other types

A tick beside the chosen option means it is correct. A cross means it is wrong; the correct one is shown if eLearn displays it. Otherwise find it yourself.

Never copy the student's selections, ticks, crosses or scores into the Questions doc.
