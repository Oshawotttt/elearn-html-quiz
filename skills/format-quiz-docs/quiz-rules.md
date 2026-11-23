# Quiz doc rules (shared by /format-quiz-docs and /generate-quiz-docs)

## Arguments

`$ARGUMENTS` may give:
- Output format: `word`/`docx` or `md`. Both named: make both. None named: default to `.docx`.
- Output location: a folder or file path. None given: write next to the source files, or to the current working directory.

For `.docx`, use the `anthropic-skills:docx` skill.

## Output: always two docs

Name them `<topic> - Questions` and `<topic> - Answers`. Take `<topic>` from the quiz title, or the source file/folder name.

1. **Questions doc**: for attempting the quiz.
   - Questions and options only.
   - No answers, hints, explanations, ticks or highlights. Nothing that gives the answer away.
   - Number every question. Label options a, b, c, d...
   - Mark multi-select questions "(Select all that apply)". Mark true/false questions as such.
   - Leave answer space (blank lines) under short-answer and open questions.
2. **Answers doc**: for checking answers.
   - Same questions, same numbering and option order.
   - Mark each correct option clearly, e.g. bold plus ✔. For multi-select, list every correct option.
   - Add a 1-2 line explanation per question where it helps.
   - Tag each answer with its source:
     - `(given)`: the quiz showed the answer.
     - `(from course: <file>)`: you found it in the course content.
     - `(own answer)`: general knowledge, not confirmed by course content. Flag these so the user can double-check.

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
