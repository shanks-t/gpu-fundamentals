# Quiz artifact format

## quiz.md

Use this structure, adapting concept map and questions to source material:

~~~markdown
# Lecture title — Mastery Quiz

Source: source files
Status: In progress

## Concept map

1. Concept

## Progress summary

| Item | Type | Status | Attempts | Hints | Score | Gap tags |
|---|---|---|---:|---:|---:|---|
| Q1 | Conceptual | Complete | 1 | 0 | 10 | — |

## Question ledger

### Q1 — Short title

Skill targets: tag, tag

Prompt:

Self-contained natural-language question. Describe source-code behavior or relevant shapes/indexing in prose; do not copy code unless the user explicitly asks for code.

Status: Pending | In progress | Complete | Review due

Attempt history:

1. Learner answer: Pending
   Hints used: 0
   Feedback: Pending
   Score: Pending
   Gap tags: Pending

Next drill: Pending
~~~

Do not prepopulate answers or scores. Add an attempt-history item when the learner responds. Update progress summary at the same time.

## quiz.ipynb

Use notebook format 4. Include:

1. A title/instructions Markdown cell naming source material and saying answers are graded in its ledger.
2. A Markdown cell describing any setup or fixture requirements.
3. Increasingly difficult exercise Markdown and empty code-cell pairs.
4. A concise exercise ledger near the end:

~~~markdown
## Exercise ledger

| Item | Skill targets | Status | Attempts | Hints | Score | Gap tags | Next drill |
|---|---|---|---:|---:|---:|---|---|
| E1 | broadcasting-shapes | Pending | 0 | 0 | — | — | — |
~~~

5. A final mastery exercise with a transfer/explanation prompt.

Do not include setup code, scaffold code, tests, solutions, or code-bearing hints unless the user explicitly requests code. State behavior checks in natural language. When grading, update its ledger row and append a concise attempt note below the table. Preserve learner code and outputs in existing notebooks.
