---
name: protein-cluster-annotation-across-seven-enrichment-categories
description: Use when you already have a FAVA protein association edge list (interactions
  in both directions) and need to detect functional modules and annotate them with
  functional enrichment.
when_to_use_negative:
- Input is raw scRNA-seq or proteomics expression data rather than an inferred FAVA
  association network — run favapy network inference first
- Goal is single-protein or single-list functional enrichment without prior network
  clustering — the seven-category stringApp analysis here operates on MCL clusters
- Clusters of interest contain fewer than five nodes — only clusters with five or
  more nodes were analyzed with stringApp enrichment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: MCL
  role: Graph clustering of the combined FAVA network at inflation 1.5 to partition
    proteins into functional clusters
- name: Cytoscape stringApp 2.0
  role: Import of the clustered network and enrichment of clusters across Pathways,
    UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular
    Functions, and Diseases
- name: favapy
  role: Upstream tool that produced the protein association network edge list this
    skill clusters and annotates
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Compute cluster count and cluster-size distribution and export the enrichment
    table in the Supplementary Table S1 layout
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:3ac6c798c05294a63bf003cfca2e4166259a91c3282a2dc57ec76dc9b11a9354
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - Cytoscape stringApp 2.0
  - MCL
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# protein-cluster-annotation-across-seven-enrichment-categories

## Summary

Partitions the combined FAVA protein association network into functional clusters using MCL graph clustering (inflation 1.5) and annotates clusters of ≥5 proteins via Cytoscape stringApp 2.0 enrichment across seven categories, reproducing the layout of Supplementary Table S1.

## When to use

Use when you already have a FAVA protein association edge list (interactions in both directions) and need to detect functional modules and annotate them with functional enrichment.

## When NOT to use

- Input is raw scRNA-seq or proteomics expression data rather than an inferred FAVA association network — run favapy network inference first
- Goal is single-protein or single-list functional enrichment without prior network clustering — the seven-category stringApp analysis here operates on MCL clusters
- Clusters of interest contain fewer than five nodes — only clusters with five or more nodes were analyzed with stringApp enrichment

## Inputs

- combined FAVA protein association network edge list from the upstream favapy run (interactions in both directions, proteinA-proteinB and proteinB-proteinA)

## Outputs

- MCL cluster assignment of proteins into functional clusters (341 clusters, 130 with >=5 nodes)
- cluster-size distribution
- enrichment_table_S1.tsv with stringApp enrichment results across seven annotation categories matching the Supplementary Table S1 column structure

## How to apply

Take the combined FAVA protein association network edge list produced by the upstream favapy run, where each interaction is provided in both directions (proteinA-proteinB and proteinB-proteinA), and run MCL graph clustering with the inflation parameter set to 1.5 to partition proteins into functional clusters. Compute the cluster count and cluster-size distribution and verify they match the reported 341 clusters with 130 clusters containing five or more nodes. Import the clustered network into Cytoscape with stringApp 2.0 and run stringApp enrichment on the clusters across the seven categories: Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, and Diseases. Export the enrichment results as a table with the column structure of Supplementary Table S1 (enrichment_table_S1.tsv); clusters with fewer than five nodes are excluded from the enrichment analysis.

## Related tools

- **MCL** (Graph clustering of the combined FAVA network at inflation 1.5 to partition proteins into functional clusters)
- **Cytoscape stringApp 2.0** (Import of the clustered network and enrichment of clusters across Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, and Diseases)
- **favapy** (Upstream tool that produced the protein association network edge list this skill clusters and annotates) — https://github.com/mikelkou/fava
- **Python** (Compute cluster count and cluster-size distribution and export the enrichment table in the Supplementary Table S1 layout)

## Examples

```
mcl combined_fava_network.edges -I 1.5 -o fava_clusters.txt
```

## Evaluation signals

- MCL clustering with inflation 1.5 reproduces exactly 341 clusters
- 130 of the clusters contain five or more nodes
- enrichment_table_S1.tsv contains results for all seven annotation categories: Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, and Diseases
- exported enrichment table column structure matches Supplementary Table S1
- clustered network loads into Cytoscape with stringApp 2.0 without node-name mismatches from the FAVA edge list

## Limitations

- Only clusters with five or more nodes are carried into stringApp enrichment; 211 of the 341 clusters fall below this size threshold and are not annotated
- Results are tied to MCL inflation 1.5 — different inflation values will not reproduce the reported 341-cluster partition
- The FAVA edge list contains each interaction in both directions, so downstream cluster counting must not treat duplicated directed edges as distinct network structure

## Evidence

- [abstract] FAVA infers networks from high-dimensional omics data: "FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods, across a diverse collection of real as well as simulated datasets"
- [supplementary] MCL clustering reproduces 341 clusters with 130 containing five or more nodes: [evidence span withheld — anchor status: not_found]
- [supplementary] seven enrichment categories in Supplementary Table S1 layout: [evidence span withheld — anchor status: not_found]
