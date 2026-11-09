---
name: dropout-noise-robustness-assessment
description: Use when evaluating how a tumor/normal cell classifier degrades under
  increasing dropout noise in scRNA-seq count matrices with known ground-truth labels,
  or when reproducing F1 benchmarks like SCEVAN's 0.948/0.943/0.909/0.824 vs CopyKAT's
  0.798/0.792/0.763/0.726.
when_to_use_negative:
- Input matrices lack ground-truth tumor/normal labels, so F1 against known labels
  cannot be computed
- The data are bulk DNA copy-number profiles or WES/WGS-derived ground truth rather
  than single-cell RNA-seq count matrices with simulated dropout
- The goal is clonal substructure/subclone calling rather than binary malignant vs
  non-malignant classification accuracy
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SCEVAN
  role: Automatic malignant/non-malignant classification being benchmarked, run via
    pipelineCNA or classifyTumorCells with ground_truth labels over 20 cores
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Comparison method providing automatic malignant/non-malignant predictions
    for the same matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:9a53b440a1c27c3107d789930ef8c3651b8d1f4c2647a16b7716e8272f7bb885
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AntonioDeFalco/SCEVAN
  tools:
  - CopyKAT v1.0.5
  - SCEVAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dropout-noise-robustness-assessment

## Summary

Benchmark robustness of malignant/non-malignant cell classification on synthetic scRNA-seq count matrices by computing mean F1 scores across dropout noise levels (none, -2, -1, 0), comparing SCEVAN against CopyKAT v1.0.5 on 500 matrices covering 322,687 cells.

## When to use

Use when evaluating how a tumor/normal cell classifier degrades under increasing dropout noise in scRNA-seq count matrices with known ground-truth labels, or when reproducing F1 benchmarks like SCEVAN's 0.948/0.943/0.909/0.824 vs CopyKAT's 0.798/0.792/0.763/0.726.

## When NOT to use

- Input matrices lack ground-truth tumor/normal labels, so F1 against known labels cannot be computed
- The data are bulk DNA copy-number profiles or WES/WGS-derived ground truth rather than single-cell RNA-seq count matrices with simulated dropout
- The goal is clonal substructure/subclone calling rather than binary malignant vs non-malignant classification accuracy

## Inputs

- 500 synthetic scRNA-seq count matrices with known tumor/normal ground-truth labels (Supplementary Data 3), grouped by dropout noise level x0 = none, -2, -1, 0
- SCEVAN R package (install_github('AntonioDeFalco/SCEVAN'))
- CopyKAT v1.0.5

## Outputs

- Per-matrix F1 scores for SCEVAN and CopyKAT at each dropout level
- benchmark_f1_scores.csv with rows = method and columns = dropout level containing mean F1 values

## How to apply

Load the 500 synthetic scRNA-seq count matrices with known tumor/normal ground-truth labels from Supplementary Data 3, organized by dropout level (x0 = none, -2, -1, 0). For each matrix run SCEVAN's automatic classification via SCEVAN::pipelineCNA or classifyTumorCells supplying the ground_truth labels (parallelized over 20 cores) and separately run CopyKAT v1.0.5, collecting predicted labels from both. Compute per-matrix F1 scores of malignant vs non-malignant predictions against the known labels, then average over all 500 matrices within each dropout level. Assemble a benchmark table (rows = method, columns = dropout level) and validate the recomputed means against the paper-reported values within a small tolerance before writing benchmark_f1_scores.csv.

## Related tools

- **SCEVAN** (Automatic malignant/non-malignant classification being benchmarked, run via pipelineCNA or classifyTumorCells with ground_truth labels over 20 cores) — https://github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Comparison method providing automatic malignant/non-malignant predictions for the same matrices)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "synthetic_x0", par_cores = 20, SUBCLONES = FALSE)
```

## Evaluation signals

- Recomputed mean F1 for SCEVAN matches 0.948, 0.943, 0.909, 0.824 at x0 = none, -2, -1, 0 within a small tolerance
- Recomputed mean F1 for CopyKAT matches 0.798, 0.792, 0.763, 0.726 at the same four dropout levels within tolerance
- benchmark_f1_scores.csv is present with a complete 2 x 4 grid of mean F1 values (both methods at all four dropout levels)
- F1 degradation is monotonic (or near-monotonic) with increasing dropout for both methods, as reported in the paper
- All 500 matrices were processed for each method with no missing predictions

## Limitations

- Results depend on synthetic matrices with simulated dropout; real-data behavior may differ from the 500-matrix simulation (322,687 cells)
- Classification F1 is only the binary tumor/normal axis; the paper's broader claims (subclone identification, speed) are not covered by this metric
- SCEVAN's aneuploidy-based assumption may not suit tumors with minimal genomic alterations (leukemia, pediatric cancers, Ependymomas), so robustness scores may not transfer to such tumors

## Evidence

- [results] SCEVAN vs CopyKAT F1 benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] SCEVAN automatic classification claim: "SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells, resulting in a practical framework to analyze tumors and their microenvironment"
- [intro] CopyKAT comparison rationale: "An approach to overcome these limitations is represented by the CopyKAT method11 that automatically classifies malignant and non-malignant cells."
- [methods] classifyTumorCells API: "classifyTumorCells(count_mtx, annot_mtx, sample, distance, par_cores, ground_truth, norm_cell_names, SEGMENTATION_CLASS, SMOOTH, beta_vega, FIXED_NORMAL_CELLS, output_dir)"
- [readme] SCEVAN installation and classification purpose: [evidence span withheld — anchor status: external_doc]
- [readme] SCEVAN installation commands: "install_github("miccec/yaGST")
install_github("AntonioDeFalco/SCEVAN")"
