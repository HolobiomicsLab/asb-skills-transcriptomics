---
name: blood-stroma-cancer-prediction-mapping
description: Use when benchmarking automated scRNA-seq annotators on tumour-microenvironment
  samples and you need tool-agnostic, three-class (blood/stroma/cancer) F1 comparisons
  such as the scATOMIC Fig. 2c evaluation.
when_to_use_negative:
- The goal is fine-grained subtype annotation (e.g., macrophage/monocyte/cDC subtypes)
  rather than a coarse three-class benchmark — use the full scATOMIC hierarchy instead
- The input sample is normal tissue rather than a tumour biopsy; scATOMIC's default
  model assumes at least some cancer cells and the normal_tissue parameter must be
  used
- You need tool-specific outputs (e.g., CNV inference for malignant cell calls) rather
  than a cross-annotator F1 comparison
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0621
tools:
- name: scATOMIC
  role: Tool under test whose hierarchical TME predictions are mapped to blood/stroma/cancer
    classes
  repo: https://github.com/abelson-lab/scATOMIC
- name: SingleR
  role: Comparator annotator; best-performing comparator with blood/stroma/cancer
    F1 of 0.92/0.97/0.76
- name: Seurat
  role: Comparator annotator trained on the same reference and applied to validation
    biopsies
- name: SingleCellNet
  role: Comparator annotator in the same benchmark
- name: scmap-cell
  role: Comparator annotator in the same benchmark
- name: CHETAH
  role: Comparator annotator in the same benchmark
- name: scType
  role: Comparator annotator in the same benchmark
- name: R
  role: Language in which all annotators are run and F1/Wilcoxon analyses are computed
    (R 4.3.2 in the paper)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:39d5a0c8ff03b28ea18e4f730048dfac5c3ac909d38fb1aa532cce488f9d85a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - CHETAH
  - R
  - Seurat
  - SingleCellNet
  - SingleR
  - scATOMIC
  - scType
  - scmap-cell
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Blood-stroma-cancer prediction mapping

## Summary

Collapses fine-grained cell-type predictions from multiple scRNA-seq annotators (scATOMIC, SingleR, Seurat, SingleCellNet, scmap-cell, CHETAH, scType) onto the three combined TME categories — blood cell, stromal cell, cancer cell — so their performance can be compared on equal footing using per-class F1 scores.

## When to use

Use when benchmarking automated scRNA-seq annotators on tumour-microenvironment samples and you need tool-agnostic, three-class (blood/stroma/cancer) F1 comparisons such as the scATOMIC Fig. 2c evaluation.

## When NOT to use

- The goal is fine-grained subtype annotation (e.g., macrophage/monocyte/cDC subtypes) rather than a coarse three-class benchmark — use the full scATOMIC hierarchy instead
- The input sample is normal tissue rather than a tumour biopsy; scATOMIC's default model assumes at least some cancer cells and the normal_tissue parameter must be used
- You need tool-specific outputs (e.g., CNV inference for malignant cell calls) rather than a cross-annotator F1 comparison

## Inputs

- Pan-cancer scATOMIC training reference (>300,000 cancer, immune, and stromal cells across 19 common cancers)
- External validation set of 225 tumour biopsies with >350,000 cells
- Fine-grained cell-type predictions from each annotator (scATOMIC, SingleR, Seurat, SingleCellNet, scmap-cell, CHETAH, scType)
- Sparse gene-by-cell count matrices used as annotator input

## Outputs

- Per-tool F1 score table for blood, stromal, and cancer classes (comparator_f1_scores.csv)
- Wilcoxon test results for cancer-cell F1 gaps (scATOMIC vs. each comparator)
- Combined three-class (blood/stroma/cancer) label assignments per cell per tool

## How to apply

Train each comparator annotator (SingleR, Seurat, SingleCellNet, scmap-cell, CHETAH, scType) plus scATOMIC on the same pan-cancer reference (>300,000 cells across 19 common cancers) and apply each to the external validation set (225 tumour biopsies, >350,000 cells) using their default R implementations. Map every tool's fine-grained labels onto the three combined categories: blood cell, stromal cell, and cancer cell, so that heterogeneous hierarchical outputs become directly comparable. Compute combined-class F1 scores per tool and verify against the reported values (scATOMIC 0.95/0.99/0.99; best comparator SingleR 0.92/0.97/0.76). Apply a Wilcoxon test to the cancer-cell F1 distributions to assess significance of the scATOMIC-vs-comparator gap, as reported for Fig. 2c (all cancer-cell comparisons P < 2 × 10−16). Write the per-tool F1 table to comparator_f1_scores.csv.

## Related tools

- **scATOMIC** (Tool under test whose hierarchical TME predictions are mapped to blood/stroma/cancer classes) — https://github.com/abelson-lab/scATOMIC
- **SingleR** (Comparator annotator; best-performing comparator with blood/stroma/cancer F1 of 0.92/0.97/0.76)
- **Seurat** (Comparator annotator trained on the same reference and applied to validation biopsies)
- **SingleCellNet** (Comparator annotator in the same benchmark)
- **scmap-cell** (Comparator annotator in the same benchmark)
- **CHETAH** (Comparator annotator in the same benchmark)
- **scType** (Comparator annotator in the same benchmark)
- **R** (Language in which all annotators are run and F1/Wilcoxon analyses are computed (R 4.3.2 in the paper))

## Examples

```
wilcox.test(f1$scATOMIC, f1$SingleR, paired = TRUE)  # cancer-cell F1 gap, Fig. 2c
```

## Evaluation signals

- Each tool's blood/stroma/cancer F1 values in the output table match the Fig. 2c reported values: scATOMIC 0.95/0.99/0.99 and SingleR 0.92/0.97/0.76
- Wilcoxon test on cancer-cell F1 scores supports scATOMIC outperforming each comparator (reported all cancer-cell comparisons P < 2 × 10−16)
- Every fine-grained prediction from every tool resolves to exactly one of the three combined categories with no unmapped labels
- All seven tools are represented in comparator_f1_scores.csv with identical reference and validation data

## Limitations

- scATOMIC's default model assumes the presence of at least some cancer cells and should not be used on normal tissue samples without the normal_tissue parameter
- Bile duct, bone, endometrial, esophageal, gallbladder, and gastric cancer classes have not been validated in external datasets and may be misclassified as other related cancers
- The three-class collapse discards fine-grained subtype information, so it only measures coarse TME compartment classification, not subtype accuracy
- Wilcoxon significance is specific to the cancer-cell F1 gap; blood and stromal differences are smaller (e.g., SingleR 0.92 vs 0.95 blood)

## Evidence

- [results] scATOMIC achieved median F1 scores of 0.95, 0.99, and 0.99 for blood, stromal, and cancer cells respectively: [evidence span withheld — anchor status: not_found]
- [abstract] scATOMIC trained on >300,000 cells across 19 cancers: "We trained scATOMIC on >300,000 cancer, immune, and stromal cells defining a pan-cancer reference across 19 common cancers"
- [discussion] outperforms other annotators on blood and stromal cells: "scATOMIC is comparable to or outperforms other existing automatic cell type annotators when classifying blood and stromal cells using our training reference."
- [readme] scATOMIC hierarchy defines the classes being mapped: "scATOMIC is designed to classify cells within the following pan cancer TME hierarchy"
