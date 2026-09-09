---
name: lecture-quiz
description: Create and run mastery-focused quizzes from lecture notes, notebooks, and code. Use when a user asks to be quizzed on course material, build a course quiz, track conceptual or coding-quiz progress, or generate targeted practice from learning gaps.
---

# Lecture Quiz

Create an evidence-based learning loop around material the user identifies. Produce two companion artifacts in the relevant lecture directory:

- quiz.md for conceptual and code-reading questions plus the graded answer ledger.
- quiz.ipynb for progressive runnable coding exercises plus the graded exercise ledger.

Use this skill for learning material and quizzes, not general code review or unrelated assessment.

## No-code teaching rule

Unless the user explicitly asks for code, communicate only in natural language. This applies to feedback, hints, explanations, follow-up questions, conceptual quizzes, and coding-exercise reviews.

- Do not provide code snippets, pseudocode, filled blanks, test code, corrected lines, shell commands, or solution-bearing examples.
- Explain the missing concept, identify the relevant relationship or invariant, and ask the learner to make the next change in their own words.
- For coding quizzes, create prose prompts and empty code cells by default. Inspect and grade code the learner submits, but describe needed changes in natural language.
- If the user explicitly asks for code, provide only the scope they requested. Do not add an unsolicited complete solution.

## Start from the material

1. Read the requested lecture material. For notebooks, inspect both markdown and code cells; for a directory, inspect relevant source files before drafting questions.
2. If quiz.md or quiz.ipynb already exists, read it first. Preserve completed answers, feedback, scores, and working learner code; extend rather than reset progress.
3. Identify actual learning objectives: data/layout assumptions, control flow, arithmetic/indexing, API boundaries, and performance trade-offs. Do not invent topics absent from the source.
4. For new artifacts, use references/quiz-format.md. For scoring, use references/scoring.md.

## Build the conceptual quiz

Progress from simple to integrative:

1. Recognition and fill-in-the-blank questions.
2. Explain-a-line or trace-the-data questions.
3. Small arithmetic or indexing derivations.
4. Implementation/performance comparisons.
5. Design or debugging questions joining multiple concepts.

Ask one question at a time when running a quiz. Adapt the next question to learner gaps; do not mechanically proceed while the prior concept remains unresolved. Make each prompt self-contained enough that its relevant code is clear.

## Build the coding notebook

The notebook must build directly on the Markdown quiz concepts, beginning with small, clearly scoped tasks and ending with writing functions or kernels from scratch. By default, include:

- prose-only setup requirements and deterministic fixture specifications;
- fill-in-the-blank exercises before open-ended code, with blank code cells for learner work;
- natural-language acceptance criteria instead of supplied test code;
- code-reading, indexing, and performance exercises when source material contains them;
- a final mastery exercise that combines important concepts;
- a per-exercise grading ledger.

Leave solutions and scaffolding code out unless the user explicitly requests code. Validate notebook JSON after creating or editing it. Do not claim code exercises pass until learner code has been executed or inspected.

## Run and record the quiz

For every completed response:

1. Record the learner answer faithfully but concisely.
2. Give direct natural-language feedback: correct parts, gaps, and the smallest needed conceptual correction. Do not show code unless explicitly requested.
3. Update status, attempt count, hint count, score, gap tags, and recommended follow-up in the corresponding artifact.
4. If incomplete, leave status In progress and give a targeted prompt or hint. Score it only when the learner explicitly finishes, moves on, or the response clearly represents a final attempt.
5. Treat a clarification request or worked example as a hint, not a failed attempt. Count a substantive submitted answer as an attempt.

Preserve original grade and attempts as history. On a later retry, add a review entry instead of overwriting prior evidence.

## Gap-driven follow-up

Use low-score or repeated-hint gap tags to create narrow review questions or coding drills. Isolate a gap with one small drill before re-asking an integrated question. At session end, summarize strengths, gaps, and the most valuable next drills from the ledgers.

Score measures demonstrated mastery of one prompt at one moment; it is not a judgment of the learner.
