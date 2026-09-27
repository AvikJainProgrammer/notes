# Software Learning Framework — Reference Guide

## Purpose

Two objectives:

1. **Dissection**: A framework to understand and reverse-engineer existing software — what technical decisions were made, and why — so that studying design patterns, architecture, etc. happens *in the context of a real project* instead of in the abstract.
2. **Construction**: A framework (or personal style) for actually building software, informed by what dissection reveals.

The dissection side is a **noun framework** (what is this thing made of). The construction side is a **verb framework** (how do I bring it into existence, and how do I know it's correct). They are separate and complementary.

---

## Part 1: DAPPL — the dissection framework

DAPPL = **D**omain, **A**rchitecture, **P**attern catalogue, **P**attern, **L**anguage.

For any given software system, ask:

- **Domain** — what problem/purpose is this solving? (e.g. DAW, e-commerce, form intake)
- **Architecture** — what's the structural shape? (microservices, layered, event-driven, pipes-and-filters, MVC, ECS, hexagonal, etc.)
- **Pattern Catalogue** — which body of literature are the patterns drawn from? (GoF, Fowler's enterprise patterns, POSA, microservices patterns, etc.) This is its own axis because different eras/domains produced different, non-overlapping pattern vocabularies.
- **Pattern** — the specific named pattern in use (Observer, DTO, Circuit Breaker, Saga, Repository, etc.)
- **Language** — the implementation language, which acts less like an independent field and more like a **lens**: it can make a pattern nearly invisible (e.g. Strategy/Visitor collapse into "just pass a function" in languages with first-class functions) or force it into heavy boilerplate (classic OOP languages like Java).

### Key properties of DAPPL

1. **Fractal / recursive**: A system is either a single component or a composition of smaller systems. Each node — whether it's "the whole platform," "one microservice," "one module inside that service," or "one class" — gets its own DAPPL tuple. Values can differ at every level (a system can be polyglot, use different architectures per service, pull from different pattern catalogues per layer).
2. **Edges matter as much as nodes**: how sibling nodes *talk to each other* (REST, gRPC, message queue, shared DB, event bus) is a distinct concern from what's happening inside a node. Worth tracking explicitly — either as a 6th field, or folded into the parent node's Architecture field.
3. **Architecture and Domain overlap**: they're not always cleanly separable in practice (e.g. "microservices" is partly a domain-shaped decision — many teams, independent deploys — and partly an architectural style).

### Why this helps

Once you can label an existing codebase this way, you know *which* pattern catalogue and *which* architectural literature to actually go study — instead of learning patterns in a vacuum disconnected from a real project.

---

## Part 2: Frameworks worth studying alongside / against DAPPL

These are established frameworks that overlap with what DAPPL is trying to do — useful for stress-testing and evolving DAPPL:

- **C4 Model** — Context → Container → Component → Code. A well-known way of formalizing "redraw the system at each zoom level," conceptually very close to DAPPL's recursion.
- **Domain-Driven Design (DDD) — bounded contexts & context maps** — each bounded context can have its own internal model/language; contexts communicate via explicitly defined context maps. Relevant both to the Domain field and to the "edges between nodes" problem.
- **arc42** — a template for documenting software architecture (worth a look for how practitioners structure this).
- **Architecture Decision Records (ADRs)** — lightweight docs that capture *why* a technical decision was made at a point in time; useful for the dissection side because they capture reasoning DAPPL alone doesn't (the "why," not just the "what").
- **Code reading / reverse engineering techniques** — general practices for onboarding onto unfamiliar codebases (tracing entry points, following data flow, static analysis tools, dependency graphs).

---

## Part 3: Pattern catalogues (and the books behind them)

There is no single canonical list of design patterns — several catalogues exist, each written for a different era/problem domain:

| Catalogue | Book | Focus | Example patterns |
|---|---|---|---|
| GoF | *Design Patterns: Elements of Reusable Object-Oriented Software* (Gamma, Helm, Johnson, Vlissides, 1994) | General OOP (C++/Smalltalk-era) | Singleton, Observer, Strategy, Factory, Decorator |
| Fowler (PoEAA) | *Patterns of Enterprise Application Architecture* (Martin Fowler, 2002) | Enterprise/distributed apps | DTO, Repository, Unit of Work, Service Layer, Gateway |
| Core J2EE Patterns | *Core J2EE Patterns* (Sun, ~2001) | Java web/enterprise | DTO, Front Controller, DAO |
| POSA | *Pattern-Oriented Software Architecture* (Buschmann et al.) | Systems/distributed | Broker, Reactor, Layers |
| Microservices patterns | *Microservices Patterns* (Chris Richardson) | Microservices-specific | Saga, Circuit Breaker, API Gateway |

Notes:
- **DTO is not in GoF** — it's a Fowler/enterprise-era pattern, which is why it felt "missing" from the classic catalogue.
- GoF is OOP-flavored because that was the dominant paradigm at the time; **functional programming has its own separate pattern vocabulary** (monads, functors, combinators) solving overlapping problems without objects — worth treating as a branch under the Language lens rather than ignoring.
- **Game development typically doesn't use MVC.** The dominant pattern is **ECS (Entity-Component-System)**: Entity = ID, Component = pure data, System = logic operating over entities with given components. Favored for performance (cache-friendly, frame-by-frame updates over thousands of objects) and composition-over-inheritance.
- **UI-heavy applications** commonly use MVC variants: MVP and MVVM (common in Android, WPF/desktop). Modern component-based frontends (React, etc.) often don't map cleanly onto MVC at all.

---

## Part 4: Development process frameworks (the "verb" side)

Separate from DAPPL — these govern *how* code gets written and shipped, not what it's made of.

### Macro / team-level process
- **Waterfall** — sequential, plan-everything-upfront (mostly legacy).
- **Agile / Scrum** — iterative sprints, adapt as you learn.
- **Kanban** — continuous flow, no fixed sprint boundaries.

### Micro / code-writing discipline
- **TDD (Test-Driven Development)** — Red → Green → Refactor: write a failing test, write minimal code to pass, refactor.
- **BDD (Behavior-Driven Development)** — like TDD but specs are written as human-readable behavior statements ("Given/When/Then"), bridging business and dev communication.
- **DDD as a practice** (not just bounded-context theory) — model-first: deeply understand the domain and encode it directly into code structure/language.

### Principles / heuristics (not sequences, more like constraints you self-check against)
- **SOLID**
- **DRY** (Don't Repeat Yourself)
- **KISS** (Keep It Simple)
- **YAGNI** (You Aren't Gonna Need It)

### Collaboration / delivery workflow
- **Git branching strategies** — GitFlow, trunk-based development
- **Code review practices**
- **CI/CD pipelines**

---

## Part 5: How the two sides connect

- Use **DAPPL** to dissect real, existing projects → this tells you which architecture, pattern catalogue, and specific patterns are actually relevant to study, in context, instead of learning patterns abstractly.
- Use the **development process frameworks** (TDD/BDD/Agile/etc.) to actually build variations of what you dissected — turning passive understanding into active practice.

---

## Part 6: Next steps / open items

- Stress-test DAPPL against 2–3 real systems, at least 2–3 zoom levels deep each, to see where the framework holds up and where it breaks.
- Decide how to formally treat "edges" (inter-node communication) — 6th field vs. folded into Architecture.
- Decide whether to build out the "verb" (process) side with the same rigor as DAPPL, or keep it deliberately loose/pragmatic (process is more team/context-dependent and risks becoming cargo-culting if over-formalized).
- Explore functional-programming pattern vocabulary as a counterweight to the OOP-heavy catalogues above.

---

*This document is a seed reference, not a finished framework — intended to be expanded as DAPPL gets stress-tested against real projects.*