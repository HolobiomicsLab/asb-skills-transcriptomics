---
name: single-cell-imputed-matrix-comparison
description: Use when you hold raw plus imputed log-transformed scRNA-seq matrices
  (e.g. Jurkat-293T, Usoskin, Kolodziejczyk, PBMC) with cell-type annotations and
  must rank imputation methods by variance stabilization within a cell type.
when_to_use_negative:
- The analysis goal is expression recovery from subsampled/masked data rather than
  intra-population variance stabilization comparison
- Cell-type annotations are unavailable, so no subpopulation can be isolated for within-type
  CV computation
- The input matrix is raw counts that have not undergone the paper's pre-processing
  pipeline (R pre-process.R produces the required format)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: AutoImpute
  role: Autoencoder-based imputation method under evaluation; its imputed matrix is
    anti-logged before CV computation
  repo: github.com/divyanshu-talwar/AutoImpute
- name: numpy
  role: Computes per-gene standard deviation/mean (coefficient of variation) and natural-log
    transform across selected cells
- name: matplotlib
  role: Draws grouped boxplots of log-CV distributions per method for each of the
    four dataset panels
- name: scikit-learn
  role: Dependency of the AutoImpute model used to produce the imputed matrices being
    compared
  repo: github.com/divyanshu-talwar/AutoImpute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:c11209b5f9d058f0fc2e2acfb69a835ff92f7c5cccdb6efd7e17990a02678b24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-imputed-matrix-comparison

## Summary

Benchmark scRNA-seq dropout-imputation methods (AutoImpute, scImpute, MAGIC, drImpute) by quantifying how well each stabilizes intra-subpopulation gene-expression variance, measured as the natural log of the per-gene coefficient of variation across cells of one randomly picked cell type (Fig. 4 protocol of the AutoImpute paper).

## When to use

Use when you hold raw plus imputed log-transformed scRNA-seq matrices (e.g. Jurkat-293T, Usoskin, Kolodziejczyk, PBMC) with cell-type annotations and must rank imputation methods by variance stabilization within a cell type.

## When NOT to use

- The analysis goal is expression recovery from subsampled/masked data rather than intra-population variance stabilization comparison
- Cell-type annotations are unavailable, so no subpopulation can be isolated for within-type CV computation
- The input matrix is raw counts that have not undergone the paper's pre-processing pipeline (R pre-process.R produces the required format)

## Inputs

- Pre-processed log-transformed expression matrices (cells x genes, .csv/.mat) for Usoskin, Kolodziejczyk, Jurkat-293T and PBMC datasets
- Imputed expression matrices for each method: AutoImpute, scImpute, MAGIC, drImpute
- Cell-type/subpopulation annotations per dataset

## Outputs

- cv_summary.csv containing per-gene CV values for the unimputed data and every imputation method
- cv_boxplots.png with four dataset panels of grouped log-CV boxplots per method (Fig. 4 replica)

## How to apply

Load the pre-processed log-transformed expression matrices for each dataset together with the corresponding matrices imputed by AutoImpute, scImpute, MAGIC and drImpute, plus cell-type annotations. Randomly pick one subpopulation (cell type) per dataset and subset both the unimputed and imputed matrices to those cells. For methods that consumed log-transformed input (e.g. AutoImpute), first anti-log back to the linear expression scale so CV is computed on comparable quantities. With numpy, compute per-gene CV (standard deviation / mean) across the selected cells for the unimputed data and every imputation method, then take the natural logarithm of the CV values to amplify fractional differences as done in the paper. Draw grouped matplotlib boxplots of log-CV per method, one panel per dataset (Usoskin, Kolodziejczyk, Jurkat-293T, PBMC), and write per-gene CV values to cv_summary.csv.

## Related tools

- **AutoImpute** (Autoencoder-based imputation method under evaluation; its imputed matrix is anti-logged before CV computation) — github.com/divyanshu-talwar/AutoImpute
- **numpy** (Computes per-gene standard deviation/mean (coefficient of variation) and natural-log transform across selected cells)
- **matplotlib** (Draws grouped boxplots of log-CV distributions per method for each of the four dataset panels)
- **scikit-learn** (Dependency of the AutoImpute model used to produce the imputed matrices being compared) — github.com/divyanshu-talwar/AutoImpute

## Examples

```
python AutoImpute.py --data jurkat.csv --imputed_save imputed_matrix  # then with numpy: cv = np.log(mat.std(axis=0) / mat.mean(axis=0))
```

## Evaluation signals

- cv_boxplots.png exists and is a valid PNG containing four dataset panels each comparing all methods
- cv_summary.csv contains per-gene CV rows for every method (AutoImpute, scImpute, MAGIC, drImpute) plus the unimputed baseline
- AutoImpute achieves the lowest median log-CV in the three datasets where the paper claims maximal variance stabilization (Jurkat, Preimplantation and PBMC)
- In the remaining datasets AutoImpute beats all methods except MAGIC, matching the reported pattern
- Imputed matrices from log-input methods were anti-logged before CV computation (sanity check: imputed values must be non-negative on linear scale)

## Limitations

- Only one randomly picked subpopulation per dataset is analyzed, so results may depend on which cell type is selected
- The paper evaluates on specific datasets (nine total; this skill covers Usoskin, Kolodziejczyk, Jurkat-293T, PBMC, Preimplantation), so the CV-stabilization ranking may not transfer to other tissues or platforms
- AutoImpute requires Python 2.7 with tensorflow and a pre-processed input matrix (columns = genes, rows = cells); raw formats must pass through the R pre-processing script first
- Comparison across methods is sensitive to whether each method's output is on log or linear scale; skipping the anti-log step for log-input methods invalidates the CV comparison

## Evidence

- [abstract] AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization"
- [intro] AutoImpute learns the inherent distribution: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
- [readme] The input to AutoImpute.py is a pre-processed count single cell matrix, with columns representing genes and rows representing cells.: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [intro] We compare the performance of AutoImpute with the existing imputation methods on nine independent datasets.: "We compare the performance of AutoImpute with the existing imputation methods on nine independent datasets."
