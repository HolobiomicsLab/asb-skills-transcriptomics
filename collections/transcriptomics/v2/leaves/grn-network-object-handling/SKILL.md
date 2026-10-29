---
name: grn-network-object-handling
description: Reference for handling scPRINT-derived gene networks loaded as GRnnData
  objects when you need differential hub detection (eigenv centrality), Louvain community
  detection, or prerank Enrichr enrichment on network communities.
when_to_use_negative:
- When the goal is to infer or pre-train the gene network itself rather than analyze
  an existing scPRINT/GRnnData network object
- When no weighted adjacency/gene-network object exists and the data is a raw expression
  matrix (need scPRINT inference first)
- When benchmarking inferred networks against ChIP-seq/perturb-seq ground truths —
  use the BenGRN benchmarking workflow instead
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0203
tools:
- name: scPRINT
  role: Source model whose averaged attention matrices produce the gene networks analyzed
    in this skill
  repo: https://github.com/cantinilab/scPRINT
- name: GRnnData
  role: Data container for the inferred gene networks loaded and manipulated (adjacency
    access, ranking)
  repo: https://github.com/cantinilab/GRnnData
- name: BenGRN
  role: Cantini-lab suite companion for benchmarking the inferred gene networks
  repo: https://github.com/jkobject/benGRN
- name: Python
  role: Runtime for NetworkX centrality/Louvain computations and Enrichr prerank enrichment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:787447eb52f7051228ff1bcff161886f8126078042823b47163cc5a847314b43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - BenGRN
  - GRnnData
  - Python
  - scPRINT
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# grn-network-object-handling

## Summary

Operates on scPRINT-inferred gene network objects (GRnnData) to compute per-gene centrality, detect communities, and rank subnetwork genes for enrichment in cell-type-specific analyses such as BPH vs normal prostate fibroblasts.

## When to use

Reference for handling scPRINT-derived gene networks loaded as GRnnData objects when you need differential hub detection (eigenv centrality), Louvain community detection, or prerank Enrichr enrichment on network communities.

## When NOT to use

- When the goal is to infer or pre-train the gene network itself rather than analyze an existing scPRINT/GRnnData network object
- When no weighted adjacency/gene-network object exists and the data is a raw expression matrix (need scPRINT inference first)
- When benchmarking inferred networks against ChIP-seq/perturb-seq ground truths — use the BenGRN benchmarking workflow instead

## Inputs

- scPRINT-derived gene network for BPH-associated fibroblasts (GRnnData object over 4000 most variable genes)
- scPRINT-derived gene network for normal fibroblasts (GRnnData object over 4000 most variable genes)
- GO_MF_2023 gene-set library for Enrichr prerank enrichment

## Outputs

- Per-gene centrality table with eigenvector centrality, target-hub (row-wise) and regulator-hub (column-wise) sums, ranked BPH vs normal
- Louvain community memberships (resolution 1.5, communities filtered to 20–200 genes)
- Enrichr prerank GO_MF_2023 enrichment terms for community 4
- Collated results CSV (bph_network_analysis_results.csv)

## How to apply

Load the scPRINT-derived gene networks (e.g. BPH-associated vs normal fibroblasts, each built over the 4000 most variable genes of the cell type) as GRnnData objects. Convert the weighted adjacency to NetworkX graphs and compute per-gene eigenvector centrality plus row-wise (target-hub) and column-wise (regulator-hub) edge-score sums, then rank genes in BPH vs normal to flag differential hubs such as PAGE4-like senescence/secreted factors. Run Louvain community detection on the BPH network at resolution 1.5, retaining only communities of 20–200 genes (communities 4 and 5 in BPH-associated fibroblasts). Rank community 4's 92 genes by eigenvector centrality and run prerank Enrichr against GO_MF_2023 gene sets; validation expects recovery of calcium/sodium/iron/metal ion-transport terms consistent with hubs SPOCK3 and HERC3. Collate centrality tables, community memberships, and enrichment terms into a single CSV (e.g. bph_network_analysis_results.csv).

## Related tools

- **scPRINT** (Source model whose averaged attention matrices produce the gene networks analyzed in this skill) — https://github.com/cantinilab/scPRINT
- **GRnnData** (Data container for the inferred gene networks loaded and manipulated (adjacency access, ranking)) — https://github.com/cantinilab/GRnnData
- **BenGRN** (Cantini-lab suite companion for benchmarking the inferred gene networks) — https://github.com/jkobject/benGRN
- **Python** (Runtime for NetworkX centrality/Louvain computations and Enrichr prerank enrichment)

## Examples

```
import networkx as nx; G = nx.from_pandas_adjacency(grn.adata.varp['weights'], create_using=nx.DiGraph); comms = nx.community.louvain_communities(G, resolution=1.5); ec = nx.eigenvector_centrality(G)
```

## Evaluation signals

- Retained Louvain communities fall within the 20–200 gene size window (communities 4 and 5 in BPH fibroblasts)
- Community 4 enrichment against GO_MF_2023 recovers calcium, sodium, iron, and metal ion-transport terms, matching SPOCK3/HERC3 hub roles
- Differential-hub ranking surfaces ion-exchange-linked senescence/secreted factors (e.g. PAGE4-like genes) in BPH vs normal networks
- Both BPH and normal networks contain 4000 genes (the most variable genes per cell type), preserving comparability
- Collated CSV contains centrality ranks, community memberships, and enrichment terms without missing entries

## Limitations

- No perfect ground truths exist for gene network inference, so hub/community calls cannot be fully validated against a gold standard
- Networks are cell-type-specific and built only over the 4000 most variable genes, so conclusions do not extend to genes outside that variable-gene set
- Community structure depends on the Louvain resolution parameter (1.5 here); changing it alters retained communities and downstream enrichment

## Evidence

- [discussion] GRnnData as analysis container: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [methods] network extraction from attention: [evidence span withheld — anchor status: not_found]
- [abstract] BPH atlas application: "scPRINT highlights the profound connections between ion exchange, senescence, and chronic inflammation"
- [discussion] no ground truths limitation: "A challenging aspect of GN inference is that no perfect ground truths exist"
- [abstract] pretraining scale context: "we present scPRINT, a large cell model for the inference of gene networks pre-trained on more than 50 million cells from the cellxgene database"
