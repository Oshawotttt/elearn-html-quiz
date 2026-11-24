# elearn-html-quiz

Claude Code plugin `quizzer`. It turns quizzes into practice material: formatted question and answer docs, generated practice quizzes, and a single-file HTML quiz page that students can attempt offline.

## Installation

In Claude Code, run:

```
/plugin install create-quiz --marketplace Oshawotttt/elearn-html-quiz
```

This adds the marketplace if it is not added yet, then installs the plugin.

Or add the marketplace and install in two steps:

```
/plugin marketplace add Oshawotttt/elearn-html-quiz
/plugin install create-quiz@create-quiz
```

From a local clone, pass the folder path instead:

```
/plugin marketplace add ./elearn-html-quiz
/plugin install create-quiz@create-quiz
```

Restart Claude Code if the commands do not show up. To update later, run `/plugin marketplace update create-quiz`.

## Skills

Each skill runs only when you type its command.

| Command | Input | Output |
|---|---|---|
| `/format-quiz-docs [word\|md] [output path]` | Quiz screenshots (including eLearn quiz reviews) or quiz text | `<name>-Questions` and `<name>-Answers` docs |
| `/generate-quiz-docs [word\|md] [output path] [count / topics]` | Example quizzes and/or course content | A new practice quiz as `<name>-Questions` and `<name>-Answers` docs |
| `/build-quiz-html <questions.md \| screenshots \| text> [output folder]` | A Questions.md, screenshots or text | `<name>.html`, plus `<name>-Questions.md` when the input was not already a Questions.md |

- Docs default to `.docx`. Pass `md` for Markdown, or both.
- The Answers doc tags each answer `(given)`, `(from course: <file>)` or `(own answer)`. Check the `(own answer)` ones.
- `/build-quiz-html` with no file uses the newest `*-Questions.md` in the working folder. The HTML holds the questions only, never the answers.

Typical flow: `/format-quiz-docs md` or `/generate-quiz-docs md`, then `/build-quiz-html <name>-Questions.md`, then send the `.html` to students.

## Question format

All three skills share one Markdown format for Questions docs: [`skills/format-quiz-docs/questions-format.md`](skills/format-quiz-docs/questions-format.md). In short:

- `# <title> - Questions`, then an optional intro paragraph.
- Each question starts with `**Question N** (P points)`. `(Select all that apply)` marks multi-select.
- Options are `- a. text` to `- e. text`. True/false uses `- a. True` and `- b. False`.
- Written questions have no options. An optional `**Word limit**: Maximum N words.` sets the limit; the default is 250.

The HTML page parses this format when it opens, so a Questions.md written by hand works too.

Other shared rules (marks, finding answers, eLearn screenshot ticks and crosses) are in [`skills/format-quiz-docs/quiz-rules.md`](skills/format-quiz-docs/quiz-rules.md).

## File names

No file the skills create has spaces. `<name>` comes from the quiz title:

1. Replace each space with `-`.
2. Drop every character other than letters, digits, `.`, `_` and `-`.
3. Collapse repeated `-`, and trim `-` from both ends. Case is kept.

Example: `Finals Practice Quiz 2 (Short)` gives `Finals-Practice-Quiz-2-Short-Questions.md`, `Finals-Practice-Quiz-2-Short-Answers.md`, `Finals-Practice-Quiz-2-Short.html` and `Finals-Practice-Quiz-2-Short-attempt.md`. The rule applies to `.md`, `.docx` and `.html`.

## The HTML quiz page

One self-contained `.html` file. Students double-click it to open it in a browser. It has a question grid, flags for unsure questions, a timer, a progress bar, a word counter for written answers and dark mode.

The page is built from [`skills/build-quiz-html/quiz-template.html`](skills/build-quiz-html/quiz-template.html): `/build-quiz-html` copies it and pastes the Questions.md into its one placeholder, `{{QUESTIONS}}`.

### Saving answers

| | Chrome, Edge | Firefox, Safari |
|---|---|---|
| In the browser | Yes | Yes |
| Autosave to a file | Yes | No: the page shows "Autosave needs Chrome or Edge" |
| Export / Copy Markdown | Yes | Yes |

- **In the browser:** answers, flags and the timer are kept in the browser's local storage, one entry per quiz title. They survive closing the tab and the browser, until the student clicks Reset or clears site data. They stay on that computer and that browser only.
- **Autosave to a file (Chrome, Edge):** on the first answer, a Save dialog opens with `<name>-attempt.md` filled in. After that, every change overwrites that file. After a reload, the first answer (or the "Resume autosave" button) asks for permission once, then autosave carries on. "Set autosave file" picks a different file.
- **Export attempt (.md):** downloads `<name>-attempt-YYYY-MM-DD-HHMM.md`. **Copy Markdown** copies the same text.

The attempt file lists every question, its options and the student's answers, plus instructions for a marker. It has no answer key. Students mark it themselves, or give it to Claude with the Answers doc.

### No network requests

The page loads nothing from the internet: no CDN, web fonts, analytics, remote images or remote saving. Everything it needs is inside the file, and it works offline.

### Sharing on Telegram

Telegram Desktop warns that opening a file may reveal your IP address. It shows this for every `.html` file, whatever the content, and the warning cannot be removed from the file. Because the page makes no network requests, opening it does not reveal the IP. Students can:

- click to proceed, which is safe;
- tick "don't ask again" for `.html` files; or
- receive the file zipped instead.

## Out of scope

- Answer key, auto-marking or instant feedback inside the HTML page.
- Autosave to a file in Firefox and Safari: those browsers have no API for it. Use Export.
- Hosting the quiz online. The page is a local file.
- Any network request from the page.
- Images in questions (tracked in [`docs/specs/TODO.md`](docs/specs/TODO.md)).
