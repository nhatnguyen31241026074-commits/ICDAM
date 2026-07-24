# AGENTS.md — ICDAM

> **Read this first.** This file is the onboarding brief for any developer or AI agent
> picking up this repo. It explains what the project is, what actually works today,
> how the pieces fit, and where to continue. Keep it updated as the code changes.

---

## 1. What this project is

**ICDAM** is a research prototype of a **hybrid LLM + solver system for Resource-Constrained
Project Scheduling (RCPSP)**. The idea being explored (aimed at an "ICDAM 2025" paper):

> Language-model *agents* negotiate/propose a project schedule, and a classical
> **OR-Tools CP-SAT** solver acts as a **verifier and repairer** — checking precedence and
> resource-capacity feasibility, feeding violations back to the LLM to fix, and falling back
> to a provably-optimal solver schedule if the LLM keeps failing. This "propose → verify →
> repair" pattern is referred to in code as the **OptiMUS-style loop**.

It runs on standard **PSPLib `j30`** benchmark instances so results are comparable to literature.

## 2. Current state (what works vs. what's aspirational)

| Component | State |
|-----------|-------|
| PSPLib `.sm` loader (`utils/psplib_loader.py`) | ✅ Works |
| CP-SAT solver + feasibility check (`solvers/rcpsp_solver.py`) | ✅ Works — this is the reliable workhorse |
| Agents + negotiation (`agents/basic_agents.py`) | ✅ Runs; LLM proposals + mock fallback |
| LLM brain (`agents/llm_brain.py`) | ✅ Gemini client with **mock fallback** when no API key / on 429 |
| Self-correction (verify→feedback→repair) loop | ✅ Implemented in `main_simulation.py` |
| Metrics collection (`utils/metrics.py`, `SimulationMetrics`) | ✅ Basic makespan/feasibility/gap |
| Evaluation tables / baselines / ablations for a paper | ⚠️ Not yet — needed before submission |
| Reproducibility (seeds pinned) | ⚠️ Not enforced |
| Tests | ❌ None (only ad-hoc `main()` demo blocks) |

## 3. Architecture & data flow

```
data/*.sm  ──►  psplib_loader  ──►  RCPSP model (jobs, precedence, resources, capacities)
                                        │
                                        ▼
              ProjectManagerAgent / WarehouseAgent  ──(prompt)──►  llm_brain (Gemini or mock)
                                        │                                   │
                                        │◄────────── proposed schedule ─────┘
                                        ▼
                          RCPSPSolver (CP-SAT) verify feasibility
                              │ feasible → accept + record metrics
                              │ violations → build feedback prompt → back to agent (repair)
                              └ repeated failure → solver-optimal fallback schedule
```

## 4. Key files (start here when reading)

- `main_simulation.py` — the orchestrator; **read this first**. Accepts an instance path as `argv[1]`.
- `solvers/rcpsp_solver.py` — CP-SAT model (`RCPSPSolver`) + parser (`RCPSPParser`). The trustworthy core.
- `agents/llm_brain.py` — LLM abstraction + mock fallback logic.
- `agents/basic_agents.py` — agent roles and their negotiation `main()` demos.
- `utils/psplib_loader.py`, `utils/plan_parser.py`, `utils/metrics.py` — IO + parsing + metrics.
- `run_benchmark_solver.py` — solver-only batch sweep (no LLM).

## 5. How to run

See `README.md`. TL;DR: create `.venv`, `pip install -r requirements.txt`, add `.env` with
`GOOGLE_API_KEY`, then `python main_simulation.py data/raw/rcpsp/j30/<instance>.sm` from repo root.
Without an API key it runs in **mock mode** (deterministic, good for CI/dev).

## 6. Known issues / gotchas (important for whoever continues)

1. **`.venv/` was committed** (~95% of tracked files). Untrack it (`git rm -r --cached .venv`) — see `IMPROVEMENTS.md`.
2. **`requirements.txt` is UTF-16** and can break `pip install`. Re-save as UTF-8 (curated file provided).
3. **Stray paste bug** in `agents/basic_agents.py:53` — an `import` statement is embedded inside a comment. Harmless but fix it.
4. **All data paths are relative to repo root** — scripts break if run from elsewhere. Anchor via `__file__` like `main_simulation.py` already does.
5. `RCPSPSolver.get_solution_details()` (`rcpsp_solver.py:307`) is brittle (assumes variable-creation order) and unused — the main flow uses `get_schedule_dict()`. Safe to delete.
6. Comments mix Vietnamese and English.

## 7. Recommended next steps (roadmap)

- [ ] Repo hygiene: untrack `.venv`, UTF-8 requirements, add `LICENSE`, fix maintainer link.
- [ ] Add `set_seed()` (random/numpy/torch-free — here just random + ortools determinism) and pin it.
- [ ] Add `tests/` — unit-test `verify_schedule` and the parser against a known `.sm` instance.
- [ ] Build the paper evaluation harness: baselines (solver-only, LLM-only, hybrid), ablation on the repair loop, a results table exported to `results/`.
- [ ] Move `run_benchmark_solver.py` config to `argparse`.
- [ ] Add a data-flow diagram + sample metrics table to the README (done in the provided README).

## 8. Conventions

- Language: Python 3.10+. Config via `.env` (never commit keys).
- Layering: agents (reasoning) never bypass the solver for feasibility guarantees.
- New experiments should write structured output to `results/` for reproducibility.
