---
name: single-cell-rna-seq-cell-type-classification
description: Use when gene-by-cell count matrices from tumour biopsies need cell-type
  labels across cancer, immune, and stromal populations, especially when CNV inference
  alone is insufficient or the tissue of origin is unknown.
when_to_use_negative:
- Input represents normal (non-cancer) tissue — the default model assumes at least
  some cancer cells are present; use the normal_tissue parameter in create_summary_matrix
  or another approach
- The cancer type is already known — the known_cancer_type parameter in create_summary_matrix
  is the appropriate path rather than open terminal classification
- Classifying blood-only or single specialized-tissue datasets where existing annotators
  suffice, or samples where cell quality thresholds (>25% mito, <500 features) would
  discard most cells
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scATOMIC
  role: core hierarchical random-forest classifier (run_scATOMIC, create_summary_matrix,
    scATOMICTree, get_new_scATOMIC_layer)
  repo: github.com/abelson-lab/scATOMIC
- name: cutoff.scATOMIC
  role: EM mixture-model package supplying confidence cutoffs for classification layers
  repo: github.com/inofechm/cutoff.scATOMIC
- name: Rmagic
  role: MAGIC imputation of the count matrix before signature scoring (requires Python
    magic-impute via reticulate)
- name: randomForest
  role: underlying random-forest models at each hierarchy layer
- name: reticulate
  role: R-Python bridge used to install and check the Python MAGIC backend (pymagic_is_available)
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:3389d10554048006d7f21de1d54f336d39510a0d7a04961b90563f22e70d24e3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rmagic
  - caret
  - cutoff.scATOMIC
  - devtools
  - dlm
  - randomForest
  - reticulate
  - scATOMIC
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# single-cell-rna-seq-cell-type-classification

## Summary

Automated classification of malignant and non-malignant cells in single-cell RNA-seq tumour microenvironment data using scATOMIC, a hierarchical random-forest classifier trained on >300,000 cells spanning 19 pan-cancer TME cell types, with MAGIC-imputed signatures and per-layer confidence cutoffs.

## When to use

Use when gene-by-cell count matrices from tumour biopsies need cell-type labels across cancer, immune, and stromal populations, especially when CNV inference alone is insufficient or the tissue of origin is unknown.

## When NOT to use

- Input represents normal (non-cancer) tissue — the default model assumes at least some cancer cells are present; use the normal_tissue parameter in create_summary_matrix or another approach
- The cancer type is already known — the known_cancer_type parameter in create_summary_matrix is the appropriate path rather than open terminal classification
- Classifying blood-only or single specialized-tissue datasets where existing annotators suffice, or samples where cell quality thresholds (>25% mito, <500 features) would discard most cells

## Inputs

- gene-by-cell raw count matrix (sparse Matrix, e.g. from Seurat object @assays$RNA@counts)
- optional ground-truth per-cell labels for F1 evaluation
- optional CNV-inference results (CopyKAT) for CNV-corrected mode

## Outputs

- detailed hierarchical annotation object from run_scATOMIC() with layer-wise per-cell predictions
- final per-cell classification from create_summary_matrix() (scATOMIC_pred labels)
- hierarchy visualization via scATOMICTree()

## How to apply

Run scATOMIC on gene-by-cell sparse matrices (convert via as(as.matrix(input_matrix), "sparseMatrix")), first filtering low-quality cells (>25% mitochondrial reads or <500 expressed features). Apply run_scATOMIC() independently per patient/biopsy sample — inter-patient effects interfere with the cancer signature scoring — with Rmagic/MAGIC imputation enabled (default TRUE) to obtain layer-wise predictions through the pan-cancer hierarchy. Aggregate intermediate results with create_summary_matrix(); for single-tumour samples set use_CNVs = FALSE, modify_results = TRUE to enforce a single cancer type, and min_prop = 0.5. Confidence cutoffs are drawn from cutoff.scATOMIC, a fork of the cutoff package that fits a finite mixture model via Expectation-Maximization to bimodal prediction scores. Correctness is judged by per-cell-type precision/recall/F1 against ground-truth labels (fivefold cross-validation median F1 0.90–0.99; external validation median F1 ≈0.99).

## Related tools

- **scATOMIC** (core hierarchical random-forest classifier (run_scATOMIC, create_summary_matrix, scATOMICTree, get_new_scATOMIC_layer)) — github.com/abelson-lab/scATOMIC
- **cutoff.scATOMIC** (EM mixture-model package supplying confidence cutoffs for classification layers) — github.com/inofechm/cutoff.scATOMIC
- **Rmagic** (MAGIC imputation of the count matrix before signature scoring (requires Python magic-impute via reticulate))
- **randomForest** (underlying random-forest models at each hierarchy layer)
- **reticulate** (R-Python bridge used to install and check the Python MAGIC backend (pymagic_is_available))

## Examples

```
results <- run_scATOMIC(raw_counts, impute = TRUE, models = scATOMIC.models); summary_mat <- create_summary_matrix(results, use_CNVs = F, modify_results = TRUE, min_prop = 0.5)
```

## Evaluation signals

- In fivefold cross-validation on the training reference, median per-cell-type F1 should fall in the reported 0.90–0.99 range (Fig. 2a)
- On training-independent external validation of 225 primary biopsies, aggregated per-sample per-cell-type median F1 should reproduce ≈0.99
- Metastatic origin majority-vote calls should match ~52/62 (83.9%) correct predictions across breast, kidney, lung, ovarian, and skin cancers
- Run scATOMIC on each biopsy independently and verify predictions exist at every hierarchy layer (non-terminal abstentions are expected for some samples)
- Confirm the Python MAGIC backend loads: Rmagic::pymagic_is_available() returns TRUE before running with impute = TRUE

## Limitations

- Inter-patient effects interfere with the cancer signature scoring module — do not pool samples; run on individual patient/biopsy samples only
- Bladder, brain, breast, colorectal, kidney, liver, lung, neuroblastoma, ovarian, pancreatic, prostate, sarcoma, and skin are externally validated; bile duct, bone, endometrial, esophageal, gallbladder, and gastric classes are not validated and may be misclassified as related cancers
- A degree of discordancy exists between scATOMIC and CNV inference (CopyKAT); combining both is recommended, and CNV inference may fail for near-diploid cancer cells or few normal control cells
- Subclassification beyond breast cancer is not yet implemented; additional work is needed for cancers of unknown primary and mucinous ovarian carcinoma
- Datasets >15,000 cells need more than the standard 16GB RAM; Windows lacks native parallelism and runs slower

## Evidence

- [methods] run_scATOMIC generates detailed hierarchical annotation: "To run scATOMIC we use the run_scATOMIC() function to generate a detailed annotation object."
- [methods] use sparse matrices: "We recommend running scATOMIC on sparse matrices to speed up performance and save memory."
- [methods] filter low-quality cells: "remove any cells with more than 25% of reads mapping to mitochondrial genes or those with fewer than 500"
- [readme] run per-sample only: "Inter-patient cancer cell specific effects can interfere with scATOMICs cancer signature scoring module and confidence in classifications."
- [methods] MAGIC imputation dependency: "In scATOMIC we use the Rmagic package to impute values."
- [methods] summary matrix step: "After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix()."
