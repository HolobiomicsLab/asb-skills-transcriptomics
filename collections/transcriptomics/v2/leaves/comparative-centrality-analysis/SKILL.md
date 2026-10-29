---
name: comparative-centrality-analysis
description: Use when signaling networks from two or more conditions or timepoints
  (e.g., Seurat objects split by an 'injury' variable into Days 0/2/5/7) must be compared
  to identify which cell types dominate source vs target roles for a signaling family
  or a single ligand–receptor mechanism.
when_to_use_negative:
- Only one tissue system/condition is available — use the single-network Centrality
  function instead of CompareCentrality, which requires a list of connectomes
- The goal is a direct quantitative edge-by-edge difference between exactly two connectomes
  requiring identical edge sets — DifferentialConnectome is the appropriate function,
  not centrality comparison
- The input lacks ligand–receptor mapping (e.g., bulk RNA-seq or an already-aggregated
  feature table) — CreateConnectome requires a Seurat object with per-cell-type expression
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: Connectome
  role: Provides CompareCentrality (and the single-network Centrality) to compute
    and compare sending- and receiving-centrality across a list of connectomes, with
    FilterConnectome-style filtration on weight.attribute, modes.include, features,
    and mechanisms.include
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Holds the per-timepoint scRNA-seq objects; SplitObject by condition variable
    and NormalizeData/ScaleData on mapped connectome genes before CreateConnectome
    builds each network in the list
- name: ggplot2
  role: Used with cowplot to plot longitudinal ligand (e.g., LOI = 'Vegfa') and receptor
    (e.g., ROI = 'Kdr') trends that corroborate the centrality ranking shifts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:188d68c3d827fdb98efcafad7059cd8ea944fe582dad1f30ccbe675e7996698e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - cowplot
  - ggplot2
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# comparative-centrality-analysis

## Summary

Graph-theoretic centrality comparison across a list of connectomes (e.g., timepoint-specific ligand–receptor networks from Seurat objects) to rank which cell types dominate producing versus receiving a given signaling mode, such as VEGF/Vegfa-mediated signaling during longitudinal tissue remodeling.

## When to use

Use when signaling networks from two or more conditions or timepoints (e.g., Seurat objects split by an 'injury' variable into Days 0/2/5/7) must be compared to identify which cell types dominate source vs target roles for a signaling family or a single ligand–receptor mechanism.

## When NOT to use

- Only one tissue system/condition is available — use the single-network Centrality function instead of CompareCentrality, which requires a list of connectomes
- The goal is a direct quantitative edge-by-edge difference between exactly two connectomes requiring identical edge sets — DifferentialConnectome is the appropriate function, not centrality comparison
- The input lacks ligand–receptor mapping (e.g., bulk RNA-seq or an already-aggregated feature table) — CreateConnectome requires a Seurat object with per-cell-type expression

## Inputs

- Seurat 3.0 objects (one per condition/timepoint) with cell-type annotations as active identities
- List of connectomes produced by CreateConnectome (edgelists with weight_norm and other edge attributes)
- Ligand–receptor reference database (FANTOM5-derived ncomms8866_mouse / ncomms8866_human or a user-provided list)

## Outputs

- Per-timepoint centrality rankings of cell types by cumulative incoming/outgoing edgeweight and Kleinberg directional centrality for the selected signaling mode
- Side-by-side comparative centrality table/plots (e.g., via ggplot2/cowplot) showing source and receiver dominance shifts across conditions
- Longitudinal ligand/receiver trend calls (e.g., producer handover from FAPs to monocytes/macrophages with endothelial reception)

## How to apply

Build a Seurat object with cell-type annotations as active identities, map ligand/receptor genes from the FANTOM5-derived database (e.g., ncomms8866_mouse or ncomms8866_human via union of Ligand.ApprovedSymbol and Receptor.ApprovedSymbol), and SplitObject by the condition/timepoint variable. For each subset run NormalizeData and ScaleData (restricted to the mapped connectome genes), then CreateConnectome (e.g., species = 'mouse', p.values = F) to yield one connectome per condition. Pass the connectome list to CompareCentrality with weight.attribute = 'weight_norm' and modes.include set to the signaling family of interest (e.g., 'VEGF'); to isolate a single mechanism, additionally restrict with features = 'Vegfa' and mechanisms.include = 'Vegfa - Flt1'. The function ranks cumulative incoming/outgoing edgeweight and Kleinberg directional centrality per cell type in each sub-network, side-by-side, so a longitudinal handover (e.g., FAPs producing Vegfa at homeostatic timepoints, recruited monocytes/macrophages taking over after injury, with endothelial cells as dominant receivers) can be read directly from the ranked output.

## Related tools

- **Connectome** (Provides CompareCentrality (and the single-network Centrality) to compute and compare sending- and receiving-centrality across a list of connectomes, with FilterConnectome-style filtration on weight.attribute, modes.include, features, and mechanisms.include) — https://github.com/msraredon/Connectome
- **Seurat** (Holds the per-timepoint scRNA-seq objects; SplitObject by condition variable and NormalizeData/ScaleData on mapped connectome genes before CreateConnectome builds each network in the list)
- **ggplot2** (Used with cowplot to plot longitudinal ligand (e.g., LOI = 'Vegfa') and receptor (e.g., ROI = 'Kdr') trends that corroborate the centrality ranking shifts)

## Examples

```
CompareCentrality(connectome.list, weight.attribute = 'weight_norm', modes.include = 'VEGF')
```

## Evaluation signals

- Output table shows FAP (fibro/adipogenic progenitors) as the dominant Vegfa source at homeostatic timepoints (Day 0/7) and monocyte/macrophage as dominant source at Day 2/5 post-injury
- Endothelial cells are identified as the dominant Vegfa receivers (e.g., via Kdr/Flt1 reception) across all timepoints, consistent with the reported longitudinal pattern
- Rankings are computed on weight_norm edgeweights and include both cumulative incoming/outgoing weight and Kleinberg directional centrality for each cell type
- Each connectome in the list was built from the same mapped ligand/receptor gene set (union of ncomms8866 ligand and receptor ApprovedSymbols) so centrality values are comparable across timepoints
- The restricted single-mechanism run (features = 'Vegfa', mechanisms.include = 'Vegfa - Flt1') yields a consistent source–receiver ordering with the family-level (modes.include = 'VEGF') result

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication; robust claims of signal transduction require extensive wet-lab experimentation
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified
- Outputs are only as good as the ground-truth database against which ligand–receptor mapping is performed; computational prediction of intracellular transduction is confounded by intracellular signaling complexity

## Evidence

- [readme] CompareCentrality role: [evidence span withheld — anchor status: external_doc]
- [methods] Centrality graph analysis: "Built into `Connectome` is a function called `Centrality`, which performs a graph-theory based analysis of the input network."
- [intro] FANTOM5 default database: "By default, Connectome uses the FANTOM5 database of ligand–receptor interactions13"
- [discussion] Evidence caveat: "strongly paired ligand and receptor expression is not direct evidence of cell–cell communication"
- [discussion] Database bound: "the outputs of Connectome, like any ligand–receptor mapping software, are only as good as the ground-truth database against which the original mapping takes place"
