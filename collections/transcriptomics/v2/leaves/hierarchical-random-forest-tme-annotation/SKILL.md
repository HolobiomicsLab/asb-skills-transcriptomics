---
name: hierarchical-random-forest-tme-annotation
description: Use when gene-by-cell scRNA-seq count matrices from individual tumour
  biopsies need pan-cancer TME cell-type labels, including malignant cells and their
  tissue of origin across 19 cancer types.
when_to_use_negative:
- 'The input is a normal (non-tumour) tissue sample: the default scATOMIC model assumes
  at least some cancer cells are present; use the normal_tissue parameter of create_summary_matrix
  instead.'
- 'The input is a pooled matrix spanning multiple patients: inter-patient cancer-cell-specific
  effects interfere with cancer signature scoring and classification confidence, so
  run scATOMIC per biopsy only.'
- The sample is a cancer type not in the validated reference (e.g. bile duct, bone,
  endometrial, esophageal, gallbladder, gastric), which may be misclassified as related
  cancers.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scATOMIC
  role: 'Core classifier: run_scATOMIC() and create_summary_matrix() produce the hierarchical
    annotations'
  repo: github.com/abelson-lab/scATOMIC
- name: Rmagic
  role: MAGIC imputation of values before signature scoring and classification
- name: cutoff.scATOMIC
  role: Forked cutoff package that fits the EM mixture model used for automatic confidence
    cutoffs
  repo: github.com/inofechm/cutoff.scATOMIC
- name: reticulate
  role: R–Python bridge so Rmagic can call the magic-impute Python module
- name: randomForest
  role: Underlying random forest models at each hierarchy layer
provenance:
  source_task_ids:
  - task_006
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:e375f6c83417ceea45c9b288553ef18bdf0a5e9e623798669df08a316183c6e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rmagic
  - Seurat
  - cutoff.scATOMIC
  - devtools
  - dlm
  - randomForest
  - reticulate
  - scATOMIC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# Hierarchical random forest TME annotation

## Summary

Annotate single cells in tumour microenvironments pan-cancer using scATOMIC, which trains hierarchically organized random forest models with elimination processes on >300,000 cancer, immune and stromal cells covering 19 common cancers, using MAGIC-imputed signatures and EM-fitted confidence cutoffs.

## When to use

Use when gene-by-cell scRNA-seq count matrices from individual tumour biopsies need pan-cancer TME cell-type labels, including malignant cells and their tissue of origin across 19 cancer types.

## When NOT to use

- The input is a normal (non-tumour) tissue sample: the default scATOMIC model assumes at least some cancer cells are present; use the normal_tissue parameter of create_summary_matrix instead.
- The input is a pooled matrix spanning multiple patients: inter-patient cancer-cell-specific effects interfere with cancer signature scoring and classification confidence, so run scATOMIC per biopsy only.
- The sample is a cancer type not in the validated reference (e.g. bile duct, bone, endometrial, esophageal, gallbladder, gastric), which may be misclassified as related cancers.

## Inputs

- Gene-by-cell sparse count matrix (sparseMatrix) for a single patient/biopsy
- Seurat object counts slot (seurat_object@assays$RNA@counts) as input matrix
- demo_lung_data primary lung cancer sample bundled with the scATOMIC package

## Outputs

- Detailed annotation object with per-cell predictions at each hierarchy layer
- Summary matrix with final scATOMIC_pred label, classification_confidence, and Cancer vs Normal calls per cell
- Per-sample majority cancer-type label / predicted primary origin

## How to apply

Convert each patient/biopsy count matrix to a sparseMatrix and remove low-quality cells: any cell with >25% of reads mapping to mitochondrial genes (pct_mt < 25) or fewer than 500 unique expressed features (nFeatureRNA > 500). Run run_scATOMIC() on each sample independently — inter-patient cancer-cell effects interfere with the cancer signature scoring module and confidence estimates — with Rmagic/MAGIC imputation enabled, hierarchical random forest models, and automatic confidence cutoffs derived from the cutoff.scATOMIC EM mixture model. Aggregate layer-wise intermediate predictions with create_summary_matrix(), setting modify_results = TRUE to enforce a single cancer type per sample and min_prop = 0.5; optionally set use_CNVs = TRUE to apply CopyKAT CNV correction. The hierarchy first separates malignant from non-malignant cells, then resolves broad immune/stromal classes and terminal cell types; cells can abstain at intermediate layers when confidence is low. Validate by comparing final labels to CopyKAT/CITE-seq-supported ground truth (median per-sample F1 ≈ 0.99 on 225 external biopsies) or, for metastatic origin prediction, by majority vote of cancer-cell annotations against known primaries (52/62, 83.9%).

## Related tools

- **scATOMIC** (Core classifier: run_scATOMIC() and create_summary_matrix() produce the hierarchical annotations) — github.com/abelson-lab/scATOMIC
- **Rmagic** (MAGIC imputation of values before signature scoring and classification)
- **cutoff.scATOMIC** (Forked cutoff package that fits the EM mixture model used for automatic confidence cutoffs) — github.com/inofechm/cutoff.scATOMIC
- **reticulate** (R–Python bridge so Rmagic can call the magic-impute Python module)
- **randomForest** (Underlying random forest models at each hierarchy layer)

## Examples

```
sparse_matrix <- as(as.matrix(input_matrix), "sparseMatrix"); results <- run_scATOMIC(raw_counts = sparse_matrix); summary_mat <- create_summary_matrix(raw_counts = sparse_matrix, modify_results = TRUE, min_prop = 0.5)
```

## Evaluation signals

- Median per-sample, per-cell-type F1 vs CopyKAT/CITE-seq ground-truth labels on the 225 external-validation biopsies reproduces ~0.99
- Majority-vote primary-origin calls on the 62 metastatic biopsies match 52/62 (83.9%) correct across breast, kidney, lung, ovarian and skin cancers
- Quality filter applied: no retained cell has >25% mitochondrial reads or fewer than 500 unique expressed features
- Rmagic::pymagic_is_available() returns TRUE so imputation is actually running
- Output summary table has one row per filtered input cell with scATOMIC_pred and classification_confidence fields; the lung demo yields a majority of confident Lung Cancer Cell labels

## Limitations

- Inter-patient effects interfere with the cancer signature scoring module and confidence, so scATOMIC must be run per patient/biopsy sample.
- Bile duct, bone, endometrial, esophageal, gallbladder and gastric cancer classes are not externally validated and may be misclassified as other related cancers.
- Datasets >15,000 cells need greater RAM; Windows does not natively parallelize and runs slower.
- Discordancy exists between scATOMIC and CNV inference; using both in conjunction is recommended, and CNV inference alone can fail on near-diploid cancers or with few normal control cells.
- Large package download may time out; set options(timeout=9999999) when installing via devtools.

## Evidence

- [abstract] Trained pan-cancer reference: "We trained scATOMIC on >300,000 cancer, immune, and stromal cells defining a pan-cancer reference across 19 common cancers and employ a hierarchical approach"
- [methods] Quality filtering thresholds: "We strongly recommend the user filter low quality cells from their dataset. We recommend to remove any cells with more than 25% of reads mapping to mitochondrial genes"
- [methods] Core classification call: "To run scATOMIC we use the run_scATOMIC() function to generate a detailed annotation object."
- [methods] Summary aggregation call: "After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix()."
- [readme] Per-sample requirement: "Inter-patient cancer cell specific effects can interfere with scATOMICs cancer signature scoring module and confidence in classifications."
- [readme] Normal-tissue scope limit: "The default scATOMIC model assumes the presence of at least some cancer cells in a biopsy and should not be used in normal tissue applications."
- [readme] Sparse matrix recommendation: "We recommend running scATOMIC on sparse matrices to speed up performance and save memory."
