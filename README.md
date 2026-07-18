# Surplus Ranking Sim

A marketplace simulation that compares ranking/recommendation strategies for surplus-food platforms (à la Too Good To Go) — evaluating how different algorithms trade off **revenue**, **waste reduction**, **fairness across stores**, and **cancellation risk** when deciding which stores to show each customer.

Built as our course project for **CSCE 2202 (Analysis of Algorithms), Fall 2025** — Team 3: Abdelrahman Abdelbaky, Abdelrahman Osama, Mohamed Anan, Yomna Othman.

## The problem

On a surplus-food marketplace, every customer sees a limited set of stores (not all of them). Which stores you choose to show shapes everything downstream: whether inventory gets sold before it's wasted, whether revenue is maximized, whether small/new stores ever get exposure, and how often orders get cancelled because a store oversold its capacity.

We built a simulated marketplace — stores with inventory, prices, ratings, and reliability; customers with price sensitivity, category preferences, and purchase history — and used it to test several ranking strategies against each other over multi-day runs.

## Strategies implemented

| Strategy | Core idea |
|---|---|
| `Greedy` | Baseline — heavily weighted toward revenue (price × rating) |
| `Hybrid` | Personalized favorites + supply-aware scoring + epsilon-greedy exploration |
| `Optimized` | Multi-objective scoring balancing supply/demand matching, expected revenue, value, fairness, personalization, and risk |
| `NearOptimal` | Adaptive weights that shift with real-time market utilization, plus Thompson Sampling for exploration |
| `RWES_T` | Lightweight reliability + revenue + personalization score with an exposure-fairness penalty |
| `Anan_Strategy` | Collaborative filtering — recommends stores based on cosine similarity between customers' learned preference vectors, with a cold-start fallback |
| `Yomna_Strategy` | Trust-first tiered selection with hard safety gates (min reliability, max utilization) before optimizing for value |
| `Hybrid_Enhanced_Strategy` | Combines NearOptimal's adaptive weighting, Anan's fairness balancing, and Yomna's safety gates |

Customer purchase decisions are modeled with a multinomial logit choice model over the displayed stores, so strategies are compared on realistic simulated demand rather than assumed click-through.

## Architecture

```
├── restaurant_api.py     # Store/Restaurant model, data generation
├── customer_api.py       # Customer model, choice model, purchase logic
├── ranking_algorithm.py  # All ranking strategies
├── simulation.py         # Marketplace simulation engine, multi-day comparison harness
├── data_loader.py        # CSV <-> model (de)serialization
├── main.py                # CLI entry point for running comparisons
├── server.py              # FastAPI backend exposing the simulation over HTTP
├── api/index.py           # Vercel serverless entry point (wraps server.py)
├── food_waste.ipynb       # Exploratory notebook / original course milestone work
├── vercel.json             # Deploy config (routes /api/* to FastAPI, rest to frontend)
└── frontend/                # React + Vite dashboard (charts, run configuration, results)
```

**Backend:** FastAPI service that runs simulations as background tasks and serves results by ID (`/run`, `/results/{id}`, `/upload` for custom store/customer CSVs).

**Frontend:** React dashboard (Recharts) for configuring a run, uploading custom data, and comparing strategies' revenue/waste/cancellation curves over time.

**Deployment:** Configured for Vercel — `api/index.py` exposes the FastAPI app as a serverless function, static frontend served alongside it.

## Running it

**CLI simulation only:**
```bash
pip install -r requirements.txt
python main.py --days 10                    # synthetic data, default 15 stores / 70 customers
python main.py --data generate --days 10     # generate + save new synthetic CSVs first
python main.py --data use_generated --stores 20 --customers 100
```

**Full app (API + dashboard):**
```bash
# backend
pip install -r requirements.txt
python -m uvicorn server:app --reload --port 8000

# frontend (separate terminal)
cd frontend
npm install
npm run dev
```

## Results

Simulation output (per-strategy revenue, waste, cancellations, fairness metrics across simulated days) is written to `simulation_results/` as CSV, and visualized in the dashboard. Run `main.py` or trigger a run from the frontend to regenerate.

## Notes

- `generated_stores.csv` / `generated_customers.csv` are sample synthetic datasets — regenerate anytime with `--data generate`.
- The `--skip-anan` flag exists because collaborative filtering is slower on larger simulated datasets; use it for quick iteration.
