---
name: lisi-integration-metric-computation
description: Use when comparing single-cell batch integration methods on multi-sample,
  multi-condition data (e.g. pancreas + lung mixtures) and you need quantitative evidence
  of batch removal (iLISI) without over-correction (cLISI, condition separation, condition-specific
  DE genes).
when_to_use_negative:
- Single-sample, single-condition datasets with no batch_label variable — iLISI measures
  batch mixing and has no technical confound to quantify
- When the goal is gene-level interpretability of what drives integration — the article
  notes 'such metrics do not provide insight into the genes or features driving integration
  and clustering'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: JOINTLY
  role: Primary integration method whose embeddings are scored for batch removal and
    over-correction robustness
  repo: https://github.com/madsen-lab/rJOINTLY
- name: fastMNN
  role: Comparison integration method producing embeddings evaluated with the same
    iLISI/cLISI and ARI/NMI framework
- name: Harmony
  role: Comparison integration method benchmarked on within-tissue mixing vs across-tissue
    retention
- name: scVI
  role: Comparison integration method noted to retain biological variation across
    conditions while removing batch effects
- name: LIGER
  role: Comparison NMF-based integration method in the multi-condition mixture evaluation
- name: Scanorama
  role: Comparison integration method in the over-correction benchmark
- name: Seurat
  role: Provides FindNeighbors/FindClusters (dims 1:15) and RunUMAP to derive cluster
    labels and embeddings from each integration reduction
  repo: https://github.com/satijalab/seurat
- name: aricode
  role: Computes ARI and NMI between cell-type labels and cluster labels as the quantitative
    evaluation
- name: UCell
  role: AddModuleScore_UCell scores the interferon-signalling module to verify across-tissue
    signal retention
  repo: https://github.com/carmonalab/UCell
- name: presto
  role: wilcoxauc Wilcoxon rank-sum test identifies the 3529 pancreas-vs-lung endothelial
    DE genes used as the over-correction readout
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:26b790c05017486c68e63dc88c102a3d1451c45377b6a7e4444cfebf5bfd0e2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - Harmony
  - JOINTLY
  - LIGER
  - Scanorama
  - Seurat
  - UCell
  - aricode
  - fastMNN
  - presto
  - scVI
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lisi-integration-metric-computation

## Summary

Computes iLISI/cLISI-style integration scores plus ARI/NMI cluster-label agreement to quantify whether a batch-integration method (JOINTLY, Harmony, fastMNN, scVI, LIGER, Scanorama) removes within-tissue batch effects while retaining across-tissue biological variability, i.e. resists over-correction.

## When to use

Use when comparing single-cell batch integration methods on multi-sample, multi-condition data (e.g. pancreas + lung mixtures) and you need quantitative evidence of batch removal (iLISI) without over-correction (cLISI, condition separation, condition-specific DE genes).

## When NOT to use

- Single-sample, single-condition datasets with no batch_label variable — iLISI measures batch mixing and has no technical confound to quantify
- When the goal is gene-level interpretability of what drives integration — the article notes 'such metrics do not provide insight into the genes or features driving integration and clustering'

## Inputs

- Merged Seurat/SingleCellExperiment object with batch_label and tissue/condition variables (e.g. pancreas-lung mixture from Zenodo record 8298157)
- Cell type label table (e.g. Human_Liver_labels.tsv with Transferred_labels column)
- Integration method embeddings/reductions (JOINTLY, fastMNN, Harmony, scVI, LIGER, Scanorama)
- Gene list for condition-specific modules (e.g. 3529 pancreas- vs lung-endothelial DE genes, interferon-signalling signature)

## Outputs

- Per-method iLISI/cLISI within-tissue integration and over-correction scores
- ARI and NMI values between celltype labels and seurat_clusters
- Condition-separation assessment (interferon module scores per cell, DE-gene separability across embeddings)
- UMAP plots colored by cluster, cell type and batch for Fig. 3A-F-style comparison

## How to apply

Merge the donor/tissue datasets (e.g. Lung and Pancreas from Zenodo record 8298157) into one Seurat object with a batch_label variable, run each integration method, and cluster each embedding with FindNeighbors/FindClusters on the reduction's dims 1:15. For every method, compute iLISI/cLISI to score within-tissue batch mixing versus cell-type mixing, and compute ARI/NMI between cell-type labels and seurat_clusters (aricode::ARI, aricode::NMI) in the same framework as the JOINTLY benchmark. Assess over-correction specifically by checking whether across-tissue biological signal survives: differential expression between pancreas- and lung-derived endothelial cells (3529 genes, wilcoxauc Wilcoxon rank-sum) and interferon-signalling AddModuleScore_UCell scores should remain separable. Accept the integration (per the task validation) only if iLISI/cLISI improve while the interferon module scores and the 3529 DE genes stay condition-separable; repeat runs are advisable since results vary between seeds (running JOINTLY up to 5 times is suggested).

## Related tools

- **JOINTLY** (Primary integration method whose embeddings are scored for batch removal and over-correction robustness) — https://github.com/madsen-lab/rJOINTLY
- **fastMNN** (Comparison integration method producing embeddings evaluated with the same iLISI/cLISI and ARI/NMI framework)
- **Harmony** (Comparison integration method benchmarked on within-tissue mixing vs across-tissue retention)
- **scVI** (Comparison integration method noted to retain biological variation across conditions while removing batch effects)
- **LIGER** (Comparison NMF-based integration method in the multi-condition mixture evaluation)
- **Scanorama** (Comparison integration method in the over-correction benchmark)
- **Seurat** (Provides FindNeighbors/FindClusters (dims 1:15) and RunUMAP to derive cluster labels and embeddings from each integration reduction) — https://github.com/satijalab/seurat
- **aricode** (Computes ARI and NMI between cell-type labels and cluster labels as the quantitative evaluation)
- **UCell** (AddModuleScore_UCell scores the interferon-signalling module to verify across-tissue signal retention) — https://github.com/carmonalab/UCell
- **presto** (wilcoxauc Wilcoxon rank-sum test identifies the 3529 pancreas-vs-lung endothelial DE genes used as the over-correction readout) — https://github.com/immunogenomics/presto

## Examples

```
liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15); liver$object <- FindClusters(liver$object, res = 0.05); aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)
```

## Evaluation signals

- iLISI/cLISI improve relative to unintegrated control, indicating within-tissue batch removal
- ARI/NMI between celltype and seurat_clusters reaches values comparable to the reference benchmark (e.g. ARI ≈ 0.74, NMI ≈ 0.63 on the human liver test set)
- Interferon-signalling module scores (AddModuleScore_UCell) remain separable between pancreas- and lung-derived endothelial cells
- The 3529 wilcoxauc DE genes between pancreas- and lung-derived endothelial cells remain distinguishable in the integrated embedding
- Per-method comparison shows the method's balance is task-dependent — re-run the full metric suite for each new integration task rather than extrapolating

## Limitations

- No method is always the best across all tasks; performance of a chosen method must be re-evaluated on any new integration task
- Integration metrics (iLISI/cLISI, ARI/NMI) do not reveal the genes or features driving integration and clustering
- For multi-condition, multi-sample datasets, consensus PCA in JOINTLY is advised against — use regular PCA — and random seeds matter: results differ between runs, sometimes requiring up to 5 JOINTLY runs choosing the best solution

## Evidence

- [intro] over-correction evaluation rationale: "We evaluate the robustness against over-correction by integrating multi-donor datasets from different tissues and find that JOINTLY removes within-tissue batch effects but retains across-tissue"
- [discussion] metrics must be re-evaluated per task: "no method is always the best across all tasks, and therefore, that it is critical to evaluate the performance of a chosen method on any new integration task."
- [discussion] metrics lack gene-level interpretability: "such metrics do not provide insight into the genes or features driving integration and clustering."
- [methods] ARI/NMI computation on clusters: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) # 0.7415981"
- [methods] module score readout for biological retention: "liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
