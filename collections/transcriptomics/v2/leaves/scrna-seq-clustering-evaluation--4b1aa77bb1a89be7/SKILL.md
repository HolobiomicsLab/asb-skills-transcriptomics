---
name: scrna-seq-clustering-evaluation
description: Use when cell embeddings from a scRNA-seq integration or clustering model
  (AnnData with 'batch_indices' and 'cell_types' obs columns) must be quantitatively
  benchmarked via ARI/NMI clustering scores and kBET batch-mixing acceptance rates.
when_to_use_negative:
- Input lacks ground-truth cell type annotations ('cell_types' column), so ARI/NMI
  against reference labels cannot be computed
- The task is supervised cell-type classification or label transfer rather than unsupervised
  clustering evaluation of learned embeddings
- Only a single batch is present and no batch-effect correction is needed, making
  kBET batch-mixing assessment uninformative
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Method under evaluation; provides 'delta'/'theta' cell embeddings scored by
    evaluate() for ARI/NMI
  repo: github.com/hui2000ji/scETM
- name: scVI
  role: Baseline producing cell embeddings benchmarked on the same six datasets
  repo: https://github.com/YosefLab/scvi-tools
- name: Seurat
  role: Baseline integration method compared via ARI and kBET
- name: Harmony
  role: Baseline batch-correction method compared via ARI and kBET
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Baseline integration method compared via ARI and kBET
  repo: https://github.com/brianhie/scanorama
- name: scVAE-GM
  role: Baseline Gaussian-mixture VAE clustering method compared via ARI and kBET
  repo: https://github.com/scvae/scvae
- name: LIGER
  role: Baseline integrative-NMF method compared via ARI and kBET
  repo: https://github.com/welch-lab/liger
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:8b027482d57ef83b947b03e1d829a8990c26f61cc5426d99e2b83e46692a6cf2
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# scrna-seq-clustering-evaluation

## Summary

Evaluate unsupervised clustering quality of single-cell embeddings (e.g. scETM's 'delta' cell embeddings) by computing ARI and NMI against ground-truth 'cell_types' labels, and batch mixing via kBET acceptance rates, as done in the scETM benchmark against seven baselines (scVI, scVI-LD, Seurat, scVAE-GM, Scanorama, Harmony, LIGER).

## When to use

Use when cell embeddings from a scRNA-seq integration or clustering model (AnnData with 'batch_indices' and 'cell_types' obs columns) must be quantitatively benchmarked via ARI/NMI clustering scores and kBET batch-mixing acceptance rates.

## When NOT to use

- Input lacks ground-truth cell type annotations ('cell_types' column), so ARI/NMI against reference labels cannot be computed
- The task is supervised cell-type classification or label transfer rather than unsupervised clustering evaluation of learned embeddings
- Only a single batch is present and no batch-effect correction is needed, making kBET batch-mixing assessment uninformative

## Inputs

- Cells-by-genes scRNA-seq AnnData objects (.h5ad) with 'batch_indices' and 'cell_types' obs annotations (MP, HP, TM, AD, MDD, MR datasets)
- Trained model cell embeddings stored in adata.obsm['delta'] (unnormalized) and adata.obsm['theta'] (normalized)
- Baseline method outputs from /scripts folder of the scETM repository (scVI, scVI-LD, Seurat, scVAE-GM, Scanorama, Harmony, LIGER)

## Outputs

- ARI and NMI clustering scores per method-dataset pair (Table 1 reproduction)
- kBET batch-mixing acceptance rates per method-dataset pair (Table 2 reproduction)
- benchmark_metrics.csv with ARI, NMI, and kBET fields for all 10 method variants across six datasets
- Embedding visualization plots (e.g. figures/scETM_MP)

## How to apply

Load each dataset as a cells-by-genes AnnData object (e.g. anndata.read_h5ad('MousePancreas.h5ad')) with batch information in adata.obs['batch_indices'] and cell type labels in adata.obs['cell_types']. Train scETM with scETM(n_vars, n_batches, enable_batch_bias=True) using UnsupervisedTrainer at n_epochs=12000, eval_every=1000, n_samplers=4, ensuring at least ~6k gradient updates on small datasets (e.g. the 1886-cell MousePancreas), and repeat with enable_batch_bias=False (scETM-λ ablation) and with the adversarial variant (scETM+adv). Extract embeddings with model.get_all_embeddings_and_nll, then call evaluate(adata, embedding_key='delta') to compute ARI and NMI against ground-truth cell types and save an embedding plot. Run the seven baselines (scVI, scVI-LD, Seurat, scVAE-GM, Scanorama, Harmony, LIGER) with the scripts in the scETM repository's /scripts folder, compute kBET acceptance rates for every method-dataset pair, and assemble all metrics into a single benchmark_metrics.csv covering the six datasets (MP, HP, TM, AD, MDD, MR). Validate by checking that reproduced scETM ARIs are consistent with published values (e.g. 0.946/0.943/0.761/0.996/0.717/0.859 across MP/HP/TM/AD/MDD/MR).

## Related tools

- **scETM** (Method under evaluation; provides 'delta'/'theta' cell embeddings scored by evaluate() for ARI/NMI) — github.com/hui2000ji/scETM
- **scVI** (Baseline producing cell embeddings benchmarked on the same six datasets) — https://github.com/YosefLab/scvi-tools
- **Seurat** (Baseline integration method compared via ARI and kBET)
- **Harmony** (Baseline batch-correction method compared via ARI and kBET) — https://github.com/immunogenomics/harmony
- **Scanorama** (Baseline integration method compared via ARI and kBET) — https://github.com/brianhie/scanorama
- **scVAE-GM** (Baseline Gaussian-mixture VAE clustering method compared via ARI and kBET) — https://github.com/scvae/scvae
- **LIGER** (Baseline integrative-NMF method compared via ARI and kBET) — https://github.com/welch-lab/liger

## Examples

```
model.get_all_embeddings_and_nll(mp); evaluate(mp, embedding_key='delta', plot_fname='scETM_MP', plot_dir='figures/scETM_MP')
```

## Evaluation signals

- benchmark_metrics.csv exists and contains ARI, NMI, and kBET fields for all 10 method variants (scETM, scETM-λ, scETM+adv, and 7 baselines) across the six datasets (MP, HP, TM, AD, MDD, MR)
- Reproduced scETM ARI values are consistent with published Table 1 values (0.946/0.943/0.761/0.996/0.717/0.859)
- Training converged: at least ~6k gradient updates were issued on small datasets (e.g. 12000 epochs for 1886-cell MousePancreas at batch size 2000)
- Embeddings are stored in the expected slots: adata.obsm['delta'] (unnormalized), adata.obsm['theta'] (normalized), adata.varm['rho'], adata.uns['alpha']
- evaluate() completed with embedding_key='delta' and produced the embedding plot files in the specified plot_dir

## Limitations

- scETM is trained via batched gradient descent; on small datasets one epoch can be a single gradient step, so too-few epochs cause non-convergence and misleadingly low ARI/NMI
- Only ARI (not NMI) is the headline clustering metric in the benchmark; Harmony and Seurat obtain slightly higher ARIs at the cost of model transferability, interpretability, and/or scalability
- scETM's current model handles only a single categorical batch variable; continuous/numerical batch variables and multi-batch-variable correction are not supported

## Evidence

- [readme] By default, scETM looks for batch information in the 'batch_indices' column of the adata.obs DataFrame, and cell type identity in the 'cell_types' column.: "By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column."
- [readme] it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model."
- [methods] evaluate(mp, embedding_key="delta", plot_fname="scETM_MP", plot_dir="figures/scETM_MP"): "evaluate(mp, embedding_key="delta", plot_fname="scETM_MP", plot_dir="figures/scETM_MP")"
- [intro] model transferrability is an important consideration. We consider a model as transferable if the learned knowledge manifested as the model parameters could benefit future data modeling.: "model transferrability is an important consideration. We consider a model as transferable if the learned knowledge manifested as the model parameters could benefit"
