---
name: knn-imputation-benchmarking
description: Use when comparing imputation methods for highly sparse APA ratio (RUD)
  matrices from spatially barcoded ST data, e.g. deciding whether stAPAminer's k-nearest-neighbor
  imputation or scDaPars better recovers APA signals.
when_to_use_negative:
- Input is already an imputed or dense APA ratio matrix — benchmarking only makes
  sense on the raw, sparse RUD matrix prior to imputation
- Data are non-spatial single-cell or image-based (MERFISH/seqFISH) — the article
  notes APA sites can only be identified from spatially barcoded ST data with existing
  tools such as scAPAtrap or Sierra
- The goal is APA site discovery rather than imputation benchmarking — use the upstream
  scAPAtrap/Sierra PA-extraction pipeline instead
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stAPAminer
  role: provides computeAPAIndex, imputeAPAIndex (KNN imputation, k=10), makeStCluster,
    computeIndex, and computePearsonIndex used to impute and score the RUD matrices
  repo: https://github.com/BMILAB/stAPAminer
- name: scDaPars
  role: competitor method; imputes the same raw RUD matrix with default nearest-neighbor
    filling as the baseline for comparison
- name: Seurat
  role: clustering engine behind makeStCluster used to evaluate clustering quality
    of each imputed matrix
- name: ggplot2
  role: draws the grouped boxplot (fill=source) comparing imputation methods and saves
    the figure and metrics table
- name: ClusterR
  role: required dependency of stAPAminer for clustering/index computation (distinctColorPalette,
    KNN support)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:cb07f29e50228f29360e07e26909d4d5f37dc1032c9408971f7457bbdcbcf168
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - ClusterR
  - Seurat
  - cluster
  - clusterSim
  - fdm2id
  - ggplot2
  - scDaPars
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# knn-imputation-benchmarking

## Summary

Benchmarks stAPAminer's KNN-based imputation of sparse spatial APA usage (RUD) matrices against scDaPars imputation on mouse olfactory bulb (MOB) ST data, scoring both by within-layer spot correlations and Seurat-based clustering quality.

## When to use

Use when comparing imputation methods for highly sparse APA ratio (RUD) matrices from spatially barcoded ST data, e.g. deciding whether stAPAminer's k-nearest-neighbor imputation or scDaPars better recovers APA signals.

## When NOT to use

- Input is already an imputed or dense APA ratio matrix — benchmarking only makes sense on the raw, sparse RUD matrix prior to imputation
- Data are non-spatial single-cell or image-based (MERFISH/seqFISH) — the article notes APA sites can only be identified from spatially barcoded ST data with existing tools such as scAPAtrap or Sierra
- The goal is APA site discovery rather than imputation benchmarking — use the upstream scAPAtrap/Sierra PA-extraction pipeline instead

## Inputs

- poly(A) site matrix as a movAPA PACdataset (APA.RDA) with rows = poly(A) sites and columns = spots
- gene-spot count matrix (count.csv)
- spot barcode coordinates table (position.txt)
- raw RUD (relative usage of distal poly(A) site) matrix from computeAPAIndex

## Outputs

- two imputed RUD matrices (stAPAminer KNN k=10 and scDaPars)
- clustering quality index per imputation method (computeIndex)
- within-layer Pearson correlation scores of spots (computePearsonIndex, source='RUD')
- grouped ggplot2 boxplot comparing stAPAminer vs scDaPars (Figure 3E/F analog) and aggregated metrics table

## How to apply

Load the raw APA data (APA.RDA), gene-spot count matrix (count.csv) and spot coordinates (position.txt) from stAPAminer's extdata, compute the raw RUD matrix with computeAPAIndex(APA, rownames(position)), and impute it with imputeAPAIndex(RUD_RAW, count, k=10, init=TRUE). Impute the same raw RUD matrix with scDaPars using its default nearest-neighbor filling on the replicate MOB dataset. Cluster both imputed matrices with makeStCluster (Seurat-based; the verification vignette uses resolution=0.6, dims=1:10, k=30, nfeatures=4500; the benchmark uses resolution=0.7) and score each with computeIndex. Compute within-layer Pearson correlations of spots per layer (GCL, GL, ONL, OPL, MCL) with computePearsonIndex(stObj, 'RUD'), then aggregate correlation and clustering indices per method into a metrics table and draw a grouped ggplot2 boxplot (fill=source) comparing stAPAminer vs scDaPars (Figure 3E and F).

## Related tools

- **stAPAminer** (provides computeAPAIndex, imputeAPAIndex (KNN imputation, k=10), makeStCluster, computeIndex, and computePearsonIndex used to impute and score the RUD matrices) — https://github.com/BMILAB/stAPAminer
- **scDaPars** (competitor method; imputes the same raw RUD matrix with default nearest-neighbor filling as the baseline for comparison)
- **Seurat** (clustering engine behind makeStCluster used to evaluate clustering quality of each imputed matrix)
- **ggplot2** (draws the grouped boxplot (fill=source) comparing imputation methods and saves the figure and metrics table)
- **ClusterR** (required dependency of stAPAminer for clustering/index computation (distinctColorPalette, KNN support))

## Examples

```
RUD_RAW <- computeAPAIndex(APA,rownames(position)); RUD <- imputeAPAIndex(RUD_RAW,count,k=10,init=TRUE)
```

## Evaluation signals

- stAPAminer-imputed matrix should outperform scDaPars on every performance indicator used (correlation and clustering index), as reported in Figure 3E and F
- computePearsonIndex(stObj, 'RUD') must return within-layer spot correlations for each morphological layer (GCL, GL, ONL, OPL, MCL) for both imputed matrices
- Both imputed matrices must be clustered under identical makeStCluster settings (resolution, dims=1:10, k=30, nfeatures=4500) so the computeIndex comparison is fair
- The metrics table must contain one row/group per imputation method with both correlation and clustering-index values feeding the grouped boxplot
- Remaining missing entries after the KNN iteration process are set to 0 — the imputed matrices should contain no NAs

## Limitations

- No method exists to determine whether an APA ratio matrix entry is truly missing before imputation
- Missing APA usage entries remaining after the entire imputation iteration process are arbitrarily set to 0
- No version numbers are reported for any of the software tools used in the comparison
- The comparison is limited to spatially barcoded ST data (MOB); applicability to MERFISH/seqFISH is not supported

## Evidence

- [results] Regardless of the performance indicators used, stAPAminer outperformed scDaPars: [evidence span withheld — anchor status: not_found]
- [abstract] an imputation model based on the k-nearest neighbors algorithm was designed to recover APA signals: "an imputation model based on the k-nearest neighbors algorithm was designed to recover APA signals"
- [methods] RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE): "RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE)"
- [readme] stObj <- makeStCluster(stObj,resolution = 0.6,dims = 1:10,k=30,nfeatures = 4500): [evidence span withheld — anchor status: external_doc]
- [discussion] If there are still missing entries after the entire iteration process, we can directly set their values to 0.: "If there are still missing entries after the entire iteration process, we can directly set their values to 0."
