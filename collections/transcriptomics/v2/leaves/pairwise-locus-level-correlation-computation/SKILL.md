---
name: pairwise-locus-level-correlation-computation
description: 'Use when evaluating BIRD predictions across multiple cell types: you
  have predicted and true DH matrices for ≥3 cell types and need pair-wise locus-level
  accuracy of differential DH, stratified by cell-type similarity.'
when_to_use_negative:
- When you need to evaluate predicted absolute DH profiles for a single cell type
  rather than differences between pairs
- When loci are not aligned between predicted and true matrices (BIRD's human genome
  hg19/hg38 coordinate system must match the DNase-seq reference)
tools:
- name: BIRD_predict
  role: Generates the predicted DH matrix (log2(x+1) transformed) that is differenced
    between cell-type pairs before correlation
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD
  role: Prediction framework whose DH output accuracy is assessed by the pairwise
    correlation computation
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Used to load .rds DNase-seq data, compute pairwise Pearson correlations, and
    aggregate by similarity quartile
  repo: https://www.r-project.org
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:93d686afe7ba4c9439ab405e467e04f52cb5c4310249025bd06687bbfa538493
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pairwise-locus-level-correlation-computation

## Summary

Computes per-pair Pearson correlations between BIRD-predicted and true DNase I hypersensitivity (DH) differences across ~1M genomic loci for every cell-type pair, optionally restricted to differential loci, to quantify how accurately BIRD predicts differential DH.

## When to use

Use when evaluating BIRD predictions across multiple cell types: you have predicted and true DH matrices for ≥3 cell types and need pair-wise locus-level accuracy of differential DH, stratified by cell-type similarity.

## When NOT to use

- When you need to evaluate predicted absolute DH profiles for a single cell type rather than differences between pairs
- When loci are not aligned between predicted and true matrices (BIRD's human genome hg19/hg38 coordinate system must match the DNase-seq reference)

## Inputs

- BIRD-predicted DH matrix in log2(x+1) scale (data-matrix output of BIRD_predict), one column per cell type, ~1M loci
- True DNase-seq profiles for the same cell types (e.g. DNase_data_167_cells.rds, 200bp windows, first three columns = locus location)
- Precomputed cell-type similarity measure or train-test distances for quartile stratification
- Locus-level DH values per cell type (log2(x+1) transformed) for computing pairwise differences

## Outputs

- Per-pair Pearson correlation of differential DH across all loci (136 values for 17 cell types)
- Per-pair Pearson correlation of differential DH across differential loci only
- Mean correlation per similarity-quartile stratum and locus subset (all vs differential loci)

## How to apply

Load the BIRD-predicted DH matrix (produced by BIRD_predict in log2(x+1) scale, e.g. via human_hg38_model.bin) and the true DNase-seq profiles (e.g. DNase_data_167_cells.rds from BIRD-data, 200bp windows with the first three columns giving locus location). Form all 136 unique pairs among the 17 test cell types and compute, per locus, the difference in DH between the two cell types for both predicted and true data. Calculate a Pearson correlation between predicted and true differential DH across all loci, and repeat on the subset of differential loci only. Optionally assign each pair to a cell-type similarity quartile (e.g. via the train-test distance R script released with BIRD) and aggregate mean correlations per stratum: higher-similarity pairs show lower correlations (about 0.42 all loci / 0.55 differential loci) while more dissimilar pairs reach about 0.60-0.75.

## Related tools

- **BIRD_predict** (Generates the predicted DH matrix (log2(x+1) transformed) that is differenced between cell-type pairs before correlation) — https://github.com/WeiqiangZhou/BIRD
- **BIRD** (Prediction framework whose DH output accuracy is assessed by the pairwise correlation computation) — https://github.com/WeiqiangZhou/BIRD
- **R** (Used to load .rds DNase-seq data, compute pairwise Pearson correlations, and aggregate by similarity quartile) — https://www.r-project.org

## Examples

```
DNase_167_cells <- readRDS("DNase_data_167_cells.rds"); diff_pred <- pred_mat[,i] - pred_mat[,j]; r_ij <- cor(diff_pred, diff_true, method="pearson")
```

## Evaluation signals

- Correlation values must lie in [-1, 1]; reproduce the reported pattern where the highest-similarity quartile has mean correlations about 0.42 (all loci) and 0.55 (differential loci)
- Remaining quartiles should show higher mean correlations (about 0.60-0.66 and 0.69-0.75), confirming better prediction for dissimilar pairs
- Exactly 136 unique cell-type pairs are formed for 17 test cell types; verify no duplicated or missing pairs
- Predicted matrix values should be within BIRD's bounded range (0 to 14 before log2(x+1) transformation; check the -u option default)
- Locus coordinates in the first three columns of the true DNase data must match the BIRD model's genome build (hg19 or hg38)

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample, so differential analysis is restricted to loci in the model's training universe
- Genomic information in the current version of BIRD is based on human genome hg19 (hg38 models are separate releases); mixing builds invalidates locus alignment
- Predictions depend on model choice: hg38 RNA-seq model trained with 174 ENCODE samples is the suggested current release; older models (70 Roadmap samples) may behave differently

## Evidence

- [readme] Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed).: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [readme] The first three columns in the data contains the location information of a genomic locus (200bp window).: [evidence span withheld — anchor status: external_doc]
- [discussion] one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
- [methods] Genomic information in the current version of BIRD is based on human genome hg19.: "Genomic information in the current version of BIRD is based on **human genome hg19**."
