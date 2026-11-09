---
name: permutation-significance-testing-of-communication-links
description: Use when inferred ligand-receptor communication networks from scRNA-seq
  (NeuronChat objects) must be filtered for statistical significance, or when assessing
  stability of significant-link calls across permutation counts.
when_to_use_negative:
- The goal is cell-cell communication based on short-range autocrine/paracrine signaling
  with protein-only ligand-receptor databases — NeuronChatDB is neural-specific and
  built for long-distance neurotransmitter signaling.
- Spatially resolved data requires distance-constrained filtering; use spatial proximity
  enrichment scores (cell/spot label permutation, 1000 permutations) rather than only
  group-label permutation.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: NeuronChat
  role: Runs the label-permutation significance test (run_NeuronChat with M permutations)
    and zeroing of non-significant links
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency of NeuronChat whose computational workflow NeuronChat reuses for
    network implementation
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:7d87b4a1146d3d5445bab9e9f3e1eb7634cf4779d3085f27854b3d2783621317
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# permutation-significance-testing-of-communication-links

## Summary

Determines which inferred neuron-neuron communication links are statistically significant by randomly permuting cell-group labels, recomputing communication strength, correcting p-values with Benjamini-Hochberg, and zeroing non-significant links (p >= 0.05).

## When to use

Use when inferred ligand-receptor communication networks from scRNA-seq (NeuronChat objects) must be filtered for statistical significance, or when assessing stability of significant-link calls across permutation counts.

## When NOT to use

- The goal is cell-cell communication based on short-range autocrine/paracrine signaling with protein-only ligand-receptor databases — NeuronChatDB is neural-specific and built for long-distance neurotransmitter signaling.
- Spatially resolved data requires distance-constrained filtering; use spatial proximity enrichment scores (cell/spot label permutation, 1000 permutations) rather than only group-label permutation.

## Inputs

- Normalized scRNA-seq count matrix (gene-by-cell, scale 10,000, pseudocount 1, natural-log transformed)
- Cell-type/subclass label vector (group.by)
- NeuronChatDB interaction database ('mouse' or 'human')
- Number of permutations M (e.g., 100 or 1000)

## Outputs

- NeuronChat object with communication probabilities and permutation-test p-values per interaction pair (x@net)
- Thresholded communication network with non-significant links (p >= 0.05) set to zero
- Comparison table (CSV) of significant-link sets across permutation counts

## How to apply

Build a NeuronChat object with createNeuronChat() on a normalized gene-by-cell matrix using the 'mouse' (or 'human') NeuronChatDB, grouping cells by subclass. Run run_NeuronChat(x, M=100), which performs a permutation test by randomly permutating group labels of cells and recalculating communication strength; Tukey-trimean-averaged ligand abundance uses geometric mean (AND logic) across functional gene groups and arithmetic mean (OR logic) within redundant groups. Correct the resulting interaction p-values with the Benjamini-Hochberg procedure, call links significant at p < 0.05, and set non-significant communication strengths to zero. To verify stability, repeat with a higher permutation count (e.g., M=1000) under identical settings and compare significant-link sets (overlap counts and per-link p-value agreement), which should show high overlap as reported in Supplementary Fig. 3.

## Related tools

- **NeuronChat** (Runs the label-permutation significance test (run_NeuronChat with M permutations) and zeroing of non-significant links) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency of NeuronChat whose computational workflow NeuronChat reuses for network implementation) — https://github.com/sqjin/CellChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx, DB='mouse', group.by = cell_group_vector); x <- run_NeuronChat(x, M=100); net_aggregated_x <- net_aggregation(x@net, method='weight')
```

## Evaluation signals

- AUROC/AUPRC against the ALM retrograde-labeling ground truth should reproduce 0.764 / 0.768 within numerical tolerance
- Significant-link sets at M=100 and M=1000 should show high overlap, matching Supplementary Fig. 3
- No inferred link should carry non-zero strength where BH-corrected p >= 0.05
- run_NeuronChat must complete without error on the cortex_data example with the 'mouse' DB
- Aggregated network from net_aggregation(x@net, method='weight') with 80%-quantile thresholding should match benchmark-default behavior

## Limitations

- Permutation-based p-value calculation has a trade-off between accuracy and computation speed (time complexity O(n) in number of permutations).
- NeuronChatDB may contain missing curated interaction entries, leading to bias in the inference.
- Genes expressed in less than 25% of cells are filtered out before inference, so sparse neurotransmitter-related genes may be dropped.

## Evidence

- [discussion] The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
- [discussion] communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [discussion] The p-value is corrected for multiple tests by using Benjamini–Hochberg procedure: "The p-value is corrected for multiple tests by using Benjamini–Hochberg procedure"
- [methods] x <- run_NeuronChat(x,M=100): "x <- run_NeuronChat(x,M=100)"
- [discussion] Because the genes expressed in less than 25% of cells are filtered out: "Because the genes expressed in less than 25% of cells are filtered out"
