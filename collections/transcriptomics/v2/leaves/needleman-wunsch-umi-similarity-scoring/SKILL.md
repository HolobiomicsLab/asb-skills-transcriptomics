---
name: needleman-wunsch-umi-similarity-scoring
description: Use when recovering/collapsing UMIs from noisy Nanopore single-cell long
  reads where fixed edit-distance collapsing (e.g., DBSCAN eps=2, Sicelore2, FLAMES)
  overestimates UMI numbers, or when needing an alignment-score-based similarity graph
  for UMI clustering.
when_to_use_negative:
- Reads come from short-read Illumina data with low per-base error rates, where fixed
  edit-distance UMI collapsing suffices
- UMI sequences are unknown/absent or have not been extracted and barcode-checked
  via the LongcellPre sliding-window and k-mer dictionary step
- The goal is generic sequence alignment rather than UMI count correction — use a
  standard aligner instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Pipeline that recovers cell barcodes/UMIs, computes Needleman-Wunsch UMI similarity,
    and runs the graph-based clustering and pruning
  repo: github.com/yuntianf/Longcell
- name: igraph
  role: R package (v1.5.1) providing the iterative Louvain algorithm used to cluster
    the NW similarity graph
- name: dbscan
  role: Fixed-edit-distance (eps=2) baseline for benchmarking UMI count inflation
    against the NW-score approach
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:4f5ffacbd921463ed17648a897ab5500c057ae99d44afff37576944928de1674
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yuntianf/Longcell
  tools:
  - Longcell
  - dbscan
  - igraph
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# needleman-wunsch-umi-similarity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores putative UMI sequences pairwise with Needleman-Wunsch alignment to build a similarity graph for error-tolerant UMI clustering in Longcell, replacing fixed-edit-distance collapsing that inflates UMI counts under Nanopore's 85-99% per-base sequencing accuracy.

## When to use

Use when recovering/collapsing UMIs from noisy Nanopore single-cell long reads where fixed edit-distance collapsing (e.g., DBSCAN eps=2, Sicelore2, FLAMES) overestimates UMI numbers, or when needing an alignment-score-based similarity graph for UMI clustering.

## When NOT to use

- Reads come from short-read Illumina data with low per-base error rates, where fixed edit-distance UMI collapsing suffices
- UMI sequences are unknown/absent or have not been extracted and barcode-checked via the LongcellPre sliding-window and k-mer dictionary step
- The goal is generic sequence alignment rather than UMI count correction — use a standard aligner instead

## Inputs

- Putative UMI sequences recovered from Nanopore long reads (e.g., 10 bp adapter UMIs, 12mer adapters upstream of cell barcodes)
- Cell barcode assignments per read (recovered via 10 bp/2 bp sliding-window search and k=6 k-mer dictionary)
- Meta-isoform group assignments per cell (isoforms interconvertible by truncation or short-exon misalignment)
- UMI length l (used for the l/2+2 and l/2+4 edge-weight cutoffs)

## Outputs

- Needleman-Wunsch pairwise similarity matrix/graph per meta-isoform group
- UMI clusters with estimated UMI counts (Louvain, min_cut > 3)
- Singleton error ratio r estimated from the 12mer adapter NW score distribution
- Pruned per-isoform UMI counts (m - p after scatter-reduction pruning)

## How to apply

Within each meta-isoform group of each cell, compute pairwise Needleman-Wunsch scores between recovered putative UMIs (length l, e.g., 10 bp adapters) and build an undirected weighted graph with UMIs as nodes and NW scores as edge weights. Filter out edges with weight below l/2+2; optionally continue splitting until the minimum edge weight exceeds l/2+4. Apply the iterative Louvain algorithm (R igraph v1.5.1), recursively splitting subgraphs until all are highly connected (min_cut > 3), to obtain UMI clusters whose count estimates stay stable across PCR amplification folds 2-50x. Downstream, estimate the error-driven singleton ratio r from the NW score distribution of 12mer adapters upstream of confidently identified cell barcodes (counting adapters scoring below k = l/2+2), then prune the left tail of the per-isoform cluster-size distribution by the r-based balance condition (removed sum >= r * remaining sum / (1-r)) to correct UMI scattering. The rationale: unlike whitelisted cell barcodes, UMIs are short and randomly generated, so alignment-score similarity tolerates Nanopore errors that break fixed edit-distance thresholds.

## Related tools

- **Longcell** (Pipeline that recovers cell barcodes/UMIs, computes Needleman-Wunsch UMI similarity, and runs the graph-based clustering and pruning) — github.com/yuntianf/Longcell
- **igraph** (R package (v1.5.1) providing the iterative Louvain algorithm used to cluster the NW similarity graph)
- **dbscan** (Fixed-edit-distance (eps=2) baseline for benchmarking UMI count inflation against the NW-score approach)

## Examples

```
g <- graph_from_adjacency_matrix(nw_scores > (l/2+2), mode="undirected", weighted=TRUE); clusters <- cluster_louvain(g)
```

## Evaluation signals

- Estimated UMI counts remain stable across simulated amplification folds 2-50x, with no inflation from UMI scattering
- The DBSCAN (eps=2) baseline overestimates UMI counts at high sequencing error while the NW-score clustering does not, matching the Fig. 1B trend
- Pruned Nanopore UMI cluster-size distribution matches the Illumina counterpart for a single-isoform gene (Supplementary Fig. 5C / Fig. 1C)
- All Louvain subgraphs are highly connected (min_cut > 3) and minimum edge weight exceeds l/2+4 at termination
- The estimated singleton ratio r satisfies the pruning balance condition removed sum >= r * remaining sum / (1-r)

## Limitations

- Fixed-edit-distance baselines overestimate UMI counts under high sequencing error, so cutoffs must scale with UMI length l
- Small UMI clusters in the left tail of the size distribution are treated as errors and pruned, assuming within-isoform cluster sizes are comparable
- Read truncation and misaligned small internal exons can still misidentify isoforms upstream of UMI clustering
- No changelog was found for the Longcell repository

## Evidence

- [methods] build a graph with UMIs as nodes and Needleman-Wunsch scores as edge weights, and filter out edges with weight below l/2+2 for UMI length l: [evidence span withheld — anchor status: not_found]
