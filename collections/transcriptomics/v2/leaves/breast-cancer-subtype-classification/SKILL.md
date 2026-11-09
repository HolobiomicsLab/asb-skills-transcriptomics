---
name: breast-cancer-subtype-classification
description: Use when scRNA-seq tumour biopsies are already annotated as Breast Cancer
  Cell by the base scATOMIC pan-cancer model and the goal is to assign ER+/HER2+/TN
  subtype labels per sample, including mixed double-positive calls.
when_to_use_negative:
- Samples not already identified as breast cancer by the base pan-cancer scATOMIC
  model — the subclassification layer expects Breast Cancer Cell inputs only
- Whole-cohort classification run on merged multi-patient matrices — the README strongly
  recommends running scATOMIC on individual patient/biopsy samples because inter-patient
  cancer-cell effects interfere with signature scoring
- Normal (non-cancerous) tissue samples — the default scATOMIC model assumes the presence
  of at least some cancer cells; use the normal_tissue parameter instead
- Subclassifying cancer types other than breast — future extension of the hierarchy
  to the other 18 cancer types is not yet implemented
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scATOMIC
  role: Provides run_scATOMIC(), create_summary_matrix(), get_new_scATOMIC_layer(),
    and classify_new_scATOMIC_layer() for pan-cancer annotation and breast subtype
    resolution
  repo: github.com/abelson-lab/scATOMIC
- name: Rmagic
  role: MAGIC imputation of expression values used by classify_new_scATOMIC_layer()
    when imputation = TRUE
- name: R
  role: Runtime environment (R 4.3.2) in which the scATOMIC pipeline executes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:7fbc19d70a51661d7a0d38a0406b8a313162a8f4475ffe8974d4aba9ecd70402
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rmagic
  - scATOMIC
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# breast-cancer-subtype-classification

## Summary

Applies the pre-trained scATOMIC breast subclassification random forest to pan-cancer-annotated breast cancer cells, resolving them into ER+, HER2+, and triple negative histological subtypes, validated at 97.4% accuracy (37/38 tumours) against immunostaining ground truth.

## When to use

Use when scRNA-seq tumour biopsies are already annotated as Breast Cancer Cell by the base scATOMIC pan-cancer model and the goal is to assign ER+/HER2+/TN subtype labels per sample, including mixed double-positive calls.

## When NOT to use

- Samples not already identified as breast cancer by the base pan-cancer scATOMIC model — the subclassification layer expects Breast Cancer Cell inputs only
- Whole-cohort classification run on merged multi-patient matrices — the README strongly recommends running scATOMIC on individual patient/biopsy samples because inter-patient cancer-cell effects interfere with signature scoring
- Normal (non-cancerous) tissue samples — the default scATOMIC model assumes the presence of at least some cancer cells; use the normal_tissue parameter instead
- Subclassifying cancer types other than breast — future extension of the hierarchy to the other 18 cancer types is not yet implemented

## Inputs

- Gene-by-cell scRNA-seq count matrices for individual breast cancer biopsy samples (sparse matrices recommended; Supplementary Data 5 for the 38-tumour validation cohort plus 2 HER2+/ER+ double-positive samples)
- Pre-trained breast subclassification random forest layer (trained on Supplementary Data 4)

## Outputs

- Per-cell subtype labels in the predicted_tissue_with_cutoff column (ER+, HER2+, or TN)
- Per-sample aggregated subtype summary table with mixed HER2+/ER+ annotations for double-positive tumours
- Overall subtyping accuracy versus immunostaining ground truth (target: 37/38, 97.4%)

## How to apply

First run the base scATOMIC pan-cancer model with run_scATOMIC() on each filtered sample (removing cells with >25% mitochondrial reads or <500 unique features) and summarize with create_summary_matrix() to isolate cells labelled Breast Cancer Cell. Then apply the pre-trained breast subclassification random forest — trained on Supplementary Data 4 via get_new_scATOMIC_layer() — with classify_new_scATOMIC_layer(), setting imputation = TRUE (Rmagic-based MAGIC imputation) and confidence_cutoff = TRUE, and take the predicted_tissue_with_cutoff labels. Aggregate per-sample subtype calls (ER+, HER2+, TN) into a summary table, recording mixed HER2+/ER+ annotations for double-positive tumours (2 in the validation cohort). Judge success by comparing subtype calls to immunostaining ground truth: the published validation correctly subtyped 37 of 38 (97.4%) single-subtype biopsies. Rationale: scATOMIC's hierarchical elimination scheme reduces transcriptomic complexity before subtype resolution, and the confidence cutoff suppresses low-confidence classifications using an EM-fitted bimodal mixture cutoff.

## Related tools

- **scATOMIC** (Provides run_scATOMIC(), create_summary_matrix(), get_new_scATOMIC_layer(), and classify_new_scATOMIC_layer() for pan-cancer annotation and breast subtype resolution) — github.com/abelson-lab/scATOMIC
- **Rmagic** (MAGIC imputation of expression values used by classify_new_scATOMIC_layer() when imputation = TRUE)
- **R** (Runtime environment (R 4.3.2) in which the scATOMIC pipeline executes)

## Examples

```
predictions <- classify_new_scATOMIC_layer(scATOMIC_results, imputation = TRUE, confidence_cutoff = TRUE); table(predictions$predicted_tissue_with_cutoff)
```

## Evaluation signals

- Subtyping accuracy against immunostaining ground truth reaches 37/38 (97.4%) on the 38 single-subtype validation tumours
- The two HER2+/ER+ double-positive tumours receive mixed HER2+/ER+ annotations rather than a single forced call
- All cells carry a label in the predicted_tissue_with_cutoff column after confidence_cutoff = TRUE, indicating sufficient classification confidence
- Low-quality cells are removed beforehand: none with >25% mitochondrial reads or <500 unique features remain in the input matrices
- Per-sample calls are run on individual biopsies, not merged multi-patient matrices, per the README recommendation

## Limitations

- Only 37/38 accuracy was achieved; 1 of 38 single-subtype tumours was misassigned, so subtyping is high but not perfect
- Subclassification beyond breast cancer is not yet implemented — the other 18 cancer types await future extension of the hierarchy
- scATOMIC should only be run on individual patient/biopsy samples; inter-patient cancer cell specific effects interfere with signature scoring
- The default model assumes the presence of at least some cancer cells and should not be used on normal tissue
- For datasets > 15,000 cells, a machine with greater than 16GB RAM is recommended

## Evidence

- [results] Extended scATOMIC breast cancer subtype validation accuracy: [evidence span withheld — anchor status: not_found]
- [methods] Confidence-cutoff subtype labels: [evidence span withheld — anchor status: not_found]
- [methods] Cell quality filtering thresholds: "remove any cells with more than 25% of reads mapping to mitochondrial genes or those with fewer than 500"
- [readme] Per-sample run recommendation: "We strongly recommend running scATOMIC only on individual patient/biopsy samples."
- [discussion] Hierarchy extension limitation: "future extensions of the core hierarchy to further subclassify the other 18 cancer types and the various core, non-malignant cell types"
