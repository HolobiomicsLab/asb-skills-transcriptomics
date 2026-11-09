---
name: gene-network-centrality-analysis
description: Use when you have weighted, cell-type-specific gene network adjacency
  matrices (e.g., GRnnData objects from scPRINT) and need to identify differential
  hub genes or functionally enriched network communities, as done for the BPH fibroblast
  ion-exchange/senescence module.
when_to_use_negative:
- When the goal is raw edge-level benchmarking against a gold standard (e.g., SERGIO-simulated
  or gwps perturb-seq edges) rather than node-level hub scoring and enrichment
- When the network is not weighted/cell-type-specific — hub centrality on binarized,
  non-specific co-expression matrices loses the per-method edge scores the ranking
  depends on
- When you only need pathway enrichment of a predefined gene list; prerank enrichment
  without a network-derived ranking does not require this skill
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0102
tools:
- name: scPRINT
  role: Source of the pretrained foundation model whose averaged attention-matrix
    gene networks are analyzed for centrality, communities, and enrichment
  repo: https://github.com/cantinilab/scPRINT
- name: GRnnData
  role: Data container for the inferred gene networks; provides the prerank setup
    for per-gene row-wise (Target-Hub) and column-wise (Regulator-Hub) edge-score
    sums used in enrichment
  repo: https://github.com/cantinilab/GRnnData
- name: BenGRN
  role: Benchmarking suite providing the prerank enrichment framework against TF and
    cell-type marker gene sets
  repo: https://github.com/jkobject/benGRN
- name: Python
  role: Implementation environment for NetworkX eigenvector centrality, Louvain community
    detection (resolution 1.5), and Enrichr prerank enrichment
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:27592c74fafa9cc049b7af1dd7c681caa7ce02a2f41c71a199269b9a3432df30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - BenGRN
  - GRnnData
  - Python
  - scPRINT
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# gene-network-centrality-analysis

## Summary

Per-gene hub scoring of cell-type-specific gene networks (e.g., scPRINT-inferred BPH vs normal prostate fibroblast networks) via eigenvector centrality and row/column-wise edge-score sums in GRnnData/NetworkX, followed by Louvain community detection and prerank gene-set enrichment to interpret differential hubs and subnetworks.

## When to use

Use when you have weighted, cell-type-specific gene network adjacency matrices (e.g., GRnnData objects from scPRINT) and need to identify differential hub genes or functionally enriched network communities, as done for the BPH fibroblast ion-exchange/senescence module.

## When NOT to use

- When the goal is raw edge-level benchmarking against a gold standard (e.g., SERGIO-simulated or gwps perturb-seq edges) rather than node-level hub scoring and enrichment
- When the network is not weighted/cell-type-specific — hub centrality on binarized, non-specific co-expression matrices loses the per-method edge scores the ranking depends on
- When you only need pathway enrichment of a predefined gene list; prerank enrichment without a network-derived ranking does not require this skill

## Inputs

- GRnnData gene-regulatory network objects (weighted adjacency matrices) for each cell type, e.g., BPH-associated and normal prostate fibroblasts over the 4000 most variable genes
- Cell-type marker gene sets (pangaoDB) and transcription factor gene sets (ENCODE) for enrichment testing
- GO_MF_2023 gene-set library for Enrichr prerank enrichment

## Outputs

- Per-gene centrality tables (eigenvector centrality, Target-Hub row sums, Regulator-Hub column sums) per network
- Differential hub rankings comparing BPH vs normal fibroblast networks
- Louvain community memberships (retained 20–200 gene communities)
- Enrichr prerank enrichment terms (e.g., ion-exchange/ion-channel and senescence/inflammation GO molecular-function terms)
- Aggregated results file, e.g., bph_network_analysis_results.csv

## How to apply

Load the inferred gene networks (built over the 4000 most variable genes per cell type) as GRnnData objects and convert their adjacency matrices to NetworkX graphs. Derive per-gene ordered scores three ways, as in the scPRINT benchmarking: eigenvector centrality via NetworkX, row-wise edge-score sums (Target-Hub), and column-wise sums (Regulator-Hub), using all genes in the network as the prerank background. Rank genes in BPH vs normal networks to flag differential hubs such as SPOCK3, HERC3, and PAGE4-like senescence/secreted factors. Run Louvain community detection on the BPH fibroblast network at resolution 1.5, retaining only communities of 20–200 genes (e.g., the 92-gene community 4). Rank community genes by eigenvector centrality and run prerank Enrichr enrichment against GO_MF_2023 gene sets; collate all tables (e.g., bph_network_analysis_results.csv). The rationale is that hub scores focus enrichment on biologically meaningful network positions, which is how scPRINT's networks were validated as enriched for TFs, cell-type markers, and ENCODE-validated targets.

## Related tools

- **scPRINT** (Source of the pretrained foundation model whose averaged attention-matrix gene networks are analyzed for centrality, communities, and enrichment) — https://github.com/cantinilab/scPRINT
- **GRnnData** (Data container for the inferred gene networks; provides the prerank setup for per-gene row-wise (Target-Hub) and column-wise (Regulator-Hub) edge-score sums used in enrichment) — https://github.com/cantinilab/GRnnData
- **BenGRN** (Benchmarking suite providing the prerank enrichment framework against TF and cell-type marker gene sets) — https://github.com/jkobject/benGRN
- **Python** (Implementation environment for NetworkX eigenvector centrality, Louvain community detection (resolution 1.5), and Enrichr prerank enrichment)

## Examples

```
import networkx as nx; G = nx.from_pandas_adjacency(grn.df, create_using=nx.DiGraph); centrality = nx.eigenvector_centrality(G, weight='weight')
```

## Evaluation signals

- Retained Louvain communities fall within the 20–200 gene size window (e.g., communities 4 and 5 in BPH-associated fibroblasts; community 4 has 92 genes)
- Prerank Enrichr of community 4 against GO_MF_2023 recovers ion-exchange/ion-channel and senescence/inflammation-associated GO molecular-function terms, consistent with the SPOCK3 and HERC3 hub roles
- Enrichment uses all genes in the network as the prerank background, and the cell-type enrichment criterion is that the correct cell type name appears in the top 10 enriched cell types
- Centrality rankings are computed for all three node scores (eigenvector centrality, row-wise Target-Hub sums, column-wise Regulator-Hub sums) and rank the expected differential hubs (e.g., SPOCK3, HERC3, PAGE4-like senescence/secreted factors) high in BPH
- Output collation contains centrality tables, community memberships, and Enrichr terms with complete coverage of all analyzed communities/networks (e.g., bph_network_analysis_results.csv)

## Limitations

- No perfect ground truths exist for gene network inference benchmarking, so centrality/hub rankings are inherently interpretive rather than validated against true regulatory edges
- ChIP-seq, perturb-seq, and literature-based ground truths used for calibration of the underlying networks remain scarce and ambiguous
- Transformer-generated graph edges (from which centrality is computed) still have unresolved explainability, and many community/enrichment calls (e.g., correct cell type in top 10) rely on manual inspection criteria

## Evidence

- [abstract] Enrichment of community 4 ion-exchange/senescence links: "scPRINT highlights the profound connections between ion exchange, senescence, and chronic inflammation"
- [discussion] GRnnData/BenGRN prerank tooling: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [methods] Network construction by attention averaging: [evidence span withheld — anchor status: not_found]
- [discussion] Prostate atlas application context: "we put scPRINT to the test on a challenging atlas of normal and senescent prostate tissues with benign prostatic hyperplasia"
