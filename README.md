# Anomalies DLRZ

This project replicates and extends parts of Dong et al., combining:

- Goyal and Welch monthly predictors
- Open Asset Pricing long-short anomaly returns
- rolling and expanding out-of-sample forecasts of excess market returns

## Files

- [01_prepare_data.ipynb](/Users/rubetron/Python-Projects/Anomalies%20DLRZ/01_prepare_data.ipynb)
  Downloads and preprocesses data, then saves prepared datasets to `outputs/prepared`.
- [02_run_models.ipynb](/Users/rubetron/Python-Projects/Anomalies%20DLRZ/02_run_models.ipynb)
  Loads prepared data, runs forecasting models, computes evaluation tables, and produces plots.
- [requirements.txt](/Users/rubetron/Python-Projects/Anomalies%20DLRZ/requirements.txt)
  Local Python dependencies.

## Workflow

1. Install dependencies:

```bash
pip install -r /Users/rubetron/Python-Projects/Anomalies\ DLRZ/requirements.txt
```

2. Run [01_prepare_data.ipynb](/Users/rubetron/Python-Projects/Anomalies%20DLRZ/01_prepare_data.ipynb)
   This creates:
   - `target_panel`
   - `gw_features`
   - `ls_features`
   - `gw_model`
   - `ls_model`
   - `ls_metadata`

   Each is saved as both CSV and parquet in `outputs/prepared`.

3. Run [02_run_models.ipynb](/Users/rubetron/Python-Projects/Anomalies%20DLRZ/02_run_models.ipynb)
   This writes forecasts to `outputs/forecasts` and summary tables to `outputs/tables`.

## Data conventions

- GW returns are treated as decimals.
- Open Asset Pricing long-short anomaly returns are downloaded in percent and converted to decimals during data preparation.
- Final reported annualized return, volatility, and CER statistics are converted back to percent for presentation.

## Forecast timing

The model frames follow the standard predictive-regression convention:

- target at month `t`
- predictors lagged to month `t-1`

So a row indexed by `1970-01` uses December 1969 predictors to explain or forecast January 1970 excess market return.

## Predictor sets

### GW

- Starts from the fixed list in the notebook
- Excludes: `cay, pce, ogap, sntm, fbm, tchi, shtint`
- Keeps only predictors with complete data from `1970-01` through the common sample end

### LS

- Default source is `deciles_vw`
- Uses the prebuilt `LS` portfolio when available
- Excludes: `ConvDebt, secureind, IndIPO, RDIPO`
- Keeps anomalies starting by `1985-01`
- Fills missing anomaly returns with the cross-sectional monthly average before converting to decimals

## Models

Implemented in `02_run_models.ipynb`:

- `Ols`
- `Enet`
  AICc-selected elastic net, matching Dong et al.'s direct ENet logic
- `EnetVal`
  validation-tuned elastic net
- `RidgeVal`
  validation-tuned ridge
- `Combine`
  simple average of univariate forecasts
- `Cenet`
  Dong et al. combination ENet based on validation-period selection of univariate forecasts
- `Avg`
  LS-only predictor-average regression

## Evaluation

The notebooks compute:

- out-of-sample `R^2_OOS`
- Clark-West statistic and p-value
- timing-portfolio annualized return
- timing-portfolio annualized volatility
- timing-portfolio Sharpe ratio
- CER difference relative to the prevailing-mean benchmark

## Notes

- The GW loader is intentionally simple and assumes the current spreadsheet structure is stable.
- If you change scaling conventions, rerun `01_prepare_data.ipynb` before rerunning models.
