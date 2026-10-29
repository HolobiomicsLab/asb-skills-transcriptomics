---
name: cross-species-scrna-seq-integration
description: Use when integrating single-cell RNA-seq datasets from different tissues
  or species where a pretrained embedding model (scETM) should be applied to a target
  AnnData without retraining, and clustering quality (ARI) and batch mixing (kBET)
  must be benchmarked.
when_to_use_negative:
- Source and target gene sets are unaligned (different/heterologous gene lists) —
  use prepare_for_transfer plus fine-tuning instead of zero-shot transfer
- The task requires a single integrated embedding from joint training on all batches
  rather than transfer of a pretrained encoder
- The target data require correcting multiple categorical batch variables or numerical
  covariates like subject age directly — scETM only handles a single categorical batch
  variable
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Pretrained topic model whose encoder is applied zero-shot to the target dataset
    to produce transferred cell embeddings (obsm['delta'])
  repo: https://github.com/hui2000ji/scETM
- name: scVI
  role: Baseline variational inference model trained and transferred on the same six
    source-target pairs for ARI/kBET comparison
  repo: https://github.com/YosefLab/scvi-tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:91f45f4630bb2179089a90127c826443befa7852940741548cfcce54308d0514
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-species-scrna-seq-integration

## Summary

Zero-shot transfer learning of a pretrained scETM model from a source scRNA-seq dataset to a cross-tissue or cross-species target dataset (e.g. MusMOp→HumM1C brain, TM(FACS)↔MP pancreas), evaluating transferred cell embeddings with ARI and kBET against scVI baselines.

## When to use

Use when integrating single-cell RNA-seq datasets from different tissues or species where a pretrained embedding model (scETM) should be applied to a target AnnData without retraining, and clustering quality (ARI) and batch mixing (kBET) must be benchmarked.

## When NOT to use

- Source and target gene sets are unaligned (different/heterologous gene lists) — use prepare_for_transfer plus fine-tuning instead of zero-shot transfer
- The task requires a single integrated embedding from joint training on all batches rather than transfer of a pretrained encoder
- The target data require correcting multiple categorical batch variables or numerical covariates like subject age directly — scETM only handles a single categorical batch variable

## Inputs

- Source AnnData cells-by-genes matrix (.h5ad), e.g. MousePancreas.h5ad, with obs['batch_indices'] and obs['cell_types']
- Target AnnData cells-by-genes matrix (.h5ad), e.g. HumanPancreas_aligned.h5ad, gene-aligned to source
- Ground-truth cell type annotations in obs['cell_types'] for ARI evaluation

## Outputs

- Target cell embeddings stored in target.obsm['delta']
- Per-task ARI and kBET values for each method (scETM, scVI, scVI-LD) across the six source-target pairs, assembled in transfer_benchmark_results.csv
- Embedding plots, e.g. figures/scETM_transfer/scETM_MP_to_HP

## How to apply

Load source and target cells-by-genes AnnData objects (.h5ad), with batch labels in adata.obs['batch_indices'] and cell identity in adata.obs['cell_types']. For pre-aligned datasets (homologous gene lists in the same order), initialize scETM with n_vars of the source and the number of unique batches, enabling batch_bias, and train with UnsupervisedTrainer for 12000 epochs (eval every 1000, n_samplers=4). Apply the trained model zero-shot to the target via get_cell_embeddings_and_nll, storing cell embeddings in obsm['delta'] — no retraining or fine-tuning for the zero-shot setting. Repeat the same train-and-transfer procedure for scVI and scVI-LD baselines on all six source-target pairs (TM(FACS)↔MP, HP↔MP, MusMOp↔HumM1C). Compute ARI between cluster assignments from transferred embeddings and ground-truth cell types, and kBET for batch mixing, assembling per-task results (e.g. scETM ARI 0.941/kBET 0.339 for TM(FACS)→MP; ARI 0.696 for MusMOp→HumM1C; ARI 0.167 for HumM1C→MusMOp) into a CSV such as transfer_benchmark_results.csv.

## Related tools

- **scETM** (Pretrained topic model whose encoder is applied zero-shot to the target dataset to produce transferred cell embeddings (obsm['delta'])) — https://github.com/hui2000ji/scETM
- **scVI** (Baseline variational inference model trained and transferred on the same six source-target pairs for ARI/kBET comparison) — https://github.com/YosefLab/scvi-tools

## Examples

```
model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_cell_embeddings_and_nll(hp, emb_names='delta')
```

## Evaluation signals

- Transferred cell embeddings appear in target.obsm['delta'] after get_cell_embeddings_and_nll without any target training epochs
- ARI of transferred clusters vs ground-truth cell types reproduces published values, e.g. scETM ARI 0.696 for MusMOp→HumM1C and ARI 0.941 (kBET 0.339) for TM(FACS)→MP
- kBET batch-mixing scores are computed for every method-task pair and recorded alongside ARI in transfer_benchmark_results.csv
- Gene order of the pre-aligned target .h5ad matches the source (same/homologous gene lists, aligned order) before transfer
- scETM zero-shot ARI on cross-species brain tasks exceeds the scVI and scVI-LD baselines on the same pairs

## Limitations

- Asymmetric performance across transfer directions: scETM achieves ARI 0.696 for MusMOp→HumM1C but only 0.167 for HumM1C→MusMOp
- scETM currently handles only a single categorical batch variable; multi-batch-variable and numerical (e.g. subject age) batch correction is future work
- Zero-shot transfer requires pre-aligned gene lists; for unaligned datasets, prepare_for_transfer discards source-unique gene parameters and randomly initializes target-unique ones, so fine-tuning is usually required before evaluation

## Evidence

- [methods] Zero-shot transfer to target dataset: "Directly apply MP-trained model on HP, storing cell embeddings to hp.obsm['delta'] (zero-shot transfer)."
- [readme] Training configuration: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [results] Cross-species transfer result: [evidence span withheld — anchor status: not_found]
- [readme] Alignment requirement: "the source and target dataset are pre-aligned, meaning they have the same / homologous gene **lists** (note that the order of the genes would have to be aligned)."
- [abstract] Scalability and transfer claim: "scETM is scalable to over 10 6 cells and confers remarkable cross-tissue and cross-species zero-shot transfer-learning performance."
