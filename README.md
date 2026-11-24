# elearn-html-quiz

Claude Code plugin `quizzer`. It turns quizzes into practice material: formatted question and answer docs, generated practice quizzes, and an HTML quiz page that students can attempt offline. The page is a single file, or a folder with the page and its images when the quiz has images.

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
| `/build-quiz-html <questions.md \| screenshots \| text> [output folder]` | A Questions.md, screenshots or text | `<name>.html`, or the folder `<name>/` with `<name>.html` and `images/` when the quiz has images; plus `<name>-Questions.md` when the input was not already a Questions.md |

- Docs default to `.docx`. Pass `md` for Markdown, or both.
- The Answers doc tags each answer `(given)`, `(from course: <file>)` or `(own answer)`. Check the `(own answer)` ones.
- `/build-quiz-html` with no file uses the newest `*-Questions.md` in the working folder. The HTML holds the questions only, never the answers.

Typical flow: `/format-quiz-docs md` or `/generate-quiz-docs md`, then `/build-quiz-html <name>-Questions.md`, then send the `.html` to students (or, for a quiz with images, zip and send the `<name>/` folder).

## Question format

All three skills share one Markdown format for Questions docs: [`skills/format-quiz-docs/questions-format.md`](skills/format-quiz-docs/questions-format.md). In short:

- `# <title> - Questions`, then an optional intro paragraph.
- Each question starts with `**Question N** (P points)`. `(Select all that apply)` marks multi-select.
- Options are `- a. text` to `- e. text`. True/false uses `- a. True` and `- b. False`.
- Written questions have no options. An optional `**Word limit**: Maximum N words.` sets the limit; the default is 250.
- A figure is an image line, `![<alt text>](images/<file>)`, in the stem or as an option's text (see **Images in questions** below).

The HTML page parses this format when it opens, so a Questions.md written by hand works too.

Other shared rules (marks, finding answers, eLearn screenshot ticks and crosses) are in [`skills/format-quiz-docs/quiz-rules.md`](skills/format-quiz-docs/quiz-rules.md).

## File names

No file the skills create has spaces. `<name>` comes from the quiz title:

1. Replace each space with `-`.
2. Drop every character other than letters, digits, `.`, `_` and `-`.
3. Collapse repeated `-`, and trim `-` from both ends. Case is kept.

Example: `Finals Practice Quiz 2 (Short)` gives `Finals-Practice-Quiz-2-Short-Questions.md`, `Finals-Practice-Quiz-2-Short-Answers.md`, `Finals-Practice-Quiz-2-Short.html` and `Finals-Practice-Quiz-2-Short-attempt.md`. The rule applies to `.md`, `.docx`, `.html` and image files.

## Images in questions

A question can show a figure: a graph, diagram, table image or code image, in the stem or as an option.

- **Supplying images:** give eLearn screenshots and Claude crops each figure out itself. The crop holds only the figure: no question or option text, ticks, crosses, highlights, scores or "Correct answer" feedback. If a crop fails 3 times, Claude asks you for that image by question number. You can also supply ready images named `Q<N>.png`, `Q<N>-2.png`, ...; they are copied, not cropped.
- **Folder and names:** images go in an `images/` folder next to the docs, named `<name>-Q<N>.png`, then `<name>-Q<N>-2.png`, ... for more images in the same question.
- **In the docs:** `.md` docs reference the file with an image line, `![<alt text>](images/<file>)`; keep the `images/` folder next to them. `.docx` docs have the images inserted. Both the Questions and the Answers doc show every image.
- **`/generate-quiz-docs`** only reuses images from example quizzes or course content. It never creates or draws one.
- **HTML page:** a quiz with images builds to a folder:

  ```
  <output folder>/<name>/
    <name>.html
    images/<name>-Q1.png ...
  ```

  The page shows each image from the `images/` folder beside it, scaled to fit. Zip the `<name>/` folder and send the zip; students unzip it and open the `.html`. If an image file is missing, the page shows its description and path instead. A quiz with no images is still one `.html` file.
- Only paths of the form `images/<file>.png|jpg|jpeg|gif|webp` become images. Anything else written as `![...](...)` shows as plain text.

## The HTML quiz page

One self-contained `.html` file, or, for a quiz with images, a folder with the `.html` and its `images/` (see **Images in questions**). Students double-click the `.html` to open it in a browser. It has a question grid, flags for unsure questions, a timer, a progress bar, a word counter for written answers and dark mode.

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

The page loads nothing from the internet: no CDN, web fonts, analytics, remote images or remote saving. Everything it needs is inside the file, plus the local `images/` folder beside it for a quiz with images, and it works offline.

### Sharing on Telegram

Telegram Desktop warns that opening a file may reveal your IP address. It shows this for every `.html` file, whatever the content, and the warning cannot be removed from the file. Because the page makes no network requests, opening it does not reveal the IP. Students can:

- click to proceed, which is safe;
- tick "don't ask again" for `.html` files; or
- receive the file zipped instead.

## Out of scope

- Answer key, auto-marking or instant feedback inside the HTML page.
- Autosave to a file in Firefox and Safari: those browsers have no API for it. Use Export.
- Hosting the quiz online. The page is a local file.
- Any network request from the page, including remote images (`http://`, `https://`, `//`).
- Embedding images inside the `.html` (base64). The page references image files in `images/`; ship the `.html` and `images/` together in one folder.
- Generating new diagrams or pictures in `/generate-quiz-docs`. It only reuses existing images.
- Zipping the output. Zip the `<name>/` folder yourself.
