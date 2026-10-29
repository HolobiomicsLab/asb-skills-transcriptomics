---
name: clustering-agreement-metric-computation
description: Use when you have predicted cluster labels and reference cell-type labels
  for the same cells and need per-dataset, per-method ARI/AMI/NMI scores for a clustering
  benchmark comparison.
when_to_use_negative:
- The dataset has no ground-truth cell-type labels, so external agreement metrics
  like ARI/AMI/NMI cannot be computed
- You need internal (reference-free) clustering validation, e.g., silhouette-based
  cluster-number prediction as done inside CASCC's predictClusterK
- Label vectors are not discrete cluster assignments (e.g., soft membership probabilities
  or continuous embeddings)
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: aricode
  role: R package used to compute ARI, AMI, and NMI for every dataset-method pair
    against ground-truth annotations
- name: CASCC
  role: Produces the predicted cluster labels being scored; benchmarked against six
    baseline methods
  repo: https://github.com/LingyiC/CASCC
- name: CASCC_benchmark
  role: Provides the benchmark code and the 15 evaluation datasets with ground-truth
    labels used in the ARI/AMI/NMI comparison
  repo: https://github.com/LingyiC/CASCC_benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:216fca6b7287526dba21eab36d445d93905e00b883d40ea4dabb82eca18ca713
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - CASCC_benchmark
  - R
  - aricode
  - cafr
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# clustering-agreement-metric-computation

## Summary

Computes label-agreement metrics (ARI, AMI, NMI) between predicted cluster assignments and ground-truth annotations to quantify scRNA-seq clustering accuracy, as done in the CASCC benchmark against six widely used methods (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN).

## When to use

Use when you have predicted cluster labels and reference cell-type labels for the same cells and need per-dataset, per-method ARI/AMI/NMI scores for a clustering benchmark comparison.

## When NOT to use

- The dataset has no ground-truth cell-type labels, so external agreement metrics like ARI/AMI/NMI cannot be computed
- You need internal (reference-free) clustering validation, e.g., silhouette-based cluster-number prediction as done inside CASCC's predictClusterK
- Label vectors are not discrete cluster assignments (e.g., soft membership probabilities or continuous embeddings)

## Inputs

- Predicted cluster label vector (one integer/factor label per cell) from CASCC or a baseline method (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN)
- Ground-truth cell-type annotation vector for the same cells from the LingyiC/CASCC_benchmark repository (15 real scRNA-seq evaluation datasets from Krzak et al. 2019)
- R session with aricode installed (R 4.0.5, Seurat 4.3.0, cafr 0.312 per the paper's sessionInfo)

## Outputs

- Per-dataset, per-method table of ARI, AMI, and NMI scores
- benchmark_ari_ami_nmi_scores.csv
- Aggregated mean ARI per method for comparison against Fig. 1b / Supplementary Table S3

## How to apply

For each dataset-method pair, align the predicted cluster label vector from the clustering method (e.g., CASCC::run.CASCC output on the normalized log-transformed count matrix, or a baseline method's assignment) with the ground-truth cell-type annotation vector over the same cells. Compute ARI, AMI, and NMI with the aricode package, which accompanies the CASCC benchmark environment, for every dataset-method pair. Aggregate all metrics into a per-dataset, per-method score table and save it as benchmark_ari_ami_nmi_scores.csv. Validate by checking that the computed values reproduce the per-dataset clustering scores in Fig. 1b and Supplementary Table S3, where CASCC improved mean ARI by 34% over the second-best method (SC3) across the 15 real scRNA-seq evaluation datasets.

## Related tools

- **aricode** (R package used to compute ARI, AMI, and NMI for every dataset-method pair against ground-truth annotations)
- **CASCC** (Produces the predicted cluster labels being scored; benchmarked against six baseline methods) — https://github.com/LingyiC/CASCC
- **CASCC_benchmark** (Provides the benchmark code and the 15 evaluation datasets with ground-truth labels used in the ARI/AMI/NMI comparison) — https://github.com/LingyiC/CASCC_benchmark

## Examples

```
library(aricode); ari <- ARI(pred_labels, true_labels); ami <- AMI(pred_labels, true_labels); nmi <- NMI(pred_labels, true_labels)
```

## Evaluation signals

- Per-dataset ARI/AMI/NMI values for CASCC and each baseline reproduce the scores in Fig. 1b and Supplementary Table S3 of the paper
- All ARI/AMI/NMI values fall in their valid ranges (ARI can be negative to 1; AMI/NMI between 0 and 1) and are exactly 1.0 only if labels match perfectly
- Mean ARI across the 15 datasets shows CASCC exceeding the second-best method (SC3) by ~34%
- Output file benchmark_ari_ami_nmi_scores.csv contains one row per dataset-method pair with all three metrics populated
- Label vectors for a given dataset are aligned over the identical cell set for prediction and ground truth (same length and ordering)

## Limitations

- Metrics require complete ground-truth annotations; results are specific to the 15 real scRNA-seq evaluation datasets from Krzak et al. 2019 used in the CASCC benchmark
- Reproducibility depends on the recorded environment (R 4.0.5, Seurat 4.3.0, cafr 0.312); version drift may change baseline cluster labels and thus metric values
- ARI/AMI/NMI compare hard cluster assignments only and do not capture clustering stability or biological interpretability (e.g., attractor correspondence)

## Evidence

- [intro] CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm.: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
- [methods] res <- CASCC::run.CASCC(Data_PDAC_peng_2k): "res <- CASCC::run.CASCC(Data_PDAC_peng_2k)"
- [readme] Code for CASCC benchmark.: [evidence span withheld — anchor status: external_doc]
