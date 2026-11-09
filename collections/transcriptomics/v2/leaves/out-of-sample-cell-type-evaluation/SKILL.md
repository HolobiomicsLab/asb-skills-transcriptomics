---
name: out-of-sample-cell-type-evaluation
description: Use when a perturbation prediction model trained on unpaired control/stimulated
  scRNA-seq data must be tested on cell types absent from training (out-of-sample
  generalization), e.g., PT_PBMC response prediction with per-cell-type held-out evaluation.
when_to_use_negative:
- Paired (same-cell) control/stimulated measurements are available, making diagonal/unpaired
  training and augmentation unnecessary
- Only one cell type is present in the data, leaving no cell types to hold out for
  out-of-sample evaluation
- The task is cross-modality translation (scRNA-seq to scATAC-seq or scADT-seq) rather
  than perturbation-response prediction across biological conditions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scButterfly
  role: Dual-aligned variational autoencoder framework; scButterfly-T (cell-type augmentation
    with optimal transport matching) builds paired training samples from unpaired
    data and the train_model_perturb.Model predicts stimulated profiles for held-out
    cell types
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for the scButterfly pipeline and evaluation metrics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:da9dca2d332ddc0e24adc85af3bf5f039ed55cb7a70e37bc01406c9f4f29ec68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# out-of-sample-cell-type-evaluation

## Summary

Evaluates single-cell perturbation-response prediction by holding out entire cell types during training and scoring predicted versus ground-truth stimulated profiles per cell type, using top-100 DEG overlap and squared Pearson correlation (R2) on real DEGs, benchmarked against scGen and scPreGAN on the PT_PBMC dataset.

## When to use

Use when a perturbation prediction model trained on unpaired control/stimulated scRNA-seq data must be tested on cell types absent from training (out-of-sample generalization), e.g., PT_PBMC response prediction with per-cell-type held-out evaluation.

## When NOT to use

- Paired (same-cell) control/stimulated measurements are available, making diagonal/unpaired training and augmentation unnecessary
- Only one cell type is present in the data, leaving no cell types to hold out for out-of-sample evaluation
- The task is cross-modality translation (scRNA-seq to scATAC-seq or scADT-seq) rather than perturbation-response prediction across biological conditions

## Inputs

- Unpaired control and stimulated PBMC (PT_PBMC) scRNA-seq count matrices as AnnData objects with cell_type annotations in adata.obs
- Train/test/validation cell-ID splits with held-out cell types for out-of-sample evaluation

## Outputs

- Predicted stimulated-expression profiles for each held-out cell type
- Per-cell-type DEG-overlap counts (common genes among top 100 DEGs) and squared Pearson correlation (R2) against ground truth
- Comparison table against scGen and scPreGAN baselines (e.g., perturbation_benchmark_results.csv)

## How to apply

Load the PT_PBMC control and stimulated scRNA-seq data with Butterfly.load_data and preprocess counts using data_processing.RNA_data_preprocessing (QC and normalization). Apply the scButterfly-T cell-type augmentation with per-cell-type optimal transport matching (butterfly.augmentation with 'cell_type_augmentation') to construct paired control/stimulated training samples from unpaired data. Train the perturbation model (scButterfly.train_model_perturb.Model) while deliberately holding out specific cell types so they never appear in training. Predict stimulated-expression profiles for held-out cell types via Model.test, then compute per cell type the number of shared genes among the top 100 DEGs and the squared Pearson correlation (R2) of predicted expression on the top 100 real DEGs. Compare against scGen and scPreGAN baselines as reported in Fig. 4h, and save per-cell-type DEG-overlap counts and R2 values to a CSV (e.g., perturbation_benchmark_results.csv).

## Related tools

- **scButterfly** (Dual-aligned variational autoencoder framework; scButterfly-T (cell-type augmentation with optimal transport matching) builds paired training samples from unpaired data and the train_model_perturb.Model predicts stimulated profiles for held-out cell types) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for the scButterfly pipeline and evaluation metrics)

## Examples

```
from scButterfly.butterfly import Butterfly; b = Butterfly(); b.load_data(ctrl_adata, stim_adata, train_id, test_id, validation_id); b.augmentation('cell_type_augmentation')
```

## Evaluation signals

- Squared Pearson correlation (R2) on the top 100 real DEGs must exceed 0.85 for every held-out cell type
- Common top-100 DEG counts per cell type should match or exceed the scGen and scPreGAN baselines reported in Fig. 4h
- Held-out cell types are verified absent from the training split before model fitting
- Per-cell-type R2 and DEG-overlap values are recorded for each method in perturbation_benchmark_results.csv
- Predicted profiles reproduce cell-type-specific response structure when compared to ground-truth stimulated profiles

## Limitations

- scButterfly requires cell-type labels for scButterfly-T augmentation; if 'cell_type' is missing from adata.obs it automatically falls back to MultiVI/cluster-based augmentation (scButterfly-C)
- Cross-cell-type and cross-organ translation performance is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters
- Cell heterogeneity information from public bulk omics data or annotated cell atlases is proposed as a future improvement and is not currently implemented

## Evidence

- [abstract] Perturbation response analysis is an explicit scButterfly application: "scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation."
- [methods] Perturbation responses prediction is performed with the scButterfly-T variant: "Perturbation responses prediction using scButterfly-T"
- [readme] scButterfly-T relies on cell-type labels found in adata.obs for augmentation: [evidence span withheld — anchor status: external_doc]
- [methods] RNA counts are preprocessed with the dedicated RNA preprocessing routine: "data_processing.RNA_data_preprocessing"
- [discussion] Cross-cell-type translation performance is a known weakness flagged for future work: "advanced machine learning techniques with adaptive and interpretable model parameters can be considered to explore novel avenues for boosting the performance of scButterfly"
