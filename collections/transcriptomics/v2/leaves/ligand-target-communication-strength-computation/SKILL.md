---
name: ligand-target-communication-strength-computation
description: Use when reproducing or auditing NeuronChat's ligand/target abundance
  and communication-strength computation for small-molecule neurotransmitter signaling
  from single-cell expression data.
when_to_use_negative:
- Input is protein-based ligand-receptor signaling (e.g., peptide ligands handled
  by CellChat/CellPhoneDB) — the AND/OR functional-grouping logic here is specific
  to non-peptide small-molecule neurotransmitters.
- You only need the aggregated, downstream network-level summary and don't need to
  reconstruct/verify how ligand/target abundance or communication strength is derived.
- Spatial data requiring the spatial proximity enrichment score / spatially constrained
  network filtering — that is a separate, additional step beyond the basic ligand-target
  computation.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: NeuronChat
  role: Implements the AND/OR ligand-target abundance computation, Tukey's trimean
    ensemble averaging, communication-strength product, permutation significance testing,
    and network aggregation via run_NeuronChat and net_aggregation.
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: NeuronChatDB
  role: Curated neural-specific interaction database (373 entries; 190 human, 183
    mouse) supplying the ligand functional gene groups (e.g., synthetic enzymes, vesicular
    transporters) and receptor subunits used for abundance computation.
  repo: https://github.com/Wei-BioMath/NeuronChat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:16292f2b9708a0b0832a2700df92495ac35fcf9ab6347ab5639edded8c76729e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - NeuronChat
  - NeuronChatDB
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ligand-target-communication-strength-computation

## Summary

Reconstructs how NeuronChat computes non-peptide neurotransmitter ligand abundance and receptor/target abundance from scRNA-seq expression (AND logic as geometric mean across functional gene groups, OR logic as arithmetic mean within redundant groups) and derives communication strength as the ligand-target product for each sender-receiver cell-group pair.

## When to use

Use when reproducing or auditing NeuronChat's ligand/target abundance and communication-strength computation for small-molecule neurotransmitter signaling from single-cell expression data.

## When NOT to use

- Input is protein-based ligand-receptor signaling (e.g., peptide ligands handled by CellChat/CellPhoneDB) — the AND/OR functional-grouping logic here is specific to non-peptide small-molecule neurotransmitters.
- You only need the aggregated, downstream network-level summary and don't need to reconstruct/verify how ligand/target abundance or communication strength is derived.
- Spatial data requiring the spatial proximity enrichment score / spatially constrained network filtering — that is a separate, additional step beyond the basic ligand-target computation.

## Inputs

- Normalized gene-by-cell count matrix (scaled to 10,000, pseudocount 1, natural-log transformed), e.g. cortex_data from Yao 2021
- Cell group/label vector (e.g., cortical cell subclass labels)
- NeuronChatDB interaction database (DB='mouse' or DB='human') with ligand and receptor gene groupings

## Outputs

- Per-interaction ligand abundance vector L_i per sender cell group
- Per-interaction target/receptor abundance vector T_j per receiver cell group
- Communication-strength matrix LT_ij (sender cell groups × receiver cell groups), with non-significant entries (p >= 0.05) set to zero

## How to apply

Load a gene-by-cell normalized count matrix (counts scaled to 10,000, pseudocount 1, natural-log transformed), group cells by a class label such as cortical subclass, and compute per-gene group averages using Tukey's trimean (Q1 + 2*median + Q3)/4 after filtering out genes expressed in fewer than 25% of cells. For each NeuronChatDB interaction entry, categorize non-peptide neurotransmitter ligand genes into biological function groups (e.g., synthetic enzymes vs. vesicular transporters); apply AND logic as the geometric mean across different functional groups and OR logic as the arithmetic mean among redundant genes within the same group — this is essential because applying AND logic to all genes (as CellChat/CellPhoneDB would) dramatically underestimates small-molecule neurotransmitter abundance. Compute receptor/target abundance analogously from receptor subunits, then set communication strength LT_ij = L_i × T_j for each sender-receiver cell-group pair. This mirrors the core computation inside run_NeuronChat(x, M=100), which adds a permutation test (randomly permuting group labels) with Benjamini–Hochberg correction and zeros out communications with p-value ≥ 0.05.

## Related tools

- **NeuronChat** (Implements the AND/OR ligand-target abundance computation, Tukey's trimean ensemble averaging, communication-strength product, permutation significance testing, and network aggregation via run_NeuronChat and net_aggregation.) — https://github.com/Wei-BioMath/NeuronChat
- **NeuronChatDB** (Curated neural-specific interaction database (373 entries; 190 human, 183 mouse) supplying the ligand functional gene groups (e.g., synthetic enzymes, vesicular transporters) and receptor subunits used for abundance computation.) — https://github.com/Wei-BioMath/NeuronChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx, DB='mouse', group.by = cell_group_vector); x <- run_NeuronChat(x, M=100); net_aggregated_x <- net_aggregation(x@net, method='weight')
```

## Evaluation signals

- Output weight matrix has dimensions equal to (number of cell groups) × (number of cell groups).
- Non-significant entries (p >= 0.05 after Benjamini–Hochberg correction) are exactly zero; significant entries are positive finite values consistent with L_i × T_j products.
- Permutation test reproducibility: rerunning run_NeuronChat(x, M=100) with the same seed yields comparable significance patterns across the M=100 label permutations.
- Ligand abundance for non-peptide neurotransmitters (e.g., glutamate, GABA) is non-trivially small — i.e., the AND/OR functional-grouping logic, not plain geometric mean across all genes, is being applied (contrast with CellChat/CellPhoneDB underestimation).
- Aggregated network via net_aggregation(x@net, method='weight') is symmetric in total link weight (sender vs. receiver totals match across the full matrix).

## Limitations

- NeuronChat estimates ligand/target abundance from transcriptomics, which can be inconsistent with actual protein or metabolite levels.
- NeuronChatDB does not contain subunit stoichiometries of heteromeric receptors, which are largely unknown, limiting target-abundance accuracy.
- The ligand-abundance model uses only expressions of synthetic enzymes and vesicular transporters; stoichiometric effects of metabolism and pathway dependency are not included.
- Curated interaction entries in NeuronChatDB may be incomplete, leading to inference bias.

## Evidence

- [discussion] AND logic across functional gene groups: "the AND logic (i.e., geometric mean) is applied among different groups of genes; since the genes within the same group are redundant for the same function, the OR logic (i.e., arithmetic mean) is"
- [discussion] Tukey's trimean ensemble average: [evidence span withheld — anchor status: not_found]
- [discussion] 25% expression filter: "Because the genes expressed in less than 25% of cells are filtered out"
- [discussion] Communication strength significance cutoff: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [readme] createNeuronChat invocation: [evidence span withheld — anchor status: external_doc]
- [methods] run_NeuronChat invocation: "x <- run_NeuronChat(x,M=100)"
