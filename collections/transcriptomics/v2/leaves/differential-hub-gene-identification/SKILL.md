---
name: differential-hub-gene-identification
description: Use when you have scPRINT-derived gene networks (GRnnData objects) for
  two cell states or conditions — e.g. BPH-associated vs normal prostate fibroblasts
  — and need to flag genes whose hub status changes between conditions.
when_to_use_negative:
- You only have raw counts or expression matrices with no inferred gene network —
  centrality cannot be computed without edges.
- The task requires causal or directed ground-truth validation; the paper notes 'no
  perfect ground truths exist' for gene network inference benchmarking, so hub scores
  are model-derived attention weights, not proven interactions.
- Networks were built on different gene sets per condition (e.g. different variable-gene
  sets), making rank comparison across genes ill-defined.
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scPRINT
  role: Pre-trained foundation model whose averaged attention matrices yield the gene
    networks in which hubs are detected
  repo: https://github.com/cantinilab/scPRINT
- name: GRnnData
  role: Data container used to load and manipulate the inferred gene networks per
    cell type
  repo: https://github.com/cantinilab/GRnnData
- name: BenGRN
  role: Benchmarking suite accompanying GRnnData for validating scPRINT-inferred gene
    networks
  repo: https://github.com/jkobject/benGRN
- name: Python
  role: Language in which NetworkX centrality computations and the analysis pipeline
    are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:e4ca368242c098dbb669b97cb26d45ca4b587efd0d9d95ca5a8c828192c66465
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - BenGRN
  - GRnnData
  - Python
  - scPRINT
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# differential-hub-gene-identification

## Summary

Identify cell-type-specific and differential hub genes by computing per-gene centrality (eigenvector centrality plus row-wise target-hub and column-wise regulator-hub edge-score sums) on scPRINT-inferred gene networks and ranking BPH vs normal fibroblast networks against each other.

## When to use

Use when you have scPRINT-derived gene networks (GRnnData objects) for two cell states or conditions — e.g. BPH-associated vs normal prostate fibroblasts — and need to flag genes whose hub status changes between conditions.

## When NOT to use

- You only have raw counts or expression matrices with no inferred gene network — centrality cannot be computed without edges.
- The task requires causal or directed ground-truth validation; the paper notes 'no perfect ground truths exist' for gene network inference benchmarking, so hub scores are model-derived attention weights, not proven interactions.
- Networks were built on different gene sets per condition (e.g. different variable-gene sets), making rank comparison across genes ill-defined.

## Inputs

- scPRINT-inferred gene network for one cell type (GRnnData object) over its 4000 most variable genes
- A second condition-matched scPRINT gene network (e.g. normal fibroblasts) as the comparison network
- Gene names/symbols indexing the network rows and columns

## Outputs

- Per-gene centrality table per condition (eigenvector centrality, target-hub row-sum, regulator-hub column-sum, and ranks)
- List of differential hub genes flagged by BPH vs normal rank comparison
- Collated CSV of centralities, community memberships, and enrichment terms (e.g. bph_network_analysis_results.csv)

## How to apply

Load the scPRINT gene networks built over the 4000 most variable genes of each cell type as GRnnData objects. Convert each network's edge weights to a NetworkX graph and compute per-gene eigenvector centrality together with row-wise (target-hub) and column-wise (regulator-hub) edge-score sums. Rank genes within each condition's network and compare the two rankings to flag differential hubs — e.g. PAGE4-like senescence/secreted factors that rise as hubs in BPH fibroblasts. Hub genes can then seed downstream analyses such as Louvain community detection (resolution 1.5, communities of 20–200 genes) followed by prerank Enrichr GO_MF_2023 enrichment, where centrality orders the gene list. In the paper this approach linked SPOCK3 and HERC3 to ion-exchange transport in BPH fibroblasts, connecting ion exchange, senescence, and chronic inflammation.

## Related tools

- **scPRINT** (Pre-trained foundation model whose averaged attention matrices yield the gene networks in which hubs are detected) — https://github.com/cantinilab/scPRINT
- **GRnnData** (Data container used to load and manipulate the inferred gene networks per cell type) — https://github.com/cantinilab/GRnnData
- **BenGRN** (Benchmarking suite accompanying GRnnData for validating scPRINT-inferred gene networks) — https://github.com/jkobject/benGRN
- **Python** (Language in which NetworkX centrality computations and the analysis pipeline are implemented)

## Examples

```
import networkx as nx; G = nx.from_pandas_adjacency(grnndata_obj.adata.to_df(), create_using=nx.DiGraph); centrality = nx.eigenvector_centrality_numpy(G, weight='weight')
```

## Evaluation signals

- Differential hubs are biologically coherent: e.g. PAGE4-like senescence/secreted factors and SPOCK3/HERC3 top the BPH centrality ranks.
- Community-4 genes ranked by eigenvector centrality enrich for ion-exchange/ion-channel GO molecular-function terms (GO_MF_2023, prerank Enrichr).
- Retained Louvain communities fall within the 20–200 gene size window (resolution 1.5).
- Both networks use a comparable gene universe (4000 most variable genes per cell type) before rank comparison.
- Results collate into bph_network_analysis_results.csv with centrality, community, and enrichment columns.

## Limitations

- No perfect ground truths exist for gene network inference, so hub calls from scPRINT attention-derived edges cannot be causally validated.
- Transformer ability to generate graphs and their explainability remain unresolved, making head/edge interpretation provisional.
- Hub identification depends on cell-state-specific networks, and most GRN inference methods do not scale to the number of genes and cells in single-cell RNA datasets — scPRINT is required for this scale.

## Evidence

- [other] Compute per-gene centrality measures in Python using NetworkX's eigenvector centrality implementation plus row-wise (target-hub) and column-wise (regulator-hub) edge-score sums, and rank genes in BPH vs normal networks: [evidence span withheld — anchor status: not_found]
- [abstract] scPRINT highlights connections between ion exchange, senescence, and chronic inflammation: "scPRINT highlights the profound connections between ion exchange, senescence, and chronic inflammation"
- [discussion] With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [discussion] we put scPRINT to the test on a challenging atlas of normal and senescent prostate tissues with benign prostatic hyperplasia: "we put scPRINT to the test on a challenging atlas of normal and senescent prostate tissues with benign prostatic hyperplasia"
