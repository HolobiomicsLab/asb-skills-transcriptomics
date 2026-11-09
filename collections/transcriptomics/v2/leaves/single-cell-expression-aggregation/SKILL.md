---
name: single-cell-expression-aggregation
description: Use when a per-cell scRNA-seq count matrix (e.g., cortex_data from Yao
  2021) must be collapsed into group-level expression per cell subclass so ligand
  and target abundances can be scored against NeuronChatDB interaction entries.
when_to_use_negative:
- Input is already pseudobulk/group-level expression with no per-cell labels — Tukey's
  trimean aggregation and the <25% gene filter cannot be applied
- Goal is a spatially constrained communication network — expression aggregation alone
  omits the spatial proximity enrichment step (e.g., 400-micron threshold for the
  MERFISH dataset)
- Data are single-cell proteomics or metabolomics rather than transcriptomics — the
  article notes NeuronChat estimates ligand/target abundance from transcriptomics
  and transcriptomics remains the main data source
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: NeuronChat
  role: Provides createNeuronChat to build the object from the normalized count matrix
    with group.by labels, and run_NeuronChat/net_aggregation to consume the aggregated
    group-level expression for communication scoring
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Its scoring rule is reimplemented within the same NeuronChat workflow on the
    identical aggregated expression, computing ligand abundance without functional
    gene grouping for benchmark comparison
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:6f74133b5134a3d93d4bf0150297ed6c09aff1b40792cd63eeaefb8e2571c78b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - CellPhoneDB
  - NeuronChat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# single-cell-expression-aggregation

## Summary

Aggregates single-cell RNA count matrices into per-cell-group ensemble expression values using Tukey's trimean within the NeuronChat workflow, supplying the ligand/target abundance inputs for neuron-neuron communication inference.

## When to use

Use when a per-cell scRNA-seq count matrix (e.g., cortex_data from Yao 2021) must be collapsed into group-level expression per cell subclass so ligand and target abundances can be scored against NeuronChatDB interaction entries.

## When NOT to use

- Input is already pseudobulk/group-level expression with no per-cell labels — Tukey's trimean aggregation and the <25% gene filter cannot be applied
- Goal is a spatially constrained communication network — expression aggregation alone omits the spatial proximity enrichment step (e.g., 400-micron threshold for the MERFISH dataset)
- Data are single-cell proteomics or metabolomics rather than transcriptomics — the article notes NeuronChat estimates ligand/target abundance from transcriptomics and transcriptomics remains the main data source

## Inputs

- Raw or normalized single-cell RNA count matrix (genes × cells), e.g., cortex_data from Yao 2021
- Per-cell group labels (e.g., 'subclass' for VISp/ALM Glutamatergic cells)
- Database selection string for NeuronChatDB ('mouse' or 'human')

## Outputs

- Group-level ensemble-averaged expression matrix (Tukey's trimean per gene per cell group)
- NeuronChat object with per-interaction communication networks (x@net) ready for run_NeuronChat and net_aggregation(method='weight')

## How to apply

First preprocess raw counts: divide each cell's RNA counts by the cell's total counts, multiply by a scale factor (10,000 by default), add a pseudocount of 1, and natural-log transform. Filter out genes expressed in less than 25% of cells, then compute the ensemble average expression per gene within each cell group (e.g., Glutamatergic subclasses in VISp/ALM) using Tukey's trimean. Build the NeuronChat object with createNeuronChat(normalized_count_mtx, DB='mouse', group.by=subclass) so subsequent ligand abundance computation applies AND logic (geometric mean) across functional gene groups and OR logic (arithmetic mean) within redundant gene groups. The resulting group-level expression feeds run_NeuronChat communication-strength calculations (ligand × target abundance) with M=100 permutation tests.

## Related tools

- **NeuronChat** (Provides createNeuronChat to build the object from the normalized count matrix with group.by labels, and run_NeuronChat/net_aggregation to consume the aggregated group-level expression for communication scoring) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Its scoring rule is reimplemented within the same NeuronChat workflow on the identical aggregated expression, computing ligand abundance without functional gene grouping for benchmark comparison) — https://github.com/sqjin/CellChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx,DB='mouse',group.by = cell_group_vector); x <- run_NeuronChat(x,M=100)
```

## Evaluation signals

- Normalized expression equals natural-log of (counts / cell total × 10,000 + 1) for every cell
- All genes expressed in fewer than 25% of cells are absent from the matrix passed to createNeuronChat
- Group-level values reproduce Tukey's trimean of member cells' log-normalized expression per gene
- After run_NeuronChat with M=100 permutations and Benjamini-Hochberg correction, all links with p >= 0.05 are exactly zero in the network
- net_aggregation(x@net, method='weight') yields a square cell-group × cell-group weight matrix over the same NeuronChatDB mouse interaction entries across methods

## Limitations

- Ligand/target abundance estimated from transcriptomics may be inconsistent with protein or metabolite levels
- Genes expressed in less than 25% of cells are filtered out, which can drop genuinely relevant lowly expressed ligands or receptors
- Permutation-based p-value computation has a trade-off between accuracy and computation speed (time complexity O(n) in the number of permutations)

## Evidence

- [discussion] The RNA counts for each cell were divided by the total counts in the cell and multiplied by a scale factor (10,000 by default), and these values are added with a pseudocount of 1 and then natural-log: "The RNA counts for each cell were divided by the total counts in the cell and multiplied by a scale factor (10,000 by default), and these values are added with a pseudocount of 1 and then natural-log"
- [discussion] the ensemble average expression in a given cell group is calculated using Tukey's trimean: [evidence span withheld — anchor status: not_found]
- [discussion] Because the genes expressed in less than 25% of cells are filtered out: "Because the genes expressed in less than 25% of cells are filtered out"
- [methods] # creat NeuronChat object; choose the database 'mouse' for mouse data; 'human' for human data: "# creat NeuronChat object; choose the database 'mouse' for mouse data; 'human' for human data"
