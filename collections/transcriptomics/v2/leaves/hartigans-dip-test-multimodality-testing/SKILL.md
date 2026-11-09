---
name: hartigans-dip-test-multimodality-testing
description: Use when PCA scores from a control-group scCRISPR-seq matrix must be
  triaged so that only multimodal, high-variance PCs are decoupled for infected proportion
  bias, while remaining PCs use observed change directly.
when_to_use_negative:
- When the dataset is expected to be homogeneous such that no cell-cluster proportion
  differences exist between perturbation (Y) and control (Z) groups — no PC will be
  multimodal and decoupling is unnecessary
- When the GMM assumption may not hold in intricate systems or disease tissues such
  as tumors, or when rare cell types are completely absent from infection, since the
  article notes these break the method's assumptions
- When input is raw unnormalized counts — PCA scores and standard deviations must
  first come from data_preprocessing (log-normalization and highly variable gene detection)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDecouple
  role: Provides the pc_selection function implementing the sdev-rank and dip-test
    PC classification
  repo: https://github.com/MengQiuchen/scDecouple
- name: diptest
  role: R package used to compute Hartigan's dip test statistic for each PC's score
    distribution
- name: R
  role: Runtime environment in which scDecouple and diptest are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:50a0b3b0ca971cbee87dc1b5f42fb36b2bddc33d7963f98af84fd42b58205561
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - diptest
  - scDecouple
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Hartigan's dip test multimodality testing

## Summary

Scores each principal component of control-group (Z) PCA scores for multimodality using Hartigan's dip test (diptest R package) and combines this with explained-spread ranking to flag PCs suspected of carrying infected proportion bias in scCRISPR-seq data, as implemented in scDecouple's pc_selection step.

## When to use

Use when PCA scores from a control-group scCRISPR-seq matrix must be triaged so that only multimodal, high-variance PCs are decoupled for infected proportion bias, while remaining PCs use observed change directly.

## When NOT to use

- When the dataset is expected to be homogeneous such that no cell-cluster proportion differences exist between perturbation (Y) and control (Z) groups — no PC will be multimodal and decoupling is unnecessary
- When the GMM assumption may not hold in intricate systems or disease tissues such as tumors, or when rare cell types are completely absent from infection, since the article notes these break the method's assumptions
- When input is raw unnormalized counts — PCA scores and standard deviations must first come from data_preprocessing (log-normalization and highly variable gene detection)

## Inputs

- Control-group PCA score matrix (cell-by-PC, Z.pca.results$x) from data_preprocessing
- Control-group PCA standard deviations vector (Z.pca.results$sdev)
- using_rank logical flag (default TRUE) selecting rank-based vs value-based thresholds
- Optional thresholds: top.sdev (30), top.multimodal (3) in rank mode; th.sdev (1), th.multimodal (0.0125) in value mode

## Outputs

- Vector of selected PCs (res.step2) for downstream scDecouple decoupling
- Per-PC sdev ranks and Hartigan's dip statistics (e.g. saved as selected_pcs.rds)

## How to apply

Run PCA on the log-normalized, highly variable control-group matrix (Z.pca.results$x scores and Z.pca.results$sdev from scDecouple's data_preprocessing). For each PC, compute its standard deviation rank (explained spread) and its multimodality score via Hartigan's dip test using the diptest R package. In rank mode (using_rank=TRUE, the default), flag PCs as candidate infection-bias PCs when the sdev rank is at most top.sdev (default 30) and the dip statistic is at most top.multimodal (default 3); in value mode (using_rank=FALSE), apply thresholds th.sdev=1 and th.multimodal=0.0125 instead. The rationale is that PCs that are both multimodal and highly explained are the ones whose distributions across cell clusters can be distorted by differing infected proportions between perturbation and control groups, so only those go to the decoupling step while the remaining PCs' observed change is taken as the cellular response. Return the classified/selected PC vector (res.step2) together with per-PC sdev ranks and dip statistics for downstream use in the scDecouple step.

## Related tools

- **scDecouple** (Provides the pc_selection function implementing the sdev-rank and dip-test PC classification) — https://github.com/MengQiuchen/scDecouple
- **diptest** (R package used to compute Hartigan's dip test statistic for each PC's score distribution)
- **R** (Runtime environment in which scDecouple and diptest are executed)

## Examples

```
res.step2 <- pc_selection(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.sdev = res.step1$Z.pca.results$sdev, using_rank = TRUE)
```

## Evaluation signals

- The output file (e.g. selected_pcs.rds) exists and parses as an RDS object
- In rank mode every PC's classification is consistent with sdev rank <= 30 and dip statistic <= 3; in value mode with sdev <= 1 and dip <= 0.0125
- Every PC is assigned to exactly one of the two classes (infection-bias candidate vs observed-change-only)
- Selected PCs feed correctly into scDecouple as the select.pcs argument, which runs without error on the test_data demo dataset

## Limitations

- GMM-based assumptions may not hold in intricate systems or disease tissues such as tumors
- Assumption that the same cell clusters exist between NT and perturbation groups may fail when rare cell types are completely absent from infection
- Threshold defaults (top.sdev=30, top.multimodal=3, th.sdev=1, th.multimodal=0.0125) are dataset-dependent; users may need to customize them
- No version number is stated for the scDecouple R package

## Evidence

- [methods] ## step2: get PCs which may have high infection bias: "## step2: get PCs which may have high infection bias"
- [methods] res.step2 <- pc_selection(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.sdev = res.step1$Z.pca.results$sdev, using_rank = TRUE): "res.step2 <- pc_selection(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.sdev = res.step1$Z.pca.results$sdev, using_rank = TRUE)"
- [discussion] including PC plots, PC variance and multimodality visualizations, cellular response heatmaps and Gene Ontology enrichment plots: "including PC plots, PC variance and multimodality visualizations, cellular response heatmaps and Gene Ontology enrichment plots"
- [readme] top.multimodal (optional): When using_rank=TRUE, this parameter sets the threshold of dip statistics. The default is 3.: "top.multimodal (optional): When using_rank=TRUE, this parameter sets the threshold of dip statistics. The default is 3."
- [readme] th.multimodal (optional): When using_rank=FALSE, this parameter sets the threshold of dip statistics. The default is 0.0125.: "th.multimodal (optional): When using_rank=FALSE, this parameter sets the threshold of dip statistics. The default is 0.0125."
