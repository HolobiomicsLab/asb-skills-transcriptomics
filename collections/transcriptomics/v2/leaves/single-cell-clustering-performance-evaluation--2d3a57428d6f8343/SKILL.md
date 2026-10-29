---
name: single-cell-clustering-performance-evaluation
description: Use when a trained scETM or p-scETM model has produced cell embeddings
  and you must quantify clustering quality (e.g., ARI/NMI on 'cell_types' labels),
  compare against competing integration tools, or check that a regularized variant
  is not compromised.
when_to_use_negative:
- No ground-truth cell type annotations ('cell_types') are available in adata.obs,
  so embedding clustering cannot be scored against reference labels
- The model has not converged — e.g., small datasets trained for fewer than ~6k gradient
  updates — making any clustering evaluation premature
- The goal is pathway-topic interpretation only (Fig 7, Supplementary Tables 16-17)
  rather than clustering benchmarking, in which case scoring pathway-topic associations
  from alpha and rho is the appropriate skill
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Provides UnsupervisedTrainer, get_all_embeddings_and_nll and the evaluate()
    function used to cluster learned cell embeddings (obsm['delta']) and score clustering
    against cell_types annotations
  repo: github.com/hui2000ji/scETM
- name: Seurat
  role: Competing integration/clustering baseline benchmarked against scETM clustering
    performance
- name: Harmony
  role: Competing batch-correction and clustering baseline in the scETM benchmarking
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Competing integration method included in the clustering benchmarking scripts
    shipped with the scETM repository
  repo: https://github.com/brianhie/scanorama
- name: LIGER
  role: Competing integrative NMF baseline for clustering comparison
  repo: https://github.com/welch-lab/liger
- name: scVI
  role: Competing deep-generative integration baseline for clustering comparison
  repo: https://github.com/YosefLab/scvi-tools
- name: scVAE-GM
  role: Competing VAE clustering baseline in the benchmark
  repo: https://github.com/scvae/scvae
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:f9cca434e976bc3acbf8271fbc4257e8bedc61e0f61863d33b1a9c8e9b22180b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - LIGER
  - Scanorama
  - Seurat
  - scETM
  - scVAE-GM
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-cell-clustering-performance-evaluation

## Summary

Evaluate scETM/p-scETM clustering quality on AnnData single-cell datasets using the scETM `evaluate` function on learned cell embeddings (delta/theta), benchmarking against baselines such as Seurat, Harmony, Scanorama, LIGER, scVI and scVAE-GM, and verifying that pathway-informed variants do not degrade clustering (Supplementary Table 15).

## When to use

Use when a trained scETM or p-scETM model has produced cell embeddings and you must quantify clustering quality (e.g., ARI/NMI on 'cell_types' labels), compare against competing integration tools, or check that a regularized variant is not compromised.

## When NOT to use

- No ground-truth cell type annotations ('cell_types') are available in adata.obs, so embedding clustering cannot be scored against reference labels
- The model has not converged — e.g., small datasets trained for fewer than ~6k gradient updates — making any clustering evaluation premature
- The goal is pathway-topic interpretation only (Fig 7, Supplementary Tables 16-17) rather than clustering benchmarking, in which case scoring pathway-topic associations from alpha and rho is the appropriate skill

## Inputs

- AnnData cells-by-genes matrix with batch information in adata.obs['batch_indices'] and cell type identity in adata.obs['cell_types'] (e.g., HumanPancreas.h5ad, MousePancreas.h5ad, HP/AD/MDD datasets)
- Trained scETM or p-scETM model checkpoint from UnsupervisedTrainer (ckpt_dir outputs)
- For p-scETM comparison: the unregularized scETM baseline clustering metrics (Supplementary Table 15)

## Outputs

- Clustering performance metrics (e.g., ARI/NMI-style scores) computed from cell embeddings versus 'cell_types' annotations
- Embedding plot files (e.g., figures/scETM_MP) written by evaluate()
- Comparative clustering table showing p-scETM is not degraded relative to baseline scETM (Supplementary Table 15)

## How to apply

After training with UnsupervisedTrainer (e.g., n_epochs=12000, eval_every=1000, n_samplers=4, ensuring at least 6k gradient updates for small datasets), call model.get_all_embeddings_and_nll(adata) to populate adata.obsm['delta'] (unnormalized cell embeddings), adata.obsm['theta'] (normalized), adata.varm['rho'] and adata.uns['alpha']. Then call evaluate(adata, embedding_key='delta', plot_fname=..., plot_dir=...) so scETM clusters the cell embeddings and scores them against the 'cell_types' column of adata.obs. For p-scETM (trainable_gene_emb_dim=0, gene embedding fixed to the pathDIP4 pathways-by-genes matrix filtered to pathways with >5 genes), run the same evaluation and compare clustering metrics to the unregularized scETM baseline to confirm fixing rho does not degrade performance (Supplementary Table 15 criterion). Rerun with identical seeds/checkpointing via UnsupervisedTrainer to keep comparisons reproducible, and benchmark competing tools (Seurat, Harmony, Scanorama, LIGER, scVI, scVAE-GM) on the same HP/AD/MDD or MousePancreas/HumanPancreas inputs.

## Related tools

- **scETM** (Provides UnsupervisedTrainer, get_all_embeddings_and_nll and the evaluate() function used to cluster learned cell embeddings (obsm['delta']) and score clustering against cell_types annotations) — github.com/hui2000ji/scETM
- **Seurat** (Competing integration/clustering baseline benchmarked against scETM clustering performance)
- **Harmony** (Competing batch-correction and clustering baseline in the scETM benchmarking) — https://github.com/immunogenomics/harmony
- **Scanorama** (Competing integration method included in the clustering benchmarking scripts shipped with the scETM repository) — https://github.com/brianhie/scanorama
- **LIGER** (Competing integrative NMF baseline for clustering comparison) — https://github.com/welch-lab/liger
- **scVI** (Competing deep-generative integration baseline for clustering comparison) — https://github.com/YosefLab/scvi-tools
- **scVAE-GM** (Competing VAE clustering baseline in the benchmark) — https://github.com/scvae/scvae

## Examples

```
from scETM import scETM, UnsupervisedTrainer, evaluate; trainer = UnsupervisedTrainer(model, mp, train_instance_name="MP", ckpt_dir="../results"); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_all_embeddings_and_nll(mp); evaluate(mp, embedding_key="delta", plot_fname="scETM_MP", plot_dir="figures/scETM_MP")
```

## Evaluation signals

- evaluate() runs without error on an AnnData whose obsm['delta'] was populated by model.get_all_embeddings_and_nll(adata) and whose obs contains 'cell_types'
- p-scETM clustering metrics match the Supplementary Table 15 criterion: fixing rho to pathway annotations does not degrade clustering versus unregularized scETM
- Training met the convergence rule of at least 6k gradient updates (e.g., n_epochs=12000, eval_every=1000 on small datasets like MousePancreas with 1886 cells)
- The embedding plot (e.g., figures/scETM_MP) is produced with the correct plot_fname/plot_dir and shows batch-corrected, cell-type-separated embeddings
- Baseline comparisons (Harmony, Scanorama, LIGER, scVI, scVAE-GM, Seurat) are run on the same AnnData with identical batch/cell-type column conventions

## Limitations

- scETM's current model handles only a single categorical batch variable; multi-batch-variable correction is future work
- For small datasets and large mini-batch sizes, one epoch can equal one gradient step, so too few epochs leads to unconverged embeddings and misleading clustering scores
- Benchmarking coverage of competing tools (R-based Seurat/Harmony/LIGER vs Python scVI/Scanorama) requires harmonized inputs and metric definitions across languages

## Evidence

- [readme] Trained model evaluated with scETM evaluate() on cell embeddings scored against cell_types: "# Evaluate the model and save the embedding plot
evaluate(mp, embedding_key="delta", plot_fname="scETM_MP", plot_dir="figures/scETM_MP")"
- [readme] Convergence rule of at least 6k gradient updates before evaluating clustering: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
- [abstract] p-scETM must not degrade clustering relative to unregularized scETM baseline (Supplementary Table 15): [evidence span withheld — anchor status: not_found]
- [readme] Batch and cell type annotation columns expected by the evaluation pipeline: "By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column."
- [intro] Competing integration methods benchmarked for clustering quality: "Harmony28 iterates between maximum diversity clustering and a linear batch correction based on the mixture-of-experts model."
