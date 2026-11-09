---
name: benchmark-runtime-measurement
description: Use when evaluating an automated cell-type annotation method against
  curated expert labels on atlas-scale tissues and you need reproducible metrics (balanced
  accuracy, label similarity) plus wall-clock run time for comparison against published
  benchmarks.
when_to_use_negative:
- The goal is annotation of a new dataset rather than benchmarking against curated
  expert labels — use census_main directly without computing accuracy/label-similarity
  metrics
- The input is not a Seurat object with a UMAP reduction (Census annotation will fail;
  the standard Seurat pipeline must be run first)
- Tissues are outside the 24 organs covered by the pretrained Tabula Sapiens model
  (e.g. tissues with no matching organ parameter) — train a custom model instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: Pretrained hierarchical gradient-boosted decision tree annotation whose predictions,
    label similarity, and run time are benchmarked against expert labels
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Builds and processes the Tabula Muris Seurat v4 objects (normalization, PCA,
    UMAP) that Census requires as input; must be loaded before the Census library
  repo: https://github.com/sjdlabgroup/Census
- name: ggpubr
  role: Publication-ready plotting of benchmark comparison figures
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:7d5397f739293ff71034210af53a102971cc4fcfe690d41ebce2cea8b66c2605
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# benchmark-runtime-measurement

## Summary

Reproduce the paper's cross-species benchmark of the Census cell-type annotation method by timing full annotation runs and computing balanced accuracy and label similarity against expert labels, e.g. the Tabula Muris mouse tissue evaluation (balanced accuracy 0.8, label similarity 0.89, run time 13 s).

## When to use

Use when evaluating an automated cell-type annotation method against curated expert labels on atlas-scale tissues and you need reproducible metrics (balanced accuracy, label similarity) plus wall-clock run time for comparison against published benchmarks.

## When NOT to use

- The goal is annotation of a new dataset rather than benchmarking against curated expert labels — use census_main directly without computing accuracy/label-similarity metrics
- The input is not a Seurat object with a UMAP reduction (Census annotation will fail; the standard Seurat pipeline must be run first)
- Tissues are outside the 24 organs covered by the pretrained Tabula Sapiens model (e.g. tissues with no matching organ parameter) — train a custom model instead

## Inputs

- Droplet and plate-based (FACS) Tabula Muris mouse tissue scRNA-seq count matrices
- Curated Tabula Muris expert cell-type label annotations per tissue
- Seurat v4 objects with a UMAP dimensionality reduction
- Deposited Census code/models from Zenodo (https://zenodo.org/records/7017103) or GitHub (sjdlabgroup/Census)

## Outputs

- Per-tissue and mean balanced accuracy of Census predictions vs. expert labels
- Label similarity scores between Census annotations and reference labels across mouse tissues
- Recorded run time of the Census annotation (benchmark: 13 s across all Tabula Muris tissues)
- Compiled metrics table, e.g. mouse_evaluation_results.csv

## How to apply

Load Seurat 4 before the Census library, as Census requires Seurat version 4. Build Seurat v4 objects from the droplet and plate-based (FACS) Tabula Muris mouse scRNA-seq tissue datasets, processing each with the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP) to produce a UMAP reduction that Census needs. Run the pretrained hierarchical gradient-boosted decision tree models via census_main, which classifies cells successively from the root node of the cell-type hierarchy until terminal classifications are reached, then applies the label-stabilizing algorithm that uses clusters and prediction contours in UMAP space to propagate higher-confidence binary prediction outcomes. Compute per-tissue and mean balanced accuracy of Census predictions against the curated Tabula Muris expert cell-type labels, compute label similarity between predicted and reference labels across tissues, and record the run time of the annotation call. Compile the mean balanced accuracy, label similarity, and run time (expected values 0.8, 0.89, and 13 s across all mouse tissues) into a results table such as mouse_evaluation_results.csv, and verify reproduced values match the paper's 44-atlas-tissue benchmark within the tolerance expected for a rerun of the deposited code.

## Related tools

- **Census** (Pretrained hierarchical gradient-boosted decision tree annotation whose predictions, label similarity, and run time are benchmarked against expert labels) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Builds and processes the Tabula Muris Seurat v4 objects (normalization, PCA, UMAP) that Census requires as input; must be loaded before the Census library) — https://github.com/sjdlabgroup/Census
- **ggpubr** (Publication-ready plotting of benchmark comparison figures) — https://github.com/kassambara/ggpubr

## Examples

```
library(Seurat); library(Census); obj = obj %>% NormalizeData() %>% ScaleData() %>% FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50); res = census_main(obj, organ = 'Lung')
```

## Evaluation signals

- Reproduced mean balanced accuracy matches the reported 0.8 on Tabula Muris droplet and plate-based tissues
- Reproduced mean label similarity matches the reported 0.89 across the mouse tissues
- Recorded run time is on the order of the reported 13 s for the mouse evaluation
- Balanced accuracy and label similarity are computed both per-tissue and as means across all tissues
- Per-tissue Census predictions agree with the curated Tabula Muris expert cell-type labels at the expected rate for a rerun of the deposited code

## Limitations

- Run-time and accuracy values are expected to match only within the tolerance of a rerun of the deposited code, not exactly
- Census requires Seurat version 4 specifically and it must be loaded prior to the Census library
- The pretrained main model identifies 175 cell-types from 24 organs; tissues outside this scope require training a custom model, which changes the benchmark interpretation

## Evidence

- [abstract] Benchmark on Tabula Muris with Census metrics: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics and naturally predicts the"
- [intro] Hierarchical classification mechanism: "The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached"
- [intro] Label-stabilizing step definition: "using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
- [readme] Seurat 4 version requirement: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] Standard Seurat processing before Census: "obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>%
  FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50)"
