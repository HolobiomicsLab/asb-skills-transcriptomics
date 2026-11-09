---
name: single-cell-cancer-signature-module-scoring
description: Use when cells have already received a cancer-type annotation from scATOMIC's
  terminal layer and you must separate malignant cells from normal cells sharing the
  same tissue of origin in a tumour biopsy.
when_to_use_negative:
- Sample is normal (non-cancer) tissue rather than a tumour biopsy — scATOMIC's default
  model assumes at least some cancer cells are present; use the normal_tissue parameter
  workflow instead
- Analysis is run on a merged multi-patient cohort — inter-patient cancer cell-specific
  effects interfere with the cancer signature scoring module and classification confidence
- Goal is to classify non-malignant immune or stromal subtypes — scATOMIC's terminal
  cancer-signature module applies only to cells predicted as cancer
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scATOMIC
  role: Runs hierarchical pan-cancer classification and hosts the cancer signature
    scoring / cell-differentiating module and create_summary_matrix() consolidation
  repo: github.com/abelson-lab/scATOMIC
- name: Seurat
  role: AddModuleScore() scores the OncoDB-derived up-/down-regulated gene programs
    in cancer-predicted cells
- name: Rmagic
  role: MAGIC imputation of count values before run_scATOMIC() classification
- name: cutoff.scATOMIC
  role: Fits an EM finite mixture model to bimodal score distributions to derive cutoff
    values separating Cancer vs Normal confidence scores
  repo: github.com/inofechm/cutoff.scATOMIC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:ea6f20ca2d2c0917e3f7b616d8aafeed2ec47e1bbe6e63055cf097158c81907f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rmagic
  - Seurat
  - cutoff.scATOMIC
  - reticulate
  - scATOMIC
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# single-cell-cancer-signature-module-scoring

## Summary

scATOMIC's cancer signature scoring and cell-differentiating module: after terminal-layer annotation, cancer-type-specific up-/down-regulated gene programs (OncoDB-derived) are scored per cell with Seurat AddModuleScore, and clusters enriched for normal cells are iteratively dropped to distinguish malignant from non-malignant tissue-specific cells.

## When to use

Use when cells have already received a cancer-type annotation from scATOMIC's terminal layer and you must separate malignant cells from normal cells sharing the same tissue of origin in a tumour biopsy.

## When NOT to use

- Sample is normal (non-cancer) tissue rather than a tumour biopsy — scATOMIC's default model assumes at least some cancer cells are present; use the normal_tissue parameter workflow instead
- Analysis is run on a merged multi-patient cohort — inter-patient cancer cell-specific effects interfere with the cancer signature scoring module and classification confidence
- Goal is to classify non-malignant immune or stromal subtypes — scATOMIC's terminal cancer-signature module applies only to cells predicted as cancer

## Inputs

- Sparse gene-by-cell count matrix (e.g., scATOMIC's demo_lung_data lung cancer demo dataset)
- scATOMIC hierarchical per-layer cell predictions from run_scATOMIC()
- OncoDB-derived cancer-type up-regulated and down-regulated gene program lists (pan_cancer_upreg / pan_cancer_downreg)

## Outputs

- Per-cell cancer signature module scores (up- and down-regulated programs)
- pan_cancer_cluster partition of cancer-predicted cells (each cluster <=20% normal cells)
- Cancer vs Normal classification_confidence label per cell
- CSV table with one row per filtered cell: layer predictions, pan_cancer_cluster, classification_confidence (cancer_vs_normal_labels.csv)

## How to apply

Filter the sparse gene-by-cell count matrix, removing cells with >25% mitochondrial reads or fewer than 500 unique expressed features, then run run_scATOMIC() (with Rmagic/MAGIC imputation) to get hierarchical predictions per cell. On cells predicted as cancer at the terminal layer, score the OncoDB-derived pan_cancer_upreg and pan_cancer_downreg gene programs (|log2FC|>1, adj P<0.01) using Seurat AddModuleScore. Cluster the signature-scored cells and iteratively drop/refine clusters whose fraction of normal cells exceeds 20% until every retained cluster has <=20% normal cells; label the final partition pan_cancer_cluster. Consolidate per-layer predictions with create_summary_matrix() (modify_results = TRUE assuming a single cancer type, min_prop = 0.5) to derive a cell-level Cancer vs Normal classification_confidence label. The rationale is that inter-patient cancer-cell-specific transcriptional effects can otherwise confound confidence, so scoring and cluster refinement are applied only within a single patient/biopsy sample.

## Related tools

- **scATOMIC** (Runs hierarchical pan-cancer classification and hosts the cancer signature scoring / cell-differentiating module and create_summary_matrix() consolidation) — github.com/abelson-lab/scATOMIC
- **Seurat** (AddModuleScore() scores the OncoDB-derived up-/down-regulated gene programs in cancer-predicted cells)
- **Rmagic** (MAGIC imputation of count values before run_scATOMIC() classification)
- **cutoff.scATOMIC** (Fits an EM finite mixture model to bimodal score distributions to derive cutoff values separating Cancer vs Normal confidence scores) — github.com/inofechm/cutoff.scATOMIC

## Examples

```
sc <- AddModuleScore(seurat_obj, features = list(scATOMIC::pan_cancer_upreg, scATOMIC::pan_cancer_downreg), name = "pan_cancer")
```

## Evaluation signals

- Output CSV exists with exactly one row per filtered input cell and contains pan_cancer_cluster and classification_confidence fields
- On the lung demo dataset, a majority of cells are labeled confident Lung Cancer Cell in the scATOMIC_pred table, matching the tutorial's reported class distribution
- Every retained pan_cancer_cluster has <=20% normal cells after the iterative drop/refine loop
- Module scores are computed only on cells predicted as cancer at the terminal layer, using gene programs with |log2FC|>1 and adj P<0.01
- create_summary_matrix() consolidation uses modify_results = TRUE (single cancer type assumed) and min_prop = 0.5

## Limitations

- Inter-patient cancer cell-specific effects can interfere with scATOMIC's cancer signature scoring module and classification confidence; scATOMIC should only be run on individual patient/biopsy samples
- A degree of discordancy still exists between scATOMIC and CNV inference; using scATOMIC in conjunction with CNV inference is recommended
- scATOMIC has been externally validated only for bladder, brain, breast, colorectal, kidney, liver, lung, neuroblastoma, ovarian, pancreatic, prostate, sarcoma, and skin cancers; bile duct, bone, endometrial, esophageal, gallbladder, and gastric classes are unvalidated and may be misclassified

## Evidence

- [readme] signature module rationale: "Inter-patient cancer cell specific effects can interfere with scATOMICS cancer signature scoring module and confidence in classifications. We strongly recommend running scATOMIC only on individual"
- [methods] filtering threshold: "We strongly recommend the user filter low quality cells from their dataset. We recommend to remove any cells with more than 25% of reads mapping to mitochondrial genes or those with fewer than 500"
- [methods] classification run: "To run scATOMIC we use the run_scATOMIC() function to generate a detailed annotation object."
- [methods] consolidation step: "After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix()."
- [methods] demo dataset: "We included a demo dataset of a primary lung cancer sample within our package under the object name ‘demo_lung_data’."
