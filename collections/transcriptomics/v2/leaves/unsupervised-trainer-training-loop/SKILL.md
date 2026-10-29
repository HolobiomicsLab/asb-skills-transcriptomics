---
name: unsupervised-trainer-training-loop
description: Use when an AnnData cells-by-genes matrix needs unsupervised scETM training
  (e.g., MousePancreas.h5ad with 'batch_indices' and 'cell_types' in adata.obs) before
  embedding extraction or zero-shot transfer to a target dataset.
when_to_use_negative:
- The goal is supervised or reference-based annotation rather than unsupervised clustering
  of the target scRNA-seq data
- The gene embedding matrix must be fixed to a pathways-by-genes matrix (p-scETM variant
  with trainable_gene_emb_dim=0), which requires a different configuration path
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Topic model trained via UnsupervisedTrainer to produce interpretable cell,
    gene, and topic embeddings from scRNA-seq counts
  repo: github.com/hui2000ji/scETM
- name: scvi-tools
  role: Baseline framework whose scVI models are trained and compared against scETM
    in the same benchmarking scripts
  repo: https://github.com/YosefLab/scvi-tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:12948ebb47534ae88d7bcfe65d746fd0278ea32786ae0abd08c0fc9c486cf860
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scETM
  - scVI
  - scvi-tools
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# unsupervised-trainer-training-loop

## Summary

Train the scETM topic model on a single-cell RNA-seq AnnData object using the UnsupervisedTrainer class for 12000 epochs with periodic evaluation, producing cell, gene, and topic embeddings suitable for clustering and zero-shot transfer.

## When to use

Use when an AnnData cells-by-genes matrix needs unsupervised scETM training (e.g., MousePancreas.h5ad with 'batch_indices' and 'cell_types' in adata.obs) before embedding extraction or zero-shot transfer to a target dataset.

## When NOT to use

- The goal is supervised or reference-based annotation rather than unsupervised clustering of the target scRNA-seq data
- The gene embedding matrix must be fixed to a pathways-by-genes matrix (p-scETM variant with trainable_gene_emb_dim=0), which requires a different configuration path

## Inputs

- AnnData object with a cells-by-genes count matrix (e.g. MousePancreas.h5ad) with 'batch_indices' and 'cell_types' columns in adata.obs
- scETM model instance initialized with adata.n_vars and adata.obs.batch_indices.nunique(), enable_batch_bias=True

## Outputs

- Trained scETM model with checkpoints/logs in ckpt_dir (e.g. ../results)
- Unnormalized cell embeddings at adata.obsm['delta'] and normalized embeddings at adata.obsm['theta']
- Gene embeddings at adata.varm['rho'] and topic embeddings at adata.uns['alpha']
- Embedding evaluation plot (e.g. figures/scETM_MP/scETM_MP) from evaluate()

## How to apply

Load the cells-by-genes AnnData object (batch info in adata.obs['batch_indices'], cell types in adata.obs['cell_types']) and initialize scETM with n_vars and the number of unique batches, enabling batch bias via enable_batch_bias=True. Instantiate UnsupervisedTrainer(model, adata, train_instance_name=..., ckpt_dir=...), which sets the random seed, optimizer, training/evaluation loop, checkpointing and logging, then call trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4) to run batched gradient descent with evaluation every 1000 epochs using 4 minibatch samplers. After training, obtain embeddings with model.get_all_embeddings_and_nll(adata): unnormalized cell embeddings go to adata.obsm['delta'], normalized to adata.obsm['theta'], gene embeddings to adata.varm['rho'], topic embeddings to adata.uns['alpha']. Rationale for epoch count: because scETM trains via batched gradient descent, small datasets (e.g. MousePancreas with 1886 cells) with large mini-batch sizes (e.g. 2000) produce only one gradient step per epoch, so epochs should be set to give at least 6k gradient updates or the model may not converge. Optionally run evaluate() to cluster on the 'delta' embedding and save an embedding plot.

## Related tools

- **scETM** (Topic model trained via UnsupervisedTrainer to produce interpretable cell, gene, and topic embeddings from scRNA-seq counts) — github.com/hui2000ji/scETM
- **scvi-tools** (Baseline framework whose scVI models are trained and compared against scETM in the same benchmarking scripts) — https://github.com/YosefLab/scvi-tools

## Examples

```
from scETM import scETM, UnsupervisedTrainer; import anndata; mp = anndata.read_h5ad('MousePancreas.h5ad'); model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); trainer = UnsupervisedTrainer(model, mp, train_instance_name='MP', ckpt_dir='../results'); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)
```

## Evaluation signals

- Evaluation runs every 1000 of the 12000 epochs (with n_samplers=4) produce loss/likelihood traces in ckpt_dir showing convergence rather than premature stopping
- Post-training, adata.obsm['delta'], adata.obsm['theta'], adata.varm['rho'], and adata.uns['alpha'] are all populated by get_all_embeddings_and_nll
- evaluate() on the 'delta' embedding yields clustering consistent with adata.obs['cell_types'] (e.g. ARI of 0.941 for scETM on TM(FACS)→MP in the benchmark)
- Total gradient updates are at least ~6k (epochs × steps-per-epoch), e.g. small datasets like MousePancreas (1886 cells) need ≥6000 epochs at batch size 2000

## Limitations

- Training one epoch on small datasets with large mini-batch sizes yields only one gradient step; too few epochs can leave the model unconverged
- GPU-enabled PyTorch should be installed before scETM to get acceptable performance; CPU training is significantly slower

## Evidence

- [readme] The trainer object will set up the random seed, optimizer, training and evaluation loop, checkpointing and logging.: "The trainer object will set up the random seed, optimizer, training and evaluation loop, checkpointing and logging."
- [readme] trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4): "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [readme] Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha'].: "Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']."
- [readme] scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object"
- [methods] model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True): "model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True)"
