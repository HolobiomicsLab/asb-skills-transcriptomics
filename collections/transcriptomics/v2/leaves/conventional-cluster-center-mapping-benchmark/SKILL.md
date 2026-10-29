---
name: conventional-cluster-center-mapping-benchmark
description: Use when evaluating whether a perturbation-effect estimator corrects
  'infected proportion bias' better than clustering-based mapping, via synthetic two-cluster
  and four-cluster scCRISPR-seq simulations with known ground truth.
when_to_use_negative:
- Input is a single real scCRISPR-seq dataset without simulated ground truth — error
  against known cellular response cannot be computed
- The perturbation data contain no cluster-composition differences between perturbation
  and control groups, so infected proportion bias is absent and the conventional mapping
  baseline is adequate
- Cell systems violate scDecouple's assumptions (intricate/tumor tissues, rare cell
  types absent from infection, or very high heterogeneity), making the comparison
  unfair to the method under test
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDecouple
  role: 'Method under test: decouples cellular response from infected proportion bias
    via run_scDecouple(Y, Z) with default parameters'
  repo: https://github.com/MengQiuchen/scDecouple
- name: Seurat
  role: Implements the conventional baseline by clustering simulated control-group
    cells and mapping perturbation effects through cluster centers
- name: tidyverse
  role: R data manipulation and aggregation of per-setting error metrics into the
    comparison table
- name: mixtools
  role: Gaussian mixture model fitting used within scDecouple's EM-based decoupling
- name: devtools
  role: Installs the developmental scDecouple R package from GitHub
  repo: https://github.com/MengQiuchen/scDecouple
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:b34d4b88c44c32c2f62a51f715c7b67836db719391fd018eabb7fbf53ca23c85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - Seurat
  - devtools
  - diptest
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# conventional-cluster-center-mapping-benchmark

## Summary

A simulation-based benchmarking strategy that contrasts scDecouple's decoupled cellular response estimates against the conventional cluster-and-map baseline (cluster perturbation/control cells, annotate clusters, map perturbation effects via cluster centers) across varied covariance angles, cluster distances, gRNA infection efficiencies and multi-cluster/multi-gRNA settings.

## When to use

Use when evaluating whether a perturbation-effect estimator corrects 'infected proportion bias' better than clustering-based mapping, via synthetic two-cluster and four-cluster scCRISPR-seq simulations with known ground truth.

## When NOT to use

- Input is a single real scCRISPR-seq dataset without simulated ground truth — error against known cellular response cannot be computed
- The perturbation data contain no cluster-composition differences between perturbation and control groups, so infected proportion bias is absent and the conventional mapping baseline is adequate
- Cell systems violate scDecouple's assumptions (intricate/tumor tissues, rare cell types absent from infection, or very high heterogeneity), making the comparison unfair to the method under test

## Inputs

- Synthetic perturbation-group cell-by-gene count matrix Y (two-cluster sweep and four-cluster/10-gRNA scenarios)
- Synthetic control-group cell-by-gene count matrix Z
- Ground-truth simulation values of the cellular response per setting
- Simulation setting grid: covariance matrix angles, inter-cluster distances, gRNA infection efficiencies, run counts (100 for two-cluster, 50 for four-cluster)

## Outputs

- Per-setting estimation errors of scDecouple and the conventional cluster-mapping strategy
- scdecouple_vs_conventional_errors.csv with entries for every simulation setting
- Aggregated comparison showing relative stability (variance) and magnitude of errors

## How to apply

Generate synthetic perturbation-group (Y) and control-group (Z) cell-by-gene count matrices, sweeping covariance matrix angles, inter-cluster distances and gRNA infection efficiencies in a two-cluster scenario (100 runs per setting), plus a four-cluster scenario with 10 gRNAs (50 runs per setting). Run scDecouple on each dataset with run_scDecouple(Y, Z) using defaults (using_rank=TRUE, top.sdev=30, top.multimodal=3, degenes.num=1500, seed_=0). Implement the conventional strategy by clustering the simulated cells (e.g. Seurat clustering on the control group), annotating clusters, and estimating perturbation effects through cluster centers/fold changes. Compute estimation error of the cellular response against the known ground-truth simulation values for both methods in every setting, aggregate errors, and write per-setting metrics to scdecouple_vs_conventional_errors.csv. Rationale: cluster proportions differ between perturbation and control groups under infection bias, so the conventional fold-change-between-cluster-centers approach conflates composition shifts with genuine cellular response, while scDecouple decouples them via maximum likelihood over a Gaussian mixture model with EM.

## Related tools

- **scDecouple** (Method under test: decouples cellular response from infected proportion bias via run_scDecouple(Y, Z) with default parameters) — https://github.com/MengQiuchen/scDecouple
- **Seurat** (Implements the conventional baseline by clustering simulated control-group cells and mapping perturbation effects through cluster centers)
- **tidyverse** (R data manipulation and aggregation of per-setting error metrics into the comparison table)
- **mixtools** (Gaussian mixture model fitting used within scDecouple's EM-based decoupling)
- **devtools** (Installs the developmental scDecouple R package from GitHub) — https://github.com/MengQiuchen/scDecouple

## Examples

```
library(scDecouple); res <- run_scDecouple(Y, Z)
```

## Evaluation signals

- scdecouple_vs_conventional_errors.csv exists with an entry for every simulation setting (all covariance angles, cluster distances, infection efficiencies, and the four-cluster 10-gRNA scenario)
- Across all two-cluster settings (100 runs each), scDecouple's mean estimation error against ground truth is lower than the conventional strategy's
- Error variability across repeated runs is smaller for scDecouple than for the conventional cluster-mapping strategy, matching the reported 'more stable estimation results and lower errors'
- The four-cluster simulation with 10 gRNAs (50 runs per setting) qualitatively reproduces the same scDecouple advantage
- Both methods' errors are computed against the same known ground-truth cellular response values, ensuring a fair paired comparison

## Limitations

- Conventional baseline conclusions depend on how clusters are annotated and mapped to cluster centers; Seurat clustering choices on simulated control cells can shift the baseline's error profile
- Simulation scenarios (Gaussian-mixture-like clusters) favor scDecouple's Gaussian mixture model assumption, which may not hold in intricate systems or disease tissues such as tumors
- The benchmark assumes the same cell clusters exist in NT and perturbation groups; it does not cover cases where rare cell types are completely absent from infection
- Simulation setup details are referenced but not fully described in all sections of the article

## Evidence

- [results] Two-cluster sweep design with 100 runs per setting: [evidence span withheld — anchor status: not_found]
- [results] Stability and error advantage of scDecouple: [evidence span withheld — anchor status: not_found]
- [intro] Rationale: infected proportion bias causes composition differences between groups: "The infection proportion bias refers to the significant differences in the proportions of cell clusters between the perturbation"
- [intro] Conventional fold-change estimator fails under bias: "the traditional perturbation effect estimation by computing the fold change (FC) in the average expression level between the perturbation group and the control group cannot reflect the"
- [readme] Default run_scDecouple invocation from README demo: "res <- run_scDecouple(Y, Z)"
