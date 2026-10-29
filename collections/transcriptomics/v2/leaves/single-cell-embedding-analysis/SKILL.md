---
name: single-cell-embedding-analysis
description: Use when scRNA-seq count data must be clustered and interpreted through
  topics — e.g. identifying topics differentially expressed between disease and control
  conditions or across cell types.
when_to_use_negative:
- The goal is batch correction alone rather than topic-level interpretation — methods
  like Harmony or Scanorama target integration without topic structure
- The input is not single-cell count data in AnnData format (e.g. bulk RNA-seq or
  pre-computed feature tables)
- Only one mini-batch of training is feasible and the model cannot reach the ~6k gradient
  updates recommended for convergence
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Trains the generative topic model and produces the cell topic mixtures (obsm['theta']),
    gene embeddings (varm['rho']), and topic embeddings (uns['alpha']) used for differential
    topic testing
  repo: https://github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:2d203163179cba95be9d128b489a84f279860c9c7a162689d3432b70f5b395c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# single-cell-embedding-analysis

## Summary

Trains the scETM generative topic model on scRNA-seq data and uses the learned cell topic mixtures, topic and gene embeddings to interpret topics via differential topic abundance tests (permutation + Bonferroni) and pathway enrichment.

## When to use

Use when scRNA-seq count data must be clustered and interpreted through topics — e.g. identifying topics differentially expressed between disease and control conditions or across cell types.

## When NOT to use

- The goal is batch correction alone rather than topic-level interpretation — methods like Harmony or Scanorama target integration without topic structure
- The input is not single-cell count data in AnnData format (e.g. bulk RNA-seq or pre-computed feature tables)
- Only one mini-batch of training is feasible and the model cannot reach the ~6k gradient updates recommended for convergence

## Inputs

- cells-by-genes count matrix as an AnnData object (.h5ad)
- adata.obs annotations for batch (batch_indices) and cell type (cell_types)

## Outputs

- normalized cell topic mixtures stored at obsm['theta']
- gene embeddings stored at varm['rho']
- topic embeddings stored at uns['alpha']
- table of topic IDs, contrasts, permutation p-values, and Bonferroni-adjusted significance

## How to apply

Load a cells-by-genes AnnData object (batch in adata.obs['batch_indices'], cell types in adata.obs['cell_types'] by default), initialize model = scETM(n_vars, n_batches, enable_batch_bias=True), and train with UnsupervisedTrainer for enough epochs to reach at least 6k gradient updates for small datasets. Call model.get_all_embeddings_and_nll(adata) to populate obsm['theta'] (normalized cell topic mixtures), varm['rho'] (gene embeddings), and uns['alpha'] (topic embeddings). For each topic, run differential topic abundance testing between labeled groups (e.g. AD vs control, suicidal vs healthy) using 100,000 label permutations, then apply Bonferroni correction over all tested topics per contrast. Inspect varm['rho'] for significant topics to confirm pathway character, e.g. mitochondrial gene enrichment for topic 58.

## Related tools

- **scETM** (Trains the generative topic model and produces the cell topic mixtures (obsm['theta']), gene embeddings (varm['rho']), and topic embeddings (uns['alpha']) used for differential topic testing) — https://github.com/hui2000ji/scETM

## Examples

```
model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); UnsupervisedTrainer(model, mp).train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_all_embeddings_and_nll(mp)
```

## Evaluation signals

- Embeddings land in the correct AnnData slots: obsm['theta'], varm['rho'], uns['alpha'] after model.get_all_embeddings_and_nll(adata)
- Results table contains topics 12 and 58 for the AD-vs-control contrast with permutation p-value at or below 1e-5
- Results table contains topics 1/52/68/70/86 for the suicidal-vs-healthy (MDD) contrast
- Cell-type contrast calls match topics 19/35/50/69/97
- Significant topics pass Bonferroni correction over all tested topics within each contrast, and topic 58 shows mitochondrial gene enrichment in varm['rho']

## Limitations

- The current model handles only a single categorical batch variable; multi-batch-variable correction and numerical batch variables (e.g. subject age, unless binned categorically) are not directly supported
- scETM is trained via batched gradient descent — on small datasets with large mini-batch sizes, one epoch may yield only one gradient step, so epoch counts must be increased or the model may not converge
- Permutation p-values from 100,000 label permutations have a resolution floor of ~1e-5

## Evidence

- [abstract] Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways.: "Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways."
- [readme] scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object.: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object."
- [readme] Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha'].: "Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']."
- [readme] it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
