---
name: clustering-metric-computation-ari-lisi-db
description: Use when comparing cell-state clustering quality of methods (e.g., ICAT
  vs Seurat/Scanorama/Harmony integration) on scRNA-seq perturbation datasets that
  have ground-truth cell labels and treatment conditions.
when_to_use_negative:
- Datasets lacking ground-truth cell labels — ARI cannot be computed and the benchmark
  loses its primary validation signal
- Single-condition data with no treatment/control structure — LISI's treatment-batch-mixing
  criterion is undefined
- Goals other than clustering-quality assessment (e.g., differential expression testing
  or trajectory inference), where ARI/LISI/DB do not measure the relevant property
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ICAT
  role: Produces the semi-supervised Louvain cluster assignments (out.obs['sslouvain'])
    whose quality is scored by ARI/LISI/DB
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Preprocesses each dataset (sc.pp.log1p, PCA, neighbor graph) and provides
    the AnnData structure holding cluster labels and obsm['X_icat']
  repo: https://github.com/BradhamLab/icat
- name: AnnData
  role: Data container that carries ground-truth labels, treatment annotations, cluster
    assignments, and the upweighted matrix used in metric computation
  repo: https://github.com/BradhamLab/icat
- name: numpy
  role: Numeric backend for metric computation over the cluster label vectors and
    transformed expression matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:9ab8dc83ed2aae6223fe530a24347b75de9f876e5cadcfb579245a0f6db6b18f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - ICAT
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# clustering-metric-computation-ari-lisi-db

## Summary

Benchmark clustering quality of scRNA-seq perturbation workflows by computing Adjusted Rand Index (ARI) against ground-truth labels, LISI for treatment-batch mixing, and the Davies-Bouldin (DB) index for cluster separation across every method-dataset pair.

## When to use

Use when comparing cell-state clustering quality of methods (e.g., ICAT vs Seurat/Scanorama/Harmony integration) on scRNA-seq perturbation datasets that have ground-truth cell labels and treatment conditions.

## When NOT to use

- Datasets lacking ground-truth cell labels — ARI cannot be computed and the benchmark loses its primary validation signal
- Single-condition data with no treatment/control structure — LISI's treatment-batch-mixing criterion is undefined
- Goals other than clustering-quality assessment (e.g., differential expression testing or trajectory inference), where ARI/LISI/DB do not measure the relevant property

## Inputs

- Preprocessed AnnData objects (log1p-normalized, PCA, neighbor graph) for the Tian CellMix (H2228/HCC827/H1975/Mixed), Kagohara (cetuximab vs PBS), and Kang (control vs IFN-beta) scRNA-seq datasets
- Ground-truth cell label annotations per dataset for ARI computation
- Cluster assignments per method (e.g., out.obs['sslouvain'] from ICAT, Louvain clusters from baseline workflows)
- Upweighted ICAT feature matrix stored in adata.obsm['X_icat']

## Outputs

- fig4_benchmark_metrics.csv: ARI, LISI, and DB values for every method-dataset pair
- fig4_benchmark.png: method-comparison plot reproducing Figure 4 of the paper

## How to apply

Retrieve and preprocess each dataset (Tian CellMix H2228/HCC827/H1975/Mixed, Kagohara cetuximab vs PBS, Kang control vs IFN-beta) in Scanpy with log1p normalization, PCA, and a neighbor graph. Run each candidate method — ICAT via models.icat with treatment labels producing semi-supervised Louvain clusters in the 'sslouvain' column and the upweighted matrix in adata.obsm['X_icat'], plus the baseline integration+Louvain workflows and the extended ICATSeurat/ICATScan/ICATHarm variants — on the same preprocessed AnnData. For every method-dataset pair, compute ARI against ground-truth cell labels, LISI to quantify treatment-batch mixing, and the Davies-Bouldin index to quantify cluster separation. Assemble the metrics into fig4_benchmark_metrics.csv and render the method-comparison plot fig4_benchmark.png reproducing Figure 4. Rationale: ARI rewards agreement with known labels, LISI rewards mixing of treatment batches, and DB rewards well-separated clusters, so together they capture the clustering-quality axis the ICAT paper claims integration workflows miss.

## Related tools

- **ICAT** (Produces the semi-supervised Louvain cluster assignments (out.obs['sslouvain']) whose quality is scored by ARI/LISI/DB) — https://github.com/BradhamLab/icat
- **scanpy** (Preprocesses each dataset (sc.pp.log1p, PCA, neighbor graph) and provides the AnnData structure holding cluster labels and obsm['X_icat']) — https://github.com/BradhamLab/icat
- **AnnData** (Data container that carries ground-truth labels, treatment annotations, cluster assignments, and the upweighted matrix used in metric computation) — https://github.com/BradhamLab/icat
- **numpy** (Numeric backend for metric computation over the cluster label vectors and transformed expression matrix)

## Examples

```
from icat import models; model = models.icat(ctrl_value='control', ncfs_kws={'reg': 1, 'sigma': 3}, neighbor_kws={'n_neighbors': 15}, cluster_kws={'resolution': 0.75}); out = model.cluster(adata, adata.obs['treatment'])
```

## Evaluation signals

- ARI values fall within [0, 1] for every method-dataset pair
- The relative ICAT-vs-integration comparison matches the trends reported in Figure 4 of the paper
- ICAT outperforms Seurat, Scanorama, and Harmony on ARI, LISI, and DB in the Kagohara and Kang real datasets
- Metrics are present for all method-dataset pairs (ICAT, Seurat, Scanorama, Harmony, ICATSeurat, ICATScan, ICATHarm) with no missing rows in fig4_benchmark_metrics.csv
- Leverage the README's hyperparameter optimization (optimize_louvain / optimize_ncfs via Calinski-Harabasz and MCC) so metric differences reflect the method, not untuned clustering parameters

## Limitations

- The ICAT_manuscript analysis 'was run on a high performance cluster with a CentOS operating system' and 'is not guaranteed to run on your system out-of-the-box'
- Metric conclusions depend on ground-truth label quality; ARI against noisy labels can misrank methods
- ICAT does not automatically compute UMAP/tsne or the metrics themselves — reduced-dimensionality checks and metric scoring must be run separately on the upweighted space

## Evidence

- [methods] Compute ARI against ground-truth cell labels, LISI for treatment-batch mixing, and the Davies-Bouldin index for cluster separation for every method-dataset pair.: [evidence span withheld — anchor status: not_found]
- [results] ICAT outperformed integration baselines on Kagohara and Kang: [evidence span withheld — anchor status: not_found]
- [other] ARI range validation: [evidence span withheld — anchor status: not_found]
- [readme] Scanpy/AnnData dependency: "ICAT makes heavy use of the excellent `scanpy` library along with the associated `AnnData` data structure."
- [readme] Upweighted matrix location for downstream scoring: "it is possible to pass the upweighted count matrix (found in `adata.obsm["X_icat"]`) to these algorithms."
