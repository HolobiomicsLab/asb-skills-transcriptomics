---
name: annotation-label-similarity-assessment
description: Use when an automated annotation output (Census celltype predictions)
  must be quantitatively compared to curated expert cell-type labels to reproduce
  benchmark metrics like balanced accuracy 0.8 and label similarity 0.89.
when_to_use_negative:
- The dataset has no curated expert cell-type labels, so neither balanced accuracy
  nor label similarity can be computed
- The input is raw count data not yet processed into a Seurat object with a UMAP reduction,
  which census_main requires
- The goal is annotation itself (e.g., cancer cell-of-origin prediction with predict_cancer=TRUE)
  rather than benchmarking annotation quality against reference labels
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Census
  role: Produces the automated hierarchical cell-type annotations whose similarity
    to expert labels is being assessed
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Builds and processes the Tabula Muris tissue Seurat objects (NormalizeData,
    RunPCA, RunUMAP) that census_main consumes; must be loaded before the Census library
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:15c196e5db1d157cae4a0d38c40106fa1859a4e8f643a9a5b33e5b2866b85719
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# annotation-label-similarity-assessment

## Summary

Evaluates automated scRNA-seq cell-type annotations (e.g., Census predictions on mouse/human tissues) against curated expert labels by computing balanced accuracy, label similarity, and annotation run time, benchmarking a 44 atlas-scale tissue evaluation such as Tabula Muris.

## When to use

Use when an automated annotation output (Census celltype predictions) must be quantitatively compared to curated expert cell-type labels to reproduce benchmark metrics like balanced accuracy 0.8 and label similarity 0.89.

## When NOT to use

- The dataset has no curated expert cell-type labels, so neither balanced accuracy nor label similarity can be computed
- The input is raw count data not yet processed into a Seurat object with a UMAP reduction, which census_main requires
- The goal is annotation itself (e.g., cancer cell-of-origin prediction with predict_cancer=TRUE) rather than benchmarking annotation quality against reference labels

## Inputs

- Seurat v4 object(s) of Tabula Muris mouse tissue scRNA-seq data (droplet and plate/FACS) with UMAP dimensionality reduction
- Curated expert/reference cell-type labels per tissue (e.g., Tabula Muris annotations)
- Census pretrained Tabula Sapiens model (175 cell-types from 24 organs)

## Outputs

- mouse_evaluation_results.csv with mean balanced accuracy, label similarity, and run time
- Per-tissue balanced accuracy values against expert labels
- Per-cell Census celltype predictions (res$pred) used for the comparison

## How to apply

Build Seurat v4 objects from the scRNA-seq samples (droplet and plate/FACS for Tabula Muris mouse tissues), loading Seurat 4 before the Census library as required, then run the pretrained census_main hierarchical gradient-boosted classification from the root node until terminal classifications are reached, including the UMAP-space label-stabilizing step. Compare the resulting per-cell celltype predictions against the curated expert labels to compute per-tissue and mean balanced accuracy across all tissues. Compute label similarity between the Census annotations and the reference labels for the same tissue set, and record the wall-clock run time of the annotation. Compile mean balanced accuracy (expected ~0.8), label similarity (~0.89), and run time (~13 s) into mouse_evaluation_results.csv and verify they match the paper's benchmark within the tolerance expected for a rerun of the deposited code.

## Related tools

- **Census** (Produces the automated hierarchical cell-type annotations whose similarity to expert labels is being assessed) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Builds and processes the Tabula Muris tissue Seurat objects (NormalizeData, RunPCA, RunUMAP) that census_main consumes; must be loaded before the Census library)

## Examples

```
library(Seurat); library(Census); res <- census_main(obj, organ = 'Lung'); compare res$pred$celltype against expert labels for balanced accuracy and label similarity
```

## Evaluation signals

- Mean balanced accuracy over all Tabula Muris mouse tissues reproduces ~0.8
- Label similarity between Census annotations and expert labels reproduces ~0.89
- Recorded Census annotation run time is ~13 s across all tissues
- Per-tissue balanced accuracy is computed for every droplet and plate-based tissue, not just the mean
- Values compiled into mouse_evaluation_results.csv match the paper's 44-tissue benchmark within rerun tolerance

## Limitations

- Census requires Seurat version 4 and must be loaded after Seurat v4 in R; other Seurat versions may not run
- The pretrained model classifies only the 175 Tabula Sapiens cell-types from 24 organs, so expert labels outside this hierarchy may not be comparable
- Evaluation depends on access to curated expert cell-type labels for the tissue; without them, similarity and balanced accuracy cannot be computed

## Evidence

- [abstract] abstract: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics"
- [readme] readme: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
