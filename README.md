# MT5 + LightGBM + ONNX

MLP has been ranked first between individual classifiers on the data I have tested, followed by LightGBM and HistGradientBoosting. On other data, an weighted ensemble of MLP 0.25, LightGBM 0.25, 0.50 HistGradientBoosting was better than MLP, LightGBM or HistGradientBoosting individually.

Contains:
- `train_mt5_lightgbm_classifier.py` — Python script for training, chronological train/test split, labeling on 3 classes and ONNX export
- `MT5_LightGBM_Classifier_ONNX_Strategy.mq5` — MQL5 EA for Strategy Tester and running in MT5

## 1. Required Python packages

In PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\activate
python -m pip install --upgrade pip
pip install MetaTrader5 pandas numpy scikit-learn lightgbm onnxmltools onnx onnxconverter-common
```

## 2. Recommended training command

### XAGUSD / M15 / horizon 8 bars

```powershell
python train_mt5_lightgbm_classifier.py --symbol XAGUSD --timeframe M15 --bars 20000 --horizon-bars 8 --train-ratio 0.70 --output-dir output_lgbm_h8
```

### Variants 4 / 8 / 12 bars

```powershell
python train_mt5_lightgbm_classifier.py --symbol XAGUSD --timeframe M15 --bars 20000 --horizon-bars 4  --train-ratio 0.70 --output-dir output_lgbm_h4
python train_mt5_lightgbm_classifier.py --symbol XAGUSD --timeframe M15 --bars 20000 --horizon-bars 8  --train-ratio 0.70 --output-dir output_lgbm_h8
python train_mt5_lightgbm_classifier.py --symbol XAGUSD --timeframe M15 --bars 20000 --horizon-bars 12 --train-ratio 0.70 --output-dir output_lgbm_h12
```

## 3. What files does the script produce

In the output folder you will have, among others:
- `ml_strategy_classifier_lightgbm.onnx`
- `model_metadata.json`
- `run_in_mt5.txt`
- `train_predictions_snapshot.csv`
- `test_predictions_snapshot.csv`

## 4. How to run in MT5

1. Copy `ml_strategy_classifier_lightgbm.onnx` to the same folder as `MT5_LightGBM_Classifier_ONNX_Strategy.mq5`
2. Recompile the EA in MetaEditor
3. Open `run_in_mt5.txt`
4. In Strategy Tester set exactly the `TEST UTC` window
5. Enter the recommended values in the inputs:
   - `InpEntryProbThreshold`
   - `InpMinProbGap`
   - `InpMaxBarsInTrade`

## 5. Starting set for filters in EA

Start from here, for comparison with your previous tests:

```text
InpEntryProbThreshold = 0.60
InpMinProbGap         = 0.15
InpUseTrendFilter     = true
InpTrendTF            = PERIOD_H1
InpTrendMAPeriod      = 100
InpTrendRequireSlope  = true
InpUseTrendDistanceFilter = false
InpUseAtrVolFilter    = true
InpAtrVolLookback     = 50
InpAtrMinPercentile   = 0.25
InpAtrMaxPercentile   = 0.85
InpUseKillSwitch      = false
```

## 6. Important notes

- For classifiers exported to ONNX, the first output is the label (`int64`), and the second is the probability tensor; the EA is already written for this format.
- If your MT5 terminal has a more sensitive ONNX version, it may need a small adjustment to the output shapes part, but this variant follows the same running type that already worked for you with RandomForest.
- The purpose here is to change only the ML model, not to rewrite all the testing logic.
