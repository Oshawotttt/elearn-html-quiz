---
name: generate-quiz-docs
description: Generates practice quizzes from example quizzes and/or course content, output as a questions-only doc and a questions-with-answers doc. Run only when the user invokes /generate-quiz-docs.
argument-hint: "[word|md] [output path] [number of questions / topics, optional]"
disable-model-invocation: true
---

# Generate quiz docs

Arguments: $ARGUMENTS

1. Read `../format-quiz-docs/quiz-rules.md` and `../format-quiz-docs/questions-format.md` (relative to this file). Follow `quiz-rules.md` for arguments, output docs, file names, marks, finding answers and eLearn rules. Follow `questions-format.md` for the layout of each question.
2. Collect inputs: example quizzes (screenshots, text, files) and/or course content. None pointed to: search the working folder for them.
3. From example quizzes, learn the style: question types (MCQ, multi-select, true/false, short answer), difficulty, wording, option count. Read eLearn screenshots with the eLearn rules so you learn the right answers, not the student's.
4. Write new questions. Defaults: 20 questions, or match the example quiz length. Use the count or topics in the arguments if given.
   - Base every question on course content. Cover the topics evenly.
   - Do not copy example questions word for word. Rephrase or test the same idea in a new way.
   - Multi-select: vary the number of correct options. Make distractors plausible.
   - Images: reuse only. A new question may use an existing image from an example quiz or the course content (a figure, graph or diagram file, or one cropped from a screenshot per **Images** in `quiz-rules.md`). Copy it to `images/<name>-Q<N>.<ext>` under the new quiz's name, following **Images** in `quiz-rules.md`. Never create, invent or draw an image. If a question would need an image that does not exist, write a different question.
5. Write the Questions doc and the Answers doc. The Answers doc repeats every image of the Questions doc, at the same spot. To mark a correct option whose text is an image, bold the letter and put ✔ after the image, e.g. `- **a.** ![Bar chart A](images/<name>-Q2.png) ✔`. In the Answers doc, cite the course file for each answer, `(from course: <file>)`, or tag `(own answer)`.
6. After the docs are created, check the answer key for patterns. Fix any found, in both docs, then check again:
   - Correct-answer letters spread roughly evenly across a, b, c, d. No letter dominates.
   - No runs: the same letter is not correct 3+ times in a row. No repeating sequence (a, b, c, d, a, b...).
   - Multi-select: the number of correct options varies. The correct set is not always the first or last options.
   - True/false: roughly balanced, no long runs.
   - The correct option is not usually the longest, most detailed, or the only one with hedging words ("usually", "may").
   - "All of the above" / "None of the above" is not always correct when it appears.
   Fix by shuffling option order or rewording options. Never change which answer is correct.
7. Report the two file paths, question count, topics covered, the final answer-letter spread, and the `images/` folder and image count if any.
