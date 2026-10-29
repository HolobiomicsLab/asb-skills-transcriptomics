---
name: supplementary-table-enrichment-result-reproduction
description: Use when a FAVA protein association network edge list must be partitioned
  into functional clusters via MCL and the cluster-level functional enrichment results
  must be reproduced to match Supplementary Table S1.
when_to_use_negative:
- The network is not the combined FAVA protein association network (e.g. a STRING
  or other literature-biased network) — the 341/130 cluster counts are specific to
  the FAVA output.
- The goal is to generate or validate the FAVA network itself rather than cluster
  and enrich an existing favapy edge list — use the upstream favapy workflow (VAE,
  cook, pairs_after_cutoff) instead.
- Only single-category enrichment is needed and the Supplementary Table S1 seven-category
  layout is not required.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: MCL
  role: Graph clustering algorithm run on the FAVA network edge list with inflation
    1.5 to partition proteins into functional clusters
- name: Cytoscape stringApp 2.0
  role: Imports the clustered network and runs enrichment on the ≥5-node clusters
    across the seven annotation categories
- name: favapy
  role: Upstream producer of the combined FAVA protein association network edge list
    that this skill clusters
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Computes the cluster count and cluster-size distribution and verifies they
    match the reported 341 clusters with 130 clusters of ≥5 nodes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:64f8204e37b74a4158285c14b4663624b72a34df5151224844fa3253a63c5f08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - Cytoscape stringApp 2.0
  - MCL
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# supplementary-table-enrichment-result-reproduction

## Summary

Reproduces the MCL graph clustering of the combined FAVA protein association network (inflation 1.5 → 341 clusters, 130 with ≥5 nodes) and the subsequent Cytoscape stringApp 2.0 functional enrichment analysis across seven annotation categories, exporting results in the Supplementary Table S1 layout.

## When to use

Use when a FAVA protein association network edge list must be partitioned into functional clusters via MCL and the cluster-level functional enrichment results must be reproduced to match Supplementary Table S1.

## When NOT to use

- The network is not the combined FAVA protein association network (e.g. a STRING or other literature-biased network) — the 341/130 cluster counts are specific to the FAVA output.
- The goal is to generate or validate the FAVA network itself rather than cluster and enrich an existing favapy edge list — use the upstream favapy workflow (VAE, cook, pairs_after_cutoff) instead.
- Only single-category enrichment is needed and the Supplementary Table S1 seven-category layout is not required.

## Inputs

- combined FAVA protein association network edge list from favapy (interactions in both directions: proteinA-proteinB and proteinB-proteinA)
- MCL inflation parameter value (1.5) from the published protocol

## Outputs

- MCL clustering of the network: 341 clusters with cluster-size distribution (130 clusters of ≥5 nodes)
- enrichment_table_S1.tsv — cluster enrichment results for Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, and Diseases in Supplementary Table S1 column layout

## How to apply

Load the combined FAVA network edge list produced by favapy, noting that interactions are stored in both directions (proteinA-proteinB and proteinB-proteinA), which preserves undirected semantics when MCL consumes the graph. Run MCL with the inflation parameter set to 1.5 to partition proteins into functional clusters, then compute the cluster count and cluster-size distribution; validation requires exactly 341 clusters with 130 clusters containing five or more nodes. Filter to the 130 clusters with ≥5 nodes, import the clustered network into Cytoscape with stringApp 2.0, and run enrichment across all seven categories: Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, and Diseases. Export the enrichment results as a table (enrichment_table_S1.tsv) matching the column structure of Supplementary Table S1; both the clustering statistics and the presence of all seven annotation categories serve as correctness checks.

## Related tools

- **MCL** (Graph clustering algorithm run on the FAVA network edge list with inflation 1.5 to partition proteins into functional clusters)
- **Cytoscape stringApp 2.0** (Imports the clustered network and runs enrichment on the ≥5-node clusters across the seven annotation categories)
- **favapy** (Upstream producer of the combined FAVA protein association network edge list that this skill clusters) — https://github.com/mikelkou/fava
- **Python** (Computes the cluster count and cluster-size distribution and verifies they match the reported 341 clusters with 130 clusters of ≥5 nodes)

## Examples

```
mcl fava_network_edges.txt -I 1.5 -o fava_clusters.txt && python -c "clusters=[l.split() for l in open('fava_clusters.txt')]; print(len(clusters), sum(1 for c in clusters if len(c)>=5))"
```

## Evaluation signals

- MCL at inflation 1.5 yields exactly 341 clusters on the combined FAVA network
- Cluster-size distribution shows 130 clusters containing five or more nodes
- enrichment_table_S1.tsv contains results for all seven annotation categories (Pathways, UniProt Keywords, Cellular Compartments, Tissues, Biological Processes, Molecular Functions, Diseases)
- Enrichment table column structure matches the Supplementary Table S1 layout
- Each exported enrichment row maps to one of the 130 clusters with ≥5 nodes

## Limitations

- Cluster counts (341 total, 130 with ≥5 nodes) are validation targets specific to the combined FAVA network; other inputs will not match these numbers.
- The stringApp enrichment step requires manual Cytoscape interaction rather than a fully scripted pipeline.
- The favapy edge list stores each interaction in both directions, so downstream analyses must account for this duplication when counting edges.

## Evidence

- [supplementary] MCL clustering with stringApp enrichment: [evidence span withheld — anchor status: not_found]
- [supplementary] seven enrichment categories: [evidence span withheld — anchor status: not_found]
- [supplementary] network edge format: [evidence span withheld — anchor status: not_found]
- [intro] latent-space correlation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
