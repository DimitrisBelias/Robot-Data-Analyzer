# Robot-Data-Analyzer
# 🤖 Smart Factory Robot Analyzer

**Aggregative analysis system for simulated robot movements in a smart factory environment.**

Built as a coursework project for *PLH211 – Software Development Tools & Systems Programming* at the Technical University of Crete (TUC), taught by Prof. Nikos Giatrakos.

---

## Overview

This project processes and analyzes a dataset of ~10 million records capturing the movements of 100 robots navigating between production stations in a simulated smart factory. The robots transport materials across 10 stations through corridors shaped by physical obstacles (machinery, rooms, offices).

The system provides an interactive CLI that supports a range of aggregative queries — from basic speed statistics to pairwise trajectory similarity and multi-criteria dominance analysis.

Implements the full analyzer in Python with a complete software development lifecycle (logging, profiling, refactoring, unit testing). 

---

## Features & Supported Analyses

| Command | Description |
|---|---|
| `avg_speed V1 V2` | Find robots with average speed in range (V1, V2] |
| `top_speed K` | Rank the K fastest robots by average speed |
| `idle_ratio MIN MAX` | Find robots idle for a proportion of time within [MIN, MAX] |
| `collisions T1 T2` | Detect collision events in time window [T1, T2] |
| `deadlocks W` | Identify deadlock sequences of at least W consecutive steps |
| `dominance` | Multi-criteria dominance ranking (speed, idle ratio, collisions) |
| `iceberg K S` | Find robots with ≥K active samples and mean displacement > S |
| `similar_robots Θ` | Pairwise cosine similarity of velocity vectors, filtered by threshold Θ |
| `proximity_events D` | Detect moments when two robots are within distance D |

---

## Architecture

The codebase follows a **Strategy Pattern** for analysis, making it straightforward to add new query types without modifying existing code.

```
┌─────────────┐      ┌──────────────────┐      ┌─────────────────────┐
│     CLI      │─────▶│  RobotManager    │─────▶│  AnalysisStrategy   │
│  (user I/O)  │      │  (Singleton,     │      │  (abstract base)    │
│              │      │   CSV parsing)   │      ├─────────────────────┤
└─────────────┘      └──────────────────┘      │ AvgSpeedAnalysis    │
                                                │ TopSpeedAnalysis    │
┌─────────────┐                                 │ IdleRatioAnalysis   │
│   Output     │◀───── results dict             │ CollisionsAnalysis  │
│   Manager    │                                │ DeadlockAnalysis    │
│  (tabulate)  │                                │ DominanceAnalysis   │
└─────────────┘                                 │ IcebergAnalysis     │
                                                │ CosineSimilarity    │
                                                │ ProximityAnalysis   │
                                                └─────────────────────┘
```

**Key design decisions:**

- **`Robot` class** — Encapsulates per-robot time series data (positions, velocities, flags) with controlled access via properties.
- **`RobotManager` (Singleton)** — Central data store; handles CSV ingestion and dispatches analysis requests to strategy objects.
- **Helper base classes** (`AvgSpeedBase`, `IdleRatioBase`, `CollisionsBase`, `IcebergBase`) — Shared computation logic reused across multiple analysis strategies.
- **`OutputManager`** — Polymorphic result rendering using `tabulate`, adapting to dicts, lists, and nested structures.

---

## Refactoring Highlights

After profiling the initial implementation, targeted refactoring was applied to address computational bottlenecks:

### Cosine Similarity — File-Based Caching
The pairwise cosine similarity computation (`O(n²)` robot pairs) was the most expensive analysis. A **caching mechanism** was introduced: results for all pairs are computed once and written to a CSV file. Subsequent queries with any threshold Θ only require filtering the cached file — reducing repeated runs from minutes to near-instant.

### Proximity Analysis — Avoiding `sqrt()`
The Euclidean distance check `√((Δx)² + (Δy)² + (Δz)²) < D` was refactored to compare **squared distances** (`(Δx)² + (Δy)² + (Δz)² < D²`), eliminating the expensive `math.sqrt()` call on every comparison. Tuple unpacking overhead was also removed.

### Average & Top Speed — Shared Caching
A caching layer was added to `compute_avg()` so that both `AvgSpeedAnalysis` and `TopSpeedAnalysis` share a single computation pass. Repeated CLI invocations reuse the cached result without recalculating.

### Logging (YAML-based)
Four dedicated loggers were configured via YAML:

- **RobotManager logger** — data ingestion events
- **Analysis logger** — strategy execution and results
- **Compute (helper) logger** — base computation internals
- **CLI/Output logger** — user interaction and display

Two handlers are used: console output and a persistent `logme.txt` file. A root logger catches unhandled log messages above WARNING level. The formatter includes timestamps, logger names, log levels, line numbers, and messages.

---

## Tech Stack

- **Language:** Python 3 (standard library only — no NumPy, Pandas, or SciPy, per course requirements)
- **Logging:** `logging` module with YAML configuration
- **Profiling:** `cProfile`, `memory_profiler`
- **Testing:** `unittest`
- **Output:** `tabulate` for formatted CLI tables

---

## Dataset

The `smart_factory_robots.csv` dataset contains ~10M rows with the following schema:

| Column | Type | Description |
|---|---|---|
| `robotID` | int | Unique robot identifier (0–99) |
| `current_time` | float | Measurement timestamp (seconds) |
| `px`, `py`, `pz` | float | 3D position |
| `vx`, `vy` | float | Velocity components |
| `goal_status` | string | e.g., "moving to station 3", "collision detected" |
| `idle` | bool | Whether the robot is idle |
| `linear` | bool | Linear (straight-line) movement |
| `rotational` | bool | Body rotation |
| `Deadlock_Bool` | bool | Deadlock state |
| `RobotBodyContact` | bool | Physical contact with another object |

---

## Course Context

**PLH211 — Software Development Tools & Systems Programming**
School of Electrical & Computer Engineering, Technical University of Crete
Academic Year 2025–2026 | Instructor: Prof. Nikos Giatrakos

The project covered the full software development lifecycle: development → logging → profiling → refactoring → profiling (post-refactor) → unit testing, with emphasis on measurable performance improvements through profiling-guided optimization.

---

## License

This project was developed for academic purposes at TUC.
