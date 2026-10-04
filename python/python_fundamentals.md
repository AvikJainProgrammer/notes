# Python Fundamentals — Learning Plan

## Objective

Understand the fundamentals of the Python **language** itself: how it works, not how it is used in industry.
By the end I should be able to look at any piece of Python and explain **what the interpreter does with it, why, and what it costs** (time and memory).

**Study files:** reading guides per module and the flashcard questions file are in [`fundamentals_study/`](fundamentals_study/). Progress is tracked in [`fundamentals_study/questions.md`](fundamentals_study/questions.md).

## Guardrails

**In scope**

- The language: syntax, semantics, the data model, the object model, scoping, functions, classes, iteration, exceptions, and so on.
- Basics **and** niche topics (e.g. metaclasses, descriptors).
- Built-in functions, built-in types and the standard library, but only where they illustrate something fundamental about the language (e.g. `collections`, `itertools`, `functools`, `dis`, `sys`, `gc`, `timeit`).
- What happens behind the scenes: how CPython compiles and interprets code (bytecode, the evaluation loop), and how memory is managed (reference counting, garbage collection, object interning, `__slots__`).
- Performance characteristics of language constructs (time and space complexity).
  - *Example:* dispatching with a dictionary instead of a long `if/elif` chain. The `if/elif` chain checks branches one by one, so it is **O(n)** in the number of branches. A dict (hash map) lookup is **O(1)** on average. Note that this is a *time* saving; the dict actually uses *more* memory than the `if` statements. Knowing both sides of a trade-off like this is part of the goal.

**Out of scope**

- Design patterns, architecture, frameworks (these come later; see `../learning_framework/dappl.md` and `object_oriented/`).
- Third-party libraries and applied topics (web scraping, image/PDF/spreadsheet handling, email, GUIs, web frameworks), unless one illustrates a language fundamental.
- Writing C extensions, packaging/distribution, and the details of other Python implementations (PyPy, etc.).

**Python version:** use the latest stable CPython (3.13 or later). Where behaviour changed recently, note which version introduced it.

## Study resources (all free)

Keep the list short and stick to it. Add a source only if it covers a gap the others don't.

### 1. Local course material: Complete Python 3 Bootcamp (Jose Portilla, Udemy)

Location: `~/Documents/projects/study_material/Complete_Python_3_Bootcamp-From-Zero-to-Hero/` (Jupyter notebooks with exercises and solutions).
Use it for the **basics and practice exercises**. In the topic list below it is referred to as **[Bootcamp NN]** (the folder number).

| Folder | Status |
|---|---|
| `00-Python Object and Data Structure Basics` | In scope |
| `01-Python Comparison Operators` | In scope |
| `02-Python Statements` | In scope |
| `03-Methods and Functions` | In scope |
| `04-Milestone Project - 1` | Optional practice |
| `05-Object Oriented Programming` | In scope |
| `06-Modules and Packages` | In scope |
| `07-Errors and Exception Handling` | In scope (unit-testing notebook optional) |
| `08-Milestone Project - 2` | Optional practice |
| `09-Built-in Functions` | In scope |
| `10-Python Decorators` | In scope |
| `11-Python Generators` | In scope |
| `12-Advanced Python Modules` | Partly: `collections`, `timeit`, `pdb` in scope; the rest out of scope |
| `13-Web-Scraping` | Out of scope |
| `14-Working-with-Images` | Out of scope |
| `15-PDFs-and-Spreadsheets` | Out of scope |
| `16-Emailing-with-Python` | Out of scope |
| `17-Advanced Python Objects and Data Structures` | In scope (incl. context managers) |
| `18-Milestone Project - 3` | Optional practice |
| `19-Bonus Material - Introduction to GUIs` | Out of scope |

### 2. Official Python documentation (docs.python.org)

The primary reference and the source of truth. Referred to below as **[Docs]**.

- *The Python Tutorial*: a quick pass to confirm the basics.
- *The Python Language Reference*: chapter 3 **Data model** (the core of "how Python works"), chapter 4 **Execution model**, chapter 5 **The import system**, and chapters 6–8 (expressions, simple statements, compound statements).
- *The Python Standard Library*: Built-in Functions, Built-in Types, Built-in Exceptions, and the modules named in each topic below.
- *Python HOWTOs*: Descriptor Guide, Functional Programming HOWTO, Sorting Techniques, Unicode HOWTO, Enum HOWTO.
- *Programming FAQ* and *Design and History FAQ*: short answers to many "why does Python do this?" questions.
- The `wiki.python.org` **TimeComplexity** page: Big-O for list, dict, set and deque operations.

### 3. *Think Python*, 3rd edition (Allen Downey)

Free online at greenteapress.com (Creative Commons licence). Optional second explanation for the basics if a Bootcamp notebook isn't clear. Referred to below as **[Think Python]**.

### 4. Internals (free)

- ***Python behind the scenes*** blog series by Victor Skvortsov (tenthousandmeters.com). Referred to below as **[PBTS #N]**, using the post number in the series.
- **Python Developer's Guide** (devguide.python.org), the internals sections.
- **`InternalDocs/`** folder in the CPython GitHub repository: the core developers' own notes on the compiler, interpreter, garbage collector, etc.
- Anthony Shaw's free Real Python article **"Your Guide to the CPython Source Code"**.

### Later (paid, optional)

Once the free plan is done, these go deeper and are better organised as reading paths:

- *Fluent Python*, 2nd edition (Luciano Ramalho, O'Reilly). Free extras: fluentpython.com and the GitHub repo `fluentpython/example-code-2e`.
- *CPython Internals* (Anthony Shaw, Real Python).

---

# Topics and learning objectives

Study the modules in order. Each module lists what I should be **able to do** at the end (learning objectives), the exact topics, and where to study them.
Tools for checking behaviour yourself: `dis.dis()` (bytecode), `sys.getsizeof()` / `tracemalloc` (memory), `timeit` (speed), `id()` / `is` (identity), `sys.getrefcount()` (references).

## Module 1 — How Python runs code

**Learning objectives**

- Explain what happens between typing `python script.py` and the program running: source → tokens → AST → bytecode → evaluation loop.
- Read simple bytecode output from `dis` and relate it back to the source.
- Explain what `.pyc` files and `__pycache__` are, and when they are reused.
- Explain the difference between CPython (the reference implementation) and Python (the language).

**Topics**

- Interpreter vs compiler; CPython's compile-then-interpret model.
- Tokenizer, parser, AST (`ast` module), compiler, code objects (`__code__`), bytecode, the evaluation loop (frames, value stack).
- `.pyc` caching and `__pycache__`.
- The REPL vs running a script vs `python -m`.
- Recent changes worth knowing exist: the specializing adaptive interpreter (3.11+), the experimental JIT and free-threaded build (3.13+).

**Sources:** [PBTS #1–#4], [Docs] `dis`, `ast`, Glossary ("bytecode"), Developer's Guide internals, `InternalDocs/`.

## Module 2 — Objects, names and memory

**Learning objectives**

- Explain "everything is an object": every object has an identity, a type and a value.
- Explain that variables are *names bound to objects*, not boxes holding values; predict the outcome of assignment, aliasing and rebinding.
- Distinguish `is` from `==`, and mutable from immutable objects, and predict the effects of mutation through an alias.
- Explain reference counting and the cyclic garbage collector, and when an object is freed.
- Explain small-integer caching and string interning, and why you must not rely on them.
- Explain shallow vs deep copy.

**Topics**

- Identity (`id`), type (`type`), value; `is` vs `==`.
- Name binding, aliasing, rebinding vs mutating; function arguments are passed by object reference ("call by sharing").
- Mutability: which built-in types are mutable or immutable, and why it matters (dict keys, default arguments, tuples containing lists).
- Reference counting, `sys.getrefcount`, the `gc` module and generational collection, `weakref`.
- Object size: `sys.getsizeof`, `tracemalloc`; the memory allocator (`pymalloc`) at a high level.
- Small-int cache, string interning (`sys.intern`).
- `copy.copy` vs `copy.deepcopy`.

**Sources:** [Bootcamp 00] (Variable Assignment), [Docs] Language Reference §3.1 "Objects, values and types", `gc`, `sys`, `copy`, `weakref`, Programming FAQ; [PBTS #5] (variables), [PBTS #6] (object system).

## Module 3 — Built-in types and their costs

**Learning objectives**

- Use every core built-in type fluently and know its main methods.
- State the time complexity of common operations on `list`, `tuple`, `dict`, `set`, `str` and `collections.deque`, and choose the right type for a job because of it.
- Explain how a dict / set works internally (hashing, hash collisions, why keys must be hashable, insertion order since 3.7).
- Explain how integers (arbitrary precision) and strings (immutable, Unicode code points) are represented, and why floats are imprecise.
- Explain the dict-vs-`if/elif` trade-off (time vs memory) and similar trade-offs, and measure them with `timeit` and `sys.getsizeof`.

**Topics**

- Numbers: `int` (arbitrary precision), `float` (IEEE 754, rounding errors), `complex`, `decimal` and `fractions` (as illustrations of exact arithmetic); integer vs true division, `//`, `%`, `**`.
- Strings: immutability, slicing, methods, f-strings and formatting, `str` vs `bytes`, Unicode and encodings.
- Lists: dynamic array, over-allocation, amortised O(1) append, O(n) insert/delete at front, slicing creates copies.
- Tuples: immutability, packing/unpacking, `namedtuple`.
- Dicts: hash tables, `__hash__` / `__eq__`, ordering, views (`keys`, `values`, `items`), `defaultdict`, `Counter`, `OrderedDict`, `ChainMap`.
- Sets and frozensets: hashing, set operations and their costs.
- `deque` vs `list` for queues.
- Booleans, `None`, truthiness (`__bool__`, `__len__`).
- Comparison operators, chained comparisons, short-circuit `and` / `or` returning operands.

**Sources:** [Bootcamp 00, 01, 17, 12] (Collections-Module, Timing your code), [Docs] Built-in Types, `collections`, Unicode HOWTO, Sorting Techniques, wiki TimeComplexity; [PBTS #8] (integers), [PBTS #9] (strings), [PBTS #10] (dictionaries); [Think Python] as an optional refresher.

## Module 4 — Statements, control flow and comprehensions

**Learning objectives**

- Write and read all control-flow statements, including `match`, and know when each is the right tool.
- Explain what a `for` loop actually does (calls `iter()` and `next()`), and the `else` clause on loops.
- Explain comprehensions vs loops vs `map`/`filter`, including their scoping and performance differences.

**Topics**

- `if` / `elif` / `else`, conditional expressions.
- `for`, `while`, `break`, `continue`, `else` on loops, `pass`.
- Structural pattern matching (`match` / `case`, 3.10+): literal, capture, class, sequence and mapping patterns, guards.
- Assignment expressions (`:=`), augmented assignment, multiple / starred unpacking.
- List, dict and set comprehensions, generator expressions; comprehension scope.
- Built-ins that express iteration: `range`, `enumerate`, `zip`, `map`, `filter`, `all`, `any`, `sorted`, `reversed`, `min` / `max` with `key`.

**Sources:** [Bootcamp 02, 09], [Docs] Tutorial ch. 4–5, Language Reference ch. 8 (compound statements), Built-in Functions, Functional Programming HOWTO.

## Module 5 — Functions, scope and closures

**Learning objectives**

- Explain that functions are first-class objects and what a function object holds (`__code__`, `__defaults__`, `__closure__`).
- Apply the LEGB scope rule, and explain `global` and `nonlocal`.
- Explain closures and late binding (e.g. lambdas created in a loop).
- Use every parameter kind correctly: positional-only (`/`), keyword-only (`*`), `*args`, `**kwargs`, defaults.
- Explain the mutable-default-argument problem and why it happens (defaults are evaluated once).

**Topics**

- `def` vs `lambda`; functions as values (passing, returning, storing in dicts — e.g. dict-based dispatch).
- Methods vs functions (bound methods).
- Scope: LEGB, `global`, `nonlocal`, `UnboundLocalError`.
- Closures and cell variables.
- Parameters and argument unpacking.
- `functools`: `partial`, `reduce`, `lru_cache` / `cache`, `wraps`.
- Recursion and the recursion limit (`sys.getrecursionlimit`); Python has no tail-call optimisation.
- Annotations / type hints as metadata (`__annotations__`), not enforcement.

**Sources:** [Bootcamp 03, 09], [Docs] Tutorial §4.7–4.9, Language Reference §4.2 "Naming and binding", `functools`, Programming FAQ (scope and default-argument entries).

## Module 6 — Iterators and generators

**Learning objectives**

- Explain the iterator protocol (`__iter__`, `__next__`, `StopIteration`) and the difference between an iterable and an iterator.
- Write generators and explain how they pause and resume (the frame is kept alive).
- Explain why generators save memory (lazy evaluation) and when that matters.
- Use `yield from`, generator `send()` / `close()`, and `itertools`.

**Topics**

- Iterable vs iterator; `iter()`, `next()`, the sentinel form of `iter`.
- Generator functions, generator expressions, laziness and memory usage.
- `yield from`, `send`, `throw`, `close`.
- `itertools`: `count`, `cycle`, `chain`, `islice`, `groupby`, `product`, `permutations`, `combinations`, `accumulate`, `tee`.

**Sources:** [Bootcamp 11], [Docs] Glossary ("iterator", "generator"), Language Reference §6.2.9 (yield expressions), `itertools`, Functional Programming HOWTO.

## Module 7 — Decorators and context managers

**Learning objectives**

- Explain that `@decorator` is syntactic sugar for `f = decorator(f)`, and write decorators with and without arguments.
- Use `functools.wraps` and explain why it is needed.
- Explain the context manager protocol (`__enter__` / `__exit__`) and what `with` guarantees, including how exceptions are passed to `__exit__`.
- Write context managers as classes and with `contextlib.contextmanager`.

**Topics**

- Function decorators, decorator factories, stacking order, class decorators.
- Built-in decorators: `@staticmethod`, `@classmethod`, `@property`, `@functools.lru_cache`.
- `with` statement, `contextlib` (`contextmanager`, `suppress`, `ExitStack`).

**Sources:** [Bootcamp 10, 17] (BONUS — With Statement Context Managers), [Docs] Language Reference §8.5 (`with`), §3.3.9 (context managers), `contextlib`, `functools`.

## Module 8 — Classes and the data model

**Learning objectives**

- Explain what happens when a class statement runs (a class is an object created at runtime).
- Explain the difference between class and instance attributes, and how attribute lookup works (instance `__dict__` → class → base classes via the MRO).
- Implement special ("dunder") methods so custom objects work with built-in operations (`len`, `+`, `==`, `in`, iteration, `with`, hashing).
- Explain inheritance, `super()` and the method resolution order (C3 linearisation).
- Explain `__slots__` and how it reduces memory.
- Use `dataclasses` and `enum` and explain what code they generate for you.

**Topics**

- Class objects vs instances, `self`, `__init__` vs `__new__`.
- Instance, class and static methods; bound vs unbound.
- Attribute lookup, `__dict__`, `__getattr__` / `__getattribute__` / `__setattr__`.
- Special methods: representation (`__repr__`, `__str__`), comparison and hashing (`__eq__`, `__lt__`, `__hash__`), arithmetic and reflected operators, container protocol (`__len__`, `__getitem__`, `__contains__`), callables (`__call__`).
- Inheritance, multiple inheritance, MRO, `super()`.
- Encapsulation conventions (`_name`, `__name` name mangling).
- `__slots__` and memory.
- Abstract base classes (`abc`) and duck typing; `typing.Protocol` as a structural alternative.
- `dataclasses`, `enum`.

**Sources:** [Bootcamp 05], [Docs] Tutorial ch. 9 (Classes), Language Reference ch. 3 (Data model — the main source for this module), `dataclasses`, `enum`, Enum HOWTO, `abc`; [PBTS #6] (object system), [PBTS #7] (attributes).

## Module 9 — Advanced object model: descriptors and metaclasses

**Learning objectives**

- Explain the descriptor protocol (`__get__`, `__set__`, `__delete__`) and data vs non-data descriptors.
- Explain how `property`, methods, `classmethod` and `staticmethod` are all implemented with descriptors.
- Explain that `type` is a metaclass and that classes are instances of `type`.
- Write a simple metaclass, and explain when `__init_subclass__` or a class decorator is the simpler choice.

**Topics**

- Descriptor protocol and attribute lookup precedence.
- `__set_name__`.
- `type(name, bases, dict)`; metaclasses, `__prepare__`, `__new__` / `__init__` on a metaclass.
- `__init_subclass__`, `__class_getitem__`.

**Sources:** [Docs] Descriptor Guide (HOWTO), Language Reference §3.3.2 (attribute access) and §3.3.3 (customizing class creation); [PBTS #7] (attributes).

## Module 10 — Errors and exceptions

**Learning objectives**

- Use `try` / `except` / `else` / `finally` correctly and explain the execution order in each case.
- Explain the built-in exception hierarchy and catch the right level of exception.
- Raise, re-raise and chain exceptions (`raise ... from ...`), and define custom exceptions.
- Explain exception groups and `except*` (3.11+).
- Read a traceback and use `pdb` to debug.

**Topics**

- `try` / `except` / `else` / `finally`, `raise`, exception chaining, `__traceback__`.
- `BaseException` vs `Exception`; common built-in exceptions.
- Custom exception classes.
- `ExceptionGroup`, `except*`, `add_note()`.
- EAFP ("easier to ask forgiveness than permission") vs LBYL ("look before you leap") — and why EAFP is idiomatic in Python.
- `pdb` / `breakpoint()`.

**Sources:** [Bootcamp 07, 12] (Python Debugger), [Docs] Tutorial ch. 8 (Errors and Exceptions), Built-in Exceptions, Language Reference §8.4 (`try`), `pdb`.

## Module 11 — Modules, packages and imports

**Learning objectives**

- Explain what `import` actually does: find → load → execute the module once → cache it in `sys.modules`.
- Explain `sys.path`, packages, `__init__.py`, relative vs absolute imports.
- Explain `if __name__ == "__main__":` and `python -m`.
- Diagnose circular imports.

**Topics**

- Modules as objects; module namespace and `__dict__`.
- `sys.modules`, `sys.path`, finders and loaders (high level), `importlib`.
- Packages, namespace packages, `__init__.py`, `__all__`.
- `__name__` and `__main__`.
- Circular imports and why they fail.

**Sources:** [Bootcamp 06], [Docs] Tutorial ch. 6 (Modules), Language Reference ch. 5 (The import system), `importlib`; [PBTS #11] (import system).

## Module 12 — Concurrency model (language level only)

**Learning objectives**

- Explain what the GIL is, why CPython has it, and its effect on CPU-bound vs I/O-bound threads.
- Explain the difference between threads, processes and async coroutines at a conceptual level.
- Explain what `async def` / `await` do and how they relate to generators.
- Know that a free-threaded (no-GIL) CPython build exists (3.13+) and what it changes.

**Topics**

- The GIL; `threading` vs `multiprocessing` (only to illustrate the GIL).
- Coroutines, `async` / `await`, the event loop (`asyncio`) at a conceptual level.
- Free-threaded CPython.

**Sources:** [Docs] `threading`, `asyncio` (conceptual overview), Python HOWTO "Python support for free threading"; [PBTS #12] (async/await), [PBTS #13] (the GIL).

## Module 13 — Measuring and optimising

**Learning objectives**

- Measure speed with `timeit` and `cProfile`, and memory with `sys.getsizeof` and `tracemalloc`, instead of guessing.
- Apply the common language-level optimisations and explain *why* each one works: dict/set lookups instead of linear searches, generators instead of lists, local-variable lookups, built-ins and comprehensions instead of manual loops, `__slots__`, string joining instead of `+=` in loops.
- Explain the time-vs-memory trade-off behind each optimisation.

**Topics**

- `timeit`, `cProfile`, `tracemalloc`, `sys.getsizeof`.
- Complexity of built-in operations (from Module 3) applied to real code.
- Why built-in functions written in C are faster than equivalent Python loops.

**Sources:** [Bootcamp 12] (Timing your code), [Docs] `timeit`, `profile`, `tracemalloc`, wiki TimeComplexity.

---

## Done when

- I can explain every learning objective above out loud without notes.
- I have done the in-scope Bootcamp exercises without looking at the solutions first.
- For every claim about speed or memory in my notes, I have checked it myself with `dis`, `timeit`, `sys.getsizeof` or `tracemalloc`.
