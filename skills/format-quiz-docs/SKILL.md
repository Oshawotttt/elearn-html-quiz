---
name: format-quiz-docs
description: Turns quiz screenshots or text (incl. eLearn quiz review screenshots) into two formatted docs, a questions-only doc and a questions-with-answers doc. Run only when the user invokes /format-quiz-docs.
argument-hint: "[word|md] [output path]"
disable-model-invocation: true
---

# Format quiz docs

Arguments: $ARGUMENTS

1. Read `quiz-rules.md` and `questions-format.md` next to this file. Follow `quiz-rules.md` for arguments, output docs, file names, marks, finding answers and eLearn rules. Follow `questions-format.md` for the layout of each question.
2. Collect the source: screenshots or text the user attached or pointed to. If none is given, ask for it.
3. Extract every question and option exactly as written. Keep the original order. Do not reword.
   - Figures (graph, diagram, table image, code image) in a question or option: follow **Images** in `quiz-rules.md`. Use the image the user supplied for that question (`Q<N>.png`, `Q<N>-2.png`, ...). Otherwise crop the figure from the screenshot yourself, per the crop steps there.
   - Ask the user only for figures you could not crop after 3 tries. List each question number in one message.
4. Work out each answer: from the source (apply the eLearn rules to screenshots), else from course content, else your own.
5. Write the Questions doc and the Answers doc in the requested format and location. Put each image in both docs, at the same spot: an image line in `.md`, the inserted image in `.docx`. To mark a correct option whose text is an image, bold the letter and put ✔ after the image, e.g. `- **a.** ![Bar chart A](images/<name>-Q2.png) ✔`.
6. Report the two file paths, how many answers are tagged `(own answer)`, the `images/` folder and image count if any, and each question number whose image you had to ask for.
