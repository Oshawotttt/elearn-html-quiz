# Questions.md format

The format of every Questions doc in Markdown. It covers every quiz that `/format-quiz-docs` and `/generate-quiz-docs` produce. `/build-quiz-html` reads it back, so keep to it exactly.

The Answers doc uses the same headings, numbering and option order (see `quiz-rules.md`).

## Layout

```markdown
# <title> - Questions

<optional intro paragraph: course, marks, time, instructions>

**Question 1** (2 points)

<stem>

- a. <option>
- b. <option>
- c. <option>
- d. <option>

**Question 2** (2 points) (Select all that apply)

<stem>

- a. <option>
- b. <option>
- c. <option>
- d. <option>
- e. <option>

**Question 3** (10 points)

<stem>

**Word limit**: Maximum 300 words.
```

## Title

- First line: `# <title> - Questions`. Exactly one `#` heading in the file.
- `<title>` is the quiz name as shown, with spaces, e.g. `# Finals Practice Quiz 2 (Short) - Questions`. Only the file name drops spaces (see `quiz-rules.md`).

## Intro

- Optional. One or more paragraphs between the title and `**Question 1**`.
- Put course-wide instructions here: total marks, time, how written answers are marked.

## Question header

- One line per question: `**Question N**`, then optional markers, in this order:
  - `(P points)`: marks for the question. `(1 point)` for 1. Leave it out when the source shows no marks.
  - `(Select all that apply)`: the question is multi-select. Leave it out for every other type.
- Number questions 1, 2, 3, ... with no gaps, in source order.
- Examples: `**Question 4**`, `**Question 4** (2 points)`, `**Question 4** (2 points) (Select all that apply)`.

## Stem

- Everything after the header and before the options (or before the next question).
- Markdown allowed: paragraphs, **bold**, *italic*, `inline code`, fenced code blocks, numbered and bulleted lists, tables.
- A bulleted list in the stem must not start with `a. ` to `e. ` after the `- `, so it is never read as options.
- Never put answers, hints or explanations in the stem.

## Options

- One line each: `- a. <text>`, `- b. <text>`, ... up to `- e. <text>`.
- Lowercase letters, in order, starting at `a`, no gaps. At least 2 options.
- Option text stays on one line. Inline Markdown is allowed (`code`, **bold**, *italic*).
- Options come after the stem, as the last block of the question.

## Question types

| Type | Header | Options |
|---|---|---|
| Single choice | `**Question N** (P points)` | `- a.` to `- e.` |
| Multi-select | `**Question N** (P points) (Select all that apply)` | `- a.` to `- e.` |
| True/false | `**Question N** (P points)` | exactly `- a. True` and `- b. False` |
| Written (short answer, essay, open) | `**Question N** (P points)` | none |

True/false is a single-choice question with the two options `True` and `False`.

## Written questions

- No options.
- Optional last line: `**Word limit**: Maximum N words.` Leave it out to use the default of 250 words.
- In a `.md` Questions doc, leave answer space under the stem with `&nbsp;` lines, one per paragraph:

```markdown
**Question 16** (10 points)

Explain why chained allocation performs badly for random access.

**Word limit**: Maximum 200 words.

&nbsp;

&nbsp;
```

## Spacing

- One blank line between the title, intro, each header, stem block and option list.
- No horizontal rules (`---`) between questions.
