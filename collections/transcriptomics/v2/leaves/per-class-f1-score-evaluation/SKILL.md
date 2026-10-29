---
name: per-class-f1-score-evaluation
description: Use when a multiclass or hierarchical cell-type classifier must be validated
  against labelled ground truth and reported per cell type, e.g. fivefold cross-validation
  on the pan-cancer training reference or the blood/stroma/cancer benchmark of Fig.
  2c.
when_to_use_negative:
- No per-cell ground-truth labels exist for the sample — F1 cannot be computed without
  held-out truth, so this applies only to stratified CV folds or externally validated
  biopsies
- Input is a normal tissue sample — the default scATOMIC model assumes at least some
  cancer cells are present and should not be used in normal tissue applications
- Data are pooled across patients — inter-patient cancer-cell effects interfere with
  scATOMIC's cancer signature scoring and classification confidence, invalidating
  per-biopsy benchmarking
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scATOMIC
  role: Hierarchical classifier under evaluation; run per fold with run_scATOMIC to
    generate labels scored by per-class F1
  repo: https://github.com/abelson-lab/scATOMIC
- name: cutoff.scATOMIC
  role: Provides the confidence cutoffs applied to scATOMIC predictions before computing
    F1
  repo: https://github.com/inofechm/cutoff.scATOMIC
- name: Rmagic
  role: MAGIC imputation of the count matrix during run_scATOMIC training/prediction
    in the cross-validation scheme
- name: caret
  role: Computes per-class precision, recall, and F1 from predicted vs. ground-truth
    labels
- name: SingleR
  role: Best-performing comparator annotator; trained on the same reference and scored
    with the same per-class F1 pipeline
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:0a67311ecc01be7e1579f8fb1b4291b11efcb9b115711f6bad37f8a023b89116
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - CHETAH
  - R
  - Rmagic
  - Seurat
  - SingleCellNet
  - SingleR
  - caret
  - cutoff.scATOMIC
  - scATOMIC
  - scType
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 52
---

# per-class-f1-score-evaluation

## Summary

Evaluates a cell-type classifier by computing per-class precision, recall, and F1 against per-cell ground-truth labels, as used to benchmark scATOMIC's pan-cancer TME annotations (median fivefold CV F1 0.90–0.99) against six comparator annotators.

## When to use

Use when a multiclass or hierarchical cell-type classifier must be validated against labelled ground truth and reported per cell type, e.g. fivefold cross-validation on the pan-cancer training reference or the blood/stroma/cancer benchmark of Fig. 2c.

## When NOT to use

- No per-cell ground-truth labels exist for the sample — F1 cannot be computed without held-out truth, so this applies only to stratified CV folds or externally validated biopsies
- Input is a normal tissue sample — the default scATOMIC model assumes at least some cancer cells are present and should not be used in normal tissue applications
- Data are pooled across patients — inter-patient cancer-cell effects interfere with scATOMIC's cancer signature scoring and classification confidence, invalidating per-biopsy benchmarking

## Inputs

- Pan-cancer training reference: gene-by-cell sparse count matrix (Matrix) with per-cell labels for cancer, immune, and stromal populations (>300,000 cells, 19 common cancers)
- External validation set: 225 tumour biopsies with >350,000 cells and ground-truth labels
- Per-cell predictions from scATOMIC and each comparator annotator (mapped to the shared label space)

## Outputs

- Per-cell-type precision, recall, and F1 scores (e.g. from caret::confusionMatrix) for each held-out fold
- Per-tool combined-class F1 scores for blood, stroma, and cancer (expected: scATOMIC 0.95/0.99/0.99; SingleR 0.92/0.97/0.76)
- comparator_f1_scores.csv with one row per tool including scATOMIC
- Wilcoxon test p-values for the cancer-cell F1 gap (reported P < 2 x 10^-16)

## How to apply

For internal validation, partition the pan-cancer training reference (gene-by-cell sparse count matrix with per-cell cancer/immune/stromal labels) into five stratified folds of ~61,100 cells that preserve cell-type proportions; train the hierarchical scATOMIC random forests per fold with run_scATOMIC (MAGIC imputation via Rmagic, confidence cutoffs from cutoff.scATOMIC), predict the held-out fold, and compute per-cell-type precision, recall, and F1 against ground truth — expect the median F1 across the hierarchy to fall in 0.90–0.99 (Fig. 2a). For external benchmarking, train each comparator (SingleR, Seurat, SingleCellNet, scmap-cell, CHETAH, scType) on the same >300,000-cell reference across 19 common cancers and apply all tools to the 225 external tumour biopsies (>350,000 cells). Map each tool's fine-grained predictions onto the three combined categories (blood cell, stromal cell, cancer cell) so F1 is comparable across annotators, then compute combined-class F1 per tool. Assess the cancer-cell F1 gap between scATOMIC and each comparator with a Wilcoxon test and write the per-tool table to comparator_f1_scores.csv.

## Related tools

- **scATOMIC** (Hierarchical classifier under evaluation; run per fold with run_scATOMIC to generate labels scored by per-class F1) — https://github.com/abelson-lab/scATOMIC
- **cutoff.scATOMIC** (Provides the confidence cutoffs applied to scATOMIC predictions before computing F1) — https://github.com/inofechm/cutoff.scATOMIC
- **Rmagic** (MAGIC imputation of the count matrix during run_scATOMIC training/prediction in the cross-validation scheme)
- **caret** (Computes per-class precision, recall, and F1 from predicted vs. ground-truth labels)
- **SingleR** (Best-performing comparator annotator; trained on the same reference and scored with the same per-class F1 pipeline)

## Examples

```
caret::confusionMatrix(factor(pred_labels), factor(truth_labels), mode = "everything")
```

## Evaluation signals

- Median per-cell-type F1 across the pan-cancer hierarchy falls within the reported 0.90–0.99 range (Fig. 2a)
- Combined-class F1 per tool matches Fig. 2c: scATOMIC 0.95 (blood) / 0.99 (stroma) / 0.99 (cancer); next-best SingleR 0.92 / 0.97 / 0.76
- Wilcoxon test on cancer-cell F1 scores supports scATOMIC outperforming every comparator (reported P < 2 x 10^-16)
- Each of the five stratified folds contains ~61,100 cells with equal cell-type proportions across folds
- comparator_f1_scores.csv is produced with a row for each of the seven tools (six comparators plus scATOMIC)

## Limitations

- scATOMIC should be run only on individual patient/biopsy samples; inter-patient cancer-cell-specific effects interfere with its cancer signature scoring and classification confidence
- Benchmark coverage is uneven: bile duct, bone, endometrial, esophageal, gallbladder, and gastric cancer classes were not validated in external datasets and may be misclassified as related cancers
- F1 on the three combined classes (blood/stroma/cancer) aggregates away fine-grained subtype errors, so fine labels must first be mapped to the shared categories

## Evidence

- [discussion] scATOMIC is comparable to or outperforms other existing automatic cell type annotators when classifying blood and stromal cells using our training reference.: "scATOMIC is comparable to or outperforms other existing automatic cell type annotators when classifying blood and stromal cells using our training reference."
- [abstract] scATOMIC's accuracy was confirmed on 225 tumour biopsies encompassing >350,000 cells: "We extensively confirm scATOMIC's accuracy on 225 tumour biopsies encompassing >350,000 cancer and a variety of TME cells."
- [abstract] trained on >300,000 cancer, immune, and stromal cells across 19 common cancers: "We trained scATOMIC on >300,000 cancer, immune, and stromal cells defining a pan-cancer reference across 19 common cancers"
- [readme] run scATOMIC only on individual patient/biopsy samples: "We strongly recommend running scATOMIC only on individual patient/biopsy samples."
- [readme] default model assumes presence of cancer cells; not for normal tissue: "The default scATOMIC model assumes the presence of at least some cancer cells in a biopsy and should not be used in normal tissue applications."
