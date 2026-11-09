---
name: circular-manifold-autoencoder-modeling
description: Use when scRNA-seq (or qPCR) expression matrices should be characterized
  for a periodic/circular latent process (e.g., cell-cycle staging with Hoechst-validated
  labels), especially where linear methods (PCA, NMF, ICA) or marker-gene methods
  (Cyclone, reCAT) underperform.
when_to_use_negative:
- The goal is linear trajectory or standard pseudo-time inference (e.g., developmental
  progression) where a circular manifold is inappropriate; Monocle or Wanderlust-style
  methods fit better
- The analysis only needs marker-gene-based cell-cycle scoring on already-annotated
  gene sets without pseudo-time
- Data cannot be meaningfully log2(x+1)-transformed, centered and scaled, e.g., non-count
  expression scales the Cyclum transform assumes
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Sinusoidal autoencoder that projects gene expression onto a circular manifold
    and infers per-cell pseudo-time; includes CyclumAutoTune for dimensionality and
    refine_labels for post-processing
  repo: github.com/KChen-lab/cyclum
- name: reCAT
  role: Model-free baseline (Gaussian mixture model pseudo-time) used for accuracy
    comparison against Cyclum's model-based circular manifold
- name: Cyclone
  role: Baseline method using PCA and relative expression of gene pairs to predict
    cell-cycle phases, compared for classification accuracy
- name: Oscope
  role: Baseline that checks gene pairs to identify circular patterns, compared against
    Cyclum
- name: f-scLVM (Slalom)
  role: Baseline latent factor model with known gene sets used for cell-cycle classification
    comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:c643194256fd0c910aaf9b47a5e5bbf050699c4e5a93f1347ba9463d899b0f4f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclone
  - Cyclum
  - Oscope
  - f-scLVM (Slalom)
  - reCAT
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# circular-manifold-autoencoder-modeling

## Summary

Fits a sinusoidal autoencoder (Cyclum) to scRNA-seq expression data to project single cells onto a circular manifold, inferring ab initio pseudo-time for periodic processes such as the cell cycle without requiring marker genes.

## When to use

Use when scRNA-seq (or qPCR) expression matrices should be characterized for a periodic/circular latent process (e.g., cell-cycle staging with Hoechst-validated labels), especially where linear methods (PCA, NMF, ICA) or marker-gene methods (Cyclone, reCAT) underperform.

## When NOT to use

- The goal is linear trajectory or standard pseudo-time inference (e.g., developmental progression) where a circular manifold is inappropriate; Monocle or Wanderlust-style methods fit better
- The analysis only needs marker-gene-based cell-cycle scoring on already-annotated gene sets without pseudo-time
- Data cannot be meaningfully log2(x+1)-transformed, centered and scaled, e.g., non-count expression scales the Cyclum transform assumes

## Inputs

- scRNA-seq or single-cell qPCR gene expression matrix (cells x genes), e.g. mESC, PC3, MDA-MB-231 or H9 data
- Optionally, externally derived cell-cycle labels (e.g., from Seurat::CellCycleScoring or scanpy.tl.score_genes_cell_cycle) for refinement

## Outputs

- Per-cell circular pseudo-time (stage) values on the inferred circular manifold
- Optimal number of linear dimensions selected by CyclumAutoTune
- Refined cell-cycle phase labels (G1/S/G2M) when using refine_labels
- Genes associated with the periodic process, scored by match between expression kinetics and inferred periodicity

## How to apply

Preprocess the expression matrix with the recommended transform: log2(x+1), then center (mean = 1) and scale (variance = 1), as advised in the Cyclum documentation. Use cyclum.tuning.CyclumAutoTune to select the number of linear dimensions so the autoencoder finds the 'most circular' manifold, then fit the sinusoidal autoencoder to project high-dimensional gene expression onto the circular manifold and infer one pseudo-time value per cell. Because Cyclum is model-based (fits data to predefined circular manifolds), it is more robust to random noise and small sample sizes than model-free approaches like reCAT's Gaussian mixture model. Optionally refine external labels (e.g., from scanpy.tl.score_genes_cell_cycle or Seurat::CellCycleScoring) along the inferred pseudotime using cyclum.postproc.refine_labels. Validate by comparing per-cell phase classifications against Hoechst-based labels; on mESC, PC3, MDA-MB-231 and H9 datasets Cyclum should outperform Slalom, Cyclone, reCAT, Oscope, Cyclops and PCA.

## Related tools

- **Cyclum** (Sinusoidal autoencoder that projects gene expression onto a circular manifold and infers per-cell pseudo-time; includes CyclumAutoTune for dimensionality and refine_labels for post-processing) — github.com/KChen-lab/cyclum
- **reCAT** (Model-free baseline (Gaussian mixture model pseudo-time) used for accuracy comparison against Cyclum's model-based circular manifold)
- **Cyclone** (Baseline method using PCA and relative expression of gene pairs to predict cell-cycle phases, compared for classification accuracy)
- **Oscope** (Baseline that checks gene pairs to identify circular patterns, compared against Cyclum)
- **f-scLVM (Slalom)** (Baseline latent factor model with known gene sets used for cell-cycle classification comparison)

## Examples

```
from cyclum.tuning import CyclumAutoTune; tunner = CyclumAutoTune(expression, linear_dims=[3,4,5,6], show_history=True); model = tunner.fit()
```

## Evaluation signals

- All per-method classification accuracies fall in [0, 1] across the four Hoechst-labeled datasets (mESC, PC3, MDA-MB-231, H9)
- Cyclum's accuracy exceeds every baseline's accuracy (Slalom, Cyclone, reCAT, Oscope, Cyclops, PCA, NMF, ICA) on each of the four datasets
- CyclumAutoTune returns a number of linear dimensions whose manifold is judged 'most circular'; pseudo-time values are cyclically consistent around the inferred circle
- refine_labels output is phase-ordered along pseudotime (e.g., ['G1','G1','G1','G1','G1','S','G2M','G2M'] from interleaved input labels)
- Runtime sanity check: a 12,280-cell x 33,694-gene dataset completes in ~40 min on a laptop with GTX 960M GPU

## Limitations

- Requires TensorFlow 1.x for the tested configuration (tested with python 3.7.4, keras 2.2.4, tensorflow 1.14.0); newer dependency versions differ from what the program was tested on
- Runtime and hardware claims are detailed only in Supplementary Note 2, given informally as a laptop with GTX 960M GPU and 2 GB graphic memory
- No explicit handling described for datasets whose latent process is non-periodic or only partially cyclic

## Evidence

- [methods] scaled (variance = 1), centered (mean = 1) log-transformed (log2(x+1)) data work well.: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [abstract] we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space: "we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space"
- [discussion] It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles.: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
- [readme] We also implemented class cyclum.tuning.CyclumAutoTune, which automatically select the proper number of linear components: [evidence span withheld — anchor status: external_doc]
- [readme] The methodology is to rely on the circular manifold, instead of the marker genes.: [evidence span withheld — anchor status: external_doc]
