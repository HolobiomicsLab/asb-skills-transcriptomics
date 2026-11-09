---
name: garnett-pretrained-classifier-annotation
description: Use when benchmarking cell-type annotation accuracy on PBMC scRNA-seq
  count matrices with known ground-truth labels and a pre-trained Garnett classifier
  is available for the tissue.
when_to_use_negative:
- No pre-trained Garnett classifier exists for the tissue/cell types in the dataset
  — marker-querying tools like EasyCellType or scCATCH are the appropriate choice
- The goal is unsupervised annotation without labeled reference data (Garnett is a
  supervised method requiring a trained classifier)
- Input is already a marker list from differential expression analysis rather than
  a count matrix — EasyCellType's GSEA/Fisher tests operate on that directly
edam_operation: http://edamontology.org/operation_2990
tools:
- name: Garnett
  role: Supervised annotation tool applied with a pre-trained PBMC classifier in cell-level
    and cluster-level extension modes
- name: Seurat
  role: Standard preprocessing pipeline (normalization, variable features, scaling,
    PCA, clustering) feeding into annotation
- name: SingleR
  role: Comparator supervised annotation method using the HumanPrimaryCellAtlasData
    reference
- name: scPred
  role: Comparator supervised method trained on 3500 sampled cells per dataset with
    prediction and no-rejection variants
- name: EasyCellType
  role: Marker-based method being benchmarked; runs GSEA and modified Fisher's exact
    test annotation
  repo: https://github.com/rx-li/EasyCellType
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:66fa174f80de61adf588a7c7498559c09f6dc6d0725452a47c8de6baed2e31cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - EasyCellType
  - Garnett
  - Seurat
  - SingleR
  - scPred
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# garnett-pretrained-classifier-annotation

## Summary

Annotates simulated or real PBMC scRNA-seq cells with the supervised tool Garnett using a pre-trained PBMC classifier in both cell-level and cluster-level extension modes, serving as a supervised benchmark against EasyCellType's marker-based GSEA and Fisher-exact annotation approaches.

## When to use

Use when benchmarking cell-type annotation accuracy on PBMC scRNA-seq count matrices with known ground-truth labels and a pre-trained Garnett classifier is available for the tissue.

## When NOT to use

- No pre-trained Garnett classifier exists for the tissue/cell types in the dataset — marker-querying tools like EasyCellType or scCATCH are the appropriate choice
- The goal is unsupervised annotation without labeled reference data (Garnett is a supervised method requiring a trained classifier)
- Input is already a marker list from differential expression analysis rather than a count matrix — EasyCellType's GSEA/Fisher tests operate on that directly

## Inputs

- Simulated or real PBMC scRNA-seq count matrix (e.g. 10X filtered gene-barcode matrix)
- Known ground-truth per-cell cell-type labels (for simulation-based benchmarks)
- Pre-trained Garnett PBMC cell-type classifier
- Seurat-processed object with clusters (for cluster-level Garnett extension)

## Outputs

- Per-cell Garnett cell-type predictions (cell-level and cluster-level variants)
- Per-cell annotation accuracy values per dataset size and tool variant
- Combined accuracy table (e.g. figure_s5_accuracy_table.csv)

## How to apply

First preprocess each count matrix with the standard Seurat pipeline (NormalizeData, FindVariableFeatures with 'vst' and nfeatures=2000, ScaleData, RunPCA, FindNeighbors/FindClusters). Annotate cells with Garnett using a pre-trained PBMC classifier in two modes: cell-level extension and cluster-level extension, where cluster-level mode applies the classifier to Seurat-identified clusters. Run the same procedure across simulated PBMC datasets of several cell counts so accuracy can be compared against other supervised tools (scPred, SingleR) and EasyCellType's hard and soft classifiers. Finally compute per-cell annotation accuracy against the simulated ground-truth labels for every tool/variant and dataset size, and write the combined results to a table such as figure_s5_accuracy_table.csv. The rationale is that supervised methods use labeled reference data, so they probe whether EasyCellType's marker-querying approach (GSEA and modified Fisher's exact test) is competitive when a matched classifier exists.

## Related tools

- **Garnett** (Supervised annotation tool applied with a pre-trained PBMC classifier in cell-level and cluster-level extension modes)
- **Seurat** (Standard preprocessing pipeline (normalization, variable features, scaling, PCA, clustering) feeding into annotation)
- **SingleR** (Comparator supervised annotation method using the HumanPrimaryCellAtlasData reference)
- **scPred** (Comparator supervised method trained on 3500 sampled cells per dataset with prediction and no-rejection variants)
- **EasyCellType** (Marker-based method being benchmarked; runs GSEA and modified Fisher's exact test annotation) — https://github.com/rx-li/EasyCellType

## Examples

```
annot <- SingleR(test = counts, ref = HumanPrimaryCellAtlasData(), labels = label.main)
```

## Evaluation signals

- Per-cell accuracy values are computed for every Garnett variant (cell-level and cluster-level) across all simulated dataset sizes and stored in figure_s5_accuracy_table.csv
- Combined accuracy table is internally consistent across datasets and tool variants (no missing tool/variant/dataset-size rows)
- Qualitative pattern of the reproduced accuracy table matches Supplementary Figure S5
- Garnett predictions can be checked against simulated ground-truth labels on a per-cell basis
- Preprocessing (Seurat pipeline) applied identically to all datasets before annotation so comparisons are fair

## Limitations

- Requires a matching pre-trained classifier; Garnett performance on the benchmark showed it generally did not outperform EasyCellType's soft classifiers on simulated PBMC data
- Depends on simulation ground truth — accuracy conclusions are relative to the simulated PBMC setting, not guaranteed for other tissues
- Software version numbers were not fully enumerated in the source article, limiting exact reproducibility

## Evidence

- [other] Annotate cells with Garnett using a pre-trained PBMC classifier in both cell-level and cluster-level extension modes.: [evidence span withheld — anchor status: not_found]
- [other] Compute per-cell annotation accuracy against the simulated ground-truth labels: [evidence span withheld — anchor status: not_found]
- [other] the supervised methods except Garnett and SingleR generally outperform EasyCellType's hard classifiers, while EasyCellType's soft classifiers are comparable to or better than the supervised methods.: [evidence span withheld — anchor status: not_found]
- [methods] We followed the standard workflow provided by `Seurat` package[@seurat] to process the PBMC data set.: "We followed the standard workflow provided by `Seurat` package[@seurat] to process the PBMC data set."
- [intro] Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
