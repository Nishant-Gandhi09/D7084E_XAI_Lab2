# D7084E XAI Lab 2: Learning with Scarce Labels

This project evaluates binary intrusion detection on CICIDS2017 when only a small fraction of the training labels is available. The notebook compares:

- A full-label Random Forest reference model.
- Labelled-only Random Forest baselines using 1%, 5%, and 10% of the training labels.
- Iterative pseudo-labelling, where confident predictions from the unlabelled pool are added to the training set.
- Co-training, where two Random Forest models use disjoint feature views and teach each other using confident predictions.
- A pseudo-labelling ablation study using confidence thresholds of 90%, 95%, and 99%.

## Project layout

```text
Lab2/
├── lab2_v04.ipynb
├── requirements.txt
├── README.md
└── data/
	└── CICIDS2017 CSV files
```

The CSV files must be placed in `data/`. The notebook discovers every `*.csv` file in that directory, normalizes the column names, and uses the `Label` column as the target.

## How the notebook works

1. It loads and concatenates the capture files, removes flow identifiers and timestamps, removes duplicate rows, converts invalid numeric values to missing values, and maps `BENIGN` to class `0` and every attack label to class `1`.
2. It creates stratified train, validation, and test partitions using a 60%/20%/20% split.
3. It creates deterministic labelled/unlabelled training pools for the 1%, 5%, and 10% label budgets. The validation set remains labelled for development comparisons, while the test set is kept untouched by the current notebook.
4. Each model uses median imputation followed by a class-balanced Random Forest with 300 trees. Imputation is fitted inside each model pipeline using only that method's training data.
5. Metrics are calculated on the validation set: accuracy, Macro-F1, attack recall, ROC-AUC, and false alarm rate (FAR).
6. The final plot compares Macro-F1 against the available label budget for the supervised baseline, pseudo-labelling, and co-training methods.

## Installation

Use Python 3.9 or newer and create an isolated environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

On Windows, activate the environment with `.venv\\Scripts\\activate` instead.

## Running the notebook

From the project directory, start Jupyter:

```bash
jupyter lab
```

Open `lab2_v04.ipynb` and run the cells from top to bottom. The notebook expects the working directory to be the project directory because it resolves the data folder with `Path.cwd() / "data"`.

Full-data execution can be computationally expensive because several experiments retrain 300-tree Random Forests over large CSV files. For a quick code check, set `USE_SAMPLE = True` in the configuration cell; the notebook will then use 25% of the cleaned rows. Set it back to `False` for reported results.

## Reproducibility and limitations

The seed is fixed at `42`, and the data splits and model random states are deterministic. The current notebook reports validation results and creates the comparison plot; it does not yet run a final test-set evaluation or save trained models. Because pseudo-labels are model-generated rather than ground truth, their quality depends on the confidence threshold and the initial labelled subset.
