# ICDAM 2025 — Benchmark-centric Hybrid LLM Multi-Agent System for SCM

Research codebase for a **hybrid multi-agent system** that combines **LLM-driven negotiation** with **symbolic optimization** (Google OR-Tools / CP-SAT) on **PSPLib-style** project scheduling benchmarks. Target direction: paper-oriented contribution around **benchmark-grounded MAS** + **solver verification** (OptiMUS-style loop).

---

## Problem statement

Supply-chain and project scheduling contexts require both:

- **Flexible reasoning** (negotiation, explanations, policy-like decisions), and  
- **Hard feasibility** (resource capacities, precedence, makespan objectives).

This project prototypes agents (e.g. warehouse vs project manager) that propose allocations via LLM, while grounding outcomes in **solver-checked** schedules where applicable.

---

## Repository structure

| Path | Purpose |
|------|---------|
| `main_simulation.py` | End-to-end simulation entry: load benchmark → instantiate agents → negotiation → solver checks |
| `agents/` | Role-based agents + `llm_brain.py` (API + **mock fallback** on quota/network errors) |
| `solvers/` | RCPSP parser + CP-SAT wrapper (`RCPSPParser`, `RCPSPSolver`) |
| `utils/` | PSPLib parsing, schedule parsing/formatting, metrics |
| `scenarios/` | Scenario configs / fixtures (as used by experiments) |
| `data/` | Benchmark instances (PSPLib `.sm` etc., when present) |
| `results/` | Run outputs / exports |
| `run_benchmark_solver.py` | Batch benchmark loop over instances → aggregated metrics |
| `requirements.txt` | Pinned Python dependencies (`google-generativeai`, `ortools`, `pandas`, …) |

---

## Key technical ideas

1. **Benchmark-first:** Load real-like RCPSP instances (PSPLib), not only toy graphs.  
2. **LLM brain with resilience:** If Gemini (or configured LLM) is unavailable (429, network), fall back to **mock mode** so experiments remain reproducible.  
3. **Verification loop:** LLM-proposed schedules / allocations can be cross-checked with OR-Tools CP-SAT (`OptiMUS` pattern referenced in code comments).  
4. **Metrics:** `SimulationMetrics` tracks experiment outputs for comparisons across solvers / agents.

---

## Setup

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS / Linux
pip install -r requirements.txt
```

Configure API keys via environment variables as expected by `agents/llm_brain.py` (see module / `.env` pattern—**never commit keys**).

Sanity scripts:

```bash
python check_setup.py
python check_models.py
```

---

## Run

### Single simulation

```bash
python main_simulation.py
```

(Adjust `data_file` path inside `main_simulation.py` or refactor to CLI args for your fork.)

### Solver benchmark sweep

```bash
python run_benchmark_solver.py
```

Tune instance directories / limits inside the script to match your `data/` layout.

---

## Dependencies (high level)

- **OR-Tools** — CP-SAT / scheduling  
- **Google Generative AI** — Gemini client  
- **pandas / numpy / matplotlib / networkx** — data + visualization  
- Optional: **OpenAI** client present in requirements for experiments

Full pins: [`requirements.txt`](requirements.txt).

---

## Status & honesty box

- Phase coverage (negotiation + solver verification) is **research-grade**, not production middleware.  
- For ICDAM-style submission: tighten **evaluation tables**, **baselines**, and **ablations**; ensure reproducible seeds and pinned hardware/API notes.

---

## Citation / references (in-code)

The codebase cites influences such as **AgentScope**, **REALM-Bench**, **OptiMUS** — see docstrings in `main_simulation.py`.

---

## Maintainer

[@nhatnguyen31241026074-commits](https://github.com/nhatnguyen31241026074-commits)

---

## License

Add an explicit `LICENSE` before external redistribution.
