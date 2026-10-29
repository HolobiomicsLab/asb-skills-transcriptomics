---
name: cite-seq-protein-based-cell-type-annotation
description: Use when a CITE-seq dataset contains cell-surface protein abundance (e.g.,
  PBMC antibody-derived tags) and trusted cell-type labels are needed as reference
  ground truth for benchmarking clustering or label-validation methods.
when_to_use_negative:
- The dataset is transcriptome-only (e.g., MCA or multiome RNA) with no cell-surface
  protein quantification — use marker-gene-based RNA annotation instead.
- The goal is de novo discovery of novel cell types rather than establishing trusted
  reference labels for benchmarking.
- Surface-protein panels lack the 10x Genomics marker genes needed to resolve the
  expected PBMC populations.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scanpy
  role: Preprocesses the RNA gene expression count matrices with the zheng17 recipe
    and provides PCA components used alongside protein-derived labels
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: pegasus
  role: Alternative automated cell-type inference via pg.infer_cell_types with the
    human_immune marker gene set
- name: scikit-learn (sklearn)
  role: Supports computation of clustering quality metrics against reference labels
  repo: https://github.com/gersteinlab/forest-fire-clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:b79187129188ba4d92474af9b6e3c858cb4420a61cb9952b7dfbdc3cddbae395
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - numba
  - numpy
  - pegasus
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cite-seq-protein-based-cell-type-annotation

## Summary

Derives high-confidence reference cell-type labels for CITE-seq PBMC datasets by annotating cells from cell-surface protein abundance using marker genes from 10x Genomics, providing ground truth for evaluating clustering methods via ARI and purity.

## When to use

Use when a CITE-seq dataset contains cell-surface protein abundance (e.g., PBMC antibody-derived tags) and trusted cell-type labels are needed as reference ground truth for benchmarking clustering or label-validation methods.

## When NOT to use

- The dataset is transcriptome-only (e.g., MCA or multiome RNA) with no cell-surface protein quantification — use marker-gene-based RNA annotation instead.
- The goal is de novo discovery of novel cell types rather than establishing trusted reference labels for benchmarking.
- Surface-protein panels lack the 10x Genomics marker genes needed to resolve the expected PBMC populations.

## Inputs

- CITE-seq PBMC count matrix with cell-surface protein (antibody-derived) measurements
- RNA gene expression count matrix for the same cells
- 10x Genomics marker gene list mapping surface proteins to cell types

## Outputs

- Per-cell reference cell-type labels derived from surface proteins
- ARI and purity scores of each clustering method against the reference labels
- ari_purity_benchmark.csv with method x dataset results and pre-/post-PEP-filter ARI comparison

## How to apply

Start from the preprocessed PBMC CITE-seq dataset (RNA counts normalized with scanpy's zheng17 recipe, clustered on the first 30 principal components). Instead of relying on RNA clusters alone, use the measured cell-surface proteins to assign cell types, matching marker genes curated by 10x Genomics against the antibody targets (e.g., CD markers distinguishing T cells, B cells, monocytes, NK cells). The resulting protein-based labels serve as reference labels against which unsupervised cluster_labels (from FFC, Louvain, Leiden, PARC) are compared. Evaluation uses adjusted Rand index and purity scores; a further refinement filters cells to those with posterior exclusion probability (PEP) < 0.1 from FFC's pval_list before recomputing ARI, yielding >20% ARI improvement. Assemble method x dataset ARI/purity values into ari_purity_benchmark.csv for pre-/post-filtering comparison.

## Related tools

- **scanpy** (Preprocesses the RNA gene expression count matrices with the zheng17 recipe and provides PCA components used alongside protein-derived labels) — https://github.com/gersteinlab/forest-fire-clustering
- **pegasus** (Alternative automated cell-type inference via pg.infer_cell_types with the human_immune marker gene set)
- **scikit-learn (sklearn)** (Supports computation of clustering quality metrics against reference labels) — https://github.com/gersteinlab/forest-fire-clustering

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.pval()
```

## Evaluation signals

- Reference labels recapitulate canonical PBMC populations separable by CD surface markers from the 10x Genomics marker list
- Clustering methods evaluated against these labels achieve positive ARI; FFC with PEP < 0.1 filtering should exceed baseline methods by >20% ARI
- Purity scores computed against protein-derived labels are consistent with ARI rankings across Louvain, Leiden, PARC, and FFC
- Post-filtering ARI (using cluster_obj.pval_list with PEP < 0.1) is strictly higher than pre-filtering ARI on the same embeddings
- Results reproducible in ari_purity_benchmark.csv covering both CITE-seq and multiome PBMC datasets

## Limitations

- The benchmark relies on 10x Genomics marker genes, so annotation quality depends on the surface-protein panel covering those markers.
- When only one cluster exists in the data, Monte Carlo internal validation cannot accurately estimate label uncertainty, weakening PEP-based filtering.
- The number of Monte Carlo trials varies with cluster number and is not fixed, introducing reproducibility variability in PEP estimates.

## Evidence

- [discussion] CITE-seq surface-protein labeling with 10x marker genes: "For CITE-seq, cell-surface proteins were used to label PBMC cell types using marker genes available from 10× Genomics."
- [discussion] PBMC preprocessing recipe: "The RNA gene expression count data for both PBMC datasets were preprocessed using the “zheng17” recipe in the scanpy package"
- [readme] PEP filtering validation API from README: "cluster_obj.pval_list # list of posterior significance values"
- [discussion] Evaluation metrics used for benchmarking: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
