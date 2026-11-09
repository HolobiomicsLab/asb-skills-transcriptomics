---
name: gene-expression-clustering-and-go-enrichment
description: Use when a Lamian lamian_test run has produced significant TDE or XDE
  genes (FDR.overall < 0.05) and you need to organize them into expression-dynamics
  clusters and interpret each cluster via GO enrichment.
when_to_use_negative:
- Differential testing has not yet been run or no genes pass FDR.overall < 0.05 —
  clustering needs a validated significant gene set first
- The data lack a pseudotime trajectory (no TSCAN/Seurat harmonization was performed),
  so genes have no fitted expression dynamics along a trajectory to cluster
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Lamian
  role: Provides clusterGene for grouping significant TDE/XDE genes and the surrounding
    differential testing framework whose results feed clustering
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Constructs the cluster-based minimum spanning tree pseudotime along which
    gene expression patterns are fitted and clustered
  repo: https://github.com/zji90/TSCAN
- name: Seurat
  role: Harmonizes samples by embedding cells into a common low-dimensional space
    prior to trajectory construction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:d8442c8af97918f0b8a2c4b5d7ab3e632465761305522bc30e05c23ffc985399
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Seurat
  - TSCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-expression-clustering-and-go-enrichment

## Summary

Clusters significant differentially-expressed (TDE/XDE) genes along pseudotime with Lamian's clusterGene and computes per-cluster GO enrichment, using FDR.overall < 0.05 to call genes and reveal coordinated biological programs (e.g. TBX21/ZEB2 and interferon programs in the COVID-19 CD8 T cell severity analysis).

## When to use

Use when a Lamian lamian_test run has produced significant TDE or XDE genes (FDR.overall < 0.05) and you need to organize them into expression-dynamics clusters and interpret each cluster via GO enrichment.

## When NOT to use

- Differential testing has not yet been run or no genes pass FDR.overall < 0.05 — clustering needs a validated significant gene set first
- The data lack a pseudotime trajectory (no TSCAN/Seurat harmonization was performed), so genes have no fitted expression dynamics along a trajectory to cluster

## Inputs

- lamian_test results object with per-gene statistics and FDR.overall values (TDE and/or XDE)
- Gene set of significant TDE/XDE genes called at FDR.overall < 0.05
- Fitted gene expression patterns along pseudotime from the Lamian model (per-sample trajectories)
- Pseudotime/cell metadata from TSCAN trajectory construction (pseudotime divided into 100 equal-length intervals for density analyses)

## Outputs

- Clusters of significant TDE/XDE genes grouped by expression dynamics (e.g. 5 TDE clusters, 14 XDE clusters in the COVID-19 CD8 T cell analysis)
- Per-cluster GO enrichment results linking each gene cluster to biological processes
- Heatmap/visualization of clustered gene expression patterns along pseudotime

## How to apply

After running lamian_test with test.type='time' (TDE) or test.type='variable', testvar=2 (XDE) using the permutation-based likelihood ratio test, collect genes passing FDR.overall < 0.05. Apply Lamian's clusterGene function to group significant genes by similarity of their fitted pseudotemporal expression patterns, so that genes with similar dynamics (e.g. early activation vs. late effector patterns) fall into the same cluster. For each resulting cluster, compute Gene Ontology enrichment to assign biological function, as done in the COVID-19 CD8 T cell analysis where the significant genes resolved into 5 TDE clusters and 14 XDE clusters harboring TBX21/ZEB2 and interferon-gene patterns. The clustering is the interpretive layer on top of differential testing; it depends on the statistical calls being correct first, which requires the XDE test to have accounted for cross-sample variability via the permutation-based LRT.

## Related tools

- **Lamian** (Provides clusterGene for grouping significant TDE/XDE genes and the surrounding differential testing framework whose results feed clustering) — https://github.com/Winnie09/Lamian
- **TSCAN** (Constructs the cluster-based minimum spanning tree pseudotime along which gene expression patterns are fitted and clustered) — https://github.com/zji90/TSCAN
- **Seurat** (Harmonizes samples by embedding cells into a common low-dimensional space prior to trajectory construction)

## Evaluation signals

- Reproducing the COVID-19 CD8 T cell analysis should yield 2195 TDE genes in 5 clusters and 1315 XDE genes in 14 clusters
- Significant gene clusters should recover the reported TBX21/ZEB2 and interferon-gene expression patterns
- Every gene entering clusterGene must satisfy FDR.overall < 0.05 from lamian_test
- Each output cluster should have non-empty GO enrichment terms that are biologically interpretable (e.g. immune response/interferon signaling)

## Limitations

- XDE results depend on the permutation-based likelihood ratio test with permuiter = 100 or higher; fewer permutations destabilize the FDR.overall calls feeding clustering
- Lamian modules do not seamlessly support Monocle2, Monocle3, or Slingshot trajectories, limiting which pseudotime inputs can feed the clustering
- The interpretive clusters are only as reliable as the underlying trajectory and pseudotime, whose inference uncertainty is not fully propagated

## Evidence

- [results] Cluster the significant genes with clusterGene and compute GO enrichment per cluster, using FDR.overall < 0.05 to call TDE/XDE genes.: [evidence span withheld — anchor status: not_found]
- [discussion] To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test.: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [readme] Pseudotime analysis based on single-cell RNA-seq (scRNA-seq) data has been widely used to study dynamic gene regulatory programs: "Pseudotime analysis based on single-cell RNA-seq (scRNA-seq) data has been widely used to study dynamic gene regulatory programs"
