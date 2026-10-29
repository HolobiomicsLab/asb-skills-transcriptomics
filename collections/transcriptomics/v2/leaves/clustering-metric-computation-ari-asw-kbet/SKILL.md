---
name: clustering-metric-computation-ari-asw-kbet
description: Use when comparing batch-integration methods on annotated scRNA-seq AnnData
  with 'batch_indices' and 'cell_types' columns and you need quantitative ARI/ASW/kBET
  scores to judge biology preservation vs. batch over-correction.
when_to_use_negative:
- Input AnnData lacks gold-standard 'cell_types' annotations — ARI/ASW against ground
  truth cannot be computed
- Data is a single batch with no 'batch_indices' variation — kBET batch-mixing is
  undefined
- Goal is interpretability of topics/pathway associations (p-scETM gene set enrichment)
  rather than integration benchmarking
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Source of cell embeddings (obsm['delta']) and the evaluate() helper used to
    cluster embeddings and score ARI/ASW/kBET
  repo: github.com/hui2000ji/scETM
- name: LIGER
  role: Competing integrative NMF method whose embeddings are scored; over-aggressive
    mixing benchmark (kBET 0.176, ARI 0.714)
  repo: https://github.com/welch-lab/liger
- name: Harmony
  role: Competing batch-correction method scored on the same ARI/ASW/kBET protocol
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Competing integration method scored on the same metrics
  repo: https://github.com/brianhie/scanorama
- name: scVI
  role: Competing VAE integration method scored on the same metrics
  repo: https://github.com/YosefLab/scvi-tools
- name: scVAE-GM
  role: Competing Gaussian-mixture VAE baseline scored on the same metrics
  repo: https://github.com/scvae/scvae
- name: AnnData
  role: Container carrying cells-by-genes matrix plus batch_indices/cell_types annotations
    and embedding slots (obsm['delta'])
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:2241d73313304b47bdb77ec6931b973daf8ddcf0af6bb796aadd262ef157c8f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - Harmony
  - LIGER
  - Scanorama
  - Seurat
  - UnsupervisedTrainer
  - scETM
  - scVAE-GM
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Clustering metric computation (ARI, ASW, kBET)

## Summary

Quantifies single-cell integration quality by clustering extracted cell embeddings and scoring cell-type accuracy (ARI), cluster coherence (ASW), and batch mixing (kBET) against gold-standard cell_types and batch_indices labels, as used in the scETM benchmark on the Mouse Retina (Macosko + Shekhar) two-batch dataset.

## When to use

Use when comparing batch-integration methods on annotated scRNA-seq AnnData with 'batch_indices' and 'cell_types' columns and you need quantitative ARI/ASW/kBET scores to judge biology preservation vs. batch over-correction.

## When NOT to use

- Input AnnData lacks gold-standard 'cell_types' annotations — ARI/ASW against ground truth cannot be computed
- Data is a single batch with no 'batch_indices' variation — kBET batch-mixing is undefined
- Goal is interpretability of topics/pathway associations (p-scETM gene set enrichment) rather than integration benchmarking

## Inputs

- Annotated cells-by-genes AnnData object (adata) with adata.obs['batch_indices'] (source study labels, e.g., Macosko vs. Shekhar) and adata.obs['cell_types'] (retinal cell identity gold annotations)
- Trained model cell embeddings (e.g., mp.obsm['delta'] unnormalized or mp.obsm['theta'] normalized from scETM)
- Batch label vector for kBET acceptance-rate testing

## Outputs

- Per-method ARI, ASW, and kBET scores assembled into a comparison table (e.g., supplementary_table7.tsv)
- Embedding UMAP/plot artifact from evaluate() (e.g., figures/scETM_MP)
- Robustness check comparing metrics computed with vs. without masked doublelets

## How to apply

Train each integration model (e.g., scETM via UnsupervisedTrainer with n_epochs=12000, eval_every=1000, n_samplers=4, enable_batch_bias=True) on the combined AnnData, then extract cell embeddings (mp.obsm['delta'] for scETM, normalized at mp.obsm['theta']). Cluster the embeddings and compute ARI and ASW against the 'cell_types' gold labels and kBET against the 'batch_indices' batch labels. Evaluate embedding_key='delta' via the scETM evaluate() helper. Assemble per-method ARI/ASW/kBET into a table (e.g., supplementary_table7.tsv) and compare against reference values — scETM ARI 0.859, ASW 0.2873, kBET 0.0656; LIGER highest kBET 0.176 with lowest ARI 0.714 — to distinguish balanced integration (scETM) from over-correction (LIGER). Re-run metrics after masking contaminating cells (e.g., 669 Shekhar doublelets) to confirm the scores are robust.

## Related tools

- **scETM** (Source of cell embeddings (obsm['delta']) and the evaluate() helper used to cluster embeddings and score ARI/ASW/kBET) — github.com/hui2000ji/scETM
- **LIGER** (Competing integrative NMF method whose embeddings are scored; over-aggressive mixing benchmark (kBET 0.176, ARI 0.714)) — https://github.com/welch-lab/liger
- **Harmony** (Competing batch-correction method scored on the same ARI/ASW/kBET protocol) — https://github.com/immunogenomics/harmony
- **Scanorama** (Competing integration method scored on the same metrics) — https://github.com/brianhie/scanorama
- **scVI** (Competing VAE integration method scored on the same metrics) — https://github.com/YosefLab/scvi-tools
- **scVAE-GM** (Competing Gaussian-mixture VAE baseline scored on the same metrics) — https://github.com/scvae/scvae
- **AnnData** (Container carrying cells-by-genes matrix plus batch_indices/cell_types annotations and embedding slots (obsm['delta']))

## Examples

```
from scETM import scETM, UnsupervisedTrainer, evaluate; trainer = UnsupervisedTrainer(model, mp); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_all_embeddings_and_nll(mp); evaluate(mp, embedding_key="delta", plot_fname="scETM_MP", plot_dir="figures/scETM_MP")
```

## Evaluation signals

- Reproduced supplementary_table7.tsv matches reported values within rounding tolerance: scETM ARI 0.859, ASW 0.2873, kBET 0.0656; LIGER kBET 0.176 / ARI 0.714; scETM+adv kBET 0.1410 / ARI 0.7720
- ARI computed against cell_types clusters above chance (>0) and scETM's embedding achieves highest ARI among compared methods
- kBET values fall in the 0–1 range where lower indicates better batch mixing; LIGER should show the highest kBET among methods
- Metrics recomputed after removing/masking the 669 annotated Shekhar doublelets agree with the full-data metrics
- Embeddings are present in mp.obsm['delta'] (and mp.obsm['theta'] normalized) after model.get_all_embeddings_and_nll(mp) before metric computation

## Limitations

- scETM's current model only handles a single categorical batch variable, so the metric suite as benchmarked assumes one batch label per cell
- Training duration matters for small datasets: scETM is trained by batched gradient descent, so the README recommends at least ~6k gradient updates or the model may not have converged when metrics are computed
- Aggressive batch mixing is not inherently better — kBET improvements can co-occur with ARI loss (LIGER), so metrics must be read jointly

## Evidence

- [readme] By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column.: "By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column."
- [methods] Trainer settings: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [methods] Embedding extraction: "model.get_all_embeddings_and_nll(mp)"
- [methods] Evaluation call: "evaluate(mp, embedding_key="delta", plot_fname="scETM_MP", plot_dir="figures/scETM_MP")"
- [readme] Convergence caveat: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model."
