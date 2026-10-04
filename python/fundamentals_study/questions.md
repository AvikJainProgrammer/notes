# Python Fundamentals — Flashcard Questions

Questions for the voice and written flashcard tools, collected as I study.
Plan: [`../python_fundamentals.md`](../python_fundamentals.md).

A question is only added here once I have read and understood the material, so **this file is also my progress tracker**.

## How to write a question

Each question records which tool it is for and its type:

- **Tool**
  - `voice` → `voice-flashcards`: short spoken prompt, short spoken answer. Can be "one answer" (extra entries are accepted variants) or "list of answers".
  - `written` → `cardmaker` (Anki) / `simple_learner` / handwritten practice: longer answers, code, fill-in-the-blank.
- **Type** (from `question-types`): `qna`, `fib` (fill in the blank), `mcq`, `list_completion`, `write_code`, `code_fib`, `descriptive_text`.

Template:

```markdown
### Q<module>.<n>
- **Tool:** voice | written
- **Type:** qna
- **Question:** ...
- **Answer:** ...
- **Accepted variants / list items:** ... (optional)
- **Source:** where this comes from
```

## Progress

| Module | Status | Questions |
|---|---|---|
| 1 — How Python runs code | Session 1 in progress | 5 |
| 2 — Objects, names and memory | Not started | 0 |
| 3 — Built-in types and their costs | Not started | 0 |
| 4 — Statements, control flow and comprehensions | Not started | 0 |
| 5 — Functions, scope and closures | Not started | 0 |
| 6 — Iterators and generators | Not started | 0 |
| 7 — Decorators and context managers | Not started | 0 |
| 8 — Classes and the data model | Not started | 0 |
| 9 — Descriptors and metaclasses | Not started | 0 |
| 10 — Errors and exceptions | Not started | 0 |
| 11 — Modules, packages and imports | Not started | 0 |
| 12 — Concurrency model | Not started | 0 |
| 13 — Measuring and optimising | Not started | 0 |

---

## Module 1 — How Python runs code

### Q1.1
- **Tool:** voice
- **Type:** qna
- **Question:** Does Python have a formal specification?
- **Answer:** No
- **Accepted variants:** No, it doesn't
- **Source:** PBTS #1, Introduction

### Q1.2
- **Tool:** voice
- **Type:** list_completion
- **Question:** Python has no formal specification. Which two things define the language instead?
- **Answer (list):**
  1. The Python Language Reference
  2. CPython, its main implementation
- **Accepted variants:** "Language Reference" / "language reference"; "CPython" / "the CPython implementation"
- **Source:** PBTS #1, Introduction

### Q1.3
- **Tool:** voice
- **Type:** list_completion
- **Question:** Name three reasons to study how CPython is implemented.
- **Answer (list):**
  1. Deeper understanding of the language and its quirks
  2. Implementation details matter in practice: performance, memory, garbage collection, threads
  3. To use the Python/C API effectively
- **Accepted variants:** item 2: "to estimate performance and inefficiencies", "to spot defects and inefficiencies"
- **Source:** PBTS #1, Introduction

### Q1.4
- **Tool:** voice
- **Type:** list_completion
- **Question:** What two things does the Python/C API let you do?
- **Answer (list):**
  1. Extend Python with C
  2. Embed Python inside C
- **Accepted variants:** item 1: "write C extensions for Python"; item 2: "embed the Python interpreter in a C program"
- **Source:** PBTS #1, Introduction

### Q1.5
- **Tool:** voice
- **Type:** list_completion
- **Question:** What are the three stages of executing a Python program, in order?
- **Answer (list):**
  1. Initialization
  2. Compilation
  3. Interpretation
- **Source:** PBTS #1, The big picture
