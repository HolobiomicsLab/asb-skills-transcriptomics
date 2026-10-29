---
name: f1-score-computation-against-ground-truth-labels
description: Use when comparing automatic malignant-vs-non-malignant classification
  methods on single-cell count matrices that come with known tumor/normal ground-truth
  labels, and you need a single harmonic-mean accuracy number per condition.
when_to_use_negative:
- Ground truth derives from bulk WES/WGS copy-number profiles rather than per-cell
  tumor/normal labels — the article notes these 'have of course different resolutions
  with respect to the single-cell data' and require resampling before comparison
- Input is real tumor data without any known malignant/non-malignant labels — F1 cannot
  be computed and unsupervised classification metrics must be used instead
- The task is clone/subclone structure evaluation rather than binary tumor/normal
  classification — F1 against binary labels is not the appropriate metric
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCEVAN
  role: Produces the automatic malignant/non-malignant predictions being scored (pipelineCNA
    / classifyTumorCells with ground_truth labels, par_cores = 20)
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Baseline automatic classifier whose malignant/non-malignant predictions are
    scored for comparison against SCEVAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:4873ccf02a070d3cbcf89df35a7e8d7fd87f9136fe089560fda0743366cf6a75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AntonioDeFalco/SCEVAN
  tools:
  - CopyKAT v1.0.5
  - SCEVAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# f1-score-computation-against-ground-truth-labels

## Summary

Benchmarking protocol that scores automatic malignant/non-malignant cell classifications (e.g., SCEVAN vs CopyKAT) by computing per-matrix F1 scores against known tumor/normal ground-truth labels on 500 synthetic scRNA-seq matrices and averaging per dropout noise level.

## When to use

Use when comparing automatic malignant-vs-non-malignant classification methods on single-cell count matrices that come with known tumor/normal ground-truth labels, and you need a single harmonic-mean accuracy number per condition.

## When NOT to use

- Ground truth derives from bulk WES/WGS copy-number profiles rather than per-cell tumor/normal labels — the article notes these 'have of course different resolutions with respect to the single-cell data' and require resampling before comparison
- Input is real tumor data without any known malignant/non-malignant labels — F1 cannot be computed and unsupervised classification metrics must be used instead
- The task is clone/subclone structure evaluation rather than binary tumor/normal classification — F1 against binary labels is not the appropriate metric

## Inputs

- Synthetic scRNA-seq raw count matrices with known tumor/normal ground-truth labels (Supplementary Data 3; 500 matrices, 322,687 cells)
- Per-matrix dropout noise level designation (x0 = none, -2, -1, 0)
- Predicted malignant/non-malignant cell labels from each classifier (SCEVAN, CopyKAT v1.0.5)

## Outputs

- Per-matrix F1 scores for each method and dropout level
- Mean F1 scores aggregated over 500 matrices per method per dropout level
- Benchmark table (benchmark_f1_scores.csv) with rows = method, columns = dropout level

## How to apply

Load the synthetic count matrices with their ground-truth labels from Supplementary Data 3, organized by dropout level (x0 = none, -2, -1, 0). For each matrix, run each classifier — SCEVAN's pipelineCNA or classifyTumorCells (supplying the ground_truth argument, parallelized over 20 cores) and CopyKAT v1.0.5 — and collect predicted cell labels. For each method, matrix, and dropout level, compute the F1 score of the malignant class from true positives, false positives, and false negatives against the known labels, then average over the 500 matrices. Assemble the recomputed means into a method-by-dropout-level table (e.g., benchmark_f1_scores.csv). Validation: the paper reports SCEVAN means of 0.948/0.943/0.909/0.824 and CopyKAT means of 0.798/0.792/0.763/0.726 for x0 = none/-2/-1/0; recomputed values should match these within a small tolerance.

## Related tools

- **SCEVAN** (Produces the automatic malignant/non-malignant predictions being scored (pipelineCNA / classifyTumorCells with ground_truth labels, par_cores = 20)) — https://github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Baseline automatic classifier whose malignant/non-malignant predictions are scored for comparison against SCEVAN)

## Examples

```
pred <- SCEVAN::classifyTumorCells(count_mtx, annot_mtx, sample = "syn01", par_cores = 20, ground_truth = truth_labels)  # then F1 = 2*TP/(2*TP+FP+FN) per matrix
```

## Evaluation signals

- Recomputed mean F1 values match the paper-reported SCEVAN 0.948/0.943/0.909/0.824 and CopyKAT 0.798/0.792/0.763/0.726 (x0 = none/-2/-1/0) within a small tolerance
- Benchmark table contains numerically complete entries for both methods at all four dropout levels (a 2x4 grid)
- Every per-matrix F1 score lies in [0, 1] and every cell receives exactly one predicted label aligned to the ground-truth label vector
- Aggregate covers all 500 synthetic matrices (322,687 cells) with no matrices silently dropped from the means
- Expected qualitative invariant: SCEVAN's mean F1 exceeds CopyKAT's at every dropout level, and both degrade monotonically as dropout increases

## Limitations

- Performance was measured on synthetic matrices with controlled dropout; aneuploidy-based methods (and thus this benchmark) may not transfer to tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas
- F1 here reflects only the binary malignant/non-malignant call; it does not assess the clonal substructure or phylogeny outputs of the tools
- Ground truth from bulk sequencing has different resolution than single-cell data and requires resampling before comparable scoring

## Evidence

- [other] Reported benchmark values: [evidence span withheld — anchor status: not_found]
- [other] Scoring procedure: [evidence span withheld — anchor status: not_found]
- [readme] Real-data headline metric: [evidence span withheld — anchor status: external_doc]
- [discussion] Compared tool versions: "are analyzed following the steps of SCEVAN Workflow (Methods) and with CopyKAT v1.0.5 and inferCNV v1.4.0."
- [methods] Key classification function accepting ground truth: "classifyTumorCells(count_mtx, annot_mtx, sample, distance, par_cores, ground_truth, norm_cell_names, SEGMENTATION_CLASS, SMOOTH, beta_vega, FIXED_NORMAL_CELLS, output_dir)"
