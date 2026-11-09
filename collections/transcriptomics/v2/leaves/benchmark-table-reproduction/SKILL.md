---
name: benchmark-table-reproduction
description: Use when a paper reports head-to-head quantitative scores (ARI/ASW/kBET)
  for multiple integration methods on a shared dataset and the goal is to regenerate
  that table from raw data within rounding tolerance.
when_to_use_negative:
- The task is a novel benchmark design rather than reproducing a published table —
  the paper's fixed epochs, seeds, and scripts must be reused, not re-tuned
- The source datasets lack gold-standard cell-type annotations, since ARI/ASW cannot
  be computed against 'cell_types'
- The dataset requires multiple categorical batch variables or multi-omic integration,
  which the focal model does not support
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Focal method; trained via UnsupervisedTrainer (12000 epochs, eval_every=1000,
    n_samplers=4, enable_batch_bias=True), cell embeddings in obsm['delta']
  repo: github.com/hui2000ji/scETM
- name: LIGER
  role: Competing integrative NMF method benchmarked for comparison table; reference
    point for over-correction (highest kBET 0.176, lowest ARI 0.714)
  repo: https://github.com/welch-lab/liger
- name: Harmony
  role: Competing batch-correction method run via the scETM repo benchmark scripts
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Competing integration method run via the scETM repo benchmark scripts
  repo: https://github.com/brianhie/scanorama
- name: scVI
  role: Competing VAE-based integration method run via the scETM repo benchmark scripts
  repo: https://github.com/YosefLab/scvi-tools
- name: scVAE-GM
  role: Competing Gaussian-mixture VAE method run via the scETM repo benchmark scripts
  repo: https://github.com/scvae/scvae
- name: Seurat
  role: Competing anchor-based integration method run via the scETM repo benchmark
    scripts
- name: AnnData
  role: Input container holding the cells-by-genes matrix with 'batch_indices' and
    'cell_types' columns
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:a681b7805e537ca7970377a8d2345e5259c75f1a477ad87e2fbd66c811b914d5
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# benchmark-table-reproduction

## Summary

Reproduce a published benchmark comparison table (e.g., Supplementary Table 7 of the scETM paper) by retraining the focal method and all competing integrative methods on the same dataset and assembling their clustering and batch-mixing metrics into a single comparison table.

## When to use

Use when a paper reports head-to-head quantitative scores (ARI/ASW/kBET) for multiple integration methods on a shared dataset and the goal is to regenerate that table from raw data within rounding tolerance.

## When NOT to use

- The task is a novel benchmark design rather than reproducing a published table — the paper's fixed epochs, seeds, and scripts must be reused, not re-tuned
- The source datasets lack gold-standard cell-type annotations, since ARI/ASW cannot be computed against 'cell_types'
- The dataset requires multiple categorical batch variables or multi-omic integration, which the focal model does not support

## Inputs

- Raw scRNA-seq data from each source study (e.g., GSE63476 Macosko, GSE81905 Shekhar) with cell-level batch and cell-type annotations
- AnnData cells-by-genes object with 'batch_indices' and 'cell_types' obs columns
- Benchmark scripts from the focal repository /scripts folder (competitor method commands)
- Published reference table values (e.g., Supplementary Table 7) with tolerances

## Outputs

- Trained embedding models for every method on the combined dataset
- Per-method ARI, ASW, and kBET scores computed against gold-standard cell types and batch labels
- Comparison table file (e.g., supplementary_table7.tsv) matching reported values within tolerance
- Doublelet-insensitivity check showing metrics with and without the 669 doublelets agree

## How to apply

Assemble a single AnnData object from the raw source studies (e.g., Mouse Retina from GSE63476 Macosko and GSE81905 Shekhar), storing batches in 'batch_indices' and gold-standard labels in 'cell_types', keeping any annotated low-quality cells (669 Shekhar doublelets) present. Train the focal method with the paper's exact settings — scETM via UnsupervisedTrainer with n_epochs=12000, eval_every=1000, n_samplers=4, enable_batch_bias=True — and the adversarial variant (scETM+adv) identically; run competitors (LIGER, Harmony, Scanorama, Seurat, scVI, scVAE-GM) with the benchmark scripts shipped in the focal repo's /scripts folder. Extract each model's cell embeddings (mp.obsm['delta'] for scETM), cluster them, and compute ARI, ASW, and kBET against the gold-standard cell types and batch labels. Assemble all metrics into a table and verify the focal method's numbers (scETM ARI 0.859, ASW 0.2873, kBET 0.0656) and the competitor extremes (LIGER kBET 0.176 / ARI 0.714; scETM+adv kBET 0.1410 / ARI 0.7720) match within rounding tolerance. Finally, confirm robustness by recomputing metrics after removing the annotated doublelets and checking the values agree.

## Related tools

- **scETM** (Focal method; trained via UnsupervisedTrainer (12000 epochs, eval_every=1000, n_samplers=4, enable_batch_bias=True), cell embeddings in obsm['delta']) — github.com/hui2000ji/scETM
- **LIGER** (Competing integrative NMF method benchmarked for comparison table; reference point for over-correction (highest kBET 0.176, lowest ARI 0.714)) — https://github.com/welch-lab/liger
- **Harmony** (Competing batch-correction method run via the scETM repo benchmark scripts) — https://github.com/immunogenomics/harmony
- **Scanorama** (Competing integration method run via the scETM repo benchmark scripts) — https://github.com/brianhie/scanorama
- **scVI** (Competing VAE-based integration method run via the scETM repo benchmark scripts) — https://github.com/YosefLab/scvi-tools
- **scVAE-GM** (Competing Gaussian-mixture VAE method run via the scETM repo benchmark scripts) — https://github.com/scvae/scvae
- **Seurat** (Competing anchor-based integration method run via the scETM repo benchmark scripts)
- **AnnData** (Input container holding the cells-by-genes matrix with 'batch_indices' and 'cell_types' columns)

## Examples

```
model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); trainer = UnsupervisedTrainer(model, mp); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_all_embeddings_and_nll(mp)
```

## Evaluation signals

- supplementary_table7.tsv reproduces reported values within rounding tolerance: scETM ARI 0.859, ASW 0.2873, kBET 0.0656
- LIGER values match the reported over-correction pattern (kBET 0.176, ARI 0.714)
- scETM+adv matches reported trade-off (kBET 0.1410, ARI 0.7720)
- ARI/ASW/kBET computed with and without the 669 annotated Shekhar doublelets agree
- Every method row in the table was trained with the paper's exact settings (e.g., n_epochs=12000, eval_every=1000, n_samplers=4, enable_batch_bias=True)

## Limitations

- Competitor results depend on using the exact benchmark commands in the scETM repo's /scripts folder; deviations may prevent matching the table within tolerance
- scETM training assumes ≥6k gradient updates — for small datasets with large mini-batch sizes the epoch count must be raised or the model may be unconverged
- Published metrics are subject to rounding; only tolerance-level agreement is expected

## Evidence

- [readme] scETM benchmark settings: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [readme] benchmark scripts: "This repository includes detailed instructions for installation and requirements, demos, and scripts used for the benchmarking of 7 other state-of-art methods."
- [readme] batch and cell type columns: "scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column"
- [methods] embedding extraction: "model.get_all_embeddings_and_nll(mp)"
- [readme] convergence guidance: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
- [intro] batch effect motivation: "Changes in gene expression due to experimental design, often referred to as batch effects18, can have a large impact on clustering"
