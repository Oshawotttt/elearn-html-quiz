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
4. Work out each answer: from the source (apply the eLearn rules to screenshots), else from course content, else your own.
5. Write the Questions doc and the Answers doc in the requested format and location.
6. Report the two file paths and how many answers are tagged `(own answer)`.
