---
name: matrix-trifactorization-cell-factor-training
description: Use when integrating multiple single-cell batches with partially missing
  modalities (mosaic scenario, e.g. scRNA-seq/scATAC-seq/protein with None placeholders)
  and you need a joint low-dimensional cell representation before clustering.
when_to_use_negative:
- When the task is horizontal/diagonal/vertical integration already handled by LIGER,
  Seurat v3/v4, CoupleNMF, MMD-MA, scDART, scAI, or MultiVI rather than a mosaic scenario
  with missing modalities
- When only a single batch/modality is available and no mosaic integration is needed
- When feature sets cannot be matched across batches (no gene intersection and no
  peak remapping possible), since each modality must share the same features per modality
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMoMaT
  role: Implements the matrix tri-factorization model (scmomat.scmomat_model) whose
    first training stage learns the cell factors
  repo: github.com/PeterZZQ/scMoMaT
- name: UMAP
  role: Downstream visualization of the post-processed cell-factor graph (scmomat.UMAP
    with min_dist=0.20, metric='precomputed')
- name: LIGER
  role: Alternative method for horizontal integration, contrasted with scMoMaT's mosaic
    tri-factorization
  repo: welch-lab/liger
- name: MultiMAP
  role: Alternative multi-omics integration method listed among tools compared against
    scMoMaT
  repo: Teichlab/MultiMAP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:076a0d3cf341d8a599090b268f135e1cf06cf873227d90d69ce3acb654eee52c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LIGER
  - MultiMAP
  - UMAP
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# matrix-trifactorization-cell-factor-training

## Summary

First-stage training of the scMoMaT matrix tri-factorization model to learn unified cell factors from mosaic single-cell multi-omics counts (e.g. the MOp_5batches dataset), producing a cell embedding for downstream KNN/Leiden clustering and marker detection.

## When to use

Use when integrating multiple single-cell batches with partially missing modalities (mosaic scenario, e.g. scRNA-seq/scATAC-seq/protein with None placeholders) and you need a joint low-dimensional cell representation before clustering.

## When NOT to use

- When the task is horizontal/diagonal/vertical integration already handled by LIGER, Seurat v3/v4, CoupleNMF, MMD-MA, scDART, scAI, or MultiVI rather than a mosaic scenario with missing modalities
- When only a single batch/modality is available and no mosaic integration is needed
- When feature sets cannot be matched across batches (no gene intersection and no peak remapping possible), since each modality must share the same features per modality

## Inputs

- counts dictionary with nbatches, feats_name, and per-modality ordered lists of (ncells, nfeatures) matrices with None placeholders for missing modality-batch pairs
- Feature-matched data matrices (gene intersection or remapped chromatin regions/peaks) preprocessed with utils.preprocess()
- Optional pseudo-count matrices derived from a connection matrix GxR.npz (e.g. pseudo-scRNA-seq inputs)

## Outputs

- Cell factors zs = model.extract_cell_factors() — a unified low-dimensional cell embedding across all batches (K=30 dimensions)
- Training loss curve (losses) confirming convergence over T=4000 iterations
- A trained scmomat model object passed to scmomat.scmomat_retrain for the second (marker-detection) stage

## How to apply

Assemble the data into the scMoMaT `counts` dictionary with keys `nbatches`, `feats_name`, and one ordered list per modality (insert None for missing modality-batch pairs), ensuring features are matched across batches (gene intersection or peak remapping) and each matrix is preprocessed with `utils.preprocess()`. Instantiate the model with `scmomat.scmomat_model(counts, K=30, device)` — K is the number of latent dimensions, recommended 20–30 — and train for T=4000 iterations via `model.train_func(T=T)` leaving lambda at its default 0.001. Monitor that training losses decrease over iterations; then extract the unified cell embedding with `model.extract_cell_factors()`. The cell factors are the output of tri-factorization on shared feature factors and batch factors, which lets scMoMaT integrate batches with unequal cell type compositions. Pseudo-count inputs, such as pseudo-scRNA-seq matrices generated via the `GxR.npz` connection matrix, can be included as additional modality entries.

## Related tools

- **scMoMaT** (Implements the matrix tri-factorization model (scmomat.scmomat_model) whose first training stage learns the cell factors) — github.com/PeterZZQ/scMoMaT
- **UMAP** (Downstream visualization of the post-processed cell-factor graph (scmomat.UMAP with min_dist=0.20, metric='precomputed'))
- **LIGER** (Alternative method for horizontal integration, contrasted with scMoMaT's mosaic tri-factorization) — welch-lab/liger
- **MultiMAP** (Alternative multi-omics integration method listed among tools compared against scMoMaT) — Teichlab/MultiMAP

## Examples

```
import scmomat; model = scmomat.scmomat_model(counts=counts, K=30, device=device); losses = model.train_func(T=4000); zs = model.extract_cell_factors()
```

## Evaluation signals

- Training losses decrease monotonically (or near-monotonically) over the T=4000 iterations, indicating convergence
- Extracted cell factors zs have one embedding per cell across all batches, with length of each modality list equal to nbatches
- Post-processed KNN graph (scmomat.post_process with n_neighbors=30) plus Leiden clustering (resolution=0.9) yields transcriptionally coherent clusters, e.g. cortical populations in MOp_5batches
- UMAP embedding of s_pair_dist (min_dist=0.20, metric='precomputed') shows batch-mixed, cell-type-separated structure
- On MOp_5batches, the downstream retrained marker scores reproduce Fig. 4e annotations (Rorb -> L4/5, Tshz2 -> near-projecting, Calb1 -> L2/3 excitatory neurons), validating the first-stage cell factors

## Limitations

- Runtime: the MOp_5batches training takes ~5 mins on an NVIDIA A40 GPU (~30 mins total for the whole workflow); CPU-only runs will be slower
- All matrices in a modality must share an identical feature set; mismatched features require prior intersection or peak remapping
- Cross-modality relationships via feature representations are not learned in this framework and are explicitly deferred to future work
- Post-processing can hit memory overflow issues; scMoMaT v0.2.2 updated the post-processing step specifically to solve this

## Evidence

- [methods] First training stage: learn cell factors with scmomat model: [evidence span withheld — anchor status: not_found]
- [methods] Extract cell factors: "zs = model.extract_cell_factors()"
- [readme] Counts dictionary structure: "`nbatches`: total number of batches
    * `feats_name`: a dictionary storing the feature names for different modalities"
- [readme] Feature matching requirement: "The data matrix in each modality (each list) should have the same set of features. You can do thie by 1. using the intersection of all genes/proteins in different batches"
- [abstract] Mosaic integration via matrix tri-factorization (abstract claim): "scMoMaT, a method that is able to integrate single cell multi-omics data under the mosaic integration scenario using matrix tri-factorization"
- [readme] Pseudo-count generation from connection matrix: "The connection matrix (`GxR.npz`) that is used to generate pseudo-count is also provided in advance"
