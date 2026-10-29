---
name: perturbation-response-prediction
description: Use when you need to predict stimulated/perturbed single-cell expression
  profiles from unpaired control scRNA-seq data (e.g., PT_PBMC) and evaluate predictions
  per cell type against ground-truth DEGs.
when_to_use_negative:
- When paired control/stimulated single-cell profiles are already available for training
  — the unpaired diagonal augmentation strategy is unnecessary
- When the task is cross-modality translation (scRNA-seq ↔ scATAC-seq or scADT-seq)
  rather than perturbation-response prediction
- When novel cell types absent from training cannot be held out, making out-of-sample
  per-cell-type evaluation impossible
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scButterfly
  role: 'Core framework: Butterfly.load_data, data_processing.RNA_data_preprocessing,
    augmentation (scButterfly-T cell-type optimal transport), train_model_perturb.Model
    construction/training, and Model.test prediction for perturbation responses'
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python
  role: Execution environment (Python 3.9 via conda) for running the scButterfly pipeline
    and computing DEG-overlap and R2 metrics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:0d14b148a2c56019fef4ab733571119a500d8c00be825c387a129c7a0cd64e69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - scButterfly
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# perturbation-response-prediction

## Summary

Predicts single-cell perturbation responses (e.g., stimulated vs control PBMC transcriptomes) from unpaired control and stimulated scRNA-seq data using scButterfly-T's dual-aligned VAEs with per-cell-type optimal transport augmentation, benchmarked against scGen and scPreGAN.

## When to use

Use when you need to predict stimulated/perturbed single-cell expression profiles from unpaired control scRNA-seq data (e.g., PT_PBMC) and evaluate predictions per cell type against ground-truth DEGs.

## When NOT to use

- When paired control/stimulated single-cell profiles are already available for training — the unpaired diagonal augmentation strategy is unnecessary
- When the task is cross-modality translation (scRNA-seq ↔ scATAC-seq or scADT-seq) rather than perturbation-response prediction
- When novel cell types absent from training cannot be held out, making out-of-sample per-cell-type evaluation impossible

## Inputs

- Unpaired control and stimulated scRNA-seq count matrices (AnnData objects, raw counts, e.g., PT_PBMC dataset)
- Cell-type annotations in adata.obs ('cell_type') for scButterfly-T augmentation
- Train/test/validation cell ID lists for held-out cell-type evaluation

## Outputs

- Predicted stimulated-expression profiles for held-out cell types (AnnData / count matrix)
- Per-cell-type DEG-overlap counts (common top-100 DEGs) and squared Pearson correlation R2 values
- perturbation_benchmark_results.csv with DEG-overlap counts and R2 values per cell type and method (scButterfly-B, scGen, scPreGAN)

## How to apply

Load the raw count matrices of control and stimulated scRNA-seq data with Butterfly.load_data, then preprocess RNA counts using data_processing.RNA_data_preprocessing for quality control and normalization. Apply the scButterfly-T cell-type augmentation (butterfly.augmentation with cell_type_augmentation), which uses per-cell-type optimal transport matching to construct paired control/stimulated training samples from unpaired data. Construct and train the perturbation model (scButterfly.train_model_perturb.Model) while holding out cell types for out-of-sample evaluation. Predict stimulated profiles for held-out cell types with Model.test, then compute per-cell-type differential expression metrics: the number of common DEGs among the top 100 and the squared Pearson correlation (R2) against ground truth. Validate that R2 for the top 100 real DEGs exceeds 0.85 for each cell type and that common top-100 DEG counts match or exceed scGen and scPreGAN baselines (Fig. 4h).

## Related tools

- **scButterfly** (Core framework: Butterfly.load_data, data_processing.RNA_data_preprocessing, augmentation (scButterfly-T cell-type optimal transport), train_model_perturb.Model construction/training, and Model.test prediction for perturbation responses) — https://github.com/BioX-NKU/scButterfly
- **Python** (Execution environment (Python 3.9 via conda) for running the scButterfly pipeline and computing DEG-overlap and R2 metrics)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id); butterfly.data_preprocessing(); butterfly.augmentation('cell_type_augmentation')
```

## Evaluation signals

- Squared Pearson correlation (R2) for the top 100 real DEGs must consistently surpass 0.85 for each cell type
- Common top-100 DEG counts per cell type should match or exceed scGen and scPreGAN baselines as reported in Fig. 4h
- Predicted stimulated profiles must correlate well with ground truth across all cell types, including held-out (out-of-sample) cell types
- Results table perturbation_benchmark_results.csv must contain per-cell-type and per-method rows with both DEG-overlap counts and R2 values

## Limitations

- Data augmentation (scButterfly-T) increases training time compared to scButterfly-B basic training
- If 'cell_type' is not present in adata.obs, cell_type_augmentation automatically falls back to MultiVI/Leiden cluster-based augmentation (scButterfly-C)
- The paper flags performance for cross-cell-type and cross-organ translation as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters
- Incorporation of prior knowledge of gene regulatory mechanisms to bridge modalities is proposed only as a future improvement

## Evidence

- [abstract] scButterfly generalizes to unpaired data training, perturbation-response analysis, and consecutive translation.: "scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation."
- [methods] Perturbation workflow: "Perturbation responses prediction using scButterfly-T"
- [intro] Unpaired diagonal training rationale: "paired multi-modal single-cell profiles may be not available for model training, requiring diagonal strategies to learn a competent translator based on unpaired data"
- [results] R2 threshold result: [evidence span withheld — anchor status: not_found]
- [readme] Augmentation API: [evidence span withheld — anchor status: external_doc]
