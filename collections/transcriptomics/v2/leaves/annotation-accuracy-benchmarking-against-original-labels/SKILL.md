---
name: annotation-accuracy-benchmarking-against-original-labels
description: Use when validating an automated cell-annotation model on held-out, independently
  published cancer scRNA-seq datasets that carry original labeled cell information
  (e.g., MCC 11,024 cells, CRC 11,717 myeloid cells, LC 45,555 T cells) so predicted
  labels can be scored against expert ground truth.
when_to_use_negative:
- Targeting tissue-specific cell types outside scWizard's TME scope (e.g., alveolar
  cells in lung cancers) — the article warns these may not be identified and fall
  in the Unknown cluster
- Evaluating on data included in model training rather than independent datasets —
  accuracy would not be comparable to the reported held-out figures
- Datasets lacking original expert cell annotations, since the consistent/inconsistent/unknown
  labeling cannot be computed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scWizard
  role: Provides the two-step hierarchical DNN annotation model (major cell types
    then Seurat-subclustered subtypes) being benchmarked; installed from GitHub with
    its Python environment and pretrained trainset
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Used in the second annotation step to define subclusters of each major cell
    type, whose majority-vote label is compared against the original study's subtype
    annotations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:4b627d192106d0178b748e629bd48005d4c73f0484ada4bd82cbbfad0f6eae18
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/scWizard
  repo_url: https://github.com/Dulab2020/scWizard
  tools:
  - Seurat
  - scWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# annotation-accuracy-benchmarking-against-original-labels

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks an automated single-cell annotation method against the original study's expert cell labels on independent cancer scRNA-seq datasets, scoring per-cluster accuracy as the fraction of cells whose predicted label is 'consistent' with the original annotation. Reproduces scWizard's reported accuracies (0.98 major cell types, 0.85 myeloid subtypes, 0.79 T/NK subtypes).

## When to use

Use when validating an automated cell-annotation model on held-out, independently published cancer scRNA-seq datasets that carry original labeled cell information (e.g., MCC 11,024 cells, CRC 11,717 myeloid cells, LC 45,555 T cells) so predicted labels can be scored against expert ground truth.

## When NOT to use

- Targeting tissue-specific cell types outside scWizard's TME scope (e.g., alveolar cells in lung cancers) — the article warns these may not be identified and fall in the Unknown cluster
- Evaluating on data included in model training rather than independent datasets — accuracy would not be comparable to the reported held-out figures
- Datasets lacking original expert cell annotations, since the consistent/inconsistent/unknown labeling cannot be computed

## Inputs

- Independent labeled cancer scRNA-seq datasets (e.g., MCC dataset, 11,024 cells with labeled cell information; CRC myeloid dataset, 11,717 cells; LC T-cell dataset, 45,555 cells) in Seurat object or counts-matrix formats (csv, rds, h5, or 10X folder)
- Pretrained scWizard trainset in H5 format (rows = genes, columns = cells), downloaded from the Baidu link and placed in app/www/python
- Original study cell-type annotations used as ground truth for consistency scoring

## Outputs

- Per-cell consistency labels ('consistent', 'inconsistent', or 'unknown') for every annotated cell
- Per-cluster annotation accuracy values (consistent cells / total cells per cluster)
- annotation_accuracy.csv assembling per-dataset accuracies for the independent and comparison datasets

## How to apply

Install scWizard from GitHub, run its 'install Python' step, and download the pretrained trainset (113,976 cells, 7 major types, 47 subtypes) from the Baidu link into the app/www/python folder. Run the two-step model: first-step major-cell-type annotation (PCA on combined training+prediction data, DNN with ReLU and softmax activation, max-likelihood assignment) on the independent MCC dataset, then second-step subtype annotation (Seurat subclustering of each major cell type, DNN prediction with majority-vote subcluster rule) on the CRC myeloid and LC T-cell datasets. Label every predicted cell as 'consistent', 'inconsistent', or 'unknown' against the original study's annotations, and compute per-cluster accuracy as consistent cells divided by total cells in each cluster. Repeat on the comparison datasets (LYM, T/NK_BRCA, T/NK_KIRC, Mye_PAAD) and assemble per-dataset accuracies into annotation_accuracy.csv, accepting reproduction within a few percent of the reported 0.98, 0.85, 0.79, >0.66, 0.71, 0.66, and 0.64 values.

## Related tools

- **scWizard** (Provides the two-step hierarchical DNN annotation model (major cell types then Seurat-subclustered subtypes) being benchmarked; installed from GitHub with its Python environment and pretrained trainset) — https://github.com/Dulab2020/scWizard
- **Seurat** (Used in the second annotation step to define subclusters of each major cell type, whose majority-vote label is compared against the original study's subtype annotations)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Reproduced MCC major-cell-type accuracy matches the reported 0.98 within a few percent
- CRC myeloid subtype accuracy matches 0.85 and LC T/NK subtype accuracy matches 0.79 within a few percent
- Comparison dataset accuracies reach >0.66 (LYM), 0.71 (T/NK_BRCA), 0.66 (T/NK_KIRC), and 0.64 (Mye_PAAD)
- Every predicted cell carries exactly one consistency label ('consistent', 'inconsistent', or 'unknown') relative to the original study's annotation
- Per-cluster accuracy equals consistent cells divided by total cells in each cell cluster, and all values assemble into annotation_accuracy.csv

## Limitations

- Tissue-specific cell types may not be identified — the article notes alveolar cells in lung cancers may be named in the Unknown cluster
- The pretrained reference covers only the 7 major cell types and 47 subtypes of the tumor microenvironment across 13 cancer types
- No software version number is stated for scWizard in the article; the trainset must be fetched from a Baidu link and Python installed before annotation

## Evidence

- [methods] MCC independent dataset size and labeled ground truth: "The independent MCC datasets including 11,024 cells with the labeled cell information were used to evaluate the major cell annotation capability of scWizard."
- [abstract] Reported accuracy targets for reproduction: "with an accuracy of 0.98 in annotating major cell types, 0.85 in annotating myeloid cell subtypes and 0.79 in annotating T/NK cell subtypes"
- [methods] DNN classifier used in the annotation step being benchmarked: "The deep neural network structures and the learning algorithms were implemented using ReLU (Rectified Linear Unit) and softmax activation function."
- [readme] Trainset placement required before running annotation: "you need to download the trainset and copy it to the "app/www/python" folder under the scwizard installation directory."
- [discussion] Tissue-specific cell types may fall into Unknown cluster: "For tissue specific cell types, scWizard may not be very applicable, such as the alveolar cells in lung cancers may not be identified and named in Unknown cluster."
