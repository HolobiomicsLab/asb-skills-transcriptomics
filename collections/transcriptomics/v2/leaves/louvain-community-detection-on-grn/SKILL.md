---
name: louvain-community-detection-on-grn
description: Use when you have a weighted gene network (e.g. from scPRINT attention-based
  inference) and want to identify coherent gene modules for pathway/gene-set enrichment,
  as done for BPH fibroblast ion-exchange communities.
when_to_use_negative:
- The network is not cell-type/disease-specific (e.g. a global literature-based network)
  — the analysis targets cell-state-specific networks like BPH-associated fibroblasts
- You need directed regulatory relationships per module rather than undirected functional
  communities of co-connected genes
- No ground truth or biological validation context exists and the goal is causal network
  inference — the paper notes no perfect ground truths exist for GN inference
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scPRINT
  role: Foundation model whose attention-derived gene networks over the 4000 most
    variable genes are the input graphs for community detection
  repo: https://github.com/cantinilab/scPRINT
- name: GRnnData
  role: Data structure for storing and handling the scPRINT-inferred gene network
    before conversion to NetworkX
  repo: https://github.com/cantinilab/GRnnData
- name: BenGRN
  role: Part of the cantinilab suite for benchmarking gene networks inferred from
    single-cell RNA sequencing, used to validate inferred networks
  repo: https://github.com/jkobject/benGRN
- name: Python
  role: Runtime for NetworkX-based centrality computation and Louvain community detection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:378e07d0ded607992f08875aefcfe9861b709fc2be2df356b270d92be374cd32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - BenGRN
  - GRnnData
  - Python
  - scPRINT
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Louvain community detection on gene regulatory networks

## Summary

Partitions a single-cell-derived gene network (e.g. a scPRINT/GRnnData network over the 4000 most variable genes) into functional gene modules using Louvain community detection at resolution 1.5, retaining only communities of 20–200 genes for downstream enrichment analysis.

## When to use

Use when you have a weighted gene network (e.g. from scPRINT attention-based inference) and want to identify coherent gene modules for pathway/gene-set enrichment, as done for BPH fibroblast ion-exchange communities.

## When NOT to use

- The network is not cell-type/disease-specific (e.g. a global literature-based network) — the analysis targets cell-state-specific networks like BPH-associated fibroblasts
- You need directed regulatory relationships per module rather than undirected functional communities of co-connected genes
- No ground truth or biological validation context exists and the goal is causal network inference — the paper notes no perfect ground truths exist for GN inference

## Inputs

- scPRINT-derived gene network for a cell type, stored as a GRnnData object (built over the 4000 most variable genes)
- Optional: matched normal/disease counterpart networks for differential-hub ranking (eigenvector centrality, target-hub and regulator-hub edge-score sums)

## Outputs

- Community membership table (genes assigned to Louvain communities, filtered to 20–200 gene communities)
- Per-gene centrality table (eigenvector centrality, row-wise target-hub and column-wise regulator-hub sums, ranked BPH vs normal)
- Enrichr prerank GO_MF_2023 enrichment terms per community
- Collated results file, e.g. bph_network_analysis_results.csv

## How to apply

Convert the inferred network (e.g. a scPRINT-derived GRnnData object built over the 4000 most variable genes of the cell type) into a Python NetworkX graph. Run Louvain community detection with resolution=1.5 on the disease-associated (e.g. BPH fibroblast) network only. Retain only communities with 20–200 genes (in the BPH analysis this kept communities 4 and 5); smaller communities are too narrow for stable enrichment and larger ones too diffuse. Rank retained-community genes by eigenvector centrality and run prerank Enrichr gene-set enrichment (e.g. against GO_MF_2023) to interpret each module — in the paper this recovered calcium/sodium/iron/metal-ion transport terms consistent with SPOCK3 and HERC3 hub roles, linking ion exchange to senescence and chronic inflammation.

## Related tools

- **scPRINT** (Foundation model whose attention-derived gene networks over the 4000 most variable genes are the input graphs for community detection) — https://github.com/cantinilab/scPRINT
- **GRnnData** (Data structure for storing and handling the scPRINT-inferred gene network before conversion to NetworkX) — https://github.com/cantinilab/GRnnData
- **BenGRN** (Part of the cantinilab suite for benchmarking gene networks inferred from single-cell RNA sequencing, used to validate inferred networks) — https://github.com/jkobject/benGRN
- **Python** (Runtime for NetworkX-based centrality computation and Louvain community detection)

## Examples

```
import networkx as nx; comms = [c for c in nx.community.louvain_communities(G, weight='weight', resolution=1.5) if 20 <= len(c) <= 200]
```

## Evaluation signals

- All retained communities fall within the 20–200 gene size window (e.g. communities 4 and 5 in BPH-associated fibroblasts)
- Prerank Enrichr enrichment of community 4 (ranked by eigenvector centrality) recovers ion-exchange/ion-channel and senescence/inflammation-associated GO molecular-function terms
- Community membership for the 92 genes of community 4 is reproducible across runs of Louvain with resolution=1.5
- Differential hub flags (e.g. PAGE4-like senescence/secreted factors) are consistent between eigenvector centrality ranking and target/regulator-hub edge-score sums

## Limitations

- No perfect ground truths exist for gene network inference, so community biology must be validated indirectly via enrichment consistency
- Only communities of 20–200 genes are interpretable via enrichment; smaller or larger modules are discarded
- Transformer explainability and the breadth of tasks remain open questions, so network-derived communities may not reflect causal regulation

## Evidence

- [other] Louvain at resolution 1.5 on BPH network: [evidence span withheld — anchor status: not_found]
- [abstract] BPH atlas finding: [evidence span withheld — anchor status: not_found]
- [discussion] GRnnData/BenGRN suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [discussion] BPH atlas tested: "we put scPRINT to the test on a challenging atlas of normal and senescent prostate tissues with benign prostatic hyperplasia"
