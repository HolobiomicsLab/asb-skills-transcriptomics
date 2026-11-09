---
name: neuronchat-object-creation-and-network-aggregation
description: Use when starting from a normalized scRNA-seq/snRNA-seq or spatial transcriptomics
  count matrix with cell-type annotations and you need inferred neural-specific ligand-receptor
  communication networks aggregated by cell group (human DB='human', mouse DB='mouse').
when_to_use_negative:
- Input is raw unnormalized counts or lacks cell-group labels, so createNeuronChat(...,
  group.by=...) cannot be populated
- Goal is small-molecule neurotransmitter inference via CellChat/CellPhoneDB without
  functional gene grouping — the paper shows this dramatically underestimates ligand
  abundance for glutamate/GABA compared to NeuronChat's grouped AND/OR logic
- 'Input is protein/metabolite-level data: NeuronChat estimates ligand/target abundance
  from transcriptomics, which may be inconsistent with protein or metabolite levels'
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: NeuronChat
  role: 'Primary R package: createNeuronChat builds the object, run_NeuronChat infers
    permutation-tested communication networks, net_aggregation collapses them over
    interaction pairs'
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency of NeuronChat (install via devtools::install_github('sqjin/CellChat'));
    the same NeuronChat computational workflow is also used to implement CellChat
    for comparison
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:24ed043d37d54aa1225cb0304afc31bdf8ecd9919197a9081918f5810b6caf99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# neuronchat-object-creation-and-network-aggregation

## Summary

Core NeuronChat workflow: build a NeuronChat object from a normalized single-cell/spatial transcriptomics matrix with cell-group labels and the neural-specific NeuronChatDB, run the permutation-based inference (M=100), then aggregate the per-interaction networks over all interaction pairs into a single group-by-group communication-strength matrix.

## When to use

Use when starting from a normalized scRNA-seq/snRNA-seq or spatial transcriptomics count matrix with cell-type annotations and you need inferred neural-specific ligand-receptor communication networks aggregated by cell group (human DB='human', mouse DB='mouse').

## When NOT to use

- Input is raw unnormalized counts or lacks cell-group labels, so createNeuronChat(..., group.by=...) cannot be populated
- Goal is small-molecule neurotransmitter inference via CellChat/CellPhoneDB without functional gene grouping — the paper shows this dramatically underestimates ligand abundance for glutamate/GABA compared to NeuronChat's grouped AND/OR logic
- Input is protein/metabolite-level data: NeuronChat estimates ligand/target abundance from transcriptomics, which may be inconsistent with protein or metabolite levels

## Inputs

- Normalized count matrix (scRNA-seq, snRNA-seq, or spatial transcriptomics), scaled to 10,000 + pseudocount 1, log-transformed
- Cell-type / cell-group annotation vector (group.by)
- NeuronChatDB interaction database selected by species ('human' or 'mouse'; 190 human and 183 mouse entries)

## Outputs

- NeuronChat object (x) with slot x@net holding per-interaction communication networks
- Aggregated group-by-group communication matrix via net_aggregation(x@net, method='weight')
- Per-interaction significance (BH-corrected permutation p-values, M=100) with non-significant links zeroed

## How to apply

Preprocess counts by scaling to 10,000 per cell, adding pseudocount 1 and natural-log transforming; note that genes expressed in fewer than 25% of cells are filtered out. Create the object with createNeuronChat(normalized_count_mtx, DB='human' for human data or 'mouse' for mouse data, group.by=cell_group_vector), then infer networks with run_NeuronChat(x, M=100), which runs 100 permutations of cell-group labels for significance testing and corrects p-values by Benjamini–Hochberg; links with p >= 0.05 have their communication strength set to zero. Aggregation over interaction pairs is done with net_aggregation(x@net, method='weight'); NeuronChat computes ensemble-average expression per group using Tukey's trimean, combines functional gene groups with AND logic (geometric mean) and redundant genes with OR logic (arithmetic mean), and scores communication strength as the product of ligand and target abundance. For multi-condition comparison, aggregate each object separately and merge with mergeNeuronChat(add.names=...) before downstream ranking or circle-plot visualization.

## Related tools

- **NeuronChat** (Primary R package: createNeuronChat builds the object, run_NeuronChat infers permutation-tested communication networks, net_aggregation collapses them over interaction pairs) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency of NeuronChat (install via devtools::install_github('sqjin/CellChat')); the same NeuronChat computational workflow is also used to implement CellChat for comparison) — https://github.com/sqjin/CellChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx,DB='human',group.by=cell_group_vector); x <- run_NeuronChat(x,M=100); net_aggregated_x <- net_aggregation(x@net,method='weight')
```

## Evaluation signals

- x@net contains one network per NeuronChatDB interaction pair; each is a group-by-group matrix sized by the number of unique cell groups in group.by
- All communication strengths in the aggregated network are non-negative and every link with BH-corrected permutation p >= 0.05 (M=100) is exactly zero
- net_aggregation(..., method='weight') returns a single square group-by-group matrix whose entries are sums of significant per-interaction weights
- Normalized input is in log scale after 10,000-per-cell scaling + pseudocount 1; genes expressed in <25% of cells are excluded from the inference
- Human data must use DB='human' (190 curated entries) and mouse data DB='mouse' (183 entries) — mismatching species silently changes the interaction set

## Limitations

- Genes expressed in less than 25% of cells are filtered out, which can drop low-abundance but biologically relevant ligands/receptors
- Ligand/target abundance is inferred from transcriptomics and may be inconsistent with protein or metabolite levels
- NeuronChatDB may contain missing curated interaction entries (e.g., subunit stoichiometries of heteromeric receptors are not recorded), leading to bias in inference
- Permutation-based p-value calculation has an accuracy/computation-speed trade-off with time complexity O(n) in the number of permutations

## Evidence

- [methods] Create NeuronChat object; species-appropriate DB: "# creat NeuronChat object; choose the database 'mouse' for mouse data; 'human' for human data"
- [methods] Run inference and aggregate: "x <- run_NeuronChat(x,M=100)"
- [methods] Aggregation step over interaction pairs: "net_aggregated_x <- net_aggregation(x@net,method = 'weight')"
- [discussion] Significance filter applied by the workflow: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [readme] Exact call sequence from the package README: [evidence span withheld — anchor status: external_doc]
