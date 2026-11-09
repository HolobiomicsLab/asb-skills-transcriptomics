---
name: mcl-graph-clustering-of-protein-networks
description: Use when you have an edge list of protein-protein functional associations
  (interactions given in both directions) and need to detect functional modules/communities
  for subsequent enrichment analysis in Cytoscape stringApp.
when_to_use_negative:
- The input is raw high-dimensional omics data (scRNA-seq/proteomics matrix) — run
  favapy first to infer the association network before clustering
- You need literature-curated physical PPIs rather than inferred functional associations;
  FAVA networks are meant to be free of literature bias and complement existing networks
- The network is tiny or the goal is single-pair interaction prediction rather than
  modular structure detection
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: favapy
  role: Produces the upstream FAVA protein association network edge list that MCL
    clusters
  repo: https://github.com/mikelkou/fava
- name: MCL
  role: Graph clustering algorithm run with inflation 1.5 to partition the FAVA network
    into functional clusters
- name: Cytoscape stringApp 2.0
  role: Used to import the clustered network and run functional enrichment on clusters
    with >=5 nodes across seven annotation categories
- name: Python
  role: Compute cluster counts and cluster-size distribution and export the enrichment
    table (enrichment_table_S1.tsv)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:6077d32a03f867c8ede53461ed84e36f1db0275658e32dba18bbe478ef9b8361
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - Cytoscape stringApp 2.0
  - MCL
  - Python
  - favapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# mcl-graph-clustering-of-protein-networks

## Summary

Partitions a dense protein functional-association network (e.g. the FAVA network inferred from scRNA-seq and proteomics data) into functional modules using the Markov Cluster Algorithm (MCL), then validates cluster count and size distribution against reported values for downstream enrichment analysis.

## When to use

Use when you have an edge list of protein-protein functional associations (interactions given in both directions) and need to detect functional modules/communities for subsequent enrichment analysis in Cytoscape stringApp.

## When NOT to use

- The input is raw high-dimensional omics data (scRNA-seq/proteomics matrix) — run favapy first to infer the association network before clustering
- You need literature-curated physical PPIs rather than inferred functional associations; FAVA networks are meant to be free of literature bias and complement existing networks
- The network is tiny or the goal is single-pair interaction prediction rather than modular structure detection

## Inputs

- Combined FAVA protein association network edge list (favapy output; interactions in both directions, proteinA-proteinB and proteinB-proteinA)
- MCL inflation parameter value (1.5 as used in the paper)

## Outputs

- 341 functional clusters covering the network proteins (130 clusters with >=5 nodes)
- Cluster-size distribution table for validation
- Clustered network imported into Cytoscape (stringApp 2.0)
- enrichment_table_S1.tsv — enrichment results in the Supplementary Table S1 column layout across seven categories

## How to apply

Load the combined FAVA protein association network edge list produced by the upstream favapy run, ensuring interactions are present in both directions (proteinA-proteinB and proteinB-proteinA). Run MCL graph clustering with the inflation parameter set to 1.5 to partition proteins into functional clusters. Compute the cluster count and the cluster-size distribution and verify they match the reported 341 clusters with 130 clusters containing five or more nodes. Keep only clusters with at least five nodes for further analysis, import the clustered network into Cytoscape, and run stringApp 2.0 enrichment across the seven annotation categories (Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, Diseases). The inflation parameter of 1.5 is the key decision point: it controls cluster granularity, and the article's reported partition is the reference against which the run is validated.

## Related tools

- **favapy** (Produces the upstream FAVA protein association network edge list that MCL clusters) — https://github.com/mikelkou/fava
- **MCL** (Graph clustering algorithm run with inflation 1.5 to partition the FAVA network into functional clusters)
- **Cytoscape stringApp 2.0** (Used to import the clustered network and run functional enrichment on clusters with >=5 nodes across seven annotation categories)
- **Python** (Compute cluster counts and cluster-size distribution and export the enrichment table (enrichment_table_S1.tsv))

## Examples

```
mcl fava_network.abc --abc -I 1.5 -o fava_clusters.txt
```

## Evaluation signals

- MCL run with inflation 1.5 yields exactly 341 clusters
- 130 of the clusters contain five or more nodes, matching the reported cluster-size distribution
- Clustered network imports cleanly into Cytoscape stringApp 2.0
- Enrichment table contains results for all seven annotation categories: Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, Diseases
- enrichment_table_S1.tsv column structure matches the Supplementary Table S1 layout

## Limitations

- Cluster partition depends on the inflation parameter; 1.5 reproduces the published 341-cluster result, other values will not match it
- Clusters with fewer than five nodes (211 of 341) are not analyzed by stringApp enrichment
- MCL operates on the favapy-inferred functional association network, which should be free of literature bias but is a predicted network, not curated physical interactions

## Evidence

- [full_text] Run MCL graph clustering on the network with inflation parameter set to 1.5 to partition proteins into functional clusters.: [evidence span withheld — anchor status: not_found]
- [full_text] Clustering the FAVA network with MCL (inflation 1.5) produced 341 clusters, of which 130 comprised five or more nodes and were further analyzed with Cytoscape stringApp 2.0 enrichment.: [evidence span withheld — anchor status: not_found]
- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] Run FAVA from the command line as follows: "Run FAVA from the command line as follows"
