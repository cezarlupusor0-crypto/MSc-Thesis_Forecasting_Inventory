# MSc thesis reproducibility package

This package accompanies **From Forecast Accuracy to Inventory Value: Evaluating Classical Forecasting Methods, Machine Learning and TiRex-2 in Retail Inventory Management**.

It contains four notebooks covering sample construction, descriptive analysis, forecasting, inventory evaluation and the focused checks requested during the final supervisor review. The supervisor-revision checks are also supplied as a command-line Python script. The package further includes the frozen 500-series sample, supporting output archives, implementation-check logs and the cost–fill-rate/Pareto figure.

## Important provenance note

The three original workflow notebooks retain the successful outputs from the author's original runs. The long ARIMA/SARIMA calculations are not rerun merely to prepare this package. One obsolete code cell that ended with a `KeyError: 'scenario'` was removed because the corrected simulation cell and its successful outputs immediately followed it. In the Chapter 4 notebook, the table and figure directories were consolidated into the setup cell and two stale `TABLE_DIR` `NameError` outputs were removed. Empty cells, widget-only display metadata and a non-fatal performance warning were also removed; analytical code and successful numerical outputs were retained. Machine-specific absolute paths in retained text outputs were replaced with neutral placeholders. The fourth notebook is a structured notebook version of `code/03_supervisor_revision_checks.py`; it starts without stored outputs and reproduces them from the supplied result archives.

The notebook metadata record Python 3.12.7. The TiRex-2 package release used for inference was 0.2.1. Exact microversions of every other dependency were not saved separately during the original long-running fits. The cleaned notebooks therefore begin with an environment-reporting cell; retain its output with any new complete execution. `requirements.txt` and `environment.yml` specify a compatible environment without pretending to reconstruct unavailable microversion information.

## Package structure

```text
reproducibility_package/
├── README.md
├── environment.yml
├── requirements.txt
├── .gitignore
├── notebooks/
│   ├── 00_chapter4_sample_construction_and_descriptive_statistics.ipynb
│   ├── 01_forecasting_and_inventory_analysis.ipynb
│   ├── 02_policy_timing_sensitivity.ipynb
│   └── 03_supervisor_revision_checks.ipynb
├── code/
│   └── 03_supervisor_revision_checks.py
├── data/
│   ├── README.md
│   └── selected_sample_ids.csv
├── supporting_outputs/
│   ├── chapter5_core_results.zip
│   ├── chapter5_quantile_results.zip
│   └── revision_checks/
└── figures/
    ├── chapter5_baseline_cost_fill_pareto.pdf
    └── chapter5_baseline_cost_fill_pareto.png
```

## 1. Create the environment

Using Conda:

```bash
conda env create -f environment.yml
conda activate cezar-thesis
```

Or using a Python 3.12 virtual environment:

```bash
python -m pip install -r requirements.txt
```

TiRex-2 and PyTorch device support can depend on the operating system and hardware. The original TiRex-2 runs used Apple's MPS device. The code also supports CPU execution, but run time will differ.

## 2. Obtain the M5 data

Download the public M5 Forecasting Accuracy data and place these files in one folder:

- `sales_train_evaluation.csv`
- `calendar.csv`
- `sell_prices.csv`
- `selected_sample_ids.csv` (copy this from `data/`)

The raw competition files are not redistributed here. On macOS or Linux, point the notebooks to the folder before starting Jupyter:

```bash
export M5_DATA_DIR="/absolute/path/to/m5_data"
```

If this variable is absent, the notebooks retain the author's original fallback path: `~/Desktop/m5_data`.

The Chapter 4 notebook writes its tables and figures to `chapter4_results` inside the data directory by default. Set `CHAPTER4_OUTPUT_DIR` to use another location. `M5_SAMPLE_FILE` can likewise point to the supplied frozen sample if it is not copied into the M5 data directory.

## 3. Run the analysis in order

1. Run `notebooks/00_chapter4_sample_construction_and_descriptive_statistics.ipynb` from top to bottom.
2. Run `notebooks/01_forecasting_and_inventory_analysis.ipynb` from top to bottom.
3. Run `notebooks/02_policy_timing_sensitivity.ipynb` from top to bottom.
4. Extract the supporting result archives and run `notebooks/03_supervisor_revision_checks.ipynb` from top to bottom. The equivalent command-line script is described below.

The Chapter 4 notebook reproduces the eligibility counts, threshold sensitivity analysis, fixed-sample checks, descriptive tables and classification figure. It reads the supplied frozen sample so that all subsequent notebooks use exactly the same 500 SKU–store series.

The forecasting-and-inventory notebook contains smoke tests and production runs. The ARIMA/SARIMA production sections are the main computational bottleneck and use checkpoints in `chapter5_results`. On the author's MacBook, the additional twelve-origin ARIMA/SARIMA run required 37.76 hours for 500 series. Do not delete valid checkpoints before a resumed run.

The policy-timing notebook reuses saved daily forecasts and does **not** refit the forecasting models. It reconstructs protection periods of 14, 21 and 28 days and evaluates six compatible `(R,L)` policies.

## 4. Reproduce the focused revision checks

Extract both archives into the package's `data` directory. Each archive contains a `chapter5_results` folder, and the contents are complementary.

```bash
python -m zipfile -e supporting_outputs/chapter5_core_results.zip data
python -m zipfile -e supporting_outputs/chapter5_quantile_results.zip data
```

Then open `notebooks/03_supervisor_revision_checks.ipynb` and run all cells from top to bottom. Alternatively, run the equivalent command-line version:

```bash
python code/03_supervisor_revision_checks.py \
  --core-dir data/chapter5_results \
  --quantile-dir data/chapter5_results \
  --sample-file data/selected_sample_ids.csv \
  --output-dir outputs/revision_checks \
  --figure-dir figures
```

This script reads saved outputs only. It produces paired accuracy intervals, ARIMA/SARIMA non-convergence sensitivities, TiRex-2 calibration intervals and monotone-rearrangement checks, terminal inventory summaries, item/store cluster bootstraps, and the baseline Pareto figure. A successful run writes `revision_analysis_checks.csv` with all checks equal to `True`.

## 5. Principal fixed settings

- sample: 500 SKU–store series, 125 per demand class;
- sample and bootstrap seed: 2026;
- forecast horizon: 28 days;
- weekly seasonal period: 7;
- ETS candidates: ETS(A,N,A), ETS(A,A,A), ETS(A,Ad,A);
- ARIMA/SARIMA search: bounded stepwise exact-likelihood search reported in Chapter 3;
- XGBoost tuning: eight predefined configurations; selected `n_estimators=300`, `learning_rate=0.03`, `max_depth=10`, row and column subsampling 0.8;
- TiRex-2 central forecast: native 0.50 quantile;
- residual-based inventory quantiles: 0.90, 0.95 and 0.99;
- paired bootstrap resamples: 2,000.

## 6. Verification philosophy

The CSV check logs are part of the analytical record, not a substitute for reviewing the code. They verify panel dimensions, forecast keys, finite and non-negative outputs, target-day arithmetic, sequential residual pools, inventory accounting identities, quantile levels, and bootstrap dimensions. The thesis reports limitations that automated checks cannot resolve, including recorded sales versus unconstrained demand, the designed rather than population-representative sample, cross-series dependence, multiple comparisons, and finite-horizon terminal inventory.
