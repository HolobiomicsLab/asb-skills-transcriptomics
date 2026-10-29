---
name: scrna-seq-cluster-downsampling
description: Use when you need to determine the minimum number of cells per cluster
  (here >=20) at which automated cell-type annotations remain statistically indistinguishable
  from full-dataset annotations, or when auditing annotation robustness on sparse
  clusters of a Seurat object with a UMAP reduction.
when_to_use_negative:
- Every cluster already has >=500 cells, so down-sampling cannot reveal any annotation
  sensitivity that was not already tested
- The Seurat object lacks a UMAP reduction or cluster annotations, so the Census label-stabilizing
  step and cluster-based down-sampling cannot be performed
- The goal is cell-of-origin inference on a full cancer dataset rather than quantifying
  annotation robustness to sample size
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: Pretrained hierarchical gradient-boosted decision tree model used to annotate
    each down-sampled, re-embedded dataset, including the UMAP-based label-stabilizing
    step
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat
  role: Loads and processes the Peng et al. 2019 dataset (must be v4, loaded before
    Census), performs per-cluster down-sampling and re-embeds each subsample in UMAP
    space
- name: ggpubr
  role: Runs the Kruskal-Wallis non-parametric test over F1 score distributions across
    down-sampling levels and reports the P value
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:c978358c2cfb49bb4aec067324a5be99da736d380c334b802c5aadc3740267e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat
  - ggpubr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# scrna-seq-cluster-downsampling

## Summary

A sensitivity-analysis skill that quantifies how many cells per UMAP cluster are needed for reliable Census cell-type annotations, by iteratively down-sampling the Peng et al. 2019 pancreatic cancer scRNA-seq dataset from 5 to 500 cells per cluster, re-annotating with the pretrained Census model, and testing F1-score differences with a Kruskal-Wallis test.

## When to use

Use when you need to determine the minimum number of cells per cluster (here >=20) at which automated cell-type annotations remain statistically indistinguishable from full-dataset annotations, or when auditing annotation robustness on sparse clusters of a Seurat object with a UMAP reduction.

## When NOT to use

- Every cluster already has >=500 cells, so down-sampling cannot reveal any annotation sensitivity that was not already tested
- The Seurat object lacks a UMAP reduction or cluster annotations, so the Census label-stabilizing step and cluster-based down-sampling cannot be performed
- The goal is cell-of-origin inference on a full cancer dataset rather than quantifying annotation robustness to sample size

## Inputs

- Peng et al. 2019 pancreatic cancer scRNA-seq dataset (demo file peng_sub.RDS from the Census Zenodo/GitHub release) with pre-computed UMAP cluster annotations
- Seurat object containing a UMAP dimensionality reduction
- Down-sampling granularity levels: 5, 10, 20, 50, 100, 500 cells per UMAP cluster

## Outputs

- downsampled_f1_scores.csv containing per-cell-type F1 scores for each down-sampling level and the Kruskal-Wallis test statistics
- Per-level F1 score distributions comparing Census annotations to original cluster-level labels

## How to apply

Load Seurat v4 before the Census library (a hard requirement of Census), then load the Peng et al. 2019 pancreatic cancer dataset with existing UMAP cluster annotations. Iteratively down-sample 5, 10, 20, 50, 100, and 500 cells per UMAP cluster, re-embed each subsample in UMAP space with the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP), and annotate each with census_main on the pretrained Tabula Sapiens model (organ='Pancreas', predict_cancer=TRUE), which applies the label-stabilizing algorithm that propagates predictions using clusters and prediction contours in UMAP space. Compute per-cell-type F1 scores against the original cluster-level labels for each down-sampling level, then run a Kruskal-Wallis non-parametric test over the F1 distributions with ggpubr, expecting P = 0.98 across levels with >=20 cells per cell-type and a performance drop below 20 cells. Write per-level F1 scores and test statistics to downsampled_f1_scores.csv.

## Related tools

- **Census** (Pretrained hierarchical gradient-boosted decision tree model used to annotate each down-sampled, re-embedded dataset, including the UMAP-based label-stabilizing step) — https://github.com/sjdlabgroup/Census
- **Seurat** (Loads and processes the Peng et al. 2019 dataset (must be v4, loaded before Census), performs per-cluster down-sampling and re-embeds each subsample in UMAP space)
- **ggpubr** (Runs the Kruskal-Wallis non-parametric test over F1 score distributions across down-sampling levels and reports the P value) — https://github.com/kassambara/ggpubr

## Examples

```
res = census_main(obj, organ = 'Pancreas', predict_cancer = T)
```

## Evaluation signals

- Kruskal-Wallis test over F1 distributions for >=20 cells per cell-type levels returns P approximately 0.98 (no significant difference)
- F1 scores visibly decrease for down-sampling levels below 20 cells per cell-type (5 and 10 cells), reproducing the reported threshold
- All six granularity levels (5, 10, 20, 50, 100, 500 cells per cluster) produce a re-embedded UMAP and a full census_main annotation before scoring
- Seurat version 4 was loaded before library(Census); the pipeline errors or misbehaves otherwise
- downsampled_f1_scores.csv contains both per-level per-cell-type F1 scores and the Kruskal-Wallis test statistics

## Limitations

- Census requires Seurat version 4 and fails to run correctly unless Seurat v4 is loaded before the Census library
- Below 20 cells per cell-type, annotation performance degrades, so conclusions at 5–10 cells per cluster are unreliable
- The pretrained model covers 175 cell-types from 24 organs; tissues outside the supported organ list require training a custom model
- The cancer model currently supports only Kidney, Large_Intestine, Liver, Lung, Mammary, and Pancreas

## Evidence

- [readme] Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R.: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [intro] New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [abstract] Census infers hierarchical cell-type relationships and uses gradient-boosted decision trees that capitalize on nodal cell-type relationships to achieve high prediction speed and accuracy.: [evidence span withheld — anchor status: not_found]
- [abstract] Census is pretrained on the Tabula Sapiens to classify 175 cell-types from 24 organs; however, users can seamlessly train their own models for customized applications.: "Census is pretrained on the Tabula Sapiens to classify 175 cell-types from 24 organs; however, users can seamlessly train their own models for customized applications."
