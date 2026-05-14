# Interpretable Network Intrusion Detection — SHAP vs. Counterfactuals

Train a Random Forest to detect web attacks in CIC-IDS2017 network flows, then explain its
predictions two different ways:

- **SHAP** (`TreeExplainer`) — feature attributions: *which features pushed this prediction toward "attack"?*
- **DiCE** counterfactuals — *what minimal changes would flip this flow back to benign?*

The final section quantitatively compares the two methods (top-*k* feature overlap and Spearman ρ
between mean |SHAP| and mean DiCE perturbation magnitude).

This repository is the artifact for the Explainable AI final project at Radboud University.

## What's in here

| File | Purpose |
| --- | --- |
| [`cic_ids2017_xai.ipynb`](cic_ids2017_xai.ipynb) | End-to-end notebook: load → train → SHAP → DiCE → compare |
| [`requirements.txt`](requirements.txt) | Python dependencies |
| [`LICENSE`](LICENSE) | MIT |

## Dataset

The notebook uses **one** CSV from CIC-IDS2017:
`Thursday-WorkingHours-Morning-WebAttacks.pcap_ISCX.csv` (≈50 MB).
It is not committed — download it from the official source and drop it next to the notebook:

- CIC-IDS2017 dataset page: <https://www.unb.ca/cic/datasets/ids-2017.html>
- The notebook expects the file at `./Thursday-WorkingHours-Morning-WebAttacks.pcap_ISCX.csv`
  (path is set in the `DATA_PATH` constant — change it if you put the file elsewhere).

## Quick start

```bash
# 1. clone
git clone https://github.com/daantje6799/interpretable-network-intrusion-detection.git
cd interpretable-network-intrusion-detection

# 2. virtual env + dependencies
python -m venv .venv
# Windows:  .venv\Scripts\activate
# macOS/Linux:  source .venv/bin/activate
pip install -r requirements.txt

# 3. add the dataset CSV to this folder (see "Dataset" above)

# 4. run
jupyter notebook cic_ids2017_xai.ipynb
```

Run the cells top to bottom. Total runtime is a few minutes on a laptop CPU
(`SAMPLE_SIZE = 50_000`, 100-tree Random Forest, depth 12).

## Reproducibility

- `RANDOM_STATE = 42` everywhere it matters (sampling, split, model).
- Stratified 80/20 train/test split preserves the benign/attack ratio.
- DiCE uses the `random` backend (the scikit-learn Random Forest is not differentiable).

## Method summary

1. **Load + clean** — strip column-name whitespace, drop infinities and NaNs.
2. **Sample + label** — stratified 50k-row sample; `BENIGN → 0`, anything else → `1`.
3. **Train** — `RandomForestClassifier(n_estimators=100, max_depth=12)`.
4. **Evaluate** — classification report and confusion matrix on the held-out 20%.
5. **SHAP** — global summary plot on a 1k-row sample, plus a local force/waterfall view of one malicious flow.
6. **DiCE** — generate two counterfactuals that flip a malicious flow to benign.
7. **Compare** — top-*k* overlap and Spearman ρ between mean |SHAP| (attack class) and mean
   counterfactual perturbation magnitude.

## License

[MIT](LICENSE)
