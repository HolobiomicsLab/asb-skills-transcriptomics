---
name: optimal-transport-cell-matching
description: Use when control and perturbed single-cell RNA-seq profiles are unpaired
  (no per-cell ground-truth pairing) and a translator must be trained to predict stimulated-expression
  profiles, especially with held-out cell types.
when_to_use_negative:
- Control and perturbed cells are already paired per cell, in which case augmentation
  matching is unnecessary overhead and basic training may suffice.
- The target data lacks cell_type annotations and cannot be confidently clustered,
  making cell_type_augmentation inapplicable (MultiVI_augmentation or scButterfly-B
  would be required instead).
- The task is standard paired scRNA-seq to scATAC-seq/scADT-seq cross-modality translation
  rather than unpaired perturbation-response prediction.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scButterfly
  role: Provides the Butterfly.augmentation API with cell_type_augmentation (scButterfly-T)
    that performs per-cell-type optimal transport matching of unpaired cells into
    paired training samples
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment (conda create scButterfly python==3.9) in which the scButterfly
    augmentation and perturbation model are executed
  repo: https://github.com/BioX-NKU/scButterfly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:3b35707ddb7a76de248061dde1758e1563b0cf9a1d3585260d26bf65ff47ef08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# optimal-transport-cell-matching

## Summary

Applies scButterfly's data augmentation with per-cell-type optimal transport matching (scButterfly-T) to construct paired training samples from unpaired single-cell datasets, enabling perturbation-response prediction on data such as the PT_PBMC control/stimulated PBMC cohorts.

## When to use

Use when control and perturbed single-cell RNA-seq profiles are unpaired (no per-cell ground-truth pairing) and a translator must be trained to predict stimulated-expression profiles, especially with held-out cell types.

## When NOT to use

- Control and perturbed cells are already paired per cell, in which case augmentation matching is unnecessary overhead and basic training may suffice.
- The target data lacks cell_type annotations and cannot be confidently clustered, making cell_type_augmentation inapplicable (MultiVI_augmentation or scButterfly-B would be required instead).
- The task is standard paired scRNA-seq to scATAC-seq/scADT-seq cross-modality translation rather than unpaired perturbation-response prediction.

## Inputs

- Unpaired AnnData count matrices of control and stimulated PBMC cells (PT_PBMC), rows = cells, columns = genes, with cell_type labels in adata.obs
- train_id / test_id / validation_id cell-ID lists used by Butterfly.load_data

## Outputs

- Paired control/stimulated training samples constructed by per-cell-type optimal transport matching
- Predicted stimulated-expression profiles for held-out cell types
- Per-cell-type top-100 DEG overlap counts and squared Pearson correlation (R2) values, e.g., saved to perturbation_benchmark_results.csv

## How to apply

After loading the unpaired scRNA-seq control and stimulated AnnData matrices and preprocessing counts (e.g., data_processing.RNA_data_preprocessing for QC and normalization), run Butterfly.augmentation with aug_type='cell_type_augmentation' so scButterfly-T matches control and perturbed cells within each cell type via optimal transport, yielding synthetic paired training samples. Choose scButterfly-B (aug_type=None) only when cell-type labels are absent and you accept basic training; scButterfly-C (MultiVI_augmentation) is used when cell_type is missing from adata.obs and Leiden/MultiVI cluster labels must be derived first. Train the perturbation model (scButterfly.train_model_perturb.Model) while holding out cell types for out-of-sample evaluation, then predict stimulated profiles with Model.test and compute per-cell-type DEG overlap (top 100) and squared Pearson correlation against ground truth. The augmentation increases training time but is credited with better prediction results, so prefer it over basic training for unpaired perturbation data.

## Related tools

- **scButterfly** (Provides the Butterfly.augmentation API with cell_type_augmentation (scButterfly-T) that performs per-cell-type optimal transport matching of unpaired cells into paired training samples) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment (conda create scButterfly python==3.9) in which the scButterfly augmentation and perturbation model are executed) — https://github.com/BioX-NKU/scButterfly

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(ctrl_adata, stim_adata, train_id, test_id); butterfly.data_preprocessing(); butterfly.augmentation('cell_type_augmentation')
```

## Evaluation signals

- Squared Pearson correlation (R2) for the top 100 real DEGs exceeds 0.85 for each held-out cell type.
- Common top-100 DEG counts per cell type match or exceed scGen and scPreGAN baselines as reported in Fig. 4h.
- Augmentation was requested (aug_type='cell_type_augmentation') and cell_type was found in adata.obs; otherwise scButterfly-T silently falls back to MultiVI_augmentation, which must be checked.
- DEG-overlap counts and R2 values per cell type and method are present in perturbation_benchmark_results.csv with no missing cell types.

## Limitations

- Augmentation increases training time compared with basic (scButterfly-B) training, trading runtime for prediction accuracy.
- If cell_type labels are missing, cell_type_augmentation automatically transfers to MultiVI_augmentation, changing the pairing basis to Leiden/MultiVI cluster labels.
- Performance for cross-cell-type and cross-organ translation is flagged in the discussion as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters.

## Evidence

- [readme] If you choose `cell_type_augmentation`, scButterfly-T (Type) will try to find `cell_type` in `adata.obs`. If failed, it will automaticly transfer to `MultiVI_augmentation`.: [evidence span withheld — anchor status: external_doc]
- [abstract] scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation.: "scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation."
- [intro] paired multi-modal single-cell profiles may be not available for model training, requiring diagonal strategies to learn a competent translator based on unpaired data: "paired multi-modal single-cell profiles may be not available for model training, requiring diagonal strategies to learn a competent translator based on unpaired data"
- [full_text] the mean squared Pearson correlation (R2) for the top 100 real DEGs consistently surpassing 0.85 for each cell type: [evidence span withheld — anchor status: not_found]
- [methods] Perturbation responses prediction using scButterfly-T: "Perturbation responses prediction using scButterfly-T"
