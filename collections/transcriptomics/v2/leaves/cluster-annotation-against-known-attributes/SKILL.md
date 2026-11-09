---
name: cluster-annotation-against-known-attributes
description: Use when an RCC (or other unsupervised) clustering result must be cross-tabulated
  with known sample attributes—such as the Biase dataset's ICM vs. trophectoderm labels—to
  biologically interpret and validate the clusters.
when_to_use_negative:
- When the transcriptomic data have no known sample attributes or annotations to compare
  against — annotation against attributes is meaningless without metadata.
- When the goal is simply to determine the optimal number of clusters rather than
  to interpret clusters against known labels; use RCC_clus alone instead.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RCC (Recursive Consensus Clustering)
  role: R package providing clusterAnnotation and clusterAttr functions to map clusters
    onto known sample attributes
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm used by RCC to recursively generate the cluster assignments
    that are then annotated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:a2e50a956a3e4c0b7f937f87ff88ef23a63375fb16bffeedf24f3ea450b0b9be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - R
  - RCC (Recursive Consensus Clustering)
  - clusterAnnotation
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cluster-annotation-against-known-attributes

## Summary

Annotate RCC's recursively derived clusters against known sample attributes (e.g., cell type, tissue of origin) using the clusterAnnotation function of the RCC R package, so that discovered subtypes can be interpreted and validated against ground-truth metadata.

## When to use

Use when an RCC (or other unsupervised) clustering result must be cross-tabulated with known sample attributes—such as the Biase dataset's ICM vs. trophectoderm labels—to biologically interpret and validate the clusters.

## When NOT to use

- When the transcriptomic data have no known sample attributes or annotations to compare against — annotation against attributes is meaningless without metadata.
- When the goal is simply to determine the optimal number of clusters rather than to interpret clusters against known labels; use RCC_clus alone instead.

## Inputs

- RCC clustering object from RCC_clus(x, config_file)
- Sample attribute/metadata file (e.g., cell-type annotation for Biase GSE57249, Pollen SRP041736, or Darmanis GSE67835/GSE84465)
- Attribute name to annotate (e.g., cell type or sample attribute column)
- Cluster level to annotate (e.g., level-2 partition)
- minSample threshold for clusterAttr

## Outputs

- Cluster-to-attribute annotation table cross-tabulating RCC clusters with known sample attributes
- Combined cluster assignment table written to rcc_cluster_assignments.csv (sample ID, dataset, attribute, RCC cluster per level)
- Output plots/annotation files written to the specified OutputDir

## How to apply

After running RCC_clus(x, config_file), which recursively applies ConsensusClusterPlus (consensus clustering with k-means) to determine the optimal number of clusters, call clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir) passing the RCC cluster assignments along with the dataset's sample-attribute metadata file (e.g., cell-type annotation for GSE57249). Use clusterAttr(x, infoFile, attribute, minSample, category, algorithm, outputDir) to characterize how a chosen attribute distributes across clusters, applying the minSample threshold to exclude sparsely populated categories. Cross-tabulate the resulting cluster-by-attribute table to verify whether RCC partitions reproduce known structure — e.g., the ICM/trophectoderm separation at level 2 of the Biase blastocyst data — and export assignments (sample ID, dataset, attribute, RCC cluster per level) to a CSV such as rcc_cluster_assignments.csv.

## Related tools

- **RCC (Recursive Consensus Clustering)** (R package providing clusterAnnotation and clusterAttr functions to map clusters onto known sample attributes) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm used by RCC to recursively generate the cluster assignments that are then annotated)

## Examples

```
clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir)
```

## Evaluation signals

- Cross-tabulation of RCC clusters with known sample attributes reproduces the reported subdivision — e.g., ICM vs. trophectoderm separation at level 2 of the Biase dataset across 1000 runs.
- BJ fibroblast data resolve into three cell-line clusters and Darmanis oligodendrocyte, endothelial and fetal quiescent cells each subdivide into three clusters, matching Figures 7–8.
- The exported rcc_cluster_assignments.csv contains sample ID, dataset, attribute, and RCC cluster per level for every cell/sample.
- Sparse attribute categories falling below the minSample threshold in clusterAttr are correctly excluded from the annotation.
- A qualitative match of the level-2 partition to the paper's Figures 6–8 constitutes success.

## Limitations

- Computational time of RCC is a limitation; it depends on the underlying base algorithm (consensus clustering with k-means), so annotation of large datasets inherits that cost.
- Analysis of large integrated datasets with millions of cells is still a next step, not yet demonstrated; combining with scanorama and geometric sketching is suggested for such cases.
- Typical pipelines rely on known annotations and manual cluster identification via visualization, which this workflow seeks to avoid but annotation still requires attribute metadata.

## Evidence

- [methods] clusterAnnotation method call: "clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir)"
- [methods] clusterAttr signature with minSample filter: "clusterAttr(x, infoFile, attribute, minSample, category, algorithm, outputDir)"
- [abstract] RCC purpose of attribute-based annotation: "We have developed Recursive Consensus Clustering (RCC), an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets."
- [results] Biase level-2 annotation result: [evidence span withheld — anchor status: not_found]
- [readme] README installation instructions: "RCC, an unsurpersived clustering algorithm for novel sub type discovery from bulk and single-cell transcriptomic data sets"
