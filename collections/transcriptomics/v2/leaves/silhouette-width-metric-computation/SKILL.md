---
name: silhouette-width-metric-computation
description: Use when evaluating single-cell integration embeddings (e.g. stored in
  adata.obsm['delta']) for whether cell-type structure is preserved (ASW) and whether
  batches are over- or under-corrected (ASW-beta).
when_to_use_negative:
- Input is raw counts without cell-type labels — silhouette width requires known cell_types
  annotations to be meaningful
- Evaluating methods on a single batch dataset — ASW-beta batch-mixing assessment
  is undefined without multiple batch_indices
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Source of the evaluate workflow that computes ARI, kBET, ASW and ASW-beta
    on cell embeddings (obsm['delta'])
  repo: github.com/hui2000ji/scETM
- name: Harmony
  role: Competing integration method whose embeddings are scored with ASW/ASW-beta
    in the benchmark
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Competing integration method whose embeddings are scored with ASW/ASW-beta
    in the benchmark
  repo: https://github.com/brianhie/scanorama
- name: scVI
  role: Competing integration method whose embeddings are scored with ASW/ASW-beta
    in the benchmark
  repo: https://github.com/YosefLab/scvi-tools
- name: LIGER
  role: Competing integration method; over-corrects batch effects, compromising ASW-beta
    on the beta-cell-removal benchmark
  repo: https://github.com/welch-lab/liger
- name: scVAE-GM
  role: Competing integration method whose embeddings are scored with ASW/ASW-beta
    in the benchmark
  repo: https://github.com/scvae/scvae
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:2b8e2d4144719b7b2b72881e34bb916fe047f0327e342f032e1612f2dfe1e586
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBKNN
  - Harmony
  - LIGER
  - Scanorama
  - Seurat
  - scANVI
  - scETM
  - scMerge2
  - scVAE-GM
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# silhouette-width-metric-computation

## Summary

Computes average silhouette width (ASW) on cell embeddings to quantify cell-type separation, and its batch-conditioned variant ASW-beta to quantify batch mixing, as done in the scETM single-cell integration benchmark (Human/Mouse Pancreas).

## When to use

Use when evaluating single-cell integration embeddings (e.g. stored in adata.obsm['delta']) for whether cell-type structure is preserved (ASW) and whether batches are over- or under-corrected (ASW-beta).

## When NOT to use

- Input is raw counts without cell-type labels — silhouette width requires known cell_types annotations to be meaningful
- Evaluating methods on a single batch dataset — ASW-beta batch-mixing assessment is undefined without multiple batch_indices

## Inputs

- AnnData object (cells-by-genes) with cell embeddings in obsm['delta'], batch indices in adata.obs['batch_indices'], and cell type labels in adata.obs['cell_types']
- Embeddings from competing integration methods (Harmony, Scanorama, Seurat, LIGER, scVI, scVAE-GM) computed on the same modified dataset

## Outputs

- Per-method metric values: ARI, kBET, ASW, ASW-beta on the cell embeddings
- Embedding plots saved by the evaluate workflow (e.g. figures/scETM_MP)
- Comparison table against Supplementary Table 5 reference values (benchmark_metrics.csv)

## How to apply

After training an integration method (scETM with UnsupervisedTrainer at n_epochs=12000, eval_every=1000, n_samplers=4, enable_batch_bias=True; or Harmony, Scanorama, Seurat, LIGER, scVI, scVAE-GM), extract the cell embeddings and compute ARI, kBET, ASW, and ASW-beta via the scETM `evaluate` workflow (evaluate(adata, embedding_key='delta')). ASW is computed on the embedding using cell_types labels and should be moderate-to-high when biological structure is preserved; ASW-beta is computed conditioned on batch_indices, where higher values indicate better batch integration but very high ASW-beta alongside degraded ASW signals over-correction (as seen for scETM+adv and LIGER on the beta-cell-removal Human Pancreas benchmark). Reference values from Supplementary Table 5 ground the interpretation: scETM achieved ARI 0.9298, kBET 0.1247, ASW 0.3525, ASW-beta 0.5370, balancing cell-type discrimination (ASW) against batch mixing (ASW-beta), while LIGER's aggressive correction yielded ARI 0.8476 and kBET 0.5978. Judge correctness by comparing computed metrics to the published table and checking the trade-off direction, not each metric in isolation.

## Related tools

- **scETM** (Source of the evaluate workflow that computes ARI, kBET, ASW and ASW-beta on cell embeddings (obsm['delta'])) — github.com/hui2000ji/scETM
- **Harmony** (Competing integration method whose embeddings are scored with ASW/ASW-beta in the benchmark) — https://github.com/immunogenomics/harmony
- **Scanorama** (Competing integration method whose embeddings are scored with ASW/ASW-beta in the benchmark) — https://github.com/brianhie/scanorama
- **scVI** (Competing integration method whose embeddings are scored with ASW/ASW-beta in the benchmark) — https://github.com/YosefLab/scvi-tools
- **LIGER** (Competing integration method; over-corrects batch effects, compromising ASW-beta on the beta-cell-removal benchmark) — https://github.com/welch-lab/liger
- **scVAE-GM** (Competing integration method whose embeddings are scored with ASW/ASW-beta in the benchmark) — https://github.com/scvae/scvae

## Examples

```
from scETM import scETM, UnsupervisedTrainer, evaluate; trainer = UnsupervisedTrainer(scETM(hp.n_vars, hp.obs.batch_indices.nunique(), enable_batch_bias=True), hp); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4); evaluate(hp, embedding_key='delta')
```

## Evaluation signals

- ASW for scETM on the beta-cell-removal Human Pancreas benchmark should reproduce 0.3525 and ASW-beta 0.5370 (Supplementary Table 5)
- ASW-beta should be high for well-batch-integrated methods, but a combined drop in ASW and ARI alongside very high batch mixing (e.g. LIGER, kBET 0.5978) indicates over-correction
- All four metrics (ARI, kBET, ASW, ASW-beta) must be computed on identical embeddings (same obsm['delta']) and the same beta-cell-removed dataset for cross-method comparability
- Computed values should be saved to benchmark_metrics.csv and matched against Supplementary Table 5 reference rows before acceptance

## Limitations

- ASW/ASW-beta must be interpreted jointly with ARI and kBET: aggressive correction methods (scETM+adv, LIGER) can score well on batch metrics while compromising cell-type structure (ASW-beta)
- Evaluation depends on correct labeling: scETM expects batch information in the 'batch_indices' column and cell types in 'cell_types' of adata.obs by default
- scETM's current model handles only a single categorical batch variable, so batch-conditioned metrics assume one categorical batch annotation

## Evidence

- [results] On the beta-cell-removal HP benchmark, scETM struck a good balance between discriminating cell types (ARI: 0.9298; ASW: 0.3525; ASW-beta: 0.5370) and integrating different batches (kBET: 0.1247): [evidence span withheld — anchor status: not_found]
- [readme] Unnormalized cell embeddings will be stored at mp.obsm['delta']: "Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta']"
- [readme] By default, scETM looks for batch information in the 'batch_indices' column of the adata.obs DataFrame, and cell type identity in the 'cell_types' column: "By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column"
- [methods] trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4): "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
