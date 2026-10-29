---
name: topic-model-unsupervised-training
description: Use when you have raw scRNA-seq count data in an AnnData object with
  batch_indices (and optionally cell_types) and need unsupervised cell embeddings
  for clustering, batch integration, or downstream benchmarking against methods like
  Harmony, LIGER, scVI, or Scanorama.
when_to_use_negative:
- The input is already a reduced embedding or feature table — scETM requires raw cells-by-genes
  count data in AnnData format
- The analysis requires correcting multiple categorical batch variables or numerical/continuous
  covariates (e.g. subject age) — the current model handles only a single categorical
  batch variable
- Multi-omic integration (scRNA-seq with protein or scATAC-seq) is needed — this is
  not yet implemented in scETM
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Generative topic model whose scETM class and UnsupervisedTrainer perform the
    unsupervised training and produce cell/gene/topic embeddings
  repo: https://github.com/hui2000ji/scETM
- name: scVI
  role: Competing probabilistic method for cell embeddings and batch-aware integration,
    benchmarked against scETM
  repo: https://github.com/YosefLab/scvi-tools
- name: Harmony
  role: Competing integration method using maximum diversity clustering and linear
    batch correction, benchmarked via scETM repo scripts
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Competing batch-correction/integration method benchmarked via scETM repo scripts
  repo: https://github.com/brianhie/scanorama
- name: LIGER
  role: Competing integrative NMF method benchmarked via scETM repo scripts
  repo: https://github.com/welch-lab/liger
- name: scVAE-GM
  role: Competing VAE method with Gaussian mixture prior benchmarked via scETM repo
    scripts
  repo: https://github.com/scvae/scvae
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:e5c4591a5a4990d613521a2122395f60cb774fea1af98113523bf78c6e5dec1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - Harmony
  - LIGER
  - Scanorama
  - Seurat
  - UnsupervisedTrainer
  - scETM
  - scVAE-GM
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# topic-model-unsupervised-training

## Summary

Train the scETM generative topic model on a cells-by-genes AnnData matrix without cell-type supervision, using batched gradient descent via UnsupervisedTrainer, to produce interpretable cell, gene, and topic embeddings for single-cell RNA-seq clustering and integration.

## When to use

Use when you have raw scRNA-seq count data in an AnnData object with batch_indices (and optionally cell_types) and need unsupervised cell embeddings for clustering, batch integration, or downstream benchmarking against methods like Harmony, LIGER, scVI, or Scanorama.

## When NOT to use

- The input is already a reduced embedding or feature table — scETM requires raw cells-by-genes count data in AnnData format
- The analysis requires correcting multiple categorical batch variables or numerical/continuous covariates (e.g. subject age) — the current model handles only a single categorical batch variable
- Multi-omic integration (scRNA-seq with protein or scATAC-seq) is needed — this is not yet implemented in scETM

## Inputs

- cells-by-genes scRNA-seq count matrix as an AnnData (.h5ad) object with adata.obs.batch_indices (required) and adata.obs.cell_types (for evaluation)
- number of batches derived from mp.obs.batch_indices.nunique()
- for p-scETM variant: a pathways-by-genes matrix from the pathDIP4 pathway database, keeping only pathways with more than 5 genes

## Outputs

- unnormalized cell embeddings stored at adata.obsm['delta']
- normalized cell embeddings stored at adata.obsm['theta']
- gene embeddings stored at adata.varm['rho']
- topic embeddings stored at adata.uns['alpha']
- model checkpoints/logs in ckpt_dir and an embedding evaluation plot (e.g. scETM_MP in figures/scETM_MP)

## How to apply

Assemble the scRNA-seq data as a cells-by-genes AnnData object with batch information in the 'batch_indices' column of adata.obs (use batch_col/cell_type_col arguments if named differently). Initialize the model as scETM(adata.n_vars, adata.obs.batch_indices.nunique(), enable_batch_bias=True) — batch bias compensates for batch effects. Instantiate UnsupervisedTrainer(model, adata, train_instance_name=..., ckpt_dir=...), which sets the random seed, optimizer, training/evaluation loop, checkpointing and logging; train with trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4). The epoch count matters: because scETM trains via batched gradient descent on small datasets with large mini-batches (e.g. MousePancreas, 1886 cells, batch size 2000), one epoch is one gradient step, so set epochs to give at least 6k gradient updates or the model may not converge. After training, call model.get_all_embeddings_and_nll(adata) to store unnormalized cell embeddings at obsm['delta'], normalized cell embeddings at obsm['theta'], gene embeddings at varm['rho'], and topic embeddings at uns['alpha'], then evaluate with evaluate(adata, embedding_key='delta', ...) to cluster and score against cell types.

## Related tools

- **scETM** (Generative topic model whose scETM class and UnsupervisedTrainer perform the unsupervised training and produce cell/gene/topic embeddings) — https://github.com/hui2000ji/scETM
- **scVI** (Competing probabilistic method for cell embeddings and batch-aware integration, benchmarked against scETM) — https://github.com/YosefLab/scvi-tools
- **Harmony** (Competing integration method using maximum diversity clustering and linear batch correction, benchmarked via scETM repo scripts) — https://github.com/immunogenomics/harmony
- **Scanorama** (Competing batch-correction/integration method benchmarked via scETM repo scripts) — https://github.com/brianhie/scanorama
- **LIGER** (Competing integrative NMF method benchmarked via scETM repo scripts) — https://github.com/welch-lab/liger
- **scVAE-GM** (Competing VAE method with Gaussian mixture prior benchmarked via scETM repo scripts) — https://github.com/scvae/scvae

## Examples

```
from scETM import scETM, UnsupervisedTrainer, evaluate; import anndata; mp = anndata.read_h5ad('MousePancreas.h5ad'); model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); trainer = UnsupervisedTrainer(model, mp, train_instance_name='MP', ckpt_dir='../results'); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_all_embeddings_and_nll(mp); evaluate(mp, embedding_key='delta', plot_fname='scETM_MP', plot_dir='figures/scETM_MP')
```

## Evaluation signals

- Trained embeddings exist at the documented slots: adata.obsm['delta'], adata.obsm['theta'], adata.varm['rho'], adata.uns['alpha'] after get_all_embeddings_and_nll
- Model has converged: number of gradient updates >= ~6000 (e.g. n_epochs=12000 with eval_every=1000 on small datasets)
- evaluate() on obsm['delta'] yields ARI/ASW clustering metrics against cell_types and kBET batch-mixing scores (e.g. scETM ARI 0.859, ASW 0.2873, kBET 0.0656 on Mouse Retina Macosko/Shekhar integration)
- Metrics are robust to annotated confounders: removing/masking the 669 Shekhar doublelets should not change reported clustering metrics
- Checkpoints and evaluation plots are written to the configured ckpt_dir and plot_dir at every eval_every interval

## Limitations

- Only a single categorical batch variable is supported; correction of multiple categorical batch variables or numerical covariates (e.g. subject age) is not implemented
- Training epochs must be sized to dataset and mini-batch so at least ~6k gradient updates occur, otherwise the model may be under-converged
- Gene embeddings pretrained on knowledge graphs (PubMed, gene regulatory network, PPI, Gene Ontology) are described as future work, not yet realized in the released model

## Evidence

- [methods] Initialize model and train unsupervised: "model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True)"
- [methods] Training loop parameters: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [methods] Embedding output locations: "Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']."
- [readme] Convergence guidance for small datasets: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
- [readme] Input data format requirement: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object."
