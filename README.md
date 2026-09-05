# Circadian Gene Signatures Do Not Replicate for Immunotherapy Response

### External Validation Across Three Independent Cohorts

[![Python](https://img.shields.io/badge/Python-3.12-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Reproducibility](https://img.shields.io/badge/Analysis-Reproducible-orange.svg)](docs/reproducibility.md)

> Reproducible research code for evaluating whether static circadian gene-expression signatures provide generalizable predictive information for immune checkpoint inhibitor (ICI) response.

---

## Overview

Circadian clock disruption has been associated with immune-cell trafficking, inflammatory signaling, macrophage polarization, and tumor immune suppression. These biological observations motivate the hypothesis that circadian gene expression may contain predictive information about response to immune checkpoint inhibitors.

This study tests that hypothesis using **three independent ICI-treated cohorts**, **407 patients**, **two cancer types**, and **five checkpoint agents**.

A prespecified **17-gene core circadian panel** is evaluated using three complementary formulations:

1. Mean per-gene z-score
2. Mean percentile rank
3. First principal component (PCA1)

The circadian signatures are evaluated both independently and in combination with established predictors, including **tumor mutational burden (TMB)** and the **T-cell inflamed gene expression profile (GEP)**.

The analysis emphasizes independent replication and strict leakage prevention rather than relying solely on within-cohort performance.

---

## Research Question

**Does a static 17-gene circadian expression signature provide reproducible incremental predictive value for immune checkpoint inhibitor response across independent cohorts?**

The central hypothesis is deliberately tested against an external replication framework rather than being accepted based on discovery-cohort performance alone.

---

## Study Design

The analysis uses three publicly available ICI-treated cohorts:

| Cohort         | Cancer Type          | Therapy      | Patients | Role                    |
| -------------- | -------------------- | ------------ | -------: | ----------------------- |
| **IMvigor210** | Urothelial carcinoma | Atezolizumab |      298 | Discovery               |
| **GSE78220**   | Melanoma             | Anti-PD-1    |       26 | Independent replication |
| **GSE176307**  | Urothelial carcinoma | Five agents  |       83 | External validation     |

**Total: 407 patients**

Response was analyzed as a binary outcome:

* `1` — Complete or Partial Response
* `0` — Progressive Disease

---

## 17-Gene Circadian Panel

The prespecified core clock panel contains:

```text
ARNTL
CLOCK
NPAS2
PER1
PER2
PER3
CRY1
CRY2
NR1D1
NR1D2
RORA
RORB
RORC
TIMELESS
TIPIN
CSNK1D
CSNK1E
```

Gene symbols are harmonized to current HGNC nomenclature before feature construction.

---

## Circadian Feature Formulations

Three formulations are evaluated:

### 1. Mean Z-Score

Each gene is standardized using the training-partition distribution and the resulting gene-level z-scores are averaged.

### 2. Mean Percentile Rank

Each gene is converted to an empirical percentile based on the training distribution, followed by averaging across the panel.

### 3. First Principal Component

PCA is fitted using the training partition and the first principal component is used as the circadian composite.

All learned transformations are restricted to the appropriate training data during validation.

---

## Machine Learning Framework

The primary classifier is:

**L2-regularized Logistic Regression**

Hyperparameter selection is performed using inner cross-validation over:

```text
C ∈ {
    0.01,
    0.0316,
    0.1,
    0.316,
    1.0,
    3.162,
    10.0
}
```

The validation framework separates:

```text
Training data
      │
      ▼
Fold-wise preprocessing
      │
      ▼
Feature construction
      │
      ▼
Inner CV hyperparameter selection
      │
      ▼
Model fitting
      │
      ▼
Held-out prediction
      │
      ▼
AUC / AUPRC
```

---

## Leakage-Controlled Analysis

A major focus of this repository is preventing optimistic performance caused by data leakage.

Four preprocessing safeguards are implemented.

### G1 — Post-treatment exclusion

Samples collected during or after treatment cannot be used as pretreatment predictors of response to that treatment.

### G2 — Duplicate patient handling

Multiple specimens from the same patient are not treated as independent observations when that would violate the independence assumption.

Duplicate specimens are collapsed after checking response-label concordance.

### G3 — Fold-wise transformations

Scaling, PCA loadings, gene-level moments, and other learned transformations are estimated using training partitions only.

No information from the validation/test partition is used to fit these transformations.

### G4 — Low-variance feature filtering

Genes with training-partition standard deviation below the predefined threshold are excluded:

```text
SD < 0.05
```

The filtering decision is made using the training partition rather than the complete dataset.

---

## Evaluation Metrics

Primary metric:

**ROC AUC**

Secondary metric:

**Area Under the Precision-Recall Curve (AUPRC)**

Uncertainty is estimated using:

```text
2,000 patient-level bootstrap replicates
```

For comparisons between disjoint cohorts, the repository uses the Hanley–McNeil approximate z-test.

---

## Main Results

The manuscript reports the following performance pattern.

### Discovery — Circadian Signatures

| Circadian formulation     |   AUC |
| ------------------------- | ----: |
| Mean z-score              | 0.518 |
| Mean percentile rank      | 0.466 |
| First principal component | 0.520 |

All confidence intervals span the null value of 0.5.

---

### Discovery — Signature Stacking

| Model                       |       AUC |
| --------------------------- | --------: |
| TMB                         |     0.748 |
| TMB + GEP                   |     0.778 |
| TMB + GEP + TIDE            |     0.775 |
| TMB + GEP + IPRES           |     0.796 |
| TMB + GEP + Subtype         | **0.824** |
| TMB + GEP + Circadian PCA   |     0.795 |
| TMB + GEP + All signatures  |     0.787 |
| TMB + GEP + All + Circadian |     0.785 |

The circadian addition produced an AUC increase of approximately **0.018**, whereas replacing the circadian component with molecular subtype produced an increase of approximately **0.047**.

---

### Independent Replication

Across the pooled independent replication cohorts:

| Formulation                  |   AUC |
| ---------------------------- | ----: |
| Full 17-gene z-score         | 0.382 |
| Full 17-gene percentile rank | 0.501 |
| Full 17-gene PCA1            | 0.403 |
| TIMELESS + TIPIN             | 0.425 |

The circadian formulations did not reproduce their discovery performance.

---

### External Positive Control

To distinguish biomarker failure from pipeline failure, a frozen **TMB + GEP** reference classifier was trained entirely on discovery data and then applied unchanged to the external cohort.

```text
AUC = 0.757
95% CI = 0.612–0.888
AUPRC = 0.515
p = 0.0005
```

The discovery-to-external difference was not statistically significant:

```text
z = 0.289
p = 0.772
```

In contrast, the circadian discovery-to-replication gap was:

```text
z = 5.158
p < 0.0001
```

This positive-control design supports the interpretation that the negative circadian result is not simply caused by a defective evaluation pipeline.

---

## Key Conclusion

The results do **not** support a reproducible predictive role for the tested static circadian gene-expression signatures in these ICI-treated cohorts.

The important distinction is:

> **Biological plausibility does not guarantee predictive generalizability.**

The study does not argue that circadian biology is irrelevant to antitumor immunity.

Instead, it suggests that a **single-timepoint bulk expression measurement** may not adequately capture the circadian phase and amplitude information implicated by the underlying biology.

Therefore, discovery-cohort improvements of this magnitude should be considered hypotheses until they are demonstrated in samples that played no role in feature selection or model development.

---

## Repository Structure

```text
circadian-ici-response-validation/
│
├── README.md
├── LICENSE
├── CITATION.cff
├── requirements.txt
├── environment.yml
├── pyproject.toml
├── .gitignore
│
├── configs/
│   └── analysis_config.yaml
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── example/
│
├── docs/
│   ├── data_sources.md
│   ├── data_format.md
│   ├── methodology.md
│   └── reproducibility.md
│
├── notebooks/
│   └── README.md
│
├── scripts/
│   ├── run_smoke_test.py
│   ├── run_discovery.py
│   ├── run_replication.py
│   ├── run_external_validation.py
│   └── generate_figures.py
│
├── src/
│   └── circadian_validation/
│       ├── constants.py
│       ├── io.py
│       ├── preprocessing.py
│       ├── features.py
│       ├── models.py
│       ├── metrics.py
│       ├── analysis.py
│       └── plotting.py
│
├── tests/
│   ├── test_features.py
│   ├── test_preprocessing.py
│   └── test_metrics.py
│
├── results/
│   ├── figures/
│   ├── tables/
│   └── metrics/
│
└── paper/
    └── manuscript.pdf
```

---

## Installation

### Option 1 — Python virtual environment

```bash
git clone https://github.com/YOUR_USERNAME/circadian-ici-response-validation.git

cd circadian-ici-response-validation

python -m venv .venv

source .venv/bin/activate

pip install --upgrade pip

pip install -r requirements.txt
```

### Option 2 — Conda

```bash
conda env create -f environment.yml

conda activate circadian-ici-validation
```

---

## Run Tests

Run the complete unit-test suite:

```bash
pytest -q
```

Run the synthetic end-to-end smoke test:

```bash
python scripts/run_smoke_test.py
```

The smoke test uses synthetic data and is intended to verify that the software pipeline executes correctly.

It is **not** intended to reproduce the manuscript's reported clinical results.

---

## Real Dataset Workflow

Patient-level source datasets are **not redistributed in this repository**.

After obtaining the datasets from their respective public sources and converting them into the canonical format described in:

```text
docs/data_format.md
```

the analysis can be executed using:

```bash
python scripts/run_discovery.py \
    --input data/processed/IMvigor210.csv
```

For independent replication:

```bash
python scripts/run_replication.py \
    --input data/processed/GSE78220.csv \
    --input2 data/processed/GSE176307.csv
```

For frozen external validation:

```bash
python scripts/run_external_validation.py \
    --discovery data/processed/IMvigor210.csv \
    --external data/processed/GSE176307.csv
```

---

## Data Availability

The repository intentionally separates:

```text
DATA ACQUISITION
        ↓
DATA PROCESSING
        ↓
ANALYSIS
        ↓
RESULTS
```

Patient-level data should only be downloaded and used according to the terms of the original data providers and publications.

See:

```text
docs/data_sources.md
docs/data_format.md
```

for the expected data structure and provenance requirements.

---

## Reproducibility

The project is designed so that learned transformations are not fitted globally before validation.

The intended workflow is:

```text
Raw public datasets
        ↓
Source-specific preprocessing
        ↓
Canonical cohort format
        ↓
Leakage safeguards
        ↓
Training-fold feature construction
        ↓
Nested model selection
        ↓
Held-out predictions
        ↓
External validation
        ↓
Statistical evaluation
        ↓
Figures and tables
```

See `docs/reproducibility.md` for the reproducibility checklist.

---

## Scientific Scope and Limitations

This repository implements the analytical framework described in the associated manuscript.

Important limitations include:

* Independent replication cohorts are relatively small.
* The tested circadian representation is static and single-timepoint.
* Bulk expression does not directly measure circadian phase or amplitude.
* The study does not establish that circadian biology has no role in immunotherapy.
* The negative replication result constrains a large reproducible predictive effect but does not exclude a small effect.
* Exact numerical reproduction requires the correctly processed source datasets.

---

## Future Directions

Potential extensions include:

* Larger independent cohorts
* Prospective validation
* Serial transcriptomic sampling
* Direct estimation of circadian phase and amplitude
* Time-aware sampling designs
* Integration with additional immune and genomic biomarkers
* Evaluation of whether dynamic circadian features outperform static expression signatures

---

## Citation

If you use this repository or build upon this analysis, please cite the associated manuscript and software repository.

See:

```text
CITATION.cff
```

---

## License

The original software in this repository is released under the MIT License.

The external datasets remain subject to their original licenses, access restrictions, and terms of use.

---

## Reproducibility Statement

This repository is intended to make the computational methodology transparent and reproducible while respecting restrictions on redistribution of patient-level data.

The central principle of the project is:

> **Independent replication is required before treating a discovery-cohort biomarker signal as generalizable evidence.**
> """

### GitHub Topics

Repo-তে এগুলোও add করো:

```text
circadian-rhythm
immunotherapy
cancer-research
immune-checkpoint-inhibitors
biomarker-validation
machine-learning
bioinformatics
transcriptomics
gene-expression
nested-cross-validation
external-validation
reproducible-research
computational-biology
```

**আমার recommendation:** repository name হিসেবে `circadian-ici-response-validation` রাখো। এটা short, academic, এবং paper-এর মূল contribution—**cross-cohort validation**—সরাসরি বোঝায়।
