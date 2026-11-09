---
name: single-cell-transfer-learning
description: Use when a trained single-cell embedding model must be applied zero-shot
  to a new tissue or species AnnData (cells-by-genes .h5ad with 'batch_indices' and
  'cell_types' obs columns) and clustering quality on the target must be quantified
  against ground-truth cell types.
when_to_use_negative:
- 'Source and target datasets are unaligned (different gene sets): use prepare_for_transfer
  plus fine-tuning instead, since the model would usually require fine-tuning before
  evaluating the target dataset'
- The goal is de novo clustering of a single dataset with no pretrained model — plain
  scETM training with UnsupervisedTrainer and evaluate() is the appropriate workflow,
  not transfer
- 'Cell type labels are unavailable on the target: ARI against ground truth cannot
  be computed, so the benchmark cannot be judged'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Pretrained generative topic model whose encoder provides zero-shot transferable
    cell embeddings (obsm['delta']) on the target dataset
  repo: github.com/hui2000ji/scETM
- name: scVI
  role: Baseline deep generative model trained and applied zero-shot on the same six
    source-target pairs for ARI/kBET comparison
  repo: https://github.com/scverse/scvi-tools
- name: scVI-LD
  role: Large-batch scVI variant used as a second baseline in the transfer-learning
    benchmark (commands available in the scripts folder)
  repo: https://github.com/scverse/scvi-tools
- name: anndata
  role: Reads the source/target .h5ad cells-by-genes matrices and holds batch and
    cell type annotations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:2b330f3e33eabb5a620940fbb9cf1eb7d88274e43fabeb639d8aca639cccff0c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - anndata
  - scETM
  - scVI
  - scVI-LD
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-transfer-learning

## Summary

Zero-shot transfer of a pretrained scETM model from a source scRNA-seq dataset to a target dataset (e.g., MousePancreas→HumanPancreas, MusMOp→HumM1C) without retraining, benchmarked against scVI and scVI-LD using ARI and kBET. This reproduces the cross-tissue and cross-species transfer-learning evaluation (Fig 4) showing scETM's transferable cell embeddings.

## When to use

Use when a trained single-cell embedding model must be applied zero-shot to a new tissue or species AnnData (cells-by-genes .h5ad with 'batch_indices' and 'cell_types' obs columns) and clustering quality on the target must be quantified against ground-truth cell types.

## When NOT to use

- Source and target datasets are unaligned (different gene sets): use prepare_for_transfer plus fine-tuning instead, since the model would usually require fine-tuning before evaluating the target dataset
- The goal is de novo clustering of a single dataset with no pretrained model — plain scETM training with UnsupervisedTrainer and evaluate() is the appropriate workflow, not transfer
- Cell type labels are unavailable on the target: ARI against ground truth cannot be computed, so the benchmark cannot be judged

## Inputs

- Source AnnData cells-by-genes matrix (.h5ad, e.g. MousePancreas.h5ad) with 'batch_indices' and 'cell_types' in adata.obs
- Target AnnData cells-by-genes matrix (.h5ad, e.g. HumanPancreas_aligned.h5ad) with gene lists aligned to the source
- Ground-truth cell type annotations ('cell_types') for ARI computation

## Outputs

- Zero-shot cell embeddings stored in target obsm['delta']
- Per-task ARI and kBET values for scETM, scVI, and scVI-LD (transfer_benchmark_results.csv)
- Embedding UMAP plots (e.g. figures/scETM_transfer/scETM_MP_to_HP)

## How to apply

Load each source and target cells-by-genes AnnData matrix (e.g., MousePancreas.h5ad, HumanPancreas.h5ad), using the 'batch_indices' column of adata.obs for batch labels and 'cell_types' for cell type identity. Initialize scETM with the number of genes (n_vars) and unique batches with enable_batch_bias=True, and train on the source with UnsupervisedTrainer for 12000 epochs, evaluating every 1000 epochs with 4 minibatch samplers. Apply the trained model to the target without retraining via get_cell_embeddings_and_nll, storing zero-shot cell embeddings in hp.obsm['delta']. Repeat the training and zero-shot application for the scVI and scVI-LD baselines on each of the six source-target pairs (TM(FACS)↔MP, HP↔MP, MusMOp↔HumM1C). Compute ARI between transferred cell-embedding-based cluster assignments and ground-truth cell types, and kBET for batch mixing, then assemble per-task results into transfer_benchmark_results.csv.

## Related tools

- **scETM** (Pretrained generative topic model whose encoder provides zero-shot transferable cell embeddings (obsm['delta']) on the target dataset) — github.com/hui2000ji/scETM
- **scVI** (Baseline deep generative model trained and applied zero-shot on the same six source-target pairs for ARI/kBET comparison) — https://github.com/scverse/scvi-tools
- **scVI-LD** (Large-batch scVI variant used as a second baseline in the transfer-learning benchmark (commands available in the scripts folder)) — https://github.com/scverse/scvi-tools
- **anndata** (Reads the source/target .h5ad cells-by-genes matrices and holds batch and cell type annotations)

## Examples

```
from scETM import scETM, UnsupervisedTrainer; model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); UnsupervisedTrainer(model, mp).train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_cell_embeddings_and_nll(hp, emb_names='delta')
```

## Evaluation signals

- Cross-species brain transfer yields ARI 0.696 for MusMOp → HumM1C and ARI 0.167 for HumM1C → MusMOp with scETM
- Cross-tissue TM(FACS)→MP transfer yields scETM ARI 0.941 and kBET 0.339
- Target AnnData contains non-empty cell embeddings in obsm['delta'] after get_cell_embeddings_and_nll with no model retraining on the target
- Per-task ARI and kBET rows for scETM, scVI, and scVI-LD are present in transfer_benchmark_results.csv for all six source-target pairs
- Training reached ≥6k gradient updates (12000 epochs at eval_every=1000, n_samplers=4) before transfer per the README convergence guidance

## Limitations

- Zero-shot transfer requires pre-aligned source and target datasets (same/homologous gene lists with aligned order); unaligned datasets need prepare_for_transfer and fine-tuning
- For small datasets (e.g. 1886-cell MousePancreas) with large mini-batch sizes, fewer than 6k gradient updates may leave the model unconverged before transfer
- Zero-shot performance is asymmetric across species direction: scETM ARI drops from 0.696 (MusMOp→HumM1C) to 0.167 (HumM1C→MusMOp)

## Evidence

- [methods] Zero-shot transfer of pretrained model to target dataset: "# Directly apply MP-trained model on HP, storing cell embeddings to hp.obsm['delta'] (zero-shot transfer)."
- [readme] README transfer learning workflow: "# Train the model on adata for 12000 epochs, and evaluate every 1000 epochs. Use 4 threads to sample minibatches."
- [intro] Transferability motivation: "model transferrability is an important consideration. We consider a model as transferable if the learned knowledge manifested as the model parameters could benefit future data modeling."
- [abstract] Scalability and zero-shot claim: "scETM is scalable to over 10 6 cells and confers remarkable cross-tissue and cross-species zero-shot transfer-learning performance."
- [readme] Alignment requirement: "In this setting, the source and target dataset are pre-aligned, meaning they have the same / homologous gene lists (note that the order of the genes would have to be aligned)."
