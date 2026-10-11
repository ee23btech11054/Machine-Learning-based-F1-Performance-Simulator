# AGENTS.md - F1 Race Strategy Simulator

## Project Overview
A machine learning project that predicts F1 lap time changes through a race and simulates pit strategies. Uses FastF1 data (2022-2025 seasons) with a gradient boosting model to predict delta lap times relative to driver baseline pace.

## Directory Structure
```
├── 01_download_data.py      # Download race laps using FastF1 (~30 min, resumable)
├── 02_build_dataset.py      # Clean laps, compute baselines, target, pit loss
├── 03_train_models.py       # Compare models, final test, save model
├── 04_backtest.py           # Backtest simulator on 2025 season
├── 05_app.py                # Interactive Streamlit app
├── strategy.py              # Simulation engine (shared by backtest and app)
├── race_animation.py        # Race data for animated race view
├── race_view.html           # Animated race view (circuit, cars, timing panel)
├── fetch_circuits.py        # Download real circuit outlines (optional)
├── common.py                # Paths, split and baseline settings
├── plot_style.py            # Chart colours and styling
├── requirements.txt         # Python dependencies
├── run_app.bat              # Windows launcher
├── data/
│   ├── raw/                 # Raw CSV files per race (2022-2025)
│   ├── circuits.json        # Circuit outlines
│   ├── laps.csv             # Processed lap data
│   ├── model_laps.csv       # Model-ready lap data
│   ├── pit_loss.csv         # Pit loss times per circuit
│   └── races.csv            # Race metadata
├── presentation/
│   ├── assets/              # Presentation assets
│   ├── build.ps1            # Build script
│   ├── build_deck.py        # Build presentation deck
│   ├── deck_analysis.py     # Presentation analysis
│   ├── F1_Race_Strategy_Simulator.pdf
│   └── F1_Race_Strategy_Simulator.pptx
└── reports/                 # Generated tables/figures (from scripts 03, 04)
```

## Pipeline Execution Order
```bash
pip install -r requirements.txt
python 01_download_data.py     # Downloads ~92 races, ~30 min, resumable
python 02_build_dataset.py     # Creates model_laps.csv, pit_loss.csv, etc.
python 03_train_models.py      # Trains and evaluates models, saves model.pkl
python 04_backtest.py          # Runs backtest on 2025, outputs to reports/
python fetch_circuits.py       # Optional: downloads real circuit outlines
streamlit run 05_app.py        # Launches interactive app
```

## Key Configuration (common.py)
- **Data splits**: Train=2022-2023, Validation=2024, Test=2025 (by season, not random)
- **Baseline**: Median of driver's first 8 clean laps (laps 2-12)
- **Target**: `delta_pct = 100 * (lap_time / baseline_time - 1)`
- **Clean lap criteria**: Green flag, dry tyres, no pit entry/exit, ≤4% slower than median

## Model Details
- **Selected**: Gradient Boosting (4 leaves, deliberately small to avoid overfitting)
- **Validation RMSE**: 0.902% pace (0.798s/lap), R²=0.454
- **Test RMSE**: 0.966% pace (0.835s/lap), R²=0.437
- **Features**: tyre_life, compound, laps_since_baseline_frac, lap_frac, total_laps, baseline_compound, baseline_tyre_life, baseline_track_temp, circuit

## Simulator (strategy.py)
1. Builds lap-by-lap tyre plan for a strategy
2. Predicts every lap time from model + driver baseline
3. Adds circuit pit-loss time per stop
4. Searches all 1-stop + grid of 2-stop strategies, ranks by predicted race time
5. Monte Carlo simulation using stint-level (0.438%) and lap-level (0.650%) errors

## Testing & Verification
No formal test suite exists. Verify by:
- Running `python 04_backtest.py` - should produce backtest results matching README
- Running `streamlit run 05_app.py` - app should launch without errors
- Checking `reports/` for output tables/figures after running scripts

## Dependencies
- fastf1, numpy, pandas, scikit-learn, matplotlib, plotly, joblib, streamlit

## Important Notes
- **Data split is by season** (not random) - laps from same race are correlated
- **Only dry races modeled** - 12 wet races dropped, 5 more produced no usable laps
- **Safety cars/weather not modeled** - backtest only valid for fully green races
- **Survivorship bias in tyre data** - model optimistic about old tyres
- **No traffic/track position** - simulator finds fastest strategy in clean air only
- **Strategy gains within model error** - read uncertainty view before trusting recommendations

## Windows
Use `run_app.bat` which sets up environment at `%USERPROFILE%\.venvs\f1sim`