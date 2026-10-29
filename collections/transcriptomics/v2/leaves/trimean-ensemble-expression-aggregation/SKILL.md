---
name: trimean-ensemble-expression-aggregation
description: Use when a gene-by-cell expression matrix with pre-defined cell-group
  labels (e.g., cortical subclasses from Yao 2021 cortex_data) must be reduced to
  robust per-group expression values before ligand/target abundance or communication-strength
  scoring.
when_to_use_negative:
- The analysis targets spatial proximity or physical-contact signaling rather than
  neurotransmitter-based neural communication; CellChat/CellPhoneDB-style paracrine
  workflows without functional gene grouping will dramatically underestimate small-molecule
  neurotransmitter ligand abundance
- Protein or metabolite abundance data are available and preferred, since transcriptomics-based
  ligand/target estimates may be inconsistent with protein or metabolite levels
- Input is a bulk expression matrix without cell-group labels, since trimean aggregation
  requires per-cell-group quartiles
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuronChat
  role: Implements the trimean-based ensemble averaging, the 25%-expression gene filter,
    and the downstream AND/OR ligand/target abundance computation inside createNeuronChat/run_NeuronChat
  repo: https://github.com/Wei-BioMath/NeuronChat
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:4c2684b292d86dd24d92df6e9b2e4a7c7f1b943b92b69cea03e6a8a37f56ae42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# trimean-ensemble-expression-aggregation

## Summary

Collapse a gene-by-cell normalized scRNA-seq matrix into robust per-gene, per-cell-group ensemble averages using Tukey's trimean, (Q1 + 2*median + Q3)/4, after filtering genes expressed in fewer than 25% of cells. This aggregated expression feeds NeuronChat's ligand/target abundance (AND/OR logic) and communication-strength computations.

## When to use

Use when a gene-by-cell expression matrix with pre-defined cell-group labels (e.g., cortical subclasses from Yao 2021 cortex_data) must be reduced to robust per-group expression values before ligand/target abundance or communication-strength scoring.

## When NOT to use

- The analysis targets spatial proximity or physical-contact signaling rather than neurotransmitter-based neural communication; CellChat/CellPhoneDB-style paracrine workflows without functional gene grouping will dramatically underestimate small-molecule neurotransmitter ligand abundance
- Protein or metabolite abundance data are available and preferred, since transcriptomics-based ligand/target estimates may be inconsistent with protein or metabolite levels
- Input is a bulk expression matrix without cell-group labels, since trimean aggregation requires per-cell-group quartiles

## Inputs

- Normalized, log-transformed gene-by-cell count matrix (counts normalized to 10,000 per cell, pseudocount 1 added, natural-log transformed)
- Cell-group label vector (e.g., mouse VISp subclass labels from cortex_data, Yao 2021)
- NeuronChat interaction database selection ('mouse' or 'human', NeuronChatDB with 183 mouse / 190 human entries)

## Outputs

- Gene-by-cell-group matrix of ensemble-average expression (Tukey's trimean per gene per group)
- Per-group ligand abundance scores (AND/OR logic over functional gene groups)
- Per-group target/receptor abundance scores from receptor subunits

## How to apply

First normalize raw counts per cell to a 10,000 scale, add a pseudocount of 1, and natural-log transform. Load the matrix into a NeuronChat object via createNeuronChat with the appropriate DB ('mouse' or 'human') and a cell_group_vector, then filter out genes expressed in less than 25% of cells. For each remaining gene and each cell group, compute Tukey's trimean, (Q1 + 2*median + Q3)/4, as the ensemble average across the group's cells. These per-group values are then consumed downstream: functional gene groups are combined with AND logic (geometric mean) across groups (e.g., synthetic enzymes vs vesicular transporters for non-peptide neurotransmitters like glutamate/GABA) and OR logic (arithmetic mean) among redundant genes within a group, and communication strength LT_i,j is the product of ligand abundance L_i and target abundance T_j. The trimean is chosen as a robust central-tendency statistic that weights the median twice relative to the quartiles, limiting the influence of outlier cells within a group.

## Related tools

- **NeuronChat** (Implements the trimean-based ensemble averaging, the 25%-expression gene filter, and the downstream AND/OR ligand/target abundance computation inside createNeuronChat/run_NeuronChat) — https://github.com/Wei-BioMath/NeuronChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx, DB='mouse', group.by = cell_group_vector); x <- run_NeuronChat(x, M=100)
```

## Evaluation signals

- Verify the trimean formula numerically: per-gene per-group value equals (Q1 + 2*median + Q3)/4 over the group's cells
- Confirm genes expressed in fewer than 25% of cells are excluded from the aggregated matrix
- Check that the downstream communication weight matrix has dimensions equal to the number of cell groups squared
- Confirm non-significant links (BH-corrected p >= 0.05 after M=100 permutations) are set to zero while significant strengths lie in the valid probability range
- Compare AND-logic ligand abundance against a naive all-genes geometric mean (as CellChat/CellPhoneDB would compute) to confirm functional grouping is applied rather than a flat minimum/geomean

## Limitations

- Ligand/target abundance estimated from transcriptomics may be inconsistent with protein or metabolite levels, since single-cell proteomics and metabolomics lag behind transcriptomics in coverage and throughput
- There is a trade-off between accuracy and computation speed for the downstream permutation-based p-value calculations, which are O(n) in the number of permutations
- NeuronChatDB may contain missing curated interaction entries and lacks heteromeric receptor subunit stoichiometries, which can bias downstream abundance-based inference

## Evidence

- [discussion] trimean ensemble average per group: [evidence span withheld — anchor status: not_found]
- [discussion] AND/OR logic for ligand abundance: "the AND logic (i.e., geometric mean) is applied among different groups of genes; since the genes within the same group are redundant for the same function, the OR logic (i.e., arithmetic mean) is"
- [discussion] 25% expression gene filter: "Because the genes expressed in less than 25% of cells are filtered out"
- [discussion] count normalization recipe: "The RNA counts for each cell were divided by the total counts in the cell and multiplied by a scale factor (10,000 by default), and these values are added with a pseudocount of 1 and then natural-log"
- [readme] createNeuronChat entry point from README: [evidence span withheld — anchor status: external_doc]
