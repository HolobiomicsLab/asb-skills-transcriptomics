---
name: cytoscape-stringapp-functional-enrichment-analysis
description: Use when a clustered protein association network (e.g. FAVA network partitioned
  by MCL with clusters of ≥5 nodes) needs functional interpretation across Pathways,
  UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular
  Functions, and Diseases.
when_to_use_negative:
- The network has not yet been clustered by MCL — enrichment requires defined protein
  clusters, not a raw edge list
- Candidate clusters contain fewer than five nodes, which the FAVA workflow excluded
  from enrichment analysis
- The goal is to infer the network itself (VAE compression and correlation in latent
  space via favapy) rather than to interpret existing clusters
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0121
tools:
- name: favapy
  role: Produces the combined FAVA protein association network edge list that is clustered
    and enriched
  repo: https://github.com/mikelkou/fava
- name: MCL
  role: Graph clustering of the FAVA network with inflation parameter 1.5 to generate
    functional clusters
- name: Cytoscape stringApp 2.0
  role: Import of the clustered network and execution of enrichment across the seven
    annotation categories
- name: Python
  role: Glue code for loading the edge list, running MCL, and computing the cluster-size
    distribution
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:a9a9215c2f3fcc8630b5bcca5eafa52f32f1cdfbc73059d5cc314e7c614f9ea9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - Cytoscape stringApp 2.0
  - MCL
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cytoscape-stringapp-functional-enrichment-analysis

## Summary

Functional enrichment of MCL-derived clusters from the combined FAVA protein association network using Cytoscape stringApp 2.0, covering seven annotation categories and exporting results in the Supplementary Table S1 layout.

## When to use

Use when a clustered protein association network (e.g. FAVA network partitioned by MCL with clusters of ≥5 nodes) needs functional interpretation across Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, and Diseases.

## When NOT to use

- The network has not yet been clustered by MCL — enrichment requires defined protein clusters, not a raw edge list
- Candidate clusters contain fewer than five nodes, which the FAVA workflow excluded from enrichment analysis
- The goal is to infer the network itself (VAE compression and correlation in latent space via favapy) rather than to interpret existing clusters

## Inputs

- Combined FAVA protein association network edge list from the upstream favapy run (interactions in both directions, proteinA-proteinB and proteinB-proteinA)
- MCL clustering of the FAVA network (inflation 1.5) with cluster-size distribution
- Clusters containing five or more nodes (130 of 341 clusters in the reference run)

## Outputs

- enrichment_table_S1.tsv — enrichment results for all seven categories in the Supplementary Table S1 column layout (Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, Diseases)
- Cluster count and cluster-size distribution statistics for the MCL partition

## How to apply

Load the combined FAVA protein association network edge list produced by favapy, where interactions are given in both directions (proteinA-proteinB and proteinB-proteinA). Run MCL graph clustering with the inflation parameter set to 1.5 to partition proteins into functional clusters, then compute the cluster count and cluster-size distribution. Retain only clusters of five or more nodes (the FAVA analysis yielded 341 clusters, of which 130 had ≥5 nodes) and import the clustered network into Cytoscape with stringApp 2.0. Run stringApp enrichment on the retained clusters across the seven categories — Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, and Diseases — and export the results as enrichment_table_S1.tsv matching the column structure of Supplementary Table S1. The rationale is that enriching larger clusters (≥5 nodes) gives interpretable functional modules, and that FAVA networks are derived from omics data rather than literature, so enrichment reveals module functions without literature-bias artifacts.

## Related tools

- **favapy** (Produces the combined FAVA protein association network edge list that is clustered and enriched) — https://github.com/mikelkou/fava
- **MCL** (Graph clustering of the FAVA network with inflation parameter 1.5 to generate functional clusters)
- **Cytoscape stringApp 2.0** (Import of the clustered network and execution of enrichment across the seven annotation categories)
- **Python** (Glue code for loading the edge list, running MCL, and computing the cluster-size distribution)

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- MCL clustering at inflation 1.5 reproduces the reported 341 clusters, of which 130 contain five or more nodes
- The exported enrichment table contains results for all seven annotation categories
- The enrichment table column structure matches the layout of Supplementary Table S1
- Every enriched cluster in the table corresponds to a cluster of five or more nodes from the MCL partition
- The edge list fed to MCL contains both directions of each interaction (proteinA-proteinB and proteinB-proteinA)

## Limitations

- Enrichment was restricted to clusters of five or more nodes, so smaller clusters are not functionally annotated
- The cluster partition depends on the MCL inflation parameter (1.5); other inflation values would yield a different number and size distribution of clusters
- stringApp enrichment is performed inside Cytoscape, so the step depends on the Cytoscape application environment rather than a pure command-line pipeline
- Networks derived from data availability can be biased — the article notes that most existing high-quality networks are heavily biased, with well-studied proteins having many more interactions than understudied proteins

## Evidence

- [abstract] FAVA infers functional association networks from high-dimensional omics data: "FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods"
- [intro] Bias of existing networks motivates enrichment on omics-derived modules: "most existing high-quality networks are heavily biased by data availability, in the sense that well-studied proteins have many more interactions than understudied proteins"
- [methods] favapy CLI produces the network used for clustering and enrichment: [evidence span withheld — anchor status: not_found]
- [readme] FAVA source and tutorials are publicly available for reproducing the network: "Source code, documentation, and tutorials for FAVA are accessible on GitHub at https://github.com/mikelkou/fava"
