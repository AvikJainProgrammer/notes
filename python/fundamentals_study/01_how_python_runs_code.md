# Module 1 — How Python runs code

Part of [`../python_fundamentals.md`](../python_fundamentals.md). Questions for this module go in [`questions.md`](questions.md).

---

## Session 1 — The big picture

### What to read (in this order)

1. **Python behind the scenes #1: how the CPython VM works** (Victor Skvortsov, Aug 2020)
   https://tenthousandmeters.com/blog/python-behind-the-scenes-1-how-the-cpython-vm-works/
   - About 4,500 words, roughly 30–40 minutes.
   - Read the whole article. The key sections are **The big picture**, **Code objects, function objects, frames** and **Architecture summary**.
   - When you reach C `struct` listings, **skim them**: read the field names and comments, don't try to understand the C. The prose around each listing explains what matters.
   - The article was written for **Python 3.9**. Some details have changed since; see "What changed since 3.9" below before or after reading.

2. **Python Glossary**, four entries only: https://docs.python.org/3/glossary.html
   - *bytecode*, *CPython*, *interpreted*, *global interpreter lock*.

3. **Hands-on exercise** (10 minutes): see "Try it yourself" below.

### What you should understand by the end of this session

Use these as a checklist. Once you can answer them out loud, tell me which questions to add to `questions.md`.

1. What are the three stages CPython goes through when you run `python script.py`?
2. What is bytecode, and why does CPython compile to bytecode instead of interpreting the source text directly?
3. What is a **code object**, and what does it contain?
4. What is a **function object**, and how is it different from a code object?
5. What is a **frame**, and why does a recursive function have many frames but only one code object?
6. What is the evaluation loop, and what does it do with bytecode?
7. What are the thread state, interpreter state and runtime state, and how do they nest?
8. What is the GIL, at a one-sentence level? (Module 12 covers it in depth.)

---

## Supplementary explanation

### The three stages, in plain words

1. **Initialization:** CPython sets itself up before running any of your code: builds built-in types and modules, sets up `sys.path`, creates the main interpreter and thread state.
2. **Compilation:** your source text is turned into **code objects** containing **bytecode** (a compact list of simple instructions for a virtual machine). The source is never executed directly.
3. **Interpretation:** the **evaluation loop** (a big loop in C, `_PyEval_EvalFrameDefault`) reads one bytecode instruction at a time and does what it says, using a **value stack** to hold intermediate results.

This is why CPython is called a *bytecode interpreter*: it compiles first, but to bytecode for its own virtual machine, not to machine code for your CPU.

### Code object vs function object vs frame — an analogy

| Thing | Analogy | What it holds | How many |
|---|---|---|---|
| **Code object** | The text of a recipe | Bytecode, constants, variable names, argument count. Immutable. | One per function body (made once, at compile time) |
| **Function object** | A recipe card pinned to a particular kitchen | A code object **plus** its context: globals, default argument values, closure variables, name | One each time `def` runs |
| **Frame** | One actual cooking session | The local variables, the value stack, which instruction you're on, the frame that called it | One per *call* |

So a recursive function like `fact(5)` has **one** code object and **one** function object, but **six** frames alive at the deepest point (`fact(5)` → `fact(4)` → … → `fact(0)`).

You can see the code object yourself: every function has `f.__code__`.

### The state hierarchy

- **Runtime state:** one per process. Holds everything shared by the whole process.
- **Interpreter state:** one per interpreter (normally just one; sub-interpreters are possible). Holds modules, `sys` settings, built-ins.
- **Thread state:** one per OS thread running Python. Holds that thread's current frame (and therefore its call stack).

### What changed since 3.9 (the article's version)

You don't need to know these in depth yet, but they explain why your own `dis` output won't match the article exactly.

- **3.11 — faster CPython.** The *specializing adaptive interpreter* (PEP 659): instructions rewrite themselves into faster, type-specialised versions after running a few times (e.g. a generic add becomes an int add). Frames became cheaper: CPython now uses a lightweight internal frame and only creates a full Python frame object when something asks for it (e.g. a traceback or `sys._getframe()`). The conceptual model (one frame per call) is unchanged.
- **3.11 — new instruction names.** `BINARY_ADD` and friends became a single `BINARY_OP`; every function starts with a `RESUME` instruction.
- **3.12 — per-interpreter GIL** (PEP 684): each sub-interpreter can have its own GIL.
- **3.13 — experimental free-threaded build** (PEP 703: CPython without the GIL) and an experimental JIT compiler (PEP 744). Both are opt-in; the normal build still has the GIL.
- **3.14 —** the free-threaded build moved from experimental to officially supported (still optional, not the default).

### Try it yourself

Your Mac has two Pythons:

- `python3` → **3.9.6** (Apple's system Python — same version as the article).
- `/opt/homebrew/bin/python3.14` → **3.14.3** (use this one for the course).

Save this as `~/scratch_dis.py` (or anywhere) and run it with **both** Pythons:

```python
import dis

def add(a, b):
    total = a + b
    return total

dis.dis(add)

c = add.__code__
print("variable names:", c.co_varnames)
print("constants:     ", c.co_consts)
print("arg count:     ", c.co_argcount)
print("bytecode bytes:", len(c.co_code))
```

What it printed on your machine when I checked:

**Python 3.9.6**
```
  3           0 LOAD_FAST                0 (a)
              2 LOAD_FAST                1 (b)
              4 BINARY_ADD
              6 STORE_FAST               2 (total)

  4           8 LOAD_FAST                2 (total)
             10 RETURN_VALUE
```

**Python 3.14.3**
```
  2           RESUME                   0

  3           LOAD_FAST_BORROW_LOAD_FAST_BORROW 1 (a, b)
              BINARY_OP                0 (+)
              STORE_FAST               2 (total)

  4           LOAD_FAST_BORROW         2 (total)
              RETURN_VALUE
```

Things to notice:

- **Read the 3.9 version as a stack machine:** push `a`, push `b`, `BINARY_ADD` pops both and pushes the sum, `STORE_FAST` pops it into `total`, then push `total` and return it. This is the evaluation loop's value stack in action.
- **3.14 is the same program, optimised:** two loads merged into one *superinstruction*, the generic `BINARY_OP (+)`, and `RESUME` at the start. Same idea, different instructions.
- **`LOAD_FAST` uses a number, not a name:** `0 (a)` means "slot 0 in the frame's local variables". Locals are looked up by position in an array, which is why local variables are fast. (This comes back in Module 5 and Module 13.)
- `co_varnames` is `('a', 'b', 'total')`: the arguments come first, then other locals.

---

## Session log

| Session | Date | Read | Questions added |
|---|---|---|---|
| 1 — The big picture | | | |
