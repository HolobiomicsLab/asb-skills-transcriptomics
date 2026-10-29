---
name: cell-type-specific-network-annotation
description: Use when you have cell-type-specific gene/GRN networks (edge-score matrices
  per cell type) and need to validate biological relevance via TF, cell-type-marker,
  and ENCODE TF-target enrichment before benchmarking inference methods.
when_to_use_negative:
- The input is raw single-cell expression data rather than inferred networks — run
  network inference (e.g., scPRINT attention-based extraction) first
- Networks are not cell-type-specific or no marker gene sets (e.g., PanglaoDB) exist
  for the cell types in question, making the cell-type enrichment criterion inapplicable
- A validated gold-standard ground-truth network exists for the task — direct benchmarking
  against ground truth is preferable to enrichment-based annotation, since 'no perfect
  ground truths exist' is why this indirect validation is used
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: scPRINT
  role: Source of the inferred cell-type-specific networks whose nodes are tested
    for TF, marker, and ENCODE-target enrichment; outperformed all other benchmarked
    methods
  repo: https://github.com/cantinilab/scPRINT
- name: BenGRN
  role: Benchmarking suite providing the prerank setup used to derive per-gene ordered
    scores from networks and run enrichment
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Benchmarking suite used together with BenGRN to process gene networks inferred
    from single-cell RNA sequencing into enrichment-testable scores
  repo: https://github.com/cantinilab/GRnnData
- name: Python
  role: Implementation environment, including NetworkX for eigenvector centrality
    computation on the network nodes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:6f26a4b558bf77855c1970df7ebdadbd2c36efabf2e6239a6d94a338adbc5e77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - BenGRN
  - GRnnData
  - Python
  - scPRINT
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-type-specific-network-annotation

## Summary

Annotates inferred cell-type-specific gene networks by testing whether their nodes are enriched for transcription factors (hubs), the correct cell type's PanglaoDB markers, and ENCODE-validated TF targets, to demonstrate the networks are biologically meaningful. Used in the scPRINT benchmark to compare scPRINT against scGPT, Geneformer, GENIE3, and DeepSEM.

## When to use

Use when you have cell-type-specific gene/GRN networks (edge-score matrices per cell type) and need to validate biological relevance via TF, cell-type-marker, and ENCODE TF-target enrichment before benchmarking inference methods.

## When NOT to use

- The input is raw single-cell expression data rather than inferred networks — run network inference (e.g., scPRINT attention-based extraction) first
- Networks are not cell-type-specific or no marker gene sets (e.g., PanglaoDB) exist for the cell types in question, making the cell-type enrichment criterion inapplicable
- A validated gold-standard ground-truth network exists for the task — direct benchmarking against ground truth is preferable to enrichment-based annotation, since 'no perfect ground truths exist' is why this indirect validation is used

## Inputs

- Cell-type-specific gene network edge-score matrices (one per inference method per cell type, e.g., scPRINT, scGPT, Geneformer, GENIE3, DeepSEM across 26 cell types)
- Transcription factor gene sets (for TF-hub enrichment)
- PanglaoDB cell-type marker gene sets
- ENCODE-validated TF-target gene sets
- List of 26 cell type labels

## Outputs

- Per-method, per-cell-type enrichment results (TF enrichment, cell-type marker enrichment, ENCODE TF-target enrichment)
- Aggregate enrichment count tables per method and cell type (e.g., correct cell type in top 10 enriched cell types)
- Per-TF enrichment statistics against ENCODE-validated targets

## How to apply

Load the inferred cell-type-specific gene networks for each method (scPRINT, scGPT, Geneformer, GENIE3, DeepSEM) across the 26 cell types. Convert each network to per-gene ordered scores in a prerank setup using BenGRN/GRnnData: sum edge scores row-wise (Target-Hub) or column-wise (Regulator-Hub), or compute eigenvector centrality with NetworkX, using all genes in the set as the prerank background. Run GSEA prerank on the node scores against transcription factor gene sets per cell type, then run prerank enrichment against PanglaoDB cell-type marker gene sets to test whether each network is enriched for its correct cell type. For each TF in each network, rank its connection scores and run prerank enrichment against that TF's ENCODE-validated target set. Finally, aggregate enrichment results per method and cell type into count tables (e.g., whether the correct cell type appears among the top 10 enriched cell types) to compare methods.

## Related tools

- **scPRINT** (Source of the inferred cell-type-specific networks whose nodes are tested for TF, marker, and ENCODE-target enrichment; outperformed all other benchmarked methods) — https://github.com/cantinilab/scPRINT
- **BenGRN** (Benchmarking suite providing the prerank setup used to derive per-gene ordered scores from networks and run enrichment) — https://github.com/jkobject/benGRN
- **GRnnData** (Benchmarking suite used together with BenGRN to process gene networks inferred from single-cell RNA sequencing into enrichment-testable scores) — https://github.com/cantinilab/GRnnData
- **Python** (Implementation environment, including NetworkX for eigenvector centrality computation on the network nodes)

## Examples

```
import networkx as nx; G = nx.from_pandas_adjacency(net_scores, create_using=nx.DiGraph); node_scores = nx.eigenvector_centrality(G)
```

## Evaluation signals

- The enrichment count table is produced for all 26 cell types for every benchmarked method
- A network is judged cell-type-enriched when its cell type name is manually identified in the top 10 cell types of the PanglaoDB enrichment results
- For ENCODE TF-target analysis, ~20% of TFs with ENCODE data in scPRINT networks show significantly enriched connections for their ENCODE-validated targets, and scPRINT should outperform the other methods
- TF node enrichment should be strong for scPRINT and scGPT while GENIE3, DeepSEM, and Geneformer v2 networks should NOT be enriched for TFs — a sanity-check contrast
- Per-gene scores cover all genes in the network set (prerank background), not a filtered subset

## Limitations

- No perfect ground truths exist for gene network inference, so enrichment-based annotation is an indirect proxy for biological meaningfulness rather than validation against known edges
- ChIP-seq, perturb-seq, and literature-based ground truths (including ENCODE targets) remain scarce and ambiguous, limiting coverage of testable TFs
- The cell-type-enrichment criterion is partly manual (cell type name identified in top 10), introducing human judgment into the acceptance decision

## Evidence

- [discussion] BenGRN and GRnnData as benchmarking suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [methods] Network extraction from attention matrices: [evidence span withheld — anchor status: not_found]
- [other] Prerank score derivation workflow: [evidence span withheld — anchor status: not_found]
- [other] ENCODE TF-target enrichment finding: [evidence span withheld — anchor status: not_found]
- [discussion] No perfect ground truths: "A challenging aspect of GN inference is that no perfect ground truths exist"
