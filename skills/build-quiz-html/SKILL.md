---
name: build-quiz-html
description: Builds a single self-contained HTML quiz page (no answers) from a Questions.md, quiz screenshots or quiz text. Run only when the user invokes /build-quiz-html.
argument-hint: "<questions.md | screenshots | text> [output folder]"
disable-model-invocation: true
---

# Build quiz HTML

Arguments: $ARGUMENTS

Files used (relative to this file):
- `quiz-template.html`: the page. It has one placeholder, `{{QUESTIONS}}`.
- `../format-quiz-docs/questions-format.md`: the Questions.md format the page reads.
- `../format-quiz-docs/quiz-rules.md`: file names (section **File names**), marks and eLearn screenshot rules.

## File names

Every file you create has no spaces. Build `<name>` from the quiz title as in **File names** of `quiz-rules.md`: replace each space with `-`, drop every character other than letters, digits, `.`, `_` and `-`, collapse repeated `-`, trim `-` from both ends, keep the case. Example: `Finals Practice Quiz 2 (Short)` gives `Finals-Practice-Quiz-2-Short.html`.

## Steps

1. **Find the input.**
   - A `.md` file given: use it.
   - Screenshots or text given: use them (step 2).
   - Nothing given: use the newest `*-Questions.md` in the working folder. None found: ask the user for the input. Stop until they answer.
   - Output folder: the one in the arguments. None given: the folder of the input file, else the working folder.
2. **Get a Questions.md in the format.** Read `../format-quiz-docs/questions-format.md`.
   - The input already follows it: use it as is.
   - Otherwise (screenshots, loose text, a `.md` in another layout): convert it. Extract every question and option exactly as written, in the original order. Read eLearn screenshots with the eLearn rules in `quiz-rules.md`, and never copy the student's ticks, crosses or scores. Include no answers, hints or explanations. Write the result to `<output folder>/<name>-Questions.md` and use that file.
3. **Copy the template.** Copy `quiz-template.html` to `<output folder>/<name>.html` with a shell copy (`cp` or `Copy-Item`). Never write or rewrite the template by hand.
4. **Prepare the question text.** Take the full text of the Questions.md, from the `# <title> - Questions` line to the end. Replace every `</script` (any case) with `<\/script`. The page reads it back as `</script`. Change nothing else.
5. **Fill the placeholder.** Use the Edit tool on `<output folder>/<name>.html`: replace `{{QUESTIONS}}` with the text from step 4. Make no other edit.
6. **Check.**
   - `{{QUESTIONS}}` no longer appears in the `.html`.
   - The number of `**Question N**` headers in the `.html` equals the number in the Questions.md.
   - The `.html` contains no answer key.
7. **Report** the `.html` path, the `-Questions.md` path if you wrote one, and the question count. Tell the user: open the file by double-clicking it; in Chrome or Edge the first answer opens a Save dialog for autosave to `<name>-attempt.md`.
