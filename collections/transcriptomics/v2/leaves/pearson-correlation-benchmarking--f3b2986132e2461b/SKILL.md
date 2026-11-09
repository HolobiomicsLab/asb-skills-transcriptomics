---
name: pearson-correlation-benchmarking
description: Use when evaluating predicted single-cell perturbation responses (e.g.,
  control vs stimulated PBMC) against ground truth by DEG overlap and squared Pearson
  correlation per cell type.
when_to_use_negative:
- Predictions cannot be paired with ground-truth stimulated profiles for the same
  cell types, so no DEG or correlation comparison is possible
- The task is cross-modality translation quality (e.g., RNA-to-ATAC) rather than perturbation-response
  prediction, where other README-integrated evaluation metrics apply
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scButterfly
  role: Generates the predicted perturbation-response profiles (scButterfly-T augmentation
    with per-cell-type optimal transport matching, Model.test predictions) whose top-100
    DEG overlap and R2 are benchmarked
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python
  role: Implementation environment (Python 3.9 conda environment) for computing DEG
    overlaps and Pearson correlations and writing the benchmark CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:8fa8c01dfb6391b0c48e7977c51089709b134ab58195b5678c5653e67bc86fcd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - scButterfly
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pearson-correlation-benchmarking

## Summary

Benchmark single-cell perturbation-response predictions by computing, per cell type, the number of common top-100 differentially expressed genes (DEGs) shared with ground truth and the squared Pearson correlation (R2) between predicted and real DEG expression, as used to validate scButterfly-B against scGen and scPreGAN on PT_PBMC.

## When to use

Use when evaluating predicted single-cell perturbation responses (e.g., control vs stimulated PBMC) against ground truth by DEG overlap and squared Pearson correlation per cell type.

## When NOT to use

- Predictions cannot be paired with ground-truth stimulated profiles for the same cell types, so no DEG or correlation comparison is possible
- The task is cross-modality translation quality (e.g., RNA-to-ATAC) rather than perturbation-response prediction, where other README-integrated evaluation metrics apply

## Inputs

- AnnData object of predicted stimulated single-cell expression profiles (cells x genes)
- AnnData object of ground-truth stimulated single-cell expression profiles (cells x genes)
- Cell-type labels for per-cell-type stratification
- Top-DEG count threshold (100 real DEGs per cell type)

## Outputs

- Per-cell-type counts of common DEGs among the top 100 for each method
- Per-cell-type mean squared Pearson correlation (R2) for the top 100 real DEGs for each method
- Benchmark comparison table (e.g., perturbation_benchmark_results.csv) covering scButterfly, scGen, and scPreGAN

## How to apply

After generating predicted stimulated-expression profiles for held-out cell types (e.g., via scButterfly's Model.test), compute differential expression separately on predicted and ground-truth profiles. For each cell type, identify the top 100 DEGs in each profile, count how many DEGs overlap between prediction and ground truth, and compute the mean squared Pearson correlation (R2) of expression across those top 100 real DEGs. Compile the DEG-overlap counts and R2 values per cell type and method (scButterfly, scGen, scPreGAN) into a results table such as perturbation_benchmark_results.csv, mirroring the article's Fig. 4h comparison. Validate that R2 exceeds 0.85 for every cell type and that top-100 DEG overlap counts match or exceed the baselines.

## Related tools

- **scButterfly** (Generates the predicted perturbation-response profiles (scButterfly-T augmentation with per-cell-type optimal transport matching, Model.test predictions) whose top-100 DEG overlap and R2 are benchmarked) — https://github.com/BioX-NKU/scButterfly
- **Python** (Implementation environment (Python 3.9 conda environment) for computing DEG overlaps and Pearson correlations and writing the benchmark CSV)

## Examples

```
from scButterfly.train_model_perturb import Model; A2R_predict = Model.test(); # then compute per-cell-type top-100 DEG overlap and squared Pearson R2 vs ground truth
```

## Evaluation signals

- Mean squared Pearson correlation (R2) across the top 100 real DEGs exceeds 0.85 for every cell type
- Common top-100 DEG counts per cell type match or exceed those of scGen and scPreGAN baselines (Fig. 4h comparison)
- Benchmark output contains one R2 value and one DEG-overlap count per cell type per method and is written to perturbation_benchmark_results.csv
- Per-cell-type stratification is applied so held-out (out-of-sample) cell types are evaluated separately

## Limitations

- The 0.85 R2 validation threshold is specific to the PT_PBMC perturbation benchmark and top-100 DEG set; it may not transfer to other datasets or DEG counts
- Benchmarking depends on the diagonal augmentation strategy matching unpaired control/stimulated cells by cell type, so mislabeled or absent cell_type annotations degrade comparability
- Performance for cross-cell-type and cross-organ translation is flagged by the authors as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters

## Evidence

- [results] The transcriptome profiles predicted by scButterfly-B correlated well with the ground truth across different cell types, with the mean squared Pearson correlation (R2) for the top 100 real DEGs consistently surpassing 0.85 for each cell type.: [evidence span withheld — anchor status: not_found]
- [methods] Perturbation responses prediction using scButterfly-T is a documented workflow step of the method.: "Perturbation responses prediction using scButterfly-T"
- [readme] scButterfly provides a predicting API whose outputs feed the DEG and correlation evaluation.: [evidence span withheld — anchor status: external_doc]
- [abstract] The method generalizes to perturbation-response analysis on unpaired data, the setting this benchmark targets.: "scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation"
