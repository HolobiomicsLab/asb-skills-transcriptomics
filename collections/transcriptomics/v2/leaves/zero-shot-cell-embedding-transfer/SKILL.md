---
name: zero-shot-cell-embedding-transfer
description: Use when a pretrained single-cell embedding model (e.g. scETM trained
  on MousePancreas.h5ad) must be applied to an aligned target dataset without retraining,
  and clustering quality of the transferred embeddings must be benchmarked.
when_to_use_negative:
- 'Source and target datasets are unaligned (different gene sets): prepare_for_transfer
  discards source-unique-gene parameters and the model usually requires fine-tuning
  before target evaluation, so pure zero-shot transfer is not appropriate.'
- The goal is de novo clustering of a single dataset rather than benchmarking transferability
  across datasets — a standard single-dataset scETM training/evaluation workflow is
  the right choice.
- 'The model must correct multiple categorical batch variables or integrate multi-omic
  data (scRNA-seq + protein/scATAC-seq): the article states these extensions are not
  yet implemented.'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Pretrained topic-model encoder trained on the source dataset and applied frozen
    to the target for zero-shot cell embedding transfer
  repo: https://github.com/hui2000ji/scETM
- name: scVI
  role: Baseline variational inference model trained and zero-shot transferred on
    the same six source-target pairs for ARI/kBET comparison
  repo: https://github.com/YosefLab/scvi-tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:5c836bea92f321905ec1386fbd469b6d53c1d5216ea372bfdec1bf7adf40a896
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - scETM
  - scVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# zero-shot-cell-embedding-transfer

## Summary

Reproduce scETM's zero-shot cross-tissue and cross-species transfer-learning evaluation (Fig 4): train the scETM encoder on a source AnnData dataset, apply the frozen model to a target dataset without retraining, and score transfer quality against scVI and scVI-LD using ARI and kBET.

## When to use

Use when a pretrained single-cell embedding model (e.g. scETM trained on MousePancreas.h5ad) must be applied to an aligned target dataset without retraining, and clustering quality of the transferred embeddings must be benchmarked.

## When NOT to use

- Source and target datasets are unaligned (different gene sets): prepare_for_transfer discards source-unique-gene parameters and the model usually requires fine-tuning before target evaluation, so pure zero-shot transfer is not appropriate.
- The goal is de novo clustering of a single dataset rather than benchmarking transferability across datasets — a standard single-dataset scETM training/evaluation workflow is the right choice.
- The model must correct multiple categorical batch variables or integrate multi-omic data (scRNA-seq + protein/scATAC-seq): the article states these extensions are not yet implemented.

## Inputs

- Source dataset AnnData cells-by-genes matrix (e.g. MousePancreas.h5ad) with obs columns 'batch_indices' and 'cell_types'
- Aligned target dataset AnnData (.h5ad, e.g. HumanPancreas_aligned.h5ad) sharing the same/homologous ordered gene list

## Outputs

- Zero-shot target cell embeddings stored in target.obsm['delta']
- Per-task ARI and kBET scores for scETM, scVI, and scVI-LD across six source-target pairs, assembled into transfer_benchmark_results.csv
- Embedding evaluation plots (e.g. figures/scETM_transfer/scETM_MP_to_HP)

## How to apply

Load each source and target cells-by-genes AnnData (.h5ad), using adata.obs['batch_indices'] for batch labels and adata.obs['cell_types'] for ground-truth identity; for cross-species or cross-tissue tasks, first align source and target to the same/homologous gene lists. Initialize scETM with n_vars and the number of unique batches (enable_batch_bias=True), then train with UnsupervisedTrainer for n_epochs=12000, eval_every=1000, n_samplers=4. Freeze the trained model and call get_cell_embeddings_and_nll on the target dataset, storing zero-shot cell embeddings in target obsm['delta']; optionally fine-tune with a new UnsupervisedTrainer (init_lr=5e-4, 800 epochs). Run scVI and scVI-LD baselines on the same source-target pairs (TM(FACS)↔MP, HP↔MP, MusMOp↔HumM1C), then compute ARI between cluster assignments of the transferred embeddings and ground-truth cell types, plus kBET for batch mixing, assembling per-task values into transfer_benchmark_results.csv.

## Related tools

- **scETM** (Pretrained topic-model encoder trained on the source dataset and applied frozen to the target for zero-shot cell embedding transfer) — https://github.com/hui2000ji/scETM
- **scVI** (Baseline variational inference model trained and zero-shot transferred on the same six source-target pairs for ARI/kBET comparison) — https://github.com/YosefLab/scvi-tools

## Examples

```
model.get_cell_embeddings_and_nll(hp, emb_names='delta'); evaluate(hp, embedding_key="delta", plot_fname="scETM_MP_to_HP", plot_dir="figures/scETM_transfer")
```

## Evaluation signals

- Zero-shot transfer yields cell embeddings in target.obsm['delta'] without any parameter updates on the target dataset.
- Cross-tissue TM(FACS)→MP scETM reaches ARI ≈ 0.941 and kBET ≈ 0.339.
- Cross-species brain transfer gives scETM ARI 0.696 for MusMOp→HumM1C and ARI 0.167 for HumM1C→MusMOp, exceeding scVI and scVI-LD.
- ARI is computed between cluster assignments derived from transferred embeddings and obs['cell_types'] ground truth; kBET quantifies batch mixing.
- Training used 12000 epochs with evaluation every 1000 epochs and 4 minibatch samplers, meeting the README guidance of at least ~6k gradient updates.

## Limitations

- Zero-shot transfer requires pre-aligned datasets: source and target must share the same or homologous gene lists with matching gene order.
- The scETM model considers only a single categorical batch variable; correction of multiple or continuous (e.g. subject age) batch variables is left as future work.
- For small datasets and large mini-batch sizes, too few epochs can leave the model unconverged; at least ~6k gradient updates are recommended before transfer.
- Multi-omic integration (scRNA-seq with protein or scATAC-seq) is not yet implemented.

## Evidence

- [methods] Zero-shot transfer of pretrained model to target dataset: [evidence span withheld — anchor status: not_found]
- [readme] Directly apply MP-trained model on HP (readme): "# Directly apply MP-trained model on HP, storing cell embeddings to hp.obsm['delta'] (zero-shot transfer)."
- [readme] Aligned gene lists requirement (readme): "the source and target dataset are pre-aligned, meaning they have the same / homologous gene **lists** (note that the order of the genes would have to be aligned)"
- [intro] Transferability motivation (intro): "model transferrability is an important consideration. We consider a model as transferable if the learned knowledge manifested as the model parameters could benefit future data modeling."
- [abstract] Zero-shot transfer performance claim (abstract): "scETM is scalable to over 10 6 cells and confers remarkable cross-tissue and cross-species zero-shot transfer-learning performance."
