---
name: silhouette-ari-purity-metric-computation
description: Use when benchmarking cell embeddings from paired multi-omics integration
  methods and you need comparable embedding (silhouette) and clustering (purity, ARI)
  scores across methods and datasets.
when_to_use_negative:
- The method provides no cell embedding (e.g. Seurat v4 WNN) — silhouette score cannot
  be computed and must be omitted
- You lack cell-type annotations, since the purity score requires an external reference
  label per cell
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: kNN graph construction (neighbors, k=20, euclidean) and Leiden clustering
    across resolutions on embeddings
- name: Mowgli
  role: Produces the cell embeddings being evaluated in the benchmark
  repo: github.com/cantinilab/mowgli
- name: Python
  role: Implementation language; metrics computed with Scikit-learn (silhouette_score,
    adjusted_rand_score)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:942c1f3a69e1cdd753bf934b3b95526ad615944d33b7443c1daf2722bd45cbd5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cobolt
  - MOFA+
  - Mowgli
  - Multigrate
  - Python
  - Scanpy
  - Seurat v4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# silhouette-ari-purity-metric-computation

## Summary

Computes three embedding/clustering quality metrics — silhouette score, kNN purity score, and Adjusted Rand Index (ARI) across Leiden resolutions — used to benchmark Mowgli against Seurat v4, MOFA+, integrative NMF, Cobolt and Multigrate on paired single-cell multi-omics datasets.

## When to use

Use when benchmarking cell embeddings from paired multi-omics integration methods and you need comparable embedding (silhouette) and clustering (purity, ARI) scores across methods and datasets.

## When NOT to use

- The method provides no cell embedding (e.g. Seurat v4 WNN) — silhouette score cannot be computed and must be omitted
- You lack cell-type annotations, since the purity score requires an external reference label per cell

## Inputs

- Cell embeddings (e.g. 50-dimensional Mowgli W matrices; 30-dim for Cobolt/integrative NMF; 15 factors for MOFA+; 15-dim for Multigrate)
- Leiden/FindClusters cluster label vectors across varying resolutions
- Reference cell-type annotations for purity score computation

## Outputs

- Per-method, per-dataset silhouette score, kNN purity score, and ARI-across-resolutions values
- Method x dataset x metric comparison table (benchmark_metrics.csv)

## How to apply

For embedding-producing methods (Mowgli, integrative NMF, MOFA+), first build a kNN graph with Scanpy's neighbors (k=20, euclidean distance) and cluster with Leiden across varying resolutions; cluster Seurat v4 with Seurat's FindClusters. Compute the silhouette score on the cell embeddings using Scikit-learn's silhouette_score, the kNN purity score on the embeddings, and the ARI across resolutions with Scikit-learn's adjusted_rand_score. Note that the silhouette score is omitted for Seurat v4 because that method provides no cell embedding. Assemble the results into a method x dataset x metric comparison table (benchmark_metrics.csv) to judge whether a method's clustering is competitive with the state-of-the-art.

## Related tools

- **Scanpy** (kNN graph construction (neighbors, k=20, euclidean) and Leiden clustering across resolutions on embeddings)
- **Mowgli** (Produces the cell embeddings being evaluated in the benchmark) — github.com/cantinilab/mowgli
- **Python** (Implementation language; metrics computed with Scikit-learn (silhouette_score, adjusted_rand_score))

## Examples

```
from sklearn.metrics import silhouette_score, adjusted_rand_score; sil = silhouette_score(emb, labels); ari = adjusted_rand_score(lev_a, lev_b)
```

## Evaluation signals

- Silhouette score computed on embeddings for all methods except Seurat v4 (excluded because no embedding is provided)
- ARI is evaluated across multiple Leiden resolutions rather than a single clustering, per the article's protocol
- Resulting metric table matches expected relative performance: Mowgli highest purity on OP CITE and comparable to MOFA+ and Multigrate on PBMC 10X
- Metrics match published patterns: Mowgli best silhouette on Liu, PBMC 10X, and OP CITE; MOFA+ best on OP Multiome and BM CITE; Seurat v4 best ARI on PBMC 10X, OP Multiome, OP CITE

## Limitations

- The silhouette score cannot be computed for Seurat because it requires an embedding of the cells not provided by this method
- Metrics are relative to clustering resolutions swept; method rankings by silhouette, ARI, and purity can disagree (e.g. Seurat v4 best by ARI but no silhouette available)

## Evidence

- [results] benchmark metrics: "We benchmarked Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate based on natural metrics for embedding and clustering performance: silhouette score, Adjusted Rand Index (ARI), and"
- [results] seurat silhouette omission: "The silhouette score cannot be com- puted for Seurat because it requires an embedding of the cells not provided by this method."
- [results] silhouette findings: "according to the silhouette score, Mowgli out- performs other methods in Liu, PBMC 10X, and OP CITE. MOFA+ performs best in the other two datasets."
- [results] ARI findings: "In terms of ARI across resolutions, Seurat v4 performs best in PBMC 10X, OP Multiome, and OP CITE. Mowgli and Seurat v4 perform comparably in the BM CITE dataset."
