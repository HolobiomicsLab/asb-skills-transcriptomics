---
name: unpaired-data-augmentation
description: Use when training scButterfly on unpaired samples — e.g., unpaired control
  and stimulated PT_PBMC scRNA-seq for perturbation-response prediction — and cell-type
  labels are available in adata.obs to enable scButterfly-T pairing.
when_to_use_negative:
- Input scRNA-seq and scATAC-seq (or control/stimulated) profiles are already genuinely
  paired cell-by-cell — the scButterfly-B basic model with aug_type=None is the appropriate
  choice
- No cell-type labels are available and training an additional MultiVI model for cluster
  labels is not feasible or not desired
- The task is plain cross-modality translation of paired multi-omics data rather than
  diagonal training on unpaired samples or perturbation-response prediction
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scButterfly
  role: Provides the augmentation API (butterfly.augmentation) implementing scButterfly-T
    cell-type-based pairing for unpaired data and scButterfly-C MultiVI-based clustering
    fallback
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for the scButterfly conda environment in which augmentation
    is executed
  repo: https://github.com/BioX-NKU/scButterfly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:2dcae3cf137bf1bf82017e142f850daf6fa5a4ce3f234f663a1d0d18178a9222
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# unpaired-data-augmentation

## Summary

scButterfly's data augmentation scheme constructs synthetic paired training samples from unpaired single-cell datasets (e.g., control vs. stimulated PBMC in the PT_PBMC perturbation setting) by matching cells within cell-type labels (scButterfly-T) or Leiden/MultiVI clusters (scButterfly-C), enabling diagonal training when no paired multi-modal profiles exist.

## When to use

Use when training scButterfly on unpaired samples — e.g., unpaired control and stimulated PT_PBMC scRNA-seq for perturbation-response prediction — and cell-type labels are available in adata.obs to enable scButterfly-T pairing.

## When NOT to use

- Input scRNA-seq and scATAC-seq (or control/stimulated) profiles are already genuinely paired cell-by-cell — the scButterfly-B basic model with aug_type=None is the appropriate choice
- No cell-type labels are available and training an additional MultiVI model for cluster labels is not feasible or not desired
- The task is plain cross-modality translation of paired multi-omics data rather than diagonal training on unpaired samples or perturbation-response prediction

## Inputs

- AnnData raw count matrix of scRNA-seq control cells (n_obs × n_vars, rows=cells, columns=genes), e.g., unpaired PT_PBMC control samples
- AnnData raw count matrix of scRNA-seq stimulated/perturbed cells from the same features but unpaired cells
- adata.obs cell-type labels ('cell_type' column) for cell_type_augmentation, or Leiden/MultiVI cluster labels as fallback
- Train/test/validation cell ID lists as passed to butterfly.load_data

## Outputs

- Synthetic paired control/stimulated training samples constructed by per-cell-type matching, consumed by butterfly.construct_model and butterfly.train_model
- Trained scButterfly-T perturbation model enabling Model.test prediction of stimulated profiles for held-out cell types

## How to apply

After loading raw count matrices via butterfly.load_data and preprocessing (e.g., data_processing.RNA_data_preprocessing for scRNA-seq QC and normalization), call butterfly.augmentation(aug_type) before construct_model. Choose 'cell_type_augmentation' (scButterfly-T) when a 'cell_type' column exists in adata.obs; scButterfly-T performs per-cell-type optimal-transport matching to build paired control/stimulated training samples from the unpaired data, which supports perturbation-response prediction with held-out cell types. If cell-type labels are absent, scButterfly-T automatically falls back to 'MultiVI_augmentation' (scButterfly-C), which trains a MultiVI model to obtain Leiden-based cluster labels. Set aug_type=None only for paired data, where the scButterfly-B basic model suffices. Note augmentation increases training time but yields better prediction results, which is why it is recommended for the unpaired perturbation benchmark against scGen and scPreGAN.

## Related tools

- **scButterfly** (Provides the augmentation API (butterfly.augmentation) implementing scButterfly-T cell-type-based pairing for unpaired data and scButterfly-C MultiVI-based clustering fallback) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for the scButterfly conda environment in which augmentation is executed) — https://github.com/BioX-NKU/scButterfly

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.augmentation('cell_type_augmentation')
```

## Evaluation signals

- For perturbation-response benchmarking on PT_PBMC, mean squared Pearson correlation (R2) for the top 100 real DEGs must exceed 0.85 for each held-out cell type
- Common top-100 DEG counts per cell type should match or exceed scGen and scPreGAN baselines (Fig. 4h comparison), saved to perturbation_benchmark_results.csv
- Confirm scButterfly-T actually engaged: 'cell_type' was found in adata.obs; otherwise verify the automatic switch to MultiVI_augmentation in logs
- Augmentation should improve prediction versus aug_type=None on the same unpaired split — the paper states augmentation promises better results at the cost of more training time

## Limitations

- Augmentation consumes additional training time compared to basic (scButterfly-B) training
- If 'cell_type' is absent from adata.obs, cell_type_augmentation automatically transfers to MultiVI_augmentation, adding a MultiVI training dependency (scvi-tools)
- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters
- Cell heterogeneity information from public bulk omics data or annotated cell atlases is not currently incorporated — noted as future work

## Evidence

- [readme] If you choose `cell_type_augmentation`, scButterfly-T (Type) will try to find `cell_type` in `adata.obs`.: [evidence span withheld — anchor status: external_doc]
- [intro] paired multi-modal single-cell profiles may be not available for model training, requiring diagonal strategies to learn a competent translator based on unpaired data: "paired multi-modal single-cell profiles may be not available for model training, requiring diagonal strategies to learn a competent translator based on unpaired data"
- [intro] an effective data augmentation scheme may further improve the translation performance, alleviate the issues of limited number of cells measured by multi-omics protocols: "an effective data augmentation scheme may further improve the translation performance, alleviate the issues of limited number of cells measured by multi-omics protocols"
- [methods] Perturbation responses prediction using scButterfly-T: "Perturbation responses prediction using scButterfly-T"
- [readme] scButterfly provide data augmentation API: [evidence span withheld — anchor status: external_doc]
