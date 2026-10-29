---
name: information-flow-ranking-of-interaction-pairs
description: Use when a NeuronChat run has produced per-interaction communication
  networks and you need to rank all significant interaction pairs by information flow
  (weight and/or count) to report the top-ranked neuron-neuron signaling interactions.
when_to_use_negative:
- The dataset is not neural tissue — NeuronChatDB is neural-specific (short-range
  autocrine/paracrine signaling frameworks like CellChat or CellPhoneDB are alternatives
  for non-neural contexts)
- The goal is multi-dataset comparison rather than single-dataset ranking — use the
  comparative-analysis workflow of NeuronChat instead of single-dataset rankNet_Neuron
  mode
- Input lacks cell-group labels or subclass annotations — ranking requires a predefined
  grouping (group.by vector) passed to createNeuronChat
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuronChat
  role: Infers per-interaction communication networks (run_NeuronChat), aggregates
    them (net_aggregation), and ranks interaction pairs by information flow (rankNet_Neuron
    with measure='weight' and 'count')
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency of NeuronChat; the same computational workflow was implemented
    for CellChat and CellPhoneDB benchmarks, where lack of functional gene grouping
    underestimates neurotransmitter ligand abundance
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:1455acb4a75e1f7ff74094cf560e011687920f762f86511ea63b49ed0e797a46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# information-flow-ranking-of-interaction-pairs

## Summary

Ranks NeuronChat-inferred ligand-receptor interaction pairs by total information flow (aggregated communication weight) to identify the dominant signaling axes, e.g., the top-10 pairs (Nrxn1-Nlgn1, Nrxn3-Nlgn1, Glu-Gria2, Glu-Grin2b, GABA-Gabra1, etc.) among 21 VISp cell subclasses.

## When to use

Use when a NeuronChat run has produced per-interaction communication networks and you need to rank all significant interaction pairs by information flow (weight and/or count) to report the top-ranked neuron-neuron signaling interactions.

## When NOT to use

- The dataset is not neural tissue — NeuronChatDB is neural-specific (short-range autocrine/paracrine signaling frameworks like CellChat or CellPhoneDB are alternatives for non-neural contexts)
- The goal is multi-dataset comparison rather than single-dataset ranking — use the comparative-analysis workflow of NeuronChat instead of single-dataset rankNet_Neuron mode
- Input lacks cell-group labels or subclass annotations — ranking requires a predefined grouping (group.by vector) passed to createNeuronChat

## Inputs

- NeuronChat object with per-interaction communication networks (output of run_NeuronChat with M=100 permutations)
- Gene-by-cell normalized count matrix from mouse VISp scRNA-seq (processed Yao 2021 cortex_data) with subclass group labels
- NeuronChatDB 'mouse' interaction database (183 curated mouse interaction entries)

## Outputs

- Ranked list of interaction pairs by total information flow (weight and count measures), e.g., top-10 table of pair names
- Aggregated intercellular communication network (net_aggregation weight matrix across 109 significant pairs)

## How to apply

After inferring networks with run_NeuronChat (M=100 permutations) on a NeuronChat object built from a gene-by-cell matrix with cells grouped by subclass using the 'mouse' interaction DB, aggregate the per-interaction networks with net_aggregation(method='weight') to obtain the total communication weight per pair. Rank all significant interaction pairs by information flow using rankNet_Neuron in single-dataset mode with both measure='weight' and measure='count' — weight captures summed communication strength (product of ligand and target abundance, where ligand abundance uses AND logic as geometric mean across functional gene groups), and count captures the number of significant links. Report the top-10 ranked pairs; in the mouse VISp analysis of the Yao 2021 cortex_data (excluding Car3, CR, DG, L2/3 IT PPP, and L5/6 IT TPE-ENT subclasses), this yields 109 significant pairs with Nrxn1-Nlgn1, Nrxn3-Nlgn1, Glu-Gria2, Glu-Grin2b, GABA-Gabra1, Glu-Gria4, GABA-Gabrb1, Glu-Grm5, GABA-Gabrg2, and Glu-Grik2 as the top 10 by weight. Only communications with p < 0.05 (Benjamini-Hochberg corrected) are retained; non-significant links are set to zero, so genes expressed in fewer than 25% of cells and weak links do not contribute to rankings.

## Related tools

- **NeuronChat** (Infers per-interaction communication networks (run_NeuronChat), aggregates them (net_aggregation), and ranks interaction pairs by information flow (rankNet_Neuron with measure='weight' and 'count')) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency of NeuronChat; the same computational workflow was implemented for CellChat and CellPhoneDB benchmarks, where lack of functional gene grouping underestimates neurotransmitter ligand abundance) — https://github.com/sqjin/CellChat

## Examples

```
x <- run_NeuronChat(x,M=100); net_aggregated_x <- net_aggregation(x@net,method = 'weight')
```

## Evaluation signals

- The VISp run yields exactly 109 significant interaction pairs among the 21 retained cell subclasses
- The top-10 weight-ranked interactions match the published ordering: Nrxn1-Nlgn1, Nrxn3-Nlgn1, Glu-Gria2, Glu-Grin2b, GABA-Gabra1, Glu-Gria4, GABA-Gabrb1, Glu-Grm5, GABA-Gabrg2, Glu-Grik2
- Ranking changes consistently when switching measure='weight' vs measure='count' (count reflects number of significant links, weight their summed strength)
- All non-significant links (BH-corrected p >= 0.05) are zero in the network before aggregation, and no gene expressed in <25% of cells contributes
- rankNet_Neuron output reproduces the published Fig. 4 rankings via output_matches_reference comparison

## Limitations

- Genes expressed in less than 25% of cells are filtered out before inference, so low-expression interactions can never be ranked
- Ligand/target abundance estimated from transcriptomics may be inconsistent with protein or metabolite levels
- NeuronChatDB does not contain subunit stoichiometries of heteromeric receptors, and missing curated interaction entries may bias inference
- There is a trade-off between accuracy and computation speed for permutation-based p-value calculations (O(n) in the number of permutations)

## Evidence

- [methods] Aggregate the communication networks over all interaction pairs: "net_aggregated_x <- net_aggregation(x@net,method = 'weight')"
- [readme] Permutation significance testing: "# calculation of communication networks  
x <- run_NeuronChat(x,M=100)"
- [results] Top-10 information flow ranking in VISp: [evidence span withheld — anchor status: not_found]
- [discussion] Significance filter for ranked pairs: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [discussion] Ligand abundance aggregation logic: "the AND logic (i.e., geometric mean) is applied among different groups of genes; since the genes within the same group are redundant for the same function, the OR logic (i.e., arithmetic mean) is"
