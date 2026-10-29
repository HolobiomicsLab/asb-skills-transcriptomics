---
name: transcription-factor-target-enrichment-analysis
description: Use when evaluating whether inferred gene regulatory network edges are
  biologically meaningful — e.g., testing if a TF's neighbors in a cell-type-specific
  network match its ENCODE-validated targets.
when_to_use_negative:
- The network has not yet been inferred/loaded — this skill validates existing networks,
  it does not produce them
- No ENCODE TF-target annotations are available for the TFs in the network, leaving
  no gene sets to prerank against
- The goal is TF-hub or cell-type-marker enrichment rather than per-TF target-set
  validation — use the corresponding prerank workflows against TF gene sets or pangaloDB
  marker sets instead
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: scPRINT
  role: Source of the cell-type-specific inferred gene networks whose TF-target connections
    are being validated
  repo: https://github.com/cantinilab/scPRINT
- name: BenGRN
  role: Benchmarking suite providing the prerank enrichment setup used to test TF
    target enrichment in inferred networks
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Gene regulatory network data structure used to derive per-gene ordered scores
    (row-wise Target-Hub / column-wise Regulator-Hub edge-score sums) for prerank
    enrichment
  repo: https://github.com/cantinilab/GRnnData
- name: scGPT
  role: Comparison method whose inferred networks are subjected to the same TF-target
    enrichment test
  repo: https://github.com/jkobject/scGPT
- name: Python
  role: Implementation environment for the enrichment workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:882c88757cb575fe6f87d5e3748f0340dabc355b222254060aaff0e2145480b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - BenGRN
  - GRnnData
  - Python
  - scGPT
  - scPRINT
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# transcription-factor-target-enrichment-analysis

## Summary

A biological-validity check on inferred cell-type-specific gene networks: for each transcription factor present in a network, rank the TF's connection scores and test whether its neighbors are enriched for the TF's ENCODE-validated gene targets using prerank enrichment, as done to benchmark scPRINT against scGPT, Geneformer, GENIE3, and DeepSEM.

## When to use

Use when evaluating whether inferred gene regulatory network edges are biologically meaningful — e.g., testing if a TF's neighbors in a cell-type-specific network match its ENCODE-validated targets.

## When NOT to use

- The network has not yet been inferred/loaded — this skill validates existing networks, it does not produce them
- No ENCODE TF-target annotations are available for the TFs in the network, leaving no gene sets to prerank against
- The goal is TF-hub or cell-type-marker enrichment rather than per-TF target-set validation — use the corresponding prerank workflows against TF gene sets or pangaloDB marker sets instead

## Inputs

- Inferred cell-type-specific weighted gene networks (adjacency matrices / GRnnData objects) per method across 26 cell types
- ENCODE-validated TF-target gene sets per transcription factor

## Outputs

- Per-TF, per-cell-type prerank enrichment results against ENCODE TF-target sets
- Aggregated count tables of significantly enriched TFs per method and cell type (e.g., % of TFs with enriched ENCODE-validated targets)

## How to apply

Load the inferred cell-type-specific gene networks produced by each benchmarked method (scPRINT, scGPT, Geneformer, GENIE3, DeepSEM) across the 26 cell types. For each transcription factor in a network, rank its connection scores to other genes and run prerank enrichment (GSEA prerank setup via BenGRN/GRnnData) against the TF's ENCODE-validated target set, using all genes in the network as the prerank background. Aggregate enrichment outcomes per method and cell type into count tables, e.g., the fraction of TFs whose ENCODE targets are significantly enriched among their network connections. The rationale is that a biologically meaningful network should connect each TF preferentially to genes it is known to regulate; in the scPRINT paper this revealed that 20% of TFs with ENCODE data have significantly enriched connections in scPRINT networks, outperforming all other benchmarked methods.

## Related tools

- **scPRINT** (Source of the cell-type-specific inferred gene networks whose TF-target connections are being validated) — https://github.com/cantinilab/scPRINT
- **BenGRN** (Benchmarking suite providing the prerank enrichment setup used to test TF target enrichment in inferred networks) — https://github.com/jkobject/benGRN
- **GRnnData** (Gene regulatory network data structure used to derive per-gene ordered scores (row-wise Target-Hub / column-wise Regulator-Hub edge-score sums) for prerank enrichment) — https://github.com/cantinilab/GRnnData
- **scGPT** (Comparison method whose inferred networks are subjected to the same TF-target enrichment test) — https://github.com/jkobject/scGPT
- **Python** (Implementation environment for the enrichment workflow)

## Examples

```
from bengrn import BenGRN; grn = GRnnData.from_adjacency(adj, genes); scores = grn.get_gene_scores(method='target_hub'); res = BenGRN.prerank(scores, gene_sets=encode_tf_targets[tf], background=grn.genes)
```

## Evaluation signals

- Enrichment count table is produced for all 26 cell types and every benchmarked method
- Each TF's prerank run uses the network's full gene set as background, so p-values are comparable across TFs and methods
- Only TFs with ENCODE-validated target data enter the denominator when computing the fraction of enriched TFs (e.g., 20% in scPRINT networks)
- Method-level ranking should reproduce the paper's ordering: scPRINT outperforms all other benchmarked methods on TF-target enrichment; scGPT and scPRINT show strong TF node enrichment while GENIE3, DeepSEM, and Geneformer v2 networks are not enriched for TFs
- Significance calls use a conventional corrected p-value cutoff of 0.05, consistent with the paper's other edge-definition thresholds

## Limitations

- ENCODE-validated TF-target sets are incomplete and ambiguous ground truths — ChIP-seq and literature-based gold standards 'remain scarce and ambiguous', so non-enrichment does not prove a network is wrong
- No perfect ground truths exist for gene network inference benchmarking, making enrichment-based validation an indirect quality signal
- Results depend on how per-gene scores are derived from the network (row-wise vs column-wise edge-score sums or eigenvector centrality), which can change enrichment outcomes

## Evidence

- [results] 20% of TFs with ENCODE data have connections significantly enriched for their ENCODE-validated gene targets, with scPRINT outperforming all other benchmarked methods: [evidence span withheld — anchor status: not_found]
- [discussion] BenGRN and GRnnData benchmarking suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [other] per-gene ordered scores via row/column sums: [evidence span withheld — anchor status: not_found]
- [discussion] no perfect ground truths caveat: "A challenging aspect of GN inference is that no perfect ground truths exist"
- [discussion] scarce and ambiguous gold standards: "ChIP-seq, perturb-seq, and literature-based ground truths remain scarce and ambiguous"
