---
name: gene-signature-enrichment-scoring
description: Use when RCC clusters (e.g. CT/PAN glioblastoma clusters or SKCM subtypes)
  must be validated against a known transcriptional signature to confirm biologically
  distinct subtype identity.
when_to_use_negative:
- Clusters have not yet been generated and annotated — score enrichment only after
  RCC_clus and clusterAnnotation define the cluster labels
- The geneSet is undefined or unrelated to the biology of the dataset — enrichment
  against an arbitrary signature cannot validate subtype identity
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: RCC (Recursive Consensus Clustering)
  role: R package whose ssgsea function performs single-sample gene set enrichment
    scoring across clusters; installed via devtools::install_github
  repo: github.com/MSCTR/RecursiveConsensusClustering
- name: clusterAnnotation
  role: Produces the annotated clusters (e.g. CT and PAN glioblastoma clusters) that
    ssgsea scores the geneSet against
  repo: github.com/MSCTR/RecursiveConsensusClustering
- name: geneMarkers
  role: Complementary step deriving marker genes per RCC cluster (e.g. four SKCM subtypes)
    that can be interpreted alongside enrichment scores
  repo: github.com/MSCTR/RecursiveConsensusClustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:64c02a4fff5c45ecf2ff843d2f6cbe6d57945f2594c348d232f91ce720c94a97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - RCC
  - RCC (Recursive Consensus Clustering)
  - clusterAnnotation
  - clusterAttr
  - geneMarkers
  - mclust
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# gene-signature-enrichment-scoring

## Summary

Quantify enrichment of a defined gene signature (e.g. the proneural geneSet) across sample groups or clusters using the ssgsea function of the RCC R package, supporting interpretation of recursive consensus clusters as biologically meaningful subtypes.

## When to use

Use when RCC clusters (e.g. CT/PAN glioblastoma clusters or SKCM subtypes) must be validated against a known transcriptional signature to confirm biologically distinct subtype identity.

## When NOT to use

- Clusters have not yet been generated and annotated — score enrichment only after RCC_clus and clusterAnnotation define the cluster labels
- The geneSet is undefined or unrelated to the biology of the dataset — enrichment against an arbitrary signature cannot validate subtype identity

## Inputs

- Expression matrix file (exprsFile) for the clustered dataset (e.g. TCGA glioblastoma transcriptome)
- Sample info/attribute file (infoFile) mapping samples to attributes and RCC cluster assignments
- Gene set file defining the signature to score (e.g. proneural geneSet)
- Attribute column name identifying the cluster/annotation variable
- Target cluster label (clus) and output directory

## Outputs

- Per-cluster ssGSEA enrichment scores for the supplied geneSet
- Enrichment result files written to the specified output directory, supporting confirmation of signature-specific clusters (e.g. proneural-signature subdivision of CT/PAN)

## How to apply

After RCC_clus assigns recursive consensus clusters and clusterAnnotation labels the clusters of interest (e.g. annotating CT and PAN clusters from TCGA glioblastoma data), run the RCC package's ssgsea function to score a supplied geneSet (e.g. the proneural signature) across the annotated clusters. Inputs are the expression matrix file (exprsFile), the sample info/attribute file (infoFile), the gene set, the attribute column, and the target cluster label, with results written to outputDir. The resulting per-cluster enrichment scores quantify whether a known signature (such as the proneural signature subdivision) is concentrated in a specific RCC cluster, providing functional validation of novel subtypes discovered by the recursive consensus clustering. Scores are interpreted against cluster composition from clusterAttr and, where relevant, marker genes from geneMarkers to confirm the reported subtype structure.

## Related tools

- **RCC (Recursive Consensus Clustering)** (R package whose ssgsea function performs single-sample gene set enrichment scoring across clusters; installed via devtools::install_github) — github.com/MSCTR/RecursiveConsensusClustering
- **clusterAnnotation** (Produces the annotated clusters (e.g. CT and PAN glioblastoma clusters) that ssgsea scores the geneSet against) — github.com/MSCTR/RecursiveConsensusClustering
- **geneMarkers** (Complementary step deriving marker genes per RCC cluster (e.g. four SKCM subtypes) that can be interpreted alongside enrichment scores) — github.com/MSCTR/RecursiveConsensusClustering

## Examples

```
R: ssgsea(x, exprsFile="gbm_exprs.txt", infoFile="gbm_info.txt", geneSet="proneural", attribute="cluster", clus="CT", outputDir="ssgsea_out")
```

## Evaluation signals

- Enrichment scores show the proneural geneSet concentrated in the annotated glioblastoma cluster expected from the reported proneural-signature subdivision of CT/PAN clusters
- Per-cluster score distribution separates the target cluster from other RCC clusters rather than being uniform across all samples
- Enrichment results are consistent with cluster composition from clusterAttr (attribute column with minSample filter) and with geneMarkers output
- ssgsea runs without errors on the paired exprsFile/infoFile and writes outputs to the specified outputDir

## Limitations

- RCC's known limitation is computational time, which is dependent on the underlying base algorithm (consensus clustering with k-means), so enrichment analysis on very large datasets inherits this cost
- ssgsea is positioned as part of the RCC pipeline on pre-clustered data; it does not itself discover clusters or select the optimal number of clusters

## Evidence

- [methods] ssgsea function signature in RCC methods: "ssgsea(x, exprsFile, infoFile, geneSet, attribute, clus, outputDir)"
- [methods] clusterAnnotation produces labeled clusters for downstream scoring: "clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir)"
- [readme] RCC package installation path for obtaining ssgsea: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
