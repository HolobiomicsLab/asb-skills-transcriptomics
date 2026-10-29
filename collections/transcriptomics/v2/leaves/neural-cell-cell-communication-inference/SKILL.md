---
name: neural-cell-cell-communication-inference
description: Use when scRNA-seq or snRNA-seq data with cell-type/subclass annotations
  from neural tissue (e.
when_to_use_negative:
- The biological system is short-range autocrine/paracrine non-neural signaling —
  existing methods like CellChat or CellPhoneDB with their protein-based ligand-receptor
  databases are appropriate, not NeuronChat's neural-specific DB
- The organism/species is not human or mouse, since NeuronChatDB contains only 190
  human and 183 mouse interaction entries
- Protein- or metabolite-level communication evidence is required rather than transcriptomic
  abundance estimates, which may be inconsistent with protein or metabolite levels
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0605
tools:
- name: NeuronChat
  role: 'Primary tool: creates NeuronChat objects, runs permutation-based communication
    inference, aggregates networks, and visualizes neural-specific signaling'
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency and general-purpose cell-cell communication method whose workflow
    NeuronChat extends; used for comparison in the article's discussion
  repo: https://github.com/sqjin/CellChat
- name: NeuronChatAnalysis2022
  role: Analysis scripts repository for reproducing the figures and results of the
    NeuronChat paper, including the spatial and tutorial workflows
  repo: https://github.com/Wei-BioMath/NeuronChatAnalysis2022
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:77c4a0b05c30acf53eb6cf5d678b9ebb7aa935e04d5609fd011627aca0cb8e37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  - NeuronChatAnalysis2022
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# neural-cell-cell-communication-inference

## Summary

Infer, visualize and analyze neural-specific cell-cell communication networks from single-cell or spatial transcriptomics using NeuronChat, which leverages the neuron-specific NeuronChatDB (373 curated interactions; 190 human, 183 mouse) including small-molecule neurotransmitter signaling.

## When to use

Use when scRNA-seq or snRNA-seq data with cell-type/subclass annotations from neural tissue (e.g., mouse VISp cortex, ASD cortical snRNA-seq) must be turned into significant ligand-receptor interaction networks including neurotransmitter signaling (Glu, GABA) that protein-based methods like CellChat miss.

## When NOT to use

- The biological system is short-range autocrine/paracrine non-neural signaling — existing methods like CellChat or CellPhoneDB with their protein-based ligand-receptor databases are appropriate, not NeuronChat's neural-specific DB
- The organism/species is not human or mouse, since NeuronChatDB contains only 190 human and 183 mouse interaction entries
- Protein- or metabolite-level communication evidence is required rather than transcriptomic abundance estimates, which may be inconsistent with protein or metabolite levels

## Inputs

- Normalized single-cell/single-nucleus RNA-seq count matrix (e.g., processed cortex_data from Yao 2021, human ASD/control snRNA-seq matrices)
- Cell-type/group annotation vector (e.g., mouse cortical cell subclass, human cortical cell type)
- Species-appropriate interaction database selection ('mouse' or 'human') from NeuronChatDB

## Outputs

- Per-interaction communication strength networks (x@net) restricted to significant links (p < 0.05 after BH correction)
- Aggregated communication network over all interaction pairs (net_aggregation output)
- Ranked interaction pairs by information flow (e.g., Nrxn1-Nlgn1, Glu-Gria2) and comparison tables (e.g., asd_vs_control_communication_comparison.csv)
- Outgoing/incoming communication pattern assignments and river/riverplot visualizations

## How to apply

Normalize RNA counts to a 10,000 scale, add a pseudocount of 1 and natural-log transform; filter genes expressed in fewer than 25% of cells. Build a NeuronChat object with createNeuronChat(normalized_count_mtx, DB='mouse' or 'human', group.by=cell_group_vector), grouping cells by subclass or cell type, then infer networks with run_NeuronChat(x, M=100), which permutes group labels 100 times and applies Benjamini-Hochberg correction, keeping communications with p < 0.05 and setting non-significant links to zero. Aggregate per-interaction networks with net_aggregation(x@net, method='weight'), rank interaction pairs with rankNet_Neuron (measure='weight' or 'count'), and decompose outgoing/incoming patterns with identifyCommunicationPattern_Neuron (e.g., k=4). Ligand abundance is computed with AND logic (geometric mean) across functional gene groups and OR logic (arithmetic mean) within redundant gene groups, using Tukey's trimean for ensemble average expression — critical for neurotransmitter signaling like Glu and GABA. For comparative analyses (e.g., ASD vs control), merge objects with mergeNeuronChat and use compareInteractions_Neuron and rankNet_Neuron(mode='comparison').

## Related tools

- **NeuronChat** (Primary tool: creates NeuronChat objects, runs permutation-based communication inference, aggregates networks, and visualizes neural-specific signaling) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency and general-purpose cell-cell communication method whose workflow NeuronChat extends; used for comparison in the article's discussion) — https://github.com/sqjin/CellChat
- **NeuronChatAnalysis2022** (Analysis scripts repository for reproducing the figures and results of the NeuronChat paper, including the spatial and tutorial workflows) — https://github.com/Wei-BioMath/NeuronChatAnalysis2022

## Examples

```
library(NeuronChat); x <- createNeuronChat(normalized_count_mtx, DB='mouse', group.by=cell_group_vector); x <- run_NeuronChat(x, M=100); net_aggregated_x <- net_aggregation(x@net, method='weight')
```

## Evaluation signals

- Reproducing the VISp analysis should detect 109 significant interaction pairs whose top-10 weight-ranked interactions (Nrxn1-Nlgn1, Nrxn3-Nlgn1, Glu-Gria2, Glu-Grin2b, GABA-Gabra1, Glu-Gria4, GABA-Gabrb1, Glu-Grm5, GABA-Gabrg2, Glu-Grik2) match published Fig. 4 results
- Aggregated VISp network should predict the retrograde-labeling projection network with AUROC ≈ 0.832 and AUPRC ≈ 0.915
- Shuffling cell-type labels while keeping graph topology should drop AUROC toward ~0.5 (VISp p = 0.010 ± 0.0036, ALM p = 0.017 ± 0.0048), confirming label specificity
- All reported communication links must satisfy the significance criterion (permutation p < 0.05 after Benjamini-Hochberg correction with M=100)
- In the ASD vs control comparison, NLGN3-to-OPC communication strength (NRXN-NLGN3 links) must be lower in ASD than control

## Limitations

- Ligand/target abundance is estimated from transcriptomics and may be inconsistent with protein or metabolite levels, since single-cell proteomics/metabolomics lag in coverage and throughput
- NeuronChatDB may contain missing curated interaction entries, leading to inference bias, and does not contain subunit stoichiometries of heteromeric receptors
- There is a trade-off between accuracy and computation speed for permutation-based p-value calculations (O(n) in number of permutations); M=100 takes roughly 4 minutes per run on large datasets
- Current model uses only expressions of synthetic enzymes and vesicular transporters for neurotransmitters; stoichiometric effects of metabolism, pathway dependency, auxiliary proteins and downstream genes are not included

## Evidence

- [intro] NeuronChatDB interaction database and workflow: "Development of NeuronChat required manual curation of a new neural-specific database containing 373 entries of intercellular molecular interactions for both human (190) and mouse (183)."
- [readme] README usage example: [evidence span withheld — anchor status: external_doc]
- [discussion] Permutation significance and correction: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
- [discussion] AND/OR logic for neurotransmitter ligand abundance: "the AND logic (i.e., geometric mean) is applied among different groups of genes; since the genes within the same group are redundant for the same function, the OR logic (i.e., arithmetic mean) is"
- [discussion] Gene filtering threshold: "Because the genes expressed in less than 25% of cells are filtered out"
- [discussion] Non-significant links zeroed: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
