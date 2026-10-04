# DAPPL Dissection Guide: `first` (Pac-Man in Godot)

A step-by-step guide for dissecting the project at `/Users/avikjain/first` using the [DAPPL framework](../dappl.md).

**How to use this guide**

- Work through the steps in order. Each step gives you questions to answer and tells you where to look.
- Try each question yourself first. Answers are hidden in collapsible **"Check your answer"** blocks.
- Write your own answers in a copy of the worksheet (see [Worksheet template](#worksheet-template)) before opening them.
- The [Reading list](#reading-list) at the end is grouped by DAPPL field, so when a step says "read X", you can find it there.

---

## Contents

1. [Project map](#1-project-map)
2. [Step 1 — Recon: find the entry point and trace one frame](#step-1--recon-find-the-entry-point-and-trace-one-frame)
3. [Step 2 — Level 0: DAPPL for the whole game](#step-2--level-0-dappl-for-the-whole-game)
4. [Step 3 — Edges: how the parts talk](#step-3--edges-how-the-parts-talk)
5. [Step 4 — Level 1: DAPPL for each script](#step-4--level-1-dappl-for-each-script)
6. [Step 5 — Level 2: zoom into the ghost AI](#step-5--level-2-zoom-into-the-ghost-ai)
7. [Step 6 — Pattern index](#step-6--pattern-index)
8. [Step 7 — Stress-test DAPPL itself](#step-7--stress-test-dappl-itself)
9. [Step 8 — Construction exercises (the verb side)](#step-8--construction-exercises-the-verb-side)
10. [Worksheet template](#worksheet-template)
11. [Reading list](#reading-list)

---

## 1. Project map

| File | Lines | Role (one line) |
|---|---|---|
| `project.godot` | 31 | Engine config: window size, main scene |
| `main.tscn` | 6 | The only real scene: one `Node2D` with `main.gd` attached |
| `node_2d.tscn` | 3 | Empty scene, not referenced anywhere (leftover) |
| `main.gd` | 283 | Game controller: builds everything, game state machine, scoring, HUD |
| `maze.gd` | 146 | Level data (ASCII map), grid queries, pellets, drawing the maze |
| `actor.gd` | 55 | Base class: grid-locked movement between tile centers |
| `player.gd` | 81 | Pac-Man: input, eating, drawing |
| `ghost.gd` | 196 | Ghosts: modes, AI target selection, drawing |

Runtime scene tree (built in code by `main.gd:_ready`, not in the editor):

```text
Main (Node2D, main.gd)
├── Maze (Node2D, maze.gd)          position = (0, TOP)
│   ├── Player (Node2D, player.gd)
│   ├── Ghost 0 "blinky" (ghost.gd)
│   ├── Ghost 1 "pinky"
│   ├── Ghost 2 "inky"
│   └── Ghost 3 "clyde"
├── CanvasLayer (HUD)
│   ├── score_label
│   ├── level_label
│   └── msg_label
└── (temporary score popup Labels, freed by a Tween)
```

Class hierarchy:

```text
Node2D
├── Maze
├── Actor
│   ├── Player
│   └── Ghost
└── (main.gd, no class_name)
```

---

## Step 1 — Recon: find the entry point and trace one frame

Before labelling anything, understand how the program runs. This is the "code reading / reverse engineering" technique from Part 2 of `dappl.md`.

**Do:**

1. Open the project in Godot and play it. Note every behaviour you see: ghosts leaving the house one at a time, ghosts turning blue, ghosts reversing direction, the tunnel on row 9, levels speeding up.
2. Find the entry point. Start at `project.godot` and follow the chain until you reach code.
3. Trace **one frame** of normal play: which function does Godot call, and in what order do things update?
4. Trace **one event**: the player eats a power pellet. List every function that runs, across every file.

**Questions:**

- Q1.1 What is the chain from engine start to the first line of game code?
- Q1.2 Which nodes have their own `_process`, and which are updated by someone else? Why might the author do that?
- Q1.3 In one frame, does the player move before or after the ghosts? Does it matter?

<details>
<summary>Check your answer</summary>

- **Q1.1** `project.godot:14` sets `run/main_scene="res://main.tscn"` → `main.tscn` has one `Node2D` with `main.gd` attached → Godot calls `main.gd:_ready()` (`main.gd:38`), which builds the maze, player, ghosts and HUD in code, then calls `_new_game()`.
- **Q1.2** Only `Main` (`main.gd:131`) and `Maze` (`maze.gd:85`, just for power-pellet blinking) have `_process`. Player and ghosts have `tick(delta)` methods that `Main` calls explicitly from `_tick_play` (`main.gd:169`). This gives **one place that controls update order**, and makes pausing trivial: if `Main` doesn't call `tick`, nothing moves.
- **Q1.3** Order in `_tick_play`: release ghosts → fright timer / scatter-chase phase → `player.tick` → each `ghost.tick` → `_check_collisions`. Collisions are checked once, after everyone has moved, so the check sees a consistent snapshot.
- **Power pellet trace:** `Actor.step` crosses a tile center → `Player._at_center` (`player.gd:53`) emits `ate(tile)` → `Main._on_player_ate` (`main.gd:207`) → `Maze.eat_at` returns `2` → score += 50, `fright_t = FRIGHT_TIME` → every `Ghost.set_frightened()` (`ghost.gd:59`) → active ghosts `reverse()`. Next frames: `_tick_play` counts `fright_t` down, sets `fright_flash`, and clears `frightened` at 0.

</details>

**Read before moving on:** Godot docs — *Nodes and Scenes*, *Idle and Physics Processing*; Nystrom — *Game Loop*, *Update Method*.

---

## Step 2 — Level 0: DAPPL for the whole game

Fill in one DAPPL tuple for the game as a whole.

**Questions:**

- **Domain** — What problem is this solving? What constraints does that domain impose (timing, frame rate, grids, input)?
- **Architecture** — What is the structural shape? Is it MVC? ECS? Layered? Something else? Where does data live, and who is allowed to change it?
- **Pattern catalogue** — Which body of literature would a game programmer draw from? Which GoF patterns also show up?
- **Pattern** — Name the 3–4 biggest patterns at this level.
- **Language** — GDScript. What does the language/engine give you for free that would be boilerplate elsewhere?

<details>
<summary>Check your answer</summary>

| Field | Value |
|---|---|
| **Domain** | Real-time arcade game (a Pac-Man clone). Tile-based world, frame-by-frame simulation, keyboard input, fixed rules copied from a known original (ghost personalities, scatter/chase timing, fright mode). |
| **Architecture** | **Godot scene tree** (nodes composed into a tree; each node has one script) + a **central game controller** (`main.gd`) that owns the game loop, state machine and all cross-object rules. Not MVC (every class draws itself), not ECS (logic lives in objects, uses inheritance). |
| **Pattern catalogue** | Mainly **Game Programming Patterns** (Nystrom). Also **GoF** (Template Method, State, Observer, Mediator, Composite). |
| **Pattern** | Game Loop + Update Method, finite state machine (`main.gd` `State` enum), Mediator (`main.gd` coordinates everyone), Composite (the scene tree), Observer (Godot signal `ate`). |
| **Language** | GDScript: Python-like syntax, optional static typing (`:=`, `: Vector2i`), built-in signals (Observer without boilerplate), `enum` + `match` for state machines, value types like `Vector2i`, `preload`, engine callbacks (`_ready`, `_process`, `_draw`). |

**Things worth noticing:**

- Everything is built **in code** rather than in the Godot editor (`main.tscn` is 6 lines). That's unusual for Godot projects; ask yourself what you gain (everything visible in one file, easy to read) and lose (no visual editing, no reusable `.tscn` for a Ghost).
- Graphics are all **procedural** (`_draw` functions), there are no image assets.

</details>

**Read:** *Godot's design philosophy*; *Why isn't Godot an ECS-based game engine?* (important — see [Step 7](#step-7--stress-test-dappl-itself)); C4 model intro (compare its "zoom levels" to what you're doing).

---

## Step 3 — Edges: how the parts talk

`dappl.md` says edges matter as much as nodes. For each pair of scripts that interact, write down **the mechanism** (direct method call, property write, signal, shared object, parent/child tree).

**Questions:**

- Q3.1 How does `Main` find out the player stepped on a pellet? Why is this one edge different from all the others?
- Q3.2 How does a ghost know where the player is? Who gave it that reference?
- Q3.3 Everyone calls into `Maze`. What role does `Maze` play for the other objects?
- Q3.4 Find a place where one object reaches directly into another's internal state. Is that a problem here?

<details>
<summary>Check your answer</summary>

| From → To | Mechanism | Where |
|---|---|---|
| Player → Main | **Signal** `ate(t)` | declared `player.gd:4`, emitted `player.gd:54`, connected `main.gd:47` |
| Main → Player / Ghost | Direct method calls (`tick`, `reset`, `release`, `set_frightened`, `on_phase_switch`) | `main.gd:111-204` |
| Main → Ghost | Direct property writes (`chase`, `speed_scale`, `fright_flash`, `frightened`) | `main.gd:121-122`, `179`, `182`, `203` |
| Main → Player | Direct property writes (`dying`, `death_t`) | `main.gd:244-245` |
| Main → Ghost / Player | **Dependency injection** of `maze`, `player`, `blinky` references at construction | `main.gd:45`, `57-58`, `65-66` |
| Actor / Player / Ghost → Maze | Direct queries (`tile_center`, `wrap_tile`, `player_can_walk`, `walls`, `house`) | `actor.gd`, `player.gd:55`, `ghost.gd:142-148` |
| Ghost → Player, Ghost → Blinky | Read-only access to `tile` and `dir` | `ghost.gd:151-164` |
| Maze → children | Scene tree parent: children's positions are relative to the maze's `(0, TOP)` offset | `main.gd:41`, `46`, `63` |

- **Q3.1** It's the only **signal**. Player doesn't need to know `Main` exists; it just announces "I reached tile t". Every other edge is `Main` calling downward. This is the common Godot guideline: **"call down, signal up"**.
- **Q3.2** `Main._ready` sets `g.player = player` and `g.blinky = ghosts[0]`. This is manual **property (setter) injection**, and `main._ready` is the **composition root** (the one place where the object graph is wired together).
- **Q3.3** `Maze` is a shared **world model / query service**: the single source of truth for walls, pellets and grid maths.
- **Q3.4** `main.gd:244-245` sets `player.dying` and `player.death_t` directly, even though `Player` has a `reset()` method for the opposite. A `player.die()` method would keep that knowledge inside `Player`. Small project, so it's a tradeoff, not a bug.

</details>

**Read:** Godot docs — *Using signals*, *Scene organization* (covers "call down, signal up" and dependency injection between nodes); Mark Seemann — *Composition Root*.

---

## Step 4 — Level 1: DAPPL for each script

Now zoom in one level (fractal property of DAPPL). Fill one tuple **per script**. Domain here means "what job does this node do inside the game".

### 4.1 `main.gd`

- What are all the jobs this file does? Count them.
- Find the state machine. How are states represented? How are transitions triggered?
- How are timers implemented? (Look for `state_t`, `fright_t`, `phase_t`.) Godot has a `Timer` node — why might the author not use it?
- `SCHEDULE`, `RELEASE_AT`, `SCATTER_CORNERS` — what's the idea behind putting rules in constant tables?

<details>
<summary>Check your answer</summary>

| Field | Value |
|---|---|
| Domain | Game rules and flow: rounds, lives, levels, score, scatter/chase schedule, fright mode, collisions, HUD |
| Architecture | Central controller / orchestrator at the root of the tree |
| Pattern catalogue | GoF + Game Programming Patterns |
| Pattern | **Mediator** (all cross-object rules live here), **State** as an enum FSM (`main.gd:3`, `131-166`), **Game Loop / Update Method** (`_tick_play`), **composition root** (`_ready`), **table-driven rules** (`main.gd:10-14`) |
| Language | `enum` + `match` for the FSM; float countdowns decremented by `delta`; `create_tween()` for the popup (`main.gd:258`); `CanvasLayer` so the HUD doesn't move with the maze |

- **Jobs:** construction/wiring, game FSM, timers, ghost release, scatter/chase phases, fright mode, scoring, collisions, HUD building, HUD drawing (lives). That's a lot — this is a **"god object" risk**, which is why it's worth practising splitting it (see [Step 8](#step-8--construction-exercises-the-verb-side)).
- **Manual timers** keep everything tied to the same `delta` and pause automatically when `_tick_play` isn't called. A `Timer` node would keep running unless you pause it too.

</details>

### 4.2 `maze.gd`

- How is the level defined? What would you change to make a new level?
- Why are `walls`, `pellets`, `house` dictionaries instead of 2D arrays?
- How does the tunnel on row 9 work?
- What does `queue_redraw()` do, and why is it called in `_process` every frame?

<details>
<summary>Check your answer</summary>

| Field | Value |
|---|---|
| Domain | The world: layout, walkability rules, pellets, grid ↔ pixel conversion |
| Architecture | Data + query service node shared by all actors |
| Pattern catalogue | Game Programming Patterns; general data-driven design |
| Pattern | **Data-driven level** (ASCII `LAYOUT`, `maze.gd:8-30`, parsed in `_ready`), **grid as a sparse set** (dict keyed by `Vector2i`), immediate-mode drawing |
| Language | `Dictionary` used as a set (`walls[t] = true`, `.has()`, `.erase()` returns a bool used in `eat_at`); `posmod` for wrap-around; `_draw` + `queue_redraw` |

- **Tunnel:** `wrap_tile` (`maze.gd:90`) uses `posmod`, so column `-1` becomes column `18`. Row 9 has no walls at its ends, so actors walk off one side and appear on the other.
- **`queue_redraw` every frame** is only needed for the blinking power pellets; it redraws the whole maze 60 times a second. Fine at this size, but a question to think about.
- **Dictionaries instead of arrays:** membership checks read nicely (`walls.has(t)`), and pellets can be removed without "empty" markers. The cost is hashing instead of indexing.

</details>

### 4.3 `actor.gd`

- `tile`, `dir`, `edge_t` — explain the movement model in your own words.
- `step()` calls `_at_center()`, and `_at_center()` does nothing here. What pattern is that?
- Why does `step` use a `while` loop instead of moving once?
- Why does `reverse()` refuse to reverse at the tunnel seam?

<details>
<summary>Check your answer</summary>

| Field | Value |
|---|---|
| Domain | Grid-locked movement shared by player and ghosts |
| Architecture | Base class in an inheritance hierarchy |
| Pattern catalogue | GoF |
| Pattern | **Template Method** — `step()` is the fixed algorithm; `_at_center()` (`actor.gd:54`) is the hook subclasses override to make decisions |
| Language | GDScript `class_name` + `extends` for inheritance; virtual-by-default methods; `_` prefix is a naming convention for "private", not enforced |

- **Movement model:** an actor is always on an edge between two tile centers; `edge_t` is progress 0→1. Decisions (turn, stop) only happen at a tile center.
- **`while` loop:** if `speed * delta` is bigger than the distance left to the next center (e.g. an eaten ghost at 260 px/s during a slow frame), it must pass through the center, make a decision, and keep going. Without the loop, fast actors would skip decision points and go through walls.
- **Link to your notes:** this is the same shape as `create_form()` in your Abstract Factory example in [factory_converstaion.md](../python_object_oriented/factory_converstaion.md) — a base-class method that calls abstract steps.

</details>

### 4.4 `player.gd`

- How does "buffered" input work (press a direction before you reach the turn)?
- What's the difference between `desired` and `dir`?
- Why does the player emit a signal instead of calling `main` or `maze.eat_at` directly?

<details>
<summary>Check your answer</summary>

| Field | Value |
|---|---|
| Domain | The player character: input, turning rules, animation |
| Architecture | Subclass of `Actor`; leaf node |
| Pattern catalogue | GoF + Game Programming Patterns |
| Pattern | Template Method hook override (`_at_center`, `player.gd:53`), **Observer** via signal `ate`, input buffering |
| Language | `signal` keyword; `Input` singleton polled every frame; `_draw` for the animated mouth |

- `desired` is what the player last pressed; `dir` is where they're actually going. At each tile center, `_at_center` tries `desired` first, then falls back to continuing in `dir`. That gives the forgiving "pre-turn" feel.
- Immediate reversal is special-cased in `tick` (`player.gd:28-30`), because reversing doesn't need to wait for a tile center.
- **Signal:** keeps `Player` reusable and ignorant of scoring rules.

</details>

### 4.5 `ghost.gd`

- List the ghost's modes. Draw the state diagram: which function causes each transition?
- `frightened` is a separate `bool`, not a `Mode`. Why?
- `personality` is an `int` checked with `match`. What pattern could replace it, and would it be worth it?

<details>
<summary>Check your answer</summary>

| Field | Value |
|---|---|
| Domain | Enemy behaviour: leaving the house, chasing, fleeing, returning when eaten |
| Architecture | Subclass of `Actor`; leaf node; configured by `Main` |
| Pattern catalogue | Game Programming Patterns + GoF |
| Pattern | **Enum FSM** (`Mode`, `ghost.gd:4`, driven in `_at_center` `ghost.gd:88-112`), **Template Method** hook override, **Type Object**-ish configuration (`personality`, `body_color`, `scatter_corner` set from tables in `main.gd`), **Strategy candidate** (`_target_tile`) |
| Language | `enum` + `match`; typed arrays (`Array[Vector2i]`); `pick_random()` |

State diagram:

```text
IN_HOUSE --release()--> EXITING --reach door_exit--> ACTIVE
                                                        |
                                             set_eaten() (player eats frightened ghost)
                                                        v
EXITING <--reach house_center-------------------- EATEN
```

- `frightened` is an **overlay** on `ACTIVE` (same movement, different target choice and speed). Making it a separate flag avoids duplicating `ACTIVE`. The flip side: `set_frightened` has to guard `EATEN` by hand (`ghost.gd:60`). This is exactly the "multiple state machines / hierarchical states" discussion in Nystrom's *State* chapter.
- `personality` → a **Strategy** (one object or `Callable` per ghost that returns a target tile). In GDScript you could store a `Callable` — this is the "Language as a lens" point in `dappl.md`: Strategy collapses to "pass a function".

</details>

---

## Step 5 — Level 2: zoom into the ghost AI

DAPPL is fractal, so zoom in again: treat the ghost AI (`_choose`, `_can_walk`, `_target_tile`) as its own system.

**Questions:**

- How does a ghost choose a direction at an intersection? Is it pathfinding?
- Why can't a ghost turn around (except when forced)?
- Explain each personality's target in plain English. Why does Inky need a reference to Blinky?
- In the original arcade game, is distance measured the same way?

<details>
<summary>Check your answer</summary>

| Field | Value |
|---|---|
| Domain | Classic Pac-Man ghost AI (the rules come from the 1980 arcade game) |
| Architecture | Pure function-style decision step inside the FSM |
| Pattern catalogue | Game AI techniques (not a design-pattern book) |
| Pattern | **Greedy target-tile steering**: at each tile center, pick the allowed direction whose next tile is closest (straight-line) to a target tile. No reversing. No pathfinding (no A*, no BFS). |
| Language | `Vector2(...).length_squared()` to avoid `sqrt`; `INF` as a starting minimum |

- **Blinky:** target = player's tile. **Pinky:** 4 tiles ahead of the player. **Inky:** take the point 2 tiles ahead of the player, draw a line from Blinky through it, and double it — so Inky depends on Blinky's position. **Clyde:** chase when more than 8 tiles away, otherwise go to his scatter corner.
- **Frightened:** random choice (`ghost.gd:128-129`).
- The emergent "personalities" come from **different targets with the same simple rule**. That's the key insight — read *Understanding Pac-Man Ghost Behavior* for the original design.
- Differences from the arcade (worth checking yourself in *The Pac-Man Dossier*): the original's tie-break order for directions, and the famous Pinky/Inky "up" overflow bug, are not reproduced here.

</details>

---

## Step 6 — Pattern index

Fill this in yourself first, then compare. It's the payoff table from `dappl.md` ("now you know which catalogue to study").

| Pattern | Where | Catalogue | Study |
|---|---|---|---|
| Game Loop | Godot engine → `main.gd:_process` | Game Programming Patterns | Nystrom: *Game Loop* |
| Update Method | `Player.tick`, `Ghost.tick` called by `_tick_play` | Game Programming Patterns | Nystrom: *Update Method* |
| State (enum FSM) | `main.gd` `State`, `ghost.gd` `Mode` | GoF / Game Programming Patterns | Nystrom: *State*; refactoring.guru: State |
| Template Method | `Actor.step` → `_at_center` hook | GoF | refactoring.guru: Template Method |
| Observer | Signal `ate` | GoF (built into Godot) | Nystrom: *Observer*; Godot: *Using signals* |
| Mediator | `main.gd` coordinating player, ghosts, maze | GoF | refactoring.guru: Mediator |
| Composite | Godot scene tree | GoF | refactoring.guru: Composite; Godot: *Nodes and Scenes* |
| Dependency injection / composition root | `main.gd:_ready` | Enterprise / general OO | Seemann: *Composition Root* |
| Type Object (partial) | ghost config tables in `main.gd:13-14` | Game Programming Patterns | Nystrom: *Type Object* |
| Strategy (candidate) | `Ghost._target_tile` personalities | GoF | refactoring.guru: Strategy |
| Data-driven level | `Maze.LAYOUT` | General game dev | Nystrom: *Type Object* intro; Red Blob Games grids |
| Spatial grid | `Maze` dicts keyed by tile | Game Programming Patterns | Nystrom: *Spatial Partition* |
| Tween | score popup `main.gd:252-261` | Engine feature | Godot: `Tween` class reference |

---

## Step 7 — Stress-test DAPPL itself

`dappl.md` Part 6 says: stress-test DAPPL against real systems and see where it breaks. This project gives you concrete findings. Write your own conclusions, then compare.

1. **"Game development typically doesn't use MVC. The dominant pattern is ECS."** — Does this project use ECS? Does Godot? What would you change in `dappl.md`?
2. **Edges as a 6th field** — After Step 3, would a separate "Edges" field have helped, or did it fit under Architecture?
3. **Language as a lens** — List every pattern that GDScript/Godot made nearly invisible.
4. **Engine as a layer** — Is "Godot" a Language, an Architecture, or something DAPPL is missing (a *Framework/Platform* field)?

<details>
<summary>Check your answer</summary>

1. No ECS here, and Godot is deliberately not ECS: it uses a **scene tree of nodes with inheritance and composition**. The DAPPL note is true for some engines (Unity DOTS, Bevy, custom C++ engines) but not all. Suggested edit: "Game engines use either ECS (Bevy, Unity DOTS) or node/scene-tree composition (Godot, classic Unity GameObjects)."
2. Edges were a large part of the understanding here ("call down, signal up", DI, the one signal). That argues for a separate Edges field.
3. Observer → `signal` keyword; Composite → the scene tree; Game Loop → engine calls `_process`; State → `enum` + `match`; Strategy → could be a `Callable`.
4. Godot decides the architecture (scene tree, callbacks, drawing model) more than GDScript does. That's a case for adding a **Framework/Platform** field, or treating "Language" as "Language + runtime".

</details>

---

## Step 8 — Construction exercises (the verb side)

Turn dissection into practice (`dappl.md` Part 5). Copy the project first so you don't break the original.

1. **Strategy:** replace `personality: int` and the `match` in `_target_tile` with a `Callable` (or small classes) per ghost. Compare readability. Which version would you keep?
2. **Factory — does it earn its place?** Extract ghost creation (`main.gd:49-66`) into a `GhostFactory` or `Ghost.create(...)` static function. Use your own test from *Observation 2* in [factory_converstaion.md](../python_object_oriented/factory_converstaion.md): does it hide real complexity, or does it add zero value?
3. **State objects:** turn `Ghost.Mode` into State classes (one per mode). Count lines before and after. When would this become worth it?
4. **Split the god object:** move the scatter/chase schedule and fright timer out of `main.gd` into a separate `GhostDirector` node. Decide how it talks to `Main` (signal up? method call down?).
5. **Encapsulation fix:** add `Player.die()` and remove the direct writes at `main.gd:244-245`.
6. **Timestep:** move ticking to `_physics_process`. Read *Fix Your Timestep!* first and write down what changes and why.
7. **Scene files:** turn `Ghost` into a `ghost.tscn` built in the editor. What got easier, what got harder?
8. **Tests:** install a Godot unit-testing addon (e.g. GUT or gdUnit4) and write tests for `Maze.wrap_tile`, `Maze.eat_at` and `Ghost._target_tile`. Try writing one test **before** a change (TDD).
9. **Housekeeping:** `node_2d.tscn` is unused. Decide whether to delete it, and write a one-paragraph ADR (Architecture Decision Record) for one decision in this project, e.g. "build the scene tree in code instead of the editor".

---

## Worksheet template

Copy this for each node you dissect.

```markdown
### Node: <name>  (zoom level: 0 / 1 / 2)

| Field | Value | Evidence (file:line) |
|---|---|---|
| Domain | | |
| Architecture | | |
| Pattern catalogue | | |
| Pattern(s) | | |
| Language (lens) | | |
| Edges (in / out) | | |

Why do I think the author chose this?

What would I do differently?

What should I read next?
```

---

## Reading list

Grouped by DAPPL field. Start with the **★** items.

### Domain — Pac-Man and grid games

- ★ Chad Birch, *Understanding Pac-Man Ghost Behavior* — how the four ghost targets create personalities. <https://gameinternals.com/understanding-pac-man-ghost-behavior>
- Jamey Pittman, *The Pac-Man Dossier* — the full rules of the original arcade game (timings, speeds, bugs). <https://pacman.holenet.info/>
- Red Blob Games, grids and graphs — how to think about tile maps, neighbours and distances. <https://www.redblobgames.com/pathfinding/grids/graphs.html>

### Architecture — Godot and games in general

- ★ Godot docs, *Nodes and Scenes*. <https://docs.godotengine.org/en/stable/getting_started/step_by_step/nodes_and_scenes.html>
- ★ Godot docs, *Godot's design philosophy*. <https://docs.godotengine.org/en/stable/getting_started/introduction/godot_design_philosophy.html>
- ★ Juan Linietsky, *Why isn't Godot an ECS-based game engine?* <https://godotengine.org/article/why-isnt-godot-ecs-based-game-engine/>
- Godot docs, *Scene organization* (call down / signal up, dependency injection between nodes). <https://docs.godotengine.org/en/stable/tutorials/best_practices/scene_organization.html>
- Godot docs, *Idle and Physics Processing*. <https://docs.godotengine.org/en/stable/tutorials/scripting/idle_and_physics_processing.html>
- Glenn Fiedler, *Fix Your Timestep!* <https://gafferongames.com/post/fix_your_timestep/>
- C4 model (for drawing the zoom levels). <https://c4model.com/>
- Architecture Decision Records. <https://adr.github.io/>

### Pattern catalogue and patterns

- ★ Robert Nystrom, *Game Programming Patterns* (free online). <https://gameprogrammingpatterns.com/>
  - Chapters for this project: *Game Loop*, *Update Method*, *State*, *Observer*, *Type Object*, *Component*, *Spatial Partition*.
- refactoring.guru — readable GoF explanations with diagrams: <https://refactoring.guru/design-patterns>
  - Template Method, State, Strategy, Observer, Mediator, Composite.
- Mark Seemann, *Composition Root*. <https://blog.ploeh.dk/2011/07/28/CompositionRoot/>
- GoF, *Design Patterns* (the book) — for the original definitions once the above make sense.

### Language — GDScript

- ★ Godot docs, *GDScript reference*. <https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html>
- Godot docs, *Static typing in GDScript*. <https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/static_typing.html>
- Godot docs, *Using signals*. <https://docs.godotengine.org/en/stable/getting_started/step_by_step/signals.html>
- Godot docs, *Custom drawing in 2D* (`_draw`, `queue_redraw`). <https://docs.godotengine.org/en/stable/tutorials/2d/custom_drawing_in_2d.html>
- Godot docs, *Godot notifications* (`_ready`, `_process` and friends). <https://docs.godotengine.org/en/stable/tutorials/best_practices/godot_notifications.html>
- Godot class reference, `Tween`. <https://docs.godotengine.org/en/stable/classes/class_tween.html>

### Suggested order

1. Godot *Nodes and Scenes* + *GDScript reference* (enough to read every line).
2. Nystrom *Game Loop*, *Update Method*, *State*, *Observer*.
3. *Understanding Pac-Man Ghost Behavior*.
4. *Why isn't Godot an ECS-based game engine?* + *Scene organization*.
5. refactoring.guru: Template Method, Strategy, Mediator.
6. Everything else as the exercises need it.
