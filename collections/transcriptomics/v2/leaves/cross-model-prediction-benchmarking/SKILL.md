---
name: cross-model-prediction-benchmarking
description: Use when comparing prediction accuracy of a new model or aggregation
  scheme (e.g., cluster/pathway-level) against a baseline model like BIRD at multiple
  DHS cluster resolutions.
when_to_use_negative:
- You want BIRD to discover new DHS locations not in the training catalog — BIRD will
  not discover new locations of DHSs on a new gene expression sample
- The input expression data uses a platform/gene-id scheme the BIRD model cannot match
  (e.g., non-ensembl ids without prior matching)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: BIRD
  role: Baseline locus-level prediction model whose performance is benchmarked against
    the pathway-level BIR(X-bar, Y-bar) model
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Generates predicted DH values (log2(x+1)) for each test sample from FPKM expression
    matrices, handling ensembl id matching and robust quantile normalization
  repo: https://github.com/WeiqiangZhou/BIRD
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:b9146234caf6fe65cc00c293722cedccb149f08f3d8f8a5f8d1b34b53bdc9a9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cross-model-prediction-benchmarking

## Summary

Benchmark alternative prediction models (e.g., pathway-level BIR(X-bar, Y-bar) aggregating co-activated DHS clusters vs. locus-level BIRD) by comparing correlation coefficients between predicted and measured DNase I hypersensitivity across genomic resolutions.

## When to use

Use when comparing prediction accuracy of a new model or aggregation scheme (e.g., cluster/pathway-level) against a baseline model like BIRD at multiple DHS cluster resolutions.

## When NOT to use

- You want BIRD to discover new DHS locations not in the training catalog — BIRD will not discover new locations of DHSs on a new gene expression sample
- The input expression data uses a platform/gene-id scheme the BIRD model cannot match (e.g., non-ensembl ids without prior matching)

## Inputs

- FPKM gene expression data matrices for 17 test cell types (gene ensembl ids in the first column) from the BIRD-data repository
- Prebuilt BIRD prediction model (e.g., human_hg38_model.bin)
- Measured DNase I hypersensitivity signals for the test cell types
- DHS cluster definitions at 1000, 2000, and 5000 cluster resolutions

## Outputs

- Per-cluster correlation coefficients (rC) between predicted and measured DH for each model
- Summary statistics (mean rC, fraction of clusters with rC > 0.5 and rC > 0.75) per model per cluster resolution
- rc_evaluation_summary.csv with per-cluster and summary results

## How to apply

Run the baseline BIRD model and the comparison model (e.g., pathway-level BIR(X-bar, Y-bar)) on the same test samples, here 17 test cell types with FPKM expression matrices from the BIRD-data repository. Match each predicted DH vector (in log2(x+1) scale) to the measured DNase-seq signal at the same genomic positions. Compute per-cluster correlation coefficients (rC) between predicted and measured DH, evaluated at 1000, 2000, and 5000 DHS cluster resolutions. Aggregate rC values into mean rC per model per resolution, plus the fraction of clusters exceeding rC = 0.5 and rC = 0.75. Accept the benchmark if the pathway-level model reproduces the reported mean rC of 0.71 versus 0.50 for BIRD at 1000 clusters, with ~84% and ~55% of clusters exceeding rC = 0.5 and 0.75 respectively, and agreement holds at 2000 and 5000 clusters.

## Related tools

- **BIRD** (Baseline locus-level prediction model whose performance is benchmarked against the pathway-level BIR(X-bar, Y-bar) model) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Generates predicted DH values (log2(x+1)) for each test sample from FPKM expression matrices, handling ensembl id matching and robust quantile normalization) — https://github.com/WeiqiangZhou/BIRD

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- Mean rC for the pathway-level model at 1000 clusters equals ~0.71 vs ~0.50 for BIRD
- ~84% of clusters exceed rC = 0.5 and ~55% exceed rC = 0.75 at the 1000-cluster resolution
- Similar agreement between predicted and measured DH holds at 2000 and 5000 cluster resolutions
- Pathway-level BIR(X-bar, Y-bar) prediction exceeds locus-level BIRD performance at all tested resolutions
- Predicted values are in log scale (log2(x+1) transformed) and bounded 0 to 14

## Limitations

- BIRD will not discover new locations of DHSs when applied to a new gene expression sample — benchmarking is restricted to known DHS loci/clusters
- BIRD's genomic information is based on the human genome build of the model (e.g., hg19 for training data in BIRD-data); genome-build mismatch invalidates matching to measured DH
- Genomic information in the current version of BIRD is based on human genome hg19 — older models may not support hg38 coordinates

## Evidence

- [results] pathway-level prediction: [evidence span withheld — anchor status: not_found]
- [methods] rC computation: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [readme] log-scale output: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [readme] gene id matching: "Input data  matching (gene id matching) is now included in the BIRD_predict program."
- [discussion] BIRD limitation: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
