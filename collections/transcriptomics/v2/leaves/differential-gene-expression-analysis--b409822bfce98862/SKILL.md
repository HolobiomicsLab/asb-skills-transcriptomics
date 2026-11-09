---
name: differential-gene-expression-analysis
description: Use when validating predicted perturbation-response transcriptomes (e.g.,
  scButterfly-B output on PT_PBMC) by comparing per-cell-type DEG profiles to ground
  truth with top-100 DEG overlap and R2 metrics.
when_to_use_negative:
- Data is not perturbation-response data (e.g., plain cross-modality scRNA/scATAC
  translation), where the correct evaluation is translation accuracy rather than DEG
  overlap
- Paired ground-truth stimulated profiles are unavailable for the held-out cell types,
  so no DEG overlap or R2 against truth can be computed
- Input counts are raw and unnormalized — RNA_data_preprocessing (QC and normalization)
  must be applied before differential expression
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scButterfly
  role: Generates predicted stimulated-expression profiles via the perturbation model
    (Model.test) whose DEG profiles are evaluated against ground truth
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python
  role: Implementation language for DEG computation, Pearson correlation, and CSV
    export (Python 3.9 conda environment)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:7b5c3621801b3f9bf1f0dd981cedcffbf36f60a27fe77f74f365841102511976
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - scButterfly
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# differential-gene-expression-analysis

## Summary

Evaluates predicted single-cell perturbation responses by computing per-cell-type differential expression against ground truth, using top-100 DEG overlap counts and squared Pearson correlation (R2) as metrics. This is the validation scheme used to benchmark scButterfly-B predictions on the PT_PBMC control/stimulated PBMC data against scGen and scPreGAN baselines.

## When to use

Use when validating predicted perturbation-response transcriptomes (e.g., scButterfly-B output on PT_PBMC) by comparing per-cell-type DEG profiles to ground truth with top-100 DEG overlap and R2 metrics.

## When NOT to use

- Data is not perturbation-response data (e.g., plain cross-modality scRNA/scATAC translation), where the correct evaluation is translation accuracy rather than DEG overlap
- Paired ground-truth stimulated profiles are unavailable for the held-out cell types, so no DEG overlap or R2 against truth can be computed
- Input counts are raw and unnormalized — RNA_data_preprocessing (QC and normalization) must be applied before differential expression

## Inputs

- Predicted stimulated scRNA-seq expression matrix (AnnData, n_obs x n_vars) from scButterfly Model.test for held-out cell types
- Ground-truth stimulated scRNA-seq expression matrix for the same cell types from the PT_PBMC dataset
- Cell-type labels in adata.obs (used to stratify the DEG computation per cell type)
- Baseline metrics for scGen and scPreGAN from Fig. 4h for comparison

## Outputs

- Per-cell-type counts of common DEGs among the top 100 predicted vs. ground-truth DEGs
- Per-cell-type squared Pearson correlation (R2) values for the top 100 real DEGs
- perturbation_benchmark_results.csv with DEG-overlap counts and R2 values per cell type and method

## How to apply

After predicting stimulated-expression profiles for held-out cell types (using scButterfly's Model.test on the perturbation model trained with scButterfly-T augmentation), compute differential expression separately for predictions and ground truth within each cell type. For each cell type, extract the top 100 DEGs from each profile, count how many DEGs are common between predicted and ground-truth lists, and compute the squared Pearson correlation (R2) of expression values over those genes. The mean squared Pearson correlation for the top 100 real DEGs must consistently exceed 0.85 for each cell type, and the common top-100 DEG counts should match or exceed scGen and scPreGAN baselines (Fig. 4h). Save the per-cell-type, per-method DEG-overlap counts and R2 values to perturbation_benchmark_results.csv.

## Related tools

- **scButterfly** (Generates predicted stimulated-expression profiles via the perturbation model (Model.test) whose DEG profiles are evaluated against ground truth) — https://github.com/BioX-NKU/scButterfly
- **Python** (Implementation language for DEG computation, Pearson correlation, and CSV export (Python 3.9 conda environment))

## Examples

```
from scButterfly.train_model_perturb import Model; A2R_predict = Model.test(model, RNA_data_stim, cell_type, test_id)
```

## Evaluation signals

- Squared Pearson correlation (R2) for the top 100 real DEGs must exceed 0.85 for each cell type
- Common top-100 DEG counts should match or exceed those of scGen and scPreGAN baselines as reported in Fig. 4h
- Metrics are stratified per cell type; every held-out cell type must pass the R2 threshold, not just the mean
- Results table perturbation_benchmark_results.csv must contain DEG-overlap counts and R2 values per cell type and method

## Limitations

- Held-out (out-of-sample) cell types require the per-cell-type optimal transport matching of scButterfly-T augmentation; without cell-type labels it falls back to MultiVI_augmentation
- Validation uses only the top 100 DEGs per cell type, so performance on lower-ranked genes is not captured
- Performance for cross-cell-type and cross-organ translation is flagged in the discussion as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters

## Evidence

- [results] R2 threshold: [evidence span withheld — anchor status: not_found]
- [methods] perturbation workflow: "Perturbation responses prediction using scButterfly-T"
- [methods] preprocessing step: "data_processing.RNA_data_preprocessing"
- [abstract] generalization claim: "scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation"
- [readme] augmentation API: [evidence span withheld — anchor status: external_doc]
