---
name: ilisi-batch-mixing-metric-computation
description: Use when you need a quantitative batch-mixing score (complementary to
  kBET rejection rates) for comparing embeddings such as BEENE vs. PCA on batch-corrected
  vs. uncorrected scRNA-seq data.
when_to_use_negative:
- When the goal is a statistical significance test of batch-label mixing rather than
  a local mixing index — kBET (R-only, run via rpy2) returns average test rejection
  rates via Pearson's chi-square and is the complementary metric
- 'When labels cannot be formatted per the API contract: batch_var must be a 1D 0/1
  array for two categories and a dense one-hot matrix for more; a string label vector
  without encoding is invalid input'
- When interpreting mixing alone would be misleading — iLISI should be compared across
  embeddings/corrections on the same data, not treated as an absolute batch-effect-free
  threshold
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BEENE
  role: beene_model class provides evaluate_batch_iLisi, which trains the model and
    computes iLISI on the test split
  repo: https://github.com/ashiq24/BEENE
- name: LISI
  role: upstream metric implementation for Local Inverse Simpson's Index used by BEENE
  repo: https://github.com/immunogenomics/LISI
- name: scikit-learn
  role: OneHotEncoder used to produce the dense one-hot batch/biological label matrices
    required when categories >2
- name: numpy
  role: expression matrix and label arrays passed as numpy arrays
- name: kBET
  role: complementary R-only batch metric; embeddings transferred via rpy2
  repo: https://github.com/theislab/kBET/
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:2895df9f43c755a4a6fe3beac00b598858f2cecad4c259e2668ae9a18bcb9093
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ashiq24/BEENE
  tools:
  - BEENE
  - LISI
  - R
  - kBET
  - numpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# ilisi-batch-mixing-metric-computation

## Summary

Computes the inverse Local Inverse Simpson's Index (iLISI) to quantify batch mixing on a learned embedding (e.g., BEENE), where higher iLISI indicates better mixing of batch labels among neighboring cells. In the BEENE workflow, iLISI is calculated on held-out test samples via beene_model.evaluate_batch_iLisi after a seeded train/validation/test split.

## When to use

Use when you need a quantitative batch-mixing score (complementary to kBET rejection rates) for comparing embeddings such as BEENE vs. PCA on batch-corrected vs. uncorrected scRNA-seq data.

## When NOT to use

- When the goal is a statistical significance test of batch-label mixing rather than a local mixing index — kBET (R-only, run via rpy2) returns average test rejection rates via Pearson's chi-square and is the complementary metric
- When labels cannot be formatted per the API contract: batch_var must be a 1D 0/1 array for two categories and a dense one-hot matrix for more; a string label vector without encoding is invalid input
- When interpreting mixing alone would be misleading — iLISI should be compared across embeddings/corrections on the same data, not treated as an absolute batch-effect-free threshold

## Inputs

- 2D numpy array of gene expression per sample (genes as columns)
- batch_var labels: 1D 0/1 array for two categories, or dense one-hot matrix (scikit-learn OneHotEncoder) for >2 categories
- bio_var biological labels (same formatting rules as batch_var)
- integer random seed for the train/validation/test split (e.g., seed=20)
- trained BEENE model (via get_hybrid_model_1, e.g., hidden layers [50,20], embedding dimension 5, reconstruction_weight=1, batch_weight=2, bio_weight=1)

## Outputs

- iLISI value for each sample in the held-out test set
- median iLISI per condition, aggregated across embeddings/correction methods into comparison tables (e.g., batch_effect_metrics.csv, method_comparison_table.csv)

## How to apply

Prepare the gene-expression matrix as a 2D numpy array (rows = samples, columns = genes) plus batch labels and biological labels. Format batch_var/bio_var as a 1D 0/1 array for two classes or a dense one-hot matrix (scikit-learn OneHotEncoder) for three or more categories, e.g., the three PBMC Chromium 10X protocols (3pV1, 3pV2, 5p). Call beene_model.evaluate_batch_iLisi(data, batch_var, bio_var, seed) — the function trains the BEENE model on the training split, selects the best model on the validation split, and returns per-sample iLISI values on the randomly chosen test set; use a fixed seed (e.g., 20) for reproducibility. Report the median iLISI on the test set, and compare across embedding spaces (BEENE, PCA, t-SNE, UMAP) and correction states (uncorrected, ComBat, Harmony); in the BEENE paper, median iLISI rises from ~1 to ~2.5 after Harmony correction on PBMC data, while PCA space yields higher iLISI (~1.76) than BEENE (1.57) on ComBat-corrected mESC data.

## Related tools

- **BEENE** (beene_model class provides evaluate_batch_iLisi, which trains the model and computes iLISI on the test split) — https://github.com/ashiq24/BEENE
- **LISI** (upstream metric implementation for Local Inverse Simpson's Index used by BEENE) — https://github.com/immunogenomics/LISI
- **scikit-learn** (OneHotEncoder used to produce the dense one-hot batch/biological label matrices required when categories >2)
- **numpy** (expression matrix and label arrays passed as numpy arrays)
- **kBET** (complementary R-only batch metric; embeddings transferred via rpy2) — https://github.com/theislab/kBET/

## Examples

```
lisi_values = my_model.evaluate_batch_iLisi(Xt, bt, yt, 20)
```

## Evaluation signals

- Returned iLISI array covers only the randomly chosen test-set samples, not the full dataset
- Using a fixed seed (e.g., 20) reproduces the same train/validation/test split across runs and across embedding comparisons
- Expected pattern on PBMC data: median iLISI rises from ~1 (uncorrected/ComBat) to ~2.5 after Harmony correction on BEENE embeddings
- On ComBat-corrected mESC data, BEENE iLISI ≈ 1.57 vs ~1.76 in PCA space; deviations beyond these ranges suggest a data- or label-formatting error
- Label arrays validate: batch_var is 1D 0/1 for 2 categories, dense one-hot for >2 (e.g., 3-column matrix for 3pV1/3pV2/5p)

## Limitations

- iLISI is computed only on the held-out test set of a random split; results depend on the chosen seed
- Two technical mESC replicates gave iLISI ~1.57–1.76 even after ComBat, so iLISI values must be interpreted relative to a baseline embedding, not absolutely
- The R LISI/kBET ecosystem requires R >= 4.1.0 with rpy2 >= 3.4.0 and Linux is recommended for rpy2 compatibility

## Evidence

- [methods] evaluate_batch_iLisi call signature: [evidence span withheld — anchor status: not_found]
- [readme] iLISI computed on held-out test samples: "returns: iLISI values of each of the samples in the randomly chosen test set using BEENE embedding"
- [readme] batch label format requirements: "batch_var: numpy array. For more than two categories, it must be one hot representation of batch labels"
- [methods] LISI as batch effect estimation metric: "LISI is a batch effect estimation metric [Link](https://github.com/immunogenomics/LISI)"
