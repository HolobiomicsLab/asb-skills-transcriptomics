---
name: kbet-batch-mixing-assessment
description: Use when benchmarking batch-correction or integration methods on scRNA-seq
  embeddings and you need a quantitative batch-mixing score (kBET acceptance rate)
  to accompany cell-type clustering metrics like ARI/NMI/ASW.
when_to_use_negative:
- Input has only one batch (no batch_indices variable) — kBET cannot measure mixing
  without batch labels
- You only need clustering accuracy and have no batch-effect concern — ARI/NMI alone
  suffices and kBET adds no signal
- Evaluating embeddings whose biological structure is unknown — kBET alone rewards
  over-correction and must be paired with ASW-beta/ARI as in the HP beta-cell-removal
  test
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Provides the cell embeddings and the evaluate() workflow on which kBET batch-mixing
    acceptance rates are computed for benchmarking
  repo: github.com/hui2000ji/scETM
- name: Harmony
  role: Baseline integration method whose embeddings are scored with kBET for batch
    mixing
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Baseline integration method whose embeddings are scored with kBET for batch
    mixing
  repo: https://github.com/brianhie/scanorama
- name: scVI
  role: Baseline deep generative method whose embeddings are scored with kBET for
    batch mixing
  repo: https://github.com/scverse/scvi-tools
- name: LIGER
  role: Baseline integrative NMF method whose embeddings are scored with kBET, exemplifying
    over-correction (kBET 0.5978) that compromises ASW-beta
  repo: https://github.com/welch-lab/liger
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:031ae39128151875ce570a35db8413e5c2795a112220c3be597ec9036ab878cc
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# kbet-batch-mixing-assessment

## Summary

Quantifies batch mixing in single-cell integration embeddings using kBET acceptance rates, so that batch-effect correction (e.g. scETM, Harmony, LIGER, Scanorama) can be benchmarked alongside clustering metrics such as ARI/NMI rather than judged on clustering alone.

## When to use

Use when benchmarking batch-correction or integration methods on scRNA-seq embeddings and you need a quantitative batch-mixing score (kBET acceptance rate) to accompany cell-type clustering metrics like ARI/NMI/ASW.

## When NOT to use

- Input has only one batch (no batch_indices variable) — kBET cannot measure mixing without batch labels
- You only need clustering accuracy and have no batch-effect concern — ARI/NMI alone suffices and kBET adds no signal
- Evaluating embeddings whose biological structure is unknown — kBET alone rewards over-correction and must be paired with ASW-beta/ARI as in the HP beta-cell-removal test

## Inputs

- Cells-by-genes AnnData (.h5ad) objects with adata.obs['batch_indices'] and adata.obs['cell_types']
- Cell embeddings from each integration method (e.g. obsm['delta'] produced by scETM's get_all_embeddings_and_nll or get_cell_embeddings_and_nll)
- Published reference kBET values (Table 2, Supplementary Table 5) for validation

## Outputs

- Per method-dataset kBET batch-mixing acceptance rates
- benchmark_metrics.csv combining ARI, NMI, ASW, ASW-beta and kBET for all method variants across the six datasets
- transfer_benchmark_results.csv with per-task ARI and kBET for cross-tissue/cross-species transfer tasks

## How to apply

Prepare each dataset as a cells-by-genes AnnData (.h5ad) object with batch labels in adata.obs['batch_indices'] and cell types in adata.obs['cell_types'], since scETM's evaluate workflow reads batch information from these columns. After each method (scETM, scETM−λ, scETM+adv, scVI, scVI-LD, Seurat, scVAE-GM, Scanorama, Harmony, LIGER) produces cell embeddings (e.g. stored in obsm['delta'] via model.get_all_embeddings_and_nll), run kBET on the embedding's k-nearest-neighbor graph to compute the batch-mixing acceptance rate per method-dataset pair. Assemble kBET values with ARI/NMI into a single benchmark_metrics.csv covering all method variants across the six datasets (MP, HP, TM, AD, MDD, MR), or into transfer_benchmark_results.csv for zero-shot transfer tasks. Judge correctness by comparing reproduced scETM kBET values against published references (e.g. kBET 0.1247 for scETM on the beta-cell-removed Human Pancreas test; kBET 0.339 for scETM TM(FACS)→MP transfer). Interpret kBET jointly with ARI and ASW-beta: high kBET with poor ASW-beta (as for LIGER, kBET 0.5978, in the beta-cell-removal test) indicates over-correction that erases biological structure.

## Related tools

- **scETM** (Provides the cell embeddings and the evaluate() workflow on which kBET batch-mixing acceptance rates are computed for benchmarking) — github.com/hui2000ji/scETM
- **Harmony** (Baseline integration method whose embeddings are scored with kBET for batch mixing) — https://github.com/immunogenomics/harmony
- **Scanorama** (Baseline integration method whose embeddings are scored with kBET for batch mixing) — https://github.com/brianhie/scanorama
- **scVI** (Baseline deep generative method whose embeddings are scored with kBET for batch mixing) — https://github.com/scverse/scvi-tools
- **LIGER** (Baseline integrative NMF method whose embeddings are scored with kBET, exemplifying over-correction (kBET 0.5978) that compromises ASW-beta) — https://github.com/welch-lab/liger

## Examples

```
from scETM import evaluate; evaluate(mp, embedding_key="delta", plot_fname="scETM_MP", plot_dir="figures/scETM_MP")
```

## Evaluation signals

- Reproduced scETM kBET matches published values, e.g. ~0.1247 on the beta-cell-removed Human Pancreas benchmark (Supplementary Table 5) and ~0.339 for TM(FACS)→MP transfer
- benchmark_metrics.csv contains ARI, NMI, and kBET fields for all 10 method variants across the six datasets
- kBET acceptance rates fall in [0,1] with higher values indicating better batch mixing
- Methods with the highest kBET (e.g. LIGER 0.5978) show degraded ASW-beta, confirming the metric is being interpreted jointly with cell-type preservation

## Limitations

- kBET rewards aggressive batch mixing; over-correction can destroy missing cell-type structure (LIGER and scETM+adv compromised ASW-beta in the beta-cell-removal test), so it must be reported alongside ASW-beta and ARI
- Requires a single categorical batch variable in adata.obs['batch_indices']; scETM/kBET do not directly handle numerical or multiple categorical batch variables
- Small datasets fit in one mini-batch produce few gradient updates per epoch, so poorly converged embeddings (fewer than ~6k gradient updates) give unstable kBET values

## Evidence

- [intro] batch effects impact clustering: "Changes in gene expression due to experimental design, often referred to as batch effects18, can have a large impact on clustering12,18–20."
- [readme] batch indices column: "By default, scETM looks for batch information in the 'batch_indices' column of the `adata.obs` DataFrame, and cell type identity in the 'cell_types' column."
- [readme] evaluate workflow: "Evaluate the model and save the embedding plot
evaluate(mp, embedding_key="delta", plot_fname="scETM_MP", plot_dir="figures/scETM_MP")"
- [readme] convergence guidance: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model."
