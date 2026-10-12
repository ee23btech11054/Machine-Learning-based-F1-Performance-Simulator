# F1 Race Strategy Simulator

A Python project that uses real Formula 1 lap data to train a predictive model, simulate pit strategies, and compare the best strategy against a team's real race decision.

This project is designed for reproducible research and practical strategy analysis. It combines data preparation, model training, strategy search, backtesting, and an interactive Streamlit app in a single workflow.

## Project overview

The workflow is:

1. Download and clean real race laps from FastF1.
2. Create a driver baseline pace and compute lap-to-lap performance deltas.
3. Train and compare candidate lap-time models.
4. Search valid pit-stop strategies.
5. Simulate race-time outcomes and compare them with real races.
6. Explore strategy trade-offs through the interactive app.

The project is organized so that reusable Python modules live in a dedicated package, while the numbered scripts remain the runnable entry points.

## Repository structure

```text
.
├── 01_download_data.py
├── 02_build_dataset.py
├── 03_train_models.py
├── 04_backtest.py
├── 05_app.py
├── f1sim/
│   ├── __init__.py
│   ├── common.py
│   ├── fetch_circuits.py
│   ├── plot_style.py
│   ├── race_animation.py
│   └── strategy.py
├── data/
│   ├── raw/
│   ├── circuits.json
│   ├── laps.csv
│   ├── model_laps.csv
│   ├── pit_loss.csv
│   └── races.csv
├── presentation/
│   ├── detailed analysis 
├── reports/
├── race_view.html
├── requirements.txt
├── run_app.bat
├── README.md
├── AGENTS.md
└── models/
```

## Requirements

- Python 3.10 or newer
- pip
- Access to the internet for FastF1 data download
- A working virtual environment is recommended

## Local setup

### 1) Clone the repository

```bash
git clone <repository-url>
cd Machine-Learning-based-F1-Performance-Simulator
```

### 2) Create and activate a virtual environment

On Linux/macOS:

```bash
python -m venv .venv
source .venv/bin/activate
```

On Windows:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 3) Install dependencies

```bash
pip install -r requirements.txt
```

## Pipeline run order

Run the scripts in this order from the project root:

```bash
python 01_download_data.py
python 02_build_dataset.py
python 03_train_models.py
python 04_backtest.py
```

### What each stage does

- `01_download_data.py` downloads raw lap and race data from FastF1 and stores it in `data/raw/`.
- `02_build_dataset.py` cleans the data, computes baseline pace, creates model features, and writes the processed data files in `data/`.
- `03_train_models.py` trains the candidate models, evaluates them, and saves the final model to `models/`.
- `04_backtest.py` evaluates the strategy search against the 2025 test season and writes outputs to `reports/04_backtest/`.

## Optional: download circuit outlines

The app can display a more realistic track layout if the circuit outlines are available. This is optional.

```bash
python f1sim/fetch_circuits.py
```

This writes a circuit map file to `data/circuits.json`. If the file is missing, the app falls back to a generic loop-shaped track.

## Run the app

Start the Streamlit app from the project root:

```bash
streamlit run 05_app.py
```

On Windows, you can also use:

```bat
run_app.bat
```

## Important project notes

- The data split is temporal, not random. Training, validation, and test seasons are separated by year.
- The model expects dry-weather race laps; the pipeline cleans out wet or incomplete race segments.
- The reusable implementation code is kept under the `f1sim/` package so the workflow stays modular and import-safe.
- The numbered scripts at the project root are the main execution entry points.

## File responsibility summary

- `01_download_data.py`: race data acquisition
- `02_build_dataset.py`: preprocessing and feature creation
- `03_train_models.py`: model comparison and model persistence
- `04_backtest.py`: strategy evaluation and summary output
- `05_app.py`: interactive visual exploration
- `f1sim/strategy.py`: strategy search and lap-time simulation logic
- `f1sim/race_animation.py`: race timeline and animation payload generation
- `f1sim/common.py`: shared paths and global configuration
- `f1sim/plot_style.py`: shared plotting style
- `f1sim/fetch_circuits.py`: optional real circuit geometry download

## Troubleshooting

### Import errors

If you see module import errors after pulling the project, make sure you are running commands from the repository root and that the package directory `f1sim` is present.

### Missing data files

If the app or pipeline complains about missing CSV files, run the earlier steps in order:

```bash
python 01_download_data.py
python 02_build_dataset.py
python 03_train_models.py
```

### App not starting

Verify dependencies are installed:

```bash
pip install -r requirements.txt
```

Then run:

```bash
streamlit run 05_app.py
```

## Contributing and usage

This project is structured for experimentation and extension. If you add new modules, keep them inside the `f1sim/` package if they are shared across scripts. Keep the numbered scripts as the execution layer for the project pipeline.

For a clean professional layout, the repository separates:

- execution scripts at the root
- reusable logic in `f1sim/`
- generated data and reports under `data/` and `reports/`
- presentation materials under `presentation/`
