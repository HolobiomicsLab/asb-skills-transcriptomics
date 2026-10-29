---
name: functional-group-geometric-mean-aggregation
description: Use when computing ligand or target abundance for NeuronChatDB interaction
  entries (especially non-peptide neurotransmitters like glutamate or GABA) from per-group
  ensemble-averaged gene expression, when contributing genes must be combined across
  functional groups.
when_to_use_negative:
- When running the CellChat or CellPhoneDB workflow on the same data — their implementation
  applies AND logic to all contributing genes without functional grouping and can
  dramatically underestimate small-molecule neurotransmitter ligand abundance
- When the desired quantity is actual neurotransmitter or receptor protein/metabolite
  abundance rather than a transcriptomics-based estimate
- When the interaction is a standard peptide/protein ligand-receptor pair with no
  functional gene grouping defined in NeuronChatDB
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuronChat
  role: Implements the functional-group AND/OR aggregation for ligand and target abundance
    within createNeuronChat/run_NeuronChat
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: NeuronChatDB
  role: Curated database (183 mouse / 190 human entries) supplying the categorized
    contributing genes and receptor subunits per interaction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:d23946ac1db48abbbb638b15a5fd840e5aed988d03d7b480f83eaa37cdd1019e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - NeuronChat
  - NeuronChatDB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# functional-group-geometric-mean-aggregation

## Summary

NeuronChat's rule for computing ligand (non-peptide neurotransmitter) and receptor/target abundance from grouped gene expression: AND logic (geometric mean) across different functional gene groups (e.g., synthetic enzymes vs. vesicular transporters) and OR logic (arithmetic mean) among redundant genes within a group. This functional grouping is what keeps small-molecule neurotransmitter ligand abundance from being underestimated.

## When to use

Use when computing ligand or target abundance for NeuronChatDB interaction entries (especially non-peptide neurotransmitters like glutamate or GABA) from per-group ensemble-averaged gene expression, when contributing genes must be combined across functional groups.

## When NOT to use

- When running the CellChat or CellPhoneDB workflow on the same data — their implementation applies AND logic to all contributing genes without functional grouping and can dramatically underestimate small-molecule neurotransmitter ligand abundance
- When the desired quantity is actual neurotransmitter or receptor protein/metabolite abundance rather than a transcriptomics-based estimate
- When the interaction is a standard peptide/protein ligand-receptor pair with no functional gene grouping defined in NeuronChatDB

## Inputs

- NeuronChat object created via createNeuronChat from a normalized gene-by-cell count matrix with cell-group labels (e.g., cortex_data from Yao 2021 with mouse NeuronChatDB)
- NeuronChatDB interaction entries with categorized contributing genes per functional group (ligand-side and receptor-subunit side)
- Per-gene, per-cell-group ensemble-averaged expression (Tukey's trimean) after the 25%-expression filter

## Outputs

- Ligand abundance values L_i per non-peptide neurotransmitter per sender cell group
- Target (receptor subunit) abundance values T_j per receiver cell group
- Communication strength entries LT_i,j feeding the x@net weight matrix

## How to apply

First compute per-gene, per-cell-group ensemble averages with Tukey's trimean, (Q1 + 2*median + Q3)/4, after filtering genes expressed in fewer than 25% of cells. For each interaction entry in NeuronChatDB, partition the contributing ligand genes into functional groups (e.g., synthesis vs. vesicular transport); combine redundant genes within each group by arithmetic mean (OR logic) and combine the group scores by geometric mean (AND logic), so a zero or near-zero value in any required functional group drives ligand abundance to near zero. Apply the analogous grouping-based computation to receptor subunits to get target abundance. Multiply ligand abundance L_i and target abundance T_j to get communication strength LT_i,j for each sender-receiver cell-group pair before permutation testing (run_NeuronChat, M=100, Benjamini-Hochberg correction, p >= 0.05 set to zero). Do NOT apply a plain geometric mean over all contributing genes without functional grouping — the article notes this (the CellChat/CellPhoneDB workflow) can dramatically underestimate ligand abundance for small-molecule neurotransmitters.

## Related tools

- **NeuronChat** (Implements the functional-group AND/OR aggregation for ligand and target abundance within createNeuronChat/run_NeuronChat) — https://github.com/Wei-BioMath/NeuronChat
- **NeuronChatDB** (Curated database (183 mouse / 190 human entries) supplying the categorized contributing genes and receptor subunits per interaction)

## Examples

```
x <- createNeuronChat(normalized_count_mtx,DB='mouse',group.by=cell_group_vector); x <- run_NeuronChat(x,M=100); net_aggregated_x <- net_aggregation(x@net,method='weight')
```

## Evaluation signals

- Ligand abundance must be near zero whenever any single functional group's (AND-logic) score is near zero — geometric mean across groups enforces this
- Within a functional group, the aggregated value is bounded by min and max of member genes (arithmetic mean/OR logic)
- Resulting communication weight matrix x@net has dimensions equal to number of cell groups squared
- After run_NeuronChat (M=100 permutations, BH correction), entries with p >= 0.05 are exactly zero and significant entries lie in the valid range
- Compare against the ungrouped (all-gene geometric mean) variant: the grouped computation should give substantially higher ligand abundance for neurotransmitter entries

## Limitations

- Transcriptomics-derived ligand/target abundance may be inconsistent with actual protein or metabolite levels
- NeuronChatDB lacks receptor subunit stoichiometries for most heteromeric receptors, and stoichiometric effects of metabolism and pathway dependency are not modeled (only synthetic enzymes and vesicular transporters are used)
- Auxiliary proteins and downstream genes are not yet incorporated, which limits prediction accuracy
- Missing curated interaction entries in NeuronChatDB can bias inference

## Evidence

- [discussion] AND/OR logic grouping: "the AND logic (i.e., geometric mean) is applied among different groups of genes; since the genes within the same group are redundant for the same function, the OR logic (i.e., arithmetic mean) is"
- [discussion] 25% expression filter before aggregation: "Because the genes expressed in less than 25% of cells are filtered out"
- [discussion] why grouping matters (CellChat/CellPhoneDB contrast): "both of CellChat and CellPhoneDB are implemented without categorizing the contributing genes into different functional groups and thus use AND logic (geometric mean or minimum) for all contributing"
- [discussion] missing stoichiometry limitation: "NeuronChatDB does not contain the information of subunit stoichiometries that are largely unknown for most heteromeric receptors"
- [readme] concrete entry point (README): [evidence span withheld — anchor status: external_doc]
