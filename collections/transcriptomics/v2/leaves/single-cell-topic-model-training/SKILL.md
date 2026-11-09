---
name: single-cell-topic-model-training
description: Use when you need to integrate and cluster large-scale scRNA-seq datasets
  while obtaining interpretable topic-pathway associations from a cells-by-genes AnnData
  matrix with batch annotations.
when_to_use_negative:
- Your input is not a raw or normalized cells-by-genes count matrix in AnnData format
  but an already-embedded or feature-summarized table
- You need to correct for multiple categorical batch variables or continuous/numerical
  batch variables (e.g., subject age); scETM currently handles only a single categorical
  batch variable
- You need multi-omic integration (scRNA-seq with protein or scATAC-seq); this is
  not yet implemented in scETM
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scETM
  role: Core generative topic model trained via UnsupervisedTrainer to learn cell,
    gene, and topic embeddings from scRNA-seq AnnData; supports batch bias, p-scETM
    pathway fixing, and zero-shot transfer
  repo: https://github.com/hui2000ji/scETM
- name: scvi-tools
  role: Competing deep-generative single-cell integration framework used as a baseline
    in the scETM benchmarking
  repo: https://github.com/YosefLab/scvi-tools
- name: Harmony
  role: Baseline batch-correction method benchmarked against scETM for single-cell
    integration
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Baseline batch-correction and integration method benchmarked against scETM
  repo: https://github.com/brianhie/scanorama
- name: LIGER
  role: Baseline integrative NMF method benchmarked against scETM
  repo: https://github.com/welch-lab/liger
- name: scVAE-GM
  role: Baseline variational auto-encoder method benchmarked against scETM
  repo: https://github.com/scvae/scvae
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:db808a66ebed0ea79454c6b910d59809358fb817b1a46ca846a14799601a53c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - Harmony
  - LIGER
  - Scanorama
  - scETM
  - scVAE-GM
  - scvi-tools
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-topic-model-training

## Summary

Train a scETM (single-cell Embedded Topic Model) generative topic model on cells-by-genes scRNA-seq AnnData matrices to learn interpretable cell, gene, and topic embeddings, optionally with pathway-fixed gene embeddings (p-scETM).

## When to use

Use when you need to integrate and cluster large-scale scRNA-seq datasets while obtaining interpretable topic-pathway associations from a cells-by-genes AnnData matrix with batch annotations.

## When NOT to use

- Your input is not a raw or normalized cells-by-genes count matrix in AnnData format but an already-embedded or feature-summarized table
- You need to correct for multiple categorical batch variables or continuous/numerical batch variables (e.g., subject age); scETM currently handles only a single categorical batch variable
- You need multi-omic integration (scRNA-seq with protein or scATAC-seq); this is not yet implemented in scETM

## Inputs

- AnnData cells-by-genes matrix (adata) with batch information in adata.obs['batch_indices'] and cell types in adata.obs['cell_types']
- Pathways-by-genes matrix from the pathDIP4 pathway database (for p-scETM), filtered to pathways with more than 5 genes
- Pre-aligned source/target AnnData files (e.g., MousePancreas_aligned.h5ad, HumanPancreas_aligned.h5ad) for transfer learning

## Outputs

- Cell embeddings (unnormalized at adata.obsm['delta'], normalized at adata.obsm['theta'])
- Gene embedding matrix rho (adata.varm['rho'])
- Topic embedding matrix alpha (adata.uns['alpha'])
- Model checkpoint files and evaluation embedding plots (e.g., figures/scETM_MP)
- Topic/pathway association tables (e.g., topic_pathway_enrichment.tsv, pathway_topic_associations.csv)

## How to apply

Prepare a cells-by-genes AnnData object with batch indices in adata.obs (scETM looks for 'batch_indices' and 'cell_types' columns by default). Initialize the model with scETM(n_genes, n_batches, enable_batch_bias=True) and train with UnsupervisedTrainer for a number of epochs yielding at least 6k gradient updates (e.g., n_epochs=12000, eval_every=1000, n_samplers=4), since scETM trains via batched gradient descent. Extract embeddings via model.get_all_embeddings_and_nll(adata): unnormalized cell embeddings at obsm['delta'], normalized at obsm['theta'], gene embeddings at varm['rho'], and topic embeddings at uns['alpha']. For the pathway-informed variant (p-scETM), fix the gene embedding matrix rho to a pathways-by-genes matrix from the pathDIP4 database (keeping only pathways with more than 5 genes) by setting trainable_gene_emb_dim to zero; the model gene set becomes the intersection of scRNA-seq genes and pathway-matrix genes. For transfer learning across datasets, use prepare_for_transfer for unaligned gene sets or directly apply the trained model to pre-aligned targets, optionally fine-tuning with a lower initial learning rate (init_lr=5e-4).

## Related tools

- **scETM** (Core generative topic model trained via UnsupervisedTrainer to learn cell, gene, and topic embeddings from scRNA-seq AnnData; supports batch bias, p-scETM pathway fixing, and zero-shot transfer) — https://github.com/hui2000ji/scETM
- **scvi-tools** (Competing deep-generative single-cell integration framework used as a baseline in the scETM benchmarking) — https://github.com/YosefLab/scvi-tools
- **Harmony** (Baseline batch-correction method benchmarked against scETM for single-cell integration) — https://github.com/immunogenomics/harmony
- **Scanorama** (Baseline batch-correction and integration method benchmarked against scETM) — https://github.com/brianhie/scanorama
- **LIGER** (Baseline integrative NMF method benchmarked against scETM) — https://github.com/welch-lab/liger
- **scVAE-GM** (Baseline variational auto-encoder method benchmarked against scETM) — https://github.com/scvae/scvae

## Examples

```
model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); trainer = UnsupervisedTrainer(model, mp, train_instance_name='MP', ckpt_dir='../results'); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_all_embeddings_and_nll(mp)
```

## Evaluation signals

- Training runs at least 6k gradient updates (small datasets like MousePancreas with 1886 cells fit in one mini-batch per epoch, so n_epochs must be large enough)
- Model evaluation via evaluate(mp, embedding_key='delta') produces embedding plots (e.g., figures/scETM_MP) without errors and with reasonable cell-type clustering of the cell embeddings
- Topic scores yield GSEA hits matching known associations, e.g., HP Topic 54 enriched in Insulin Processing with BH-adjusted q-value of 0, and AD topics in Reactome Amyloid Fiber Formation / KEGG AD
- For p-scETM, pathway-topic association table contains expected hits (Insulin Signaling among top HP pathways; Topic 9 associated with Alzheimer Disease-Amyloid Secretase) and clustering metrics are not degraded versus unregularized scETM baseline
- For transfer learning, zero-shot application stores target cell embeddings in the target adata.obsm['delta'] and evaluation is run on the transferred embeddings

## Limitations

- The current model handles only a single categorical batch variable; multi-batch-variable correction is future work, and continuous batch variables (e.g., subject age) must be binned into categorical ranges
- Multi-omic integration (scRNA-seq plus protein or scATAC-seq) is not yet implemented
- Gene embeddings pretrained on knowledge graphs (PubMed, gene regulatory network, PPI, Gene Ontology) are a suggested future direction, not realized in the current model
- For small datasets and large mini-batch sizes, one epoch yields only one gradient step; too few epochs risks a non-converged model
- In p-scETM with trainable_gene_emb_dim=0, the model gene set is restricted to the intersection of scRNA-seq genes and pathway-matrix genes

## Evidence

- [methods] model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True): "model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True)"
- [readme] it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
- [methods] We only keep pathways that contain more than 5 genes.: "We only keep pathways that contain more than 5 genes."
- [abstract] Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways.: "Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways."
- [readme] The prepare_for_transfer function discards the parameters tied to the source-dataset-unique genes and randomly initializes those tied to the target-dataset-unique genes.: "The `prepare_for_transfer` function discards the parameters tied to the source-dataset-unique genes and randomly initializes those tied to the target-dataset-unique genes."
