# CADD_Seminar_2025

## QSAR Modelling of D2 Dopamine Receptor Antagonists

Predicting ligand binding affinity (pKi) for the dopamine D2 receptor with classical QSAR
machine learning — and showing why strong internal test scores don't survive an external dataset.

## Data

DRD2 bioactivity from [GPCRdb](https://gpcrdb.org/) (aggregating ChEMBL and Guide to Pharmacology).
Restricted to Ki only, then salt-stripped, canonicalised, and deduplicated with RDKit →
**6,636 unique compounds**. IC50 values were excluded: they depend on radioligand concentration
and the data lacks the metadata for a Cheng–Prusoff conversion. External evaluation used a
467-compound challenge set held out entirely from training and tuning.

## Approach

Two representations — **RDKit descriptors** (216 physicochemical features) and **Morgan
fingerprints** (substructure bit vectors) — each paired with **XGBoost** and an **MLP**.
80/20 random split, fixed seed.

## Results

| Model | RMSE | R² |
|---|---|---|
| RDKit + XGBoost | **0.608** | **0.658** |
| Morgan + XGBoost | 0.617 | 0.647 |
| Morgan + MLP | 0.663 | 0.593 |
| RDKit + MLP | 0.758 | 0.453 |
| RDKit + XGBoost on **challenge set** | 1.056 | 0.088 |

Internal RMSE near 0.6 log units is close to the experimental noise floor of aggregated
ChEMBL-style data (~0.6 log units, Landrum & Riniker 2024). On the challenge set performance
collapses: predictions cluster in a narrow pKi band regardless of true activity — regression to
the mean, the signature of a model outside its applicability domain.

## Why it fails

A fuzzy-matching pipeline (Levenshtein, >75% similarity) merged 878 fragmented assay
descriptions into 254 semantic clusters. Retraining on the cleanest cluster did not close the
gap — which is the informative result. The bottleneck is representation, not label noise:
2D features can't distinguish enantiomers, ignore conformational flexibility, and can't
recognise novel scaffolds that fit the same pocket.

**Next step:** move from 2D topology to 3D geometry — RDKit conformer generation with
shape-aware descriptors, plus an explicit applicability-domain check.

## Running it

```bash
pip install rdkit pandas numpy scikit-learn xgboost matplotlib seaborn scipy
jupyter lab final_talktorial.ipynb
```

## Attribution

Group project for the Computer-Aided Drug Design Seminar 2025, [Volkamer Lab](https://volkamerlab.org/),
Saarland University. Authors: Abdelsalam Helala, Abdulrahman Walid, Ahmed Hassaan, Saleha Attar,
Maham Ayesha, Fouzia Nasreen. Original repo: `volkamerlab/CADDSeminar_2025`.
