---
name: hierarchical-cell-type-annotation-scoring
description: Use when a gene-by-cell scRNA-seq count matrix from a tumour biopsy must
  be annotated into cancer, immune, and stromal cell types/subtypes across a pan-cancer
  hierarchy with intermediate per-class confidence scores.
when_to_use_negative:
- Input samples represent normal tissue without cancer cells — use the normal_tissue
  parameter, since the default scATOMIC model assumes presence of at least some cancer
  cells
- Analysis goal is annotating whole clusters rather than individual cells from a heterogeneous
  tumour microenvironment, where manual cluster-based annotation is used
- The count matrix is dense/non-sparse and has not been quality filtered (mitochondrial
  percentage and feature count thresholds not applied)
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scATOMIC
  role: Provides run_scATOMIC(), create_summary_matrix(), pre-trained hierarchical
    random forest models, and the demo_lung_data dataset for hierarchical annotation
    scoring
  repo: https://github.com/abelson-lab/scATOMIC
- name: cutoff.scATOMIC
  role: Fork of the cutoff package implementing the finite mixture model (EM algorithm)
    used to derive bimodal-distribution cut-off values for intermediate group scores
  repo: https://github.com/inofechm/cutoff.scATOMIC
- name: Rmagic
  role: Used inside scATOMIC to impute values (MAGIC) supporting the cancer signature
    scoring module
  repo: https://cran.r-project.org/package=Rmagic
- name: R
  role: Runtime environment (v4.3.2 tested) hosting scATOMIC, agrmt, cutoff.scATOMIC
    and dependency packages
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:e0c78b92e3b8a3d120f02d7f047286f4ee2a8d4a887d4db19fb42343beca3ff0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - R
  - Rmagic
  - agrmt
  - cutoff.scATOMIC
  - scATOMIC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# hierarchical-cell-type-annotation-scoring

## Summary

Implements scATOMIC's hierarchical, model-based scoring scheme for annotating malignant and non-malignant cells in pan-cancer single-cell RNA-seq data, where cells traverse a hierarchy of pre-trained random forest models and per-layer class scores determine terminal labels or fallback to parental classifications.

## When to use

Use when a gene-by-cell scRNA-seq count matrix from a tumour biopsy must be annotated into cancer, immune, and stromal cell types/subtypes across a pan-cancer hierarchy with intermediate per-class confidence scores.

## When NOT to use

- Input samples represent normal tissue without cancer cells — use the normal_tissue parameter, since the default scATOMIC model assumes presence of at least some cancer cells
- Analysis goal is annotating whole clusters rather than individual cells from a heterogeneous tumour microenvironment, where manual cluster-based annotation is used
- The count matrix is dense/non-sparse and has not been quality filtered (mitochondrial percentage and feature count thresholds not applied)

## Inputs

- Sparse gene-by-cell count matrix from a single patient/biopsy scRNA-seq sample (e.g. the packaged demo_lung_data lung cancer dataset)
- Optionally a Seurat object from which counts are extracted via seurat_object@assays$RNA@counts
- Quality-filtered cells (>25% mitochondrial reads removed, <500 unique expressed features removed)

## Outputs

- Detailed annotation object with hierarchical per-layer class scores from run_scATOMIC()
- Summary matrix of intermediate per-layer results from create_summary_matrix()
- Per-parent-node cutoff_table.csv containing class labels and cut-off values in [0, 0.7]
- Cell-level labels where sub-threshold cells are relabelled unclassified at the corresponding hierarchy layer

## How to apply

Convert the count matrix to a sparse matrix and run run_scATOMIC(), which generates per-layer prediction score lists as cells descend the hierarchical scheme of elimination models. Summarize intermediate per-layer per-class results with create_summary_matrix(), which mirrors the median_score_class_layer_* columns. To reproduce confidence cut-offs for intermediate group scores (IGS), extract each parent node's class score distributions via score_class()/add_class_per_layer(), assess modality with the 'ajus' criterion from the agrmt package, and compute cut-offs with automatic_threshold(): mean minus 3 SD for unimodal distributions (unimodal_nsd = 3) and second-component mean minus 2 SD for bimodal distributions (bimodal_nsd = 2), capped at 0.7; all layers can be batched with get_auto_threshold(predictions, mc.cores, unimodal_nsd = 3, bimodal_nsd = 2). Apply cut-offs with add_unclassified_automatic_threshold() so cells scoring below their class cut-off are relabelled unclassified at that layer, and write the per-parent-node table to cutoff_table.csv. The rationale is that hierarchical models with elimination processes reduce transcriptomic complexity of the TME multi-cellular system, and MAGIC-imputed scores (via Rmagic) support the cancer signature scoring module.

## Related tools

- **scATOMIC** (Provides run_scATOMIC(), create_summary_matrix(), pre-trained hierarchical random forest models, and the demo_lung_data dataset for hierarchical annotation scoring) — https://github.com/abelson-lab/scATOMIC
- **cutoff.scATOMIC** (Fork of the cutoff package implementing the finite mixture model (EM algorithm) used to derive bimodal-distribution cut-off values for intermediate group scores) — https://github.com/inofechm/cutoff.scATOMIC
- **Rmagic** (Used inside scATOMIC to impute values (MAGIC) supporting the cancer signature scoring module) — https://cran.r-project.org/package=Rmagic
- **R** (Runtime environment (v4.3.2 tested) hosting scATOMIC, agrmt, cutoff.scATOMIC and dependency packages)

## Examples

```
devtools::install_github("inofechm/cutoff.scATOMIC", force = T); cutoffs <- get_auto_threshold(predictions, mc.cores = 8, unimodal_nsd = 3, bimodal_nsd = 2)
```

## Evaluation signals

- cutoff_table.csv exists and contains one row per parent-node class with a populated cut-off field
- Every derived cut-off value lies within [0, 0.7] (the maximum cap)
- Applying the cut-offs converts all sub-threshold cells to an unclassified label at the corresponding hierarchy layer
- Unimodal classes yield cut-offs at mean − 3 SD and bimodal classes at second-component mean − 2 SD, consistent with the ajus modality assessment
- create_summary_matrix() returns median_score_class_layer_* columns matching the per-layer scores produced by run_scATOMIC()

## Limitations

- Inter-patient cancer cell-specific effects can interfere with scATOMIC's cancer signature scoring module and confidence in classifications; scATOMIC should only be run on individual patient/biopsy samples
- Datasets with more than 15,000 cells require a machine with greater RAM than a standard 16GB laptop; Windows lacks native parallelism and runs slower
- The default model assumes the presence of at least some cancer cells and should not be used in normal tissue applications without the normal_tissue parameter
- In samples with near-diploid cancer cells or few normal control cells, and where discordancy exists with CNV inference, scATOMIC-based classification may fall short and joint use with CNV inference is recommended

## Evidence

- [intro] Hierarchical elimination models reduce complexity: "We devise a structured scheme that uses hierarchically organized models and elimination processes, reducing the transcriptomic complexity of the TME multi-cellular system to improve cell"
- [methods] run_scATOMIC generates the annotation object: "To run scATOMIC we use the run_scATOMIC() function to generate a detailed annotation object."
- [methods] Summary of intermediate per-layer results: "After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix()."
- [methods] Demo lung cancer dataset packaged with scATOMIC: "We included a demo dataset of a primary lung cancer sample within our package under the object name ‘demo_lung_data’."
- [readme] Fork of cutoff package used for bimodal cut-offs: [evidence span withheld — anchor status: external_doc]
- [readme] Run on individual patient/biopsy samples only: "Inter-patient cancer cell specific effects can interfere with scATOMICs cancer signature scoring module and confidence in classifications."
