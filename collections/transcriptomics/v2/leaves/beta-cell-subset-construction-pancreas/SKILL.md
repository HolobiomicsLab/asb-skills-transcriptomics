---
name: beta-cell-subset-construction-pancreas
description: Use when benchmarking scRNA-seq integration methods (scETM, Harmony,
  Scanorama, Seurat, LIGER, scVI, scVAE-GM) for over-correction, by removing a cell
  type (beta cells) from all Human Pancreas batches except CelSeq2.
when_to_use_negative:
- Input AnnData lacks obs['batch_indices'] or obs['cell_types'] annotations and column
  mapping (batch_col/cell_type_col) is not possible
- The goal is unbiased clustering/integration on the complete Human Pancreas dataset
  — removing beta cells changes the benchmark question to over-correction testing
- The missing cell type is not batch-restricted (removal must leave one batch with
  the cell type intact, otherwise ASW-beta cannot diagnose over-correction)
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Model trained on the beta-cell-removed subset (UnsupervisedTrainer, enable_batch_bias=True);
    its cell embeddings in obsm['delta'] are the primary evaluation target
  repo: https://github.com/hui2000ji/scETM
- name: Harmony
  role: Competing integration method run on the same modified dataset for benchmark
    comparison
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Competing integration method run on the same modified dataset for benchmark
    comparison
  repo: https://github.com/brianhie/scanorama
- name: LIGER
  role: Competing integration method (integrative NMF) whose over-correction on the
    subset is a reference finding (ARI 0.8476, kBET 0.5978)
  repo: https://github.com/welch-lab/liger
- name: scVI
  role: Competing integration method run on the same modified dataset for benchmark
    comparison
  repo: https://github.com/YosefLab/scvi-tools
- name: scVAE-GM
  role: Competing VAE-based integration method run on the same modified dataset for
    benchmark comparison
  repo: https://github.com/scvae/scvae
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:325030f5e3bf01e7753feb9d10f29ca7f9f1ee477d97a5cc3abf9f6a2328d092
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - Harmony
  - LIGER
  - Scanorama
  - Seurat
  - scETM
  - scVAE-GM
  - scVI
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# beta-cell-subset-construction-pancreas

## Summary

Constructs the beta-cell-removed Human Pancreas benchmark: beta cells are dropped from 4 of 5 batches (retained only in CelSeq2, per Supplementary Table 4) so that integration methods that over-correct batch effects fail to preserve the missing beta-cell-type structure, scored with ARI/kBET/ASW/ASW-beta against Supplementary Table 5.

## When to use

Use when benchmarking scRNA-seq integration methods (scETM, Harmony, Scanorama, Seurat, LIGER, scVI, scVAE-GM) for over-correction, by removing a cell type (beta cells) from all Human Pancreas batches except CelSeq2.

## When NOT to use

- Input AnnData lacks obs['batch_indices'] or obs['cell_types'] annotations and column mapping (batch_col/cell_type_col) is not possible
- The goal is unbiased clustering/integration on the complete Human Pancreas dataset — removing beta cells changes the benchmark question to over-correction testing
- The missing cell type is not batch-restricted (removal must leave one batch with the cell type intact, otherwise ASW-beta cannot diagnose over-correction)

## Inputs

- Human Pancreas cells-by-genes AnnData object (.h5ad) with obs['batch_indices'] (5 batches) and obs['cell_types'] annotations
- Supplementary Table 4 (beta-cell removal counts per batch) and Supplementary Table 5 (reference ARI/kBET/ASW/ASW-beta values)

## Outputs

- Beta-cell-removed Human Pancreas AnnData subset (beta cells retained only in the CelSeq2 batch)
- Cell embeddings per method (e.g., scETM in obsm['delta'])
- benchmark_metrics.csv comparing ARI, kBET, ASW, ASW-beta to Supplementary Table 5

## How to apply

Load the Human Pancreas cells-by-genes AnnData object (scETM reads batch information from obs['batch_indices'] and cell type identity from obs['cell_types']; pass batch_col/cell_type_col if stored elsewhere). Build the modified dataset per Supplementary Table 4 by dropping beta cells from all 5 batches except CelSeq2, keeping all non-beta cells unchanged. Train scETM on this subset with UnsupervisedTrainer (n_epochs=12000, eval_every=1000, n_samplers=4, enable_batch_bias=True) and its adversarial variant scETM+adv, storing cell embeddings in obsm['delta'], and run the competing methods (Harmony, Scanorama, Seurat, LIGER, scVI, scVAE-GM) from the scETM repo's benchmarking scripts on the same modified data. Compute ARI, kBET, ASW and ASW-beta for each method's embeddings via the evaluate workflow, compare to Supplementary Table 5 (scETM: ARI 0.9298, kBET 0.1247, ASW 0.3525, ASW-beta 0.5370; LIGER: ARI 0.8476, kBET 0.5978) and save benchmark_metrics.csv. Rationale: methods that aggressively mix batches (e.g., scETM+adv, LIGER) merge beta-like cells into other clusters, which shows up as compromised ASW-beta — the skill's decision point is flagging such over-correction.

## Related tools

- **scETM** (Model trained on the beta-cell-removed subset (UnsupervisedTrainer, enable_batch_bias=True); its cell embeddings in obsm['delta'] are the primary evaluation target) — https://github.com/hui2000ji/scETM
- **Harmony** (Competing integration method run on the same modified dataset for benchmark comparison) — https://github.com/immunogenomics/harmony
- **Scanorama** (Competing integration method run on the same modified dataset for benchmark comparison) — https://github.com/brianhie/scanorama
- **LIGER** (Competing integration method (integrative NMF) whose over-correction on the subset is a reference finding (ARI 0.8476, kBET 0.5978)) — https://github.com/welch-lab/liger
- **scVI** (Competing integration method run on the same modified dataset for benchmark comparison) — https://github.com/YosefLab/scvi-tools
- **scVAE-GM** (Competing VAE-based integration method run on the same modified dataset for benchmark comparison) — https://github.com/scvae/scvae

## Examples

```
import anndata; hp = anndata.read_h5ad('HumanPancreas.h5ad'); subset = hp[~((hp.obs['cell_types']=='beta') & (hp.obs['batch_indices']!=celseq2_idx))].copy()
```

## Evaluation signals

- Subset check: beta-cell count is zero in the 4 non-CelSeq2 batches and unchanged (fully retained) in CelSeq2, matching Supplementary Table 4 removal counts
- All non-beta cells are preserved: per-batch non-beta cell counts in the subset AnnData equal those in the original HumanPancreas object
- scETM metrics on the subset reproduce Supplementary Table 5 within tolerance: ARI ~0.9298, kBET ~0.1247, ASW ~0.3525, ASW-beta ~0.5370
- Over-correction diagnosis: methods with aggressive correction (scETM+adv, LIGER) show compromised ASW-beta and worse kBET (LIGER reference: ARI 0.8476, kBET 0.5978)
- benchmark_metrics.csv contains a row per method with all four metrics compared to the Supplementary Table 5 reference values

## Limitations

- scETM's current model handles only a single categorical batch variable, so the benchmark design is restricted to one batch key; multi-batch-variable correction is future work
- scETM trains via batched gradient descent — for small datasets (e.g., MousePancreas, 1886 cells), epochs must provide at least ~6k gradient updates or the model may not converge
- Results depend on training configuration (n_epochs=12000, eval_every=1000, n_samplers=4, enable_batch_bias=True) and the specific Supplementary Table 4 removal scheme; other removal patterns are not covered by the reference values

## Evidence

- [readme] Data format: batch_indices and cell_types columns: "By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column."
- [methods] scETM input format: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object"
- [methods] Training configuration used on the subset: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [methods] Model initialization with batch bias: "model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True)"
