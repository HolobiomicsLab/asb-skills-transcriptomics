---
name: clustering-metric-computation-nmi-ari-asw
description: Use when evaluating predicted single-cell cluster assignments against
  ground-truth cell type labels, or comparing learned cell embeddings of scMGCA vs
  baselines (scziDesk, scDeepCluster, DCA, DEC, DESC, scVI, scCAEs, scGNN, scGAE,
  GraphSCC, Seurat, SHARP).
when_to_use_negative:
- No ground-truth cell type labels exist for the dataset — NMI/ARI against true labels
  cannot be computed (only ASW on embeddings is possible)
- You are scoring raw count matrices rather than a method's learned embeddings — ASW
  depends on the latent representation each method produces
- Evaluating marker-gene or pathway analysis outputs (e.g. PDAC differential expression)
  rather than cluster assignments
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: Computes NMI and ARI between predicted and ground-truth cell type labels
- name: scMGCA
  role: Method under evaluation whose predicted labels and latent embeddings are scored
    with NMI/ARI/ASW
  repo: https://pypi.org/project/scMGCA
- name: SHARP
  role: Baseline clustering method whose output is validated with ARI against reference
    labels (provides a runnable ARI example)
  repo: https://github.com/shibiaowan/SHARP
- name: scziDesk
  role: Baseline deep soft K-means clustering method scored in the same benchmark
    table
  repo: https://github.com/xuebaliang/scziDesk
- name: scDeepCluster
  role: Baseline model-based deep embedding clustering method scored in the same benchmark
    table
  repo: https://github.com/ttgump/scDeepCluster
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:1870e9aeb517e96211b77a0054d2aa34c4e8343712a2f01366418d8c7f725ae6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - DESC
  - SCANPY
  - SHARP
  - Seurat
  - anndata
  - numpy
  - pandas
  - scCAEs
  - scDeepCluster
  - scMGCA
  - scVI
  - scikit-learn
  - scipy
  - scziDesk
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# clustering-metric-computation-nmi-ari-asw

## Summary

Computes the three clustering quality metrics (NMI, ARI, ASW) used by scMGCA to benchmark single-cell clustering methods against ground-truth cell type labels across 20 real scRNA-seq datasets from multiple platforms.

## When to use

Use when evaluating predicted single-cell cluster assignments against ground-truth cell type labels, or comparing learned cell embeddings of scMGCA vs baselines (scziDesk, scDeepCluster, DCA, DEC, DESC, scVI, scCAEs, scGNN, scGAE, GraphSCC, Seurat, SHARP).

## When NOT to use

- No ground-truth cell type labels exist for the dataset — NMI/ARI against true labels cannot be computed (only ASW on embeddings is possible)
- You are scoring raw count matrices rather than a method's learned embeddings — ASW depends on the latent representation each method produces
- Evaluating marker-gene or pathway analysis outputs (e.g. PDAC differential expression) rather than cluster assignments

## Inputs

- Predicted cluster label vector for each cell (from scMGCA or one of the 12 baseline methods)
- Ground-truth cell type label vector for each cell from the 20 real scRNA-seq datasets
- Method-specific learned low-dimensional cell embeddings (needed for ASW)

## Outputs

- Per-dataset NMI, ARI, and ASW values for each of the 13 methods
- Row-wise mean NMI, mean ARI, and mean ASW across the 20 datasets (assembled into clustering_benchmark_results.csv)

## How to apply

For each of the 13 methods (scMGCA plus 12 baselines) run on the same preprocessed datasets (genes not expressed filtered, normalized matrix, top d=500 highly variable genes), compute NMI and ARI with scikit-learn by comparing predicted labels to the ground-truth cell type labels. Compute ASW (average silhouette width) on each method's learned embeddings rather than raw counts. Assemble per-dataset scores plus row-wise means across all 20 datasets into a single table (clustering_benchmark_results.csv); the reference result is scMGCA reaching mean NMI 0.8304, ARI 0.8278, and ASW 0.5827, with the highest per-dataset values on 15 of the 20 datasets.

## Related tools

- **scikit-learn** (Computes NMI and ARI between predicted and ground-truth cell type labels)
- **scMGCA** (Method under evaluation whose predicted labels and latent embeddings are scored with NMI/ARI/ASW) — https://pypi.org/project/scMGCA
- **SHARP** (Baseline clustering method whose output is validated with ARI against reference labels (provides a runnable ARI example)) — https://github.com/shibiaowan/SHARP
- **scziDesk** (Baseline deep soft K-means clustering method scored in the same benchmark table) — https://github.com/xuebaliang/scziDesk
- **scDeepCluster** (Baseline model-based deep embedding clustering method scored in the same benchmark table) — https://github.com/ttgump/scDeepCluster

## Examples

```
res = SHARP(scExp_tpm, rN.seed=2103); ARI(label, res)
```

## Evaluation signals

- scMGCA's row-wise means across the 20 datasets should reach approximately NMI 0.8304, ARI 0.8278, and ASW 0.5827
- scMGCA should have the highest metric value on 15 of the 20 datasets (e.g. highest ASW on 'QS Trachea' and 'QS Heart')
- NMI/ARI must be computed with scikit-learn against ground-truth labels, not against cluster centroids or self-consistency measures
- ASW must be calculated on each method's learned embedding space; the table must contain all three metrics for all 13 methods on all 20 datasets
- Sanity check against the SHARP README example: ARI on the TPM pancreas example should be about 0.9675143 when reproducibility seed rN.seed=2103 is set

## Limitations

- ASW depends on each method's latent embedding, so differences in embedding dimensionality or scale between methods can influence the metric
- Sample size was not predetermined (maximum available cells per dataset); datasets with >25,000 cells are split into batches, which may affect reproducibility of the per-dataset scores
- High dropout rates in scRNA-seq data and unstated software versions (Python, TensorFlow, SCANPY, R packages) complicate exact reproduction; the Source Data file is referenced without an explicit URL

## Evidence

- [results] Across the 20 datasets, scMGCA achieved the highest mean NMI, ARI and ASW of all 13 methods, reaching 0.8304, 0.8278, and 0.5827, respectively, and had the highest values for 15 of the 20 datasets.: [evidence span withheld — anchor status: not_found]
- [results] Compared labels with true labels on named datasets.: [evidence span withheld — anchor status: not_found]
- [readme] SHARP README demonstrates checking performance with ARI against reference labels.: [evidence span withheld — anchor status: external_doc]
- [results] Preprocessing pipeline that produces the inputs the metrics are computed on.: "We first normalized the scRNA-seq gene expression matrix and selected the highly variable genes"
