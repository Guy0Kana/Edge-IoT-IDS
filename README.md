# Lightweight Intrusion Detection for Edge IoT on TON_IoT

Machine learning models that detect cyberattacks in IoT network traffic, built on the network portion of the TON_IoT dataset and evaluated with edge deployment in mind (small models, features a constrained device could compute, inference cost measured alongside accuracy).

## Objectives

1. Explore and clean the TON_IoT network flow data.
2. Train baseline models (logistic regression, random forest, LightGBM) for binary detection (attack vs. normal) and multiclass attack typing.
3. Test whether high scores are genuine by checking for leakage (identifier columns, duplicates, grouped data).
4. Compare a full feature set with a reduced, device-computable feature set, and measure model size and inference time.

## Dataset

| Item | Detail |
|---|---|
| Name | TON_IoT network dataset |
| Source | https://research.unsw.edu.au/projects/toniot-datasets |
| Mirror used for download | https://huggingface.co/datasets/codymlewis/TON_IoT_network |
| File used | `TODO: file name` |
| Size | 211,043 rows x 44 columns |
| Targets | `label` (binary: 0 normal, 1 attack), `type` (attack name) |
| License | Free for academic use; cite the authors' papers (see Citations) |

**Classes (before deduplication):** normal 50,000; backdoor, ddos, dos, injection, password, ransomware, scanning, xss 20,000 each; mitm 1,043.

## Findings so far

- `label` and `type` are redundant (`label = 0` only for `normal`). One is the target and the other must be excluded from the features.
- The file is sorted by attack type, with one contiguous block per type. Splits must be stratified random, not positional.
- 20,569 rows (9.7%) are exact duplicates, concentrated in ransomware (-5,265), xss (-4,863), normal (-7,960) and backdoor (-1,289). After deduplication there are 190,474 rows.
- `mitm` is by far the smallest class (about 1,041 rows) and is expected to be the hardest to detect.
- Hidden missing values (`-`): `TODO`
- Identifier columns (IPs, ports): `TODO`
- Results of baseline models: `TODO`
- Effect of removing suspicious features: `TODO`
- Edge feature set vs. full feature set: `TODO`

## Method

1. **Exploration:** label balance, file ordering, duplicates, placeholder values, identifier columns, feature behavior per class.
2. **Cleaning:** deduplicate before splitting, convert `-` to a proper missing category, handle identifier and high-cardinality columns, encode categoricals.
3. **Splitting:** stratified train/validation/test split; the test set is used once at the end.
4. **Modeling:** logistic regression, random forest and LightGBM for binary and multiclass tasks.
5. **Evaluation:** precision, recall, F1, PR-AUC (binary) and macro-F1 with per-class recall (multiclass). Accuracy is not used as the main metric because the attack share is about 78%.
6. **Edge analysis:** model file size, inference time per sample and memory for each candidate, compared against performance.

## Repository structure

```
.
├── README.md
├── notebook.ipynb      # main analysis
└── data/               # not included; see Dataset for download
```

## How to reproduce

```bash
hf download codymlewis/TON_IoT_network --repo-type=dataset --local-dir ./data
pip install pandas numpy scikit-learn lightgbm matplotlib seaborn
jupyter notebook notebook.ipynb
```

## Limitations

- The data comes from a simulated IIoT testbed with attacks launched by the researchers, and has not been validated on live traffic.
- The class balance reflects how the dataset was built, not real network conditions, where benign traffic dominates.
- Flow features are Zeek-style records; a deployed system would need an upstream monitoring tool to compute them from packets.
- No cross-dataset testing yet: `TODO: add or remove`.
- Latency is measured on `TODO: hardware`, not on real edge devices.
