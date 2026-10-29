---
name: balanced-accuracy-benchmark-computation
description: Use when evaluating an automated cell-type annotation method against
  curated expert labels on atlas-scale mouse or human tissue datasets, when balanced
  accuracy and label-similarity metrics must be reproduced from deposited code, or
  when benchmarking Census against annotators on droplet or FACS.
when_to_use_negative:
- Input is not a Seurat object containing a UMAP dimensionality reduction — census_main
  requires one
- Tissue falls outside the 24 Tabula Sapiens organs Census can classify (e.g. tissues
  not among Bladder, Blood, Bone_Marrow, ... Vasculature) and no custom model has
  been trained
- Goal is cancer-of-origin inference rather than normal-tissue annotation benchmarking
  — that requires the organ-specific cancer models with predict_cancer=T, a different
  workflow
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: Hierarchical gradient-boosted decision tree annotator; runs successive classification
    from the hierarchy root and applies the UMAP-based label-stabilizing algorithm
    to produce predictions scored by balanced accuracy and label similarity
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Builds and processes the Tabula Muris tissue datasets into Seurat objects
    with UMAP reductions, the required input format for census_main
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
  source_content_hash: sha256:0490abc53150dc9b9ef860a960f22c9093c18eb6a0b8bf364232bdfc99555e1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# balanced-accuracy-benchmark-computation

## Summary

Reproduce Census's cross-species benchmark on Tabula Muris mouse tissues by annotating droplet and plate-based (FACS) scRNA-seq datasets with the pretrained Census hierarchical model and computing mean balanced accuracy (0.8), label similarity (0.89), and run time (13 s) against expert cell-type labels.

## When to use

Use when evaluating an automated cell-type annotation method against curated expert labels on atlas-scale mouse or human tissue datasets, when balanced accuracy and label-similarity metrics must be reproduced from deposited code, or when benchmarking Census against state-of-the-art annotators on droplet or FACS data.

## When NOT to use

- Input is not a Seurat object containing a UMAP dimensionality reduction — census_main requires one
- Tissue falls outside the 24 Tabula Sapiens organs Census can classify (e.g. tissues not among Bladder, Blood, Bone_Marrow, ... Vasculature) and no custom model has been trained
- Goal is cancer-of-origin inference rather than normal-tissue annotation benchmarking — that requires the organ-specific cancer models with predict_cancer=T, a different workflow

## Inputs

- Tabula Muris mouse scRNA-seq tissue datasets (droplet and plate-based/FACS)
- Seurat v4 objects with a UMAP dimensionality reduction
- Curated Tabula Muris expert cell-type labels per tissue
- Deposited Census code (Zenodo record 7017103 or GitHub sjdlabgroup/Census)

## Outputs

- Per-tissue and mean balanced accuracy of Census predictions vs. expert labels
- Label similarity scores across mouse tissues
- Census annotation run time
- mouse_evaluation_results.csv with mean balanced accuracy (0.8), label similarity (0.89), and run time (13 s)

## How to apply

Build Seurat v4 objects from the Tabula Muris droplet and plate-based (FACS) tissue datasets, processing each with the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50) so a UMAP reduction is present, and load Seurat 4 before the Census library as required. Train the Census hierarchical gradient-boosted decision tree models on the mouse reference and run census_main with the appropriate organ so successive classification proceeds from the root node of the cell-type hierarchy until terminal classifications are reached. Apply the Census label-stabilizing algorithm, which uses clusters and prediction contours in UMAP space to propagate higher-confidence binary prediction outcomes. Compute per-tissue and mean balanced accuracy of the predictions against the curated Tabula Muris expert labels, compute label similarity across tissues, and record the annotation run time. Compile the mean balanced accuracy, label similarity, and run time into mouse_evaluation_results.csv and compare to the paper's reported values (0.8, 0.89, 13 s) within rerun tolerance.

## Related tools

- **Census** (Hierarchical gradient-boosted decision tree annotator; runs successive classification from the hierarchy root and applies the UMAP-based label-stabilizing algorithm to produce predictions scored by balanced accuracy and label similarity) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Builds and processes the Tabula Muris tissue datasets into Seurat objects with UMAP reductions, the required input format for census_main)
- **ggpubr** (Publication-ready plotting of benchmark comparison figures) — https://github.com/kassambara/ggpubr

## Examples

```
remotes::install_github('sjdlabgroup/Census', ref = 'main'); res <- census_main(obj, organ = 'Lung')
```

## Evaluation signals

- Mean balanced accuracy over all Tabula Muris tissues equals 0.8
- Mean label similarity across mouse tissues equals 0.89
- Recorded Census annotation run time is approximately 13 s across all tissues
- Per-tissue balanced accuracies match the benchmark of the 44 atlas-scale tissues reported in the paper within tolerance expected for a rerun of the deposited code
- Seurat 4 is loaded before the Census library and every input object has a UMAP reduction before census_main is called

## Limitations

- Census requires Seurat version 4 specifically and Seurat must be loaded prior to the Census library in R
- The pretrained model classifies only 175 cell-types from 24 organs; tissues outside these organs require training a custom model
- Benchmark results depend on the label-stabilizing step using clusters and prediction contours in UMAP space, so run times and predictions may vary slightly across reruns

## Evidence

- [abstract] Benchmark scope of the 44 atlas-scale tissues: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics"
- [abstract] Hierarchical GDT speed and accuracy rationale: "Census infers hierarchical cell-type relationships and uses gradient-boosted \decision trees that capitalize on nodal cell-type relationships to achieve high prediction speed and accuracy."
- [intro] Root-to-terminal classification procedure: "The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached"
- [readme] Seurat v4 loading requirement: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] Label-stabilizing algorithm using UMAP contours: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
