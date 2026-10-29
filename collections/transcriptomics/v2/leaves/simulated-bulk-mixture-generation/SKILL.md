---
name: simulated-bulk-mixture-generation
description: Use when benchmarking deconvolution or cell-type proportion estimation
  and you need synthetic bulk profiles with known true mixing proportions, e.g., 100
  random mixtures over the five GSE28490 cell types.
when_to_use_negative:
- The evaluation data are real bulk tissues without known true proportions — simulated
  ground truth cannot be constructed from them.
- The goal is marker gene detection itself rather than deconvolution benchmarking
  — use the COT pipeline (cos_pipeline / obtain_subtype_markers) instead.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3523
tools:
- name: COT
  role: Provides the COT-derived marker gene set and the subtype expression profiles
    used as the pure components of the simulated mixtures
  repo: https://github.com/MintaYLu/COT
- name: pandas
  role: Loads the GSE28490 cell-type expression profiles and assembles the simulated
    mixtures and summary tables
- name: NumPy
  role: Draws random mixture proportions and linearly combines cell-type profiles
    into bulk profiles
- name: scikit-learn
  role: Performs marker-gene-restricted least-squares deconvolution of the simulated
    mixtures
- name: SciPy
  role: Computes sample-wise correlations between true and estimated proportions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:75c30bad412869405d911834114d3d1acc9da9712d379e9674449a1402b60854
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MintaYLu/COT
  tools:
  - COT
  - NumPy
  - Python
  - SciPy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# simulated-bulk-mixture-generation

## Summary

Generates in-silico bulk expression mixtures by combining known cell-type expression profiles (e.g., the five GSE28490 benchmark cell types) with randomly drawn mixture proportions, providing ground-truth proportions for benchmarking marker-gene-guided deconvolution.

## When to use

Use when benchmarking deconvolution or cell-type proportion estimation and you need synthetic bulk profiles with known true mixing proportions, e.g., 100 random mixtures over the five GSE28490 cell types.

## When NOT to use

- The evaluation data are real bulk tissues without known true proportions — simulated ground truth cannot be constructed from them.
- The goal is marker gene detection itself rather than deconvolution benchmarking — use the COT pipeline (cos_pipeline / obtain_subtype_markers) instead.

## Inputs

- GSE28490 benchmark dataset (purified cell-type expression profiles of five cell types) downloaded from GEO
- Number of simulated mixtures (100) and random mixture proportions over the five cell types

## Outputs

- Matrix of 100 simulated bulk expression profiles (NumPy array / pandas DataFrame)
- Ground-truth cell-type proportion matrix (100 samples x 5 cell types) used to compute RMSE and sample-wise correlations (Supplementary Table S2)

## How to apply

Load the purified cell-type expression profiles (GSE28490 from GEO) with pandas, then draw random cell-type mixture proportions for each of 100 simulated bulk samples and combine the five cell-type profiles with NumPy to form the bulk matrix. Retain the true proportions per sample as the ground truth for downstream RMSE and sample-wise correlation evaluation. The same simulated mixtures are then deconvolved under competing marker gene sets (COT-derived via COT.obtain_subtype_markers(), one-vs-rest t-test-derived, and a priori) so that differences in RMSE/correlation are attributable to the MG sets rather than the mixtures. Note that COT's input should be batch-corrected subtype expression profiles before marker detection on the source profiles.

## Related tools

- **COT** (Provides the COT-derived marker gene set and the subtype expression profiles used as the pure components of the simulated mixtures) — https://github.com/MintaYLu/COT
- **pandas** (Loads the GSE28490 cell-type expression profiles and assembles the simulated mixtures and summary tables)
- **NumPy** (Draws random mixture proportions and linearly combines cell-type profiles into bulk profiles)
- **scikit-learn** (Performs marker-gene-restricted least-squares deconvolution of the simulated mixtures)
- **SciPy** (Computes sample-wise correlations between true and estimated proportions)

## Examples

```
cot = COT(df_raw=df_raw, normalization=False); cot.cos_pipeline(subtype_label=subtype_label, top=2)
```

## Evaluation signals

- Each simulated bulk profile equals the weighted sum of the five GSE28490 cell-type profiles with per-sample proportions summing to 1.
- Ground-truth proportion matrix has shape 100 x 5 and matches the proportions used to build each mixture.
- Downstream deconvolution of the COT-MG set yields the lowest RMSE and highest sample-wise correlation versus OVR t-test and a priori MG sets (Supplementary Table S2 reproducible in supplementary_table_s2.csv).
- Random proportions are drawn independently per sample so mixtures span the simplex rather than clustering near uniform.

## Limitations

- COT cannot calculate p-values when there are too few genes; the input subtype profiles should have sufficient genes (see the Example_GSE28490 vignette).
- COT input must be batch-corrected; uncorrected profiles can bias both marker detection and the simulated mixtures.
- Random mixtures are one simulation scheme; results may not transfer to real bulk tissues with unknown or correlated proportions.

## Evidence

- [supplementary] Benchmark dataset used in this paper can be downloaded from Gene Expression Omnibus (https://www.ncbi.nlm.nih.gov/geo/) under the accession number: GSE28490.: "Benchmark dataset used in this paper can be downloaded from Gene Expression Omnibus (https://www.ncbi.nlm.nih.gov/geo/) under the accession number: GSE28490."
- [results] COT-MG deconvolution benchmark: [evidence span withheld — anchor status: not_found]
- [readme] COT input must be batch-corrected: "Note that the input of COT should be batch-corrected."
- [readme] COT workflow via pipeline: "cot = COT(df_raw=df_raw, normalization=False)
    cot.cos_pipeline(subtype_label=subtype_label, top=2)"
