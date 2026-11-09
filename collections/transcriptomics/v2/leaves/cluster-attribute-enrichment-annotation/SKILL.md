---
name: cluster-attribute-enrichment-annotation
description: Use when RCC_clus has produced recursive consensus cluster assignments
  and you must validate clusters against sample metadata (attribute columns with a
  minSample cutoff) or assign subtype labels to clusters.
when_to_use_negative:
- Clusters have not yet been generated — run RCC_clus with the package config_file
  first, since clusterAttr and clusterAnnotation operate on RCC cluster assignments.
- No sample attribute metadata is available for comparison — clusterAttr requires
  an attribute column in the sample info file and a minSample-filtered category.
- Input is a small dataset with a known number of clusters where recursive subtype
  discovery offers no added value over a simpler baseline.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: RCC (Recursive Consensus Clustering)
  role: Parent R package providing clusterAttr and clusterAnnotation for post-clustering
    attribute enrichment and cluster labeling
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: clusterAttr
  role: Compares RCC clusters against sample attributes with a minSample filter to
    assess cluster-attribute correspondence
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: clusterAnnotation
  role: Annotates specific clusters (e.g., CT and PAN glioblastoma clusters) to confirm
    subtype signatures
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ssgsea
  role: Quantifies gene-set (e.g., proneural) enrichment in annotated clusters
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: geneMarkers
  role: Derives marker genes for subtypes (e.g., four SKCM melanoma subtypes) to validate
    cluster identity
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:090524a5b823c695d6c3964770fd59a5d4f0c071e9630ae20fd77cef38133aa7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - RCC
  - RCC (Recursive Consensus Clustering)
  - clusterAnnotation
  - clusterAttr
  - geneMarkers
  - mclust
  - ssgsea
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cluster-attribute-enrichment-annotation

## Summary

Post-clustering characterization steps of the RCC package (clusterAttr and clusterAnnotation) that compare recursive consensus clusters against known sample attributes and annotate biologically meaningful cluster labels (e.g., CT/PAN proneural subdivision in TCGA glioblastoma).

## When to use

Use when RCC_clus has produced recursive consensus cluster assignments and you must validate clusters against sample metadata (attribute columns with a minSample cutoff) or assign subtype labels to clusters.

## When NOT to use

- Clusters have not yet been generated — run RCC_clus with the package config_file first, since clusterAttr and clusterAnnotation operate on RCC cluster assignments.
- No sample attribute metadata is available for comparison — clusterAttr requires an attribute column in the sample info file and a minSample-filtered category.
- Input is a small dataset with a known number of clusters where recursive subtype discovery offers no added value over a simpler baseline.

## Inputs

- RCC cluster assignment object from RCC_clus (recursive consensus cluster assignments)
- Sample info/attribute file (infoFile) with attribute columns (e.g., cancer type, tissue, subtype)
- Expression matrix (exprsFile) used for downstream annotation and enrichment
- minSample threshold and attribute/category selections
- Proneural geneSet for signature quantification via ssgsea

## Outputs

- Cluster-by-attribute enrichment/composition table per dataset
- Annotated cluster labels (e.g., CT/PAN clusters with proneural subdivision in TCGA glioblastoma)
- ssgsea signature enrichment scores for annotated clusters
- geneMarkers subtype marker gene lists (e.g., 4 SKCM metastatic melanoma subtype markers)

## How to apply

After running RCC_clus(x, config_file) to obtain cluster assignments, call clusterAttr(x, infoFile, attribute, minSample, category, algorithm, outputDir) to test enrichment of each cluster against the chosen attribute column from the sample info file, applying the minSample filter to exclude sparse categories. For datasets such as TCGA pan-cancer (including SKCM melanoma) and Ivy GAP glioblastoma, this reveals whether recovered clusters correspond to known sample attributes or novel subtypes. Use clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir) to assign labels to specific clusters — e.g., annotating the CT and PAN clusters in TCGA glioblastoma to confirm the reported proneural-signature subdivision, which can then be quantified with ssgsea using the proneural geneSet. Acceptance is judged by whether cluster-attribute compositions match the published structures (proneural subdivision of CT/PAN, 8 GTEx lung clusters vs. 2 by tSNE+k-means, 4 SKCM subtypes with distinct geneMarkers).

## Related tools

- **RCC (Recursive Consensus Clustering)** (Parent R package providing clusterAttr and clusterAnnotation for post-clustering attribute enrichment and cluster labeling) — https://github.com/MSCTR/RecursiveConsensusClustering
- **clusterAttr** (Compares RCC clusters against sample attributes with a minSample filter to assess cluster-attribute correspondence) — https://github.com/MSCTR/RecursiveConsensusClustering
- **clusterAnnotation** (Annotates specific clusters (e.g., CT and PAN glioblastoma clusters) to confirm subtype signatures) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ssgsea** (Quantifies gene-set (e.g., proneural) enrichment in annotated clusters) — https://github.com/MSCTR/RecursiveConsensusClustering
- **geneMarkers** (Derives marker genes for subtypes (e.g., four SKCM melanoma subtypes) to validate cluster identity) — https://github.com/MSCTR/RecursiveConsensusClustering

## Examples

```
clusterAttr(x, infoFile, attribute, minSample, category, algorithm, outputDir); clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir)
```

## Evaluation signals

- Annotated TCGA glioblastoma CT/PAN clusters reproduce the reported proneural-signature subdivision, confirmed by ssgsea enrichment of the proneural geneSet.
- Cluster-attribute compositions from clusterAttr match the published structures across Ivy GAP, GTEx, and TCGA SKCM datasets.
- GTEx lung-tissue samples recover eight RCC clusters versus two clusters by the tSNE + k-means baseline.
- geneMarkers on TCGA SKCM yield marker genes for four metastatic melanoma subtypes that match the reported subtype markers.
- minSample filtering removes sparse attribute categories so enrichment results are not driven by tiny sample groups.

## Limitations

- RCC inherits the computational time of its underlying base algorithm (consensus clustering with k-means), which constrains how quickly cluster-attribute iterations can be run.
- Typical pipelines relying on known annotations and manual cluster identification through visualization lead to partial exploration of data — the very limitation this skill's automated attribute comparison mitigates.
- Analysis of large integrated datasets with millions of cells remains an unaddressed next step, limiting scalability of annotation workflows.

## Evidence

- [methods] clusterAttr(x, infoFile, attribute, minSample, category, algorithm, outputDir): "clusterAttr(x, infoFile, attribute, minSample, category, algorithm, outputDir)"
- [methods] clusterAnnotation signature: "clusterAnnotation(x, attribute, cluster, OutputDir, file, clus, outputDir)"
- [discussion] RCC enables recapitulation of complex findings: "with just a few commands we can recapitulate findings obtained through multi-step complex analysis and also identify additional subtypes not described in the original studies"
- [readme] README installation instructions: "RCC, an unsurpersived clustering algorithm for novel sub type discovery from bulk and single-cell transcriptomic data sets"
- [intro] prior pan-cancer analyses lacked subtype resolution: "Two major publications that analyzed this transcriptome data were able to largely find clusters with tissue-specific cancers and not the subtypes within each cancer type with clinical relevance"
