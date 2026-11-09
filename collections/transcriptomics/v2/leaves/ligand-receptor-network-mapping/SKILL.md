---
name: ligand-receptor-network-mapping
description: Use when you have a heterogeneous single-cell RNA-seq dataset with cell-type
  annotations and want to infer cell–cell communication as ligand–receptor edges (e.g.,
  VEGF, PDGF, DLL3-NOTCH signaling) between cell identities.
when_to_use_negative:
- Strongly paired ligand and receptor expression is not direct evidence of cell–cell
  communication; do not treat mapped edges as confirmed signaling without follow-up
  validation
- Suspension-based single-cell sequencing does not preserve spatial histologic information,
  so the method cannot verify spatial proximity of candidate interactions
- Output quality is bounded by the ground-truth ligand–receptor database; do not use
  for mechanisms absent from the reference list unless a user-provided database is
  supplied
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: Connectome
  role: 'R package performing the ligand–receptor mapping: CreateConnectome, FilterConnectome,
    Centrality, NetworkPlot, CircosPlot, SignalScatter'
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Provides the single-cell object, active identity slot, NormalizeData and ScaleData
    pre-processing that CreateConnectome consumes
- name: SeuratData
  role: Distributes the panc8 and ifnb example datasets used to demonstrate the connectome
    workflow
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:00bce380ce50138bf165113ff8de3cb7b1d4c8824402791f8765432a4fa0676b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  - cowplot
  - ggplot2
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 27
---

# ligand-receptor-network-mapping

## Summary

Maps ligand–receptor signaling edges between cell types in a Seurat single-cell RNA-seq object using the Connectome R package against the FANTOM5-derived ncomms8866 database (or a user-supplied mechanism list), producing a weighted, directional edgelist of cell-type-to-cell-type signaling.

## When to use

Use when you have a heterogeneous single-cell RNA-seq dataset with cell-type annotations and want to infer cell–cell communication as ligand–receptor edges (e.g., VEGF, PDGF, DLL3-NOTCH signaling) between cell identities.

## When NOT to use

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication; do not treat mapped edges as confirmed signaling without follow-up validation
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so the method cannot verify spatial proximity of candidate interactions
- Output quality is bounded by the ground-truth ligand–receptor database; do not use for mechanisms absent from the reference list unless a user-provided database is supplied

## Inputs

- Seurat 3.0 object with active identity slot defining cell types (e.g., celltype or cell_annotation)
- Ligand–receptor reference database: Connectome built-in ncomms8866_human / ncomms8866_mouse (FANTOM5-derived) or a user-provided ligand–receptor pair list
- Dataset examples from the paper: panc8 pancreas, ifnb PBMCs (SeuratData), GSE143437 DeMicheli mouse muscle atlas

## Outputs

- Connectome object: edgelist of cell-type-to-celltype edges per ligand–receptor mechanism with edge attributes (ligand/receptor z-scores, min.pct, weight_norm, weight_sc)
- Filtered connectome edgelist after FilterConnectome (min.pct, min.z thresholds)

## How to apply

Build a Seurat 3.0 object with cell_annotation set as active identities, run NormalizeData, then restrict to the union of Ligand.ApprovedSymbol and Receptor.ApprovedSymbol genes mapped from Connectome::ncomms8866_human (or ncomms8866_mouse for mouse data) and scale those genes with ScaleData. Call CreateConnectome with species='human' or 'mouse' (FANTOM5/Ramilowski 2015 ncomms8866 mechanisms by default), optionally setting min.cells.per.ident (e.g., 75) to drop sparse clusters and p.values=FALSE for speed; the active identity slot defines the network nodes. The output edgelist carries multiple edge attributes (ligand/receptor z-scores, percent expression, weight_norm, weight_sc) which are then reduced with FilterConnectome using thresholds such as min.pct=0.1, min.z=0.25, remove.na=TRUE. The ground-truth database can be customized to remove low-relevance connections or add newly discovered mechanisms. Downstream Centrality, NetworkPlot, or CircosPlot functions consume this edgelist.

## Related tools

- **Connectome** (R package performing the ligand–receptor mapping: CreateConnectome, FilterConnectome, Centrality, NetworkPlot, CircosPlot, SignalScatter) — https://github.com/msraredon/Connectome
- **Seurat** (Provides the single-cell object, active identity slot, NormalizeData and ScaleData pre-processing that CreateConnectome consumes)
- **SeuratData** (Distributes the panc8 and ifnb example datasets used to demonstrate the connectome workflow)

## Examples

```
panc8.con <- CreateConnectome(panc8, species = 'human', min.cells.per.ident = 75, p.values = F, calculate.DOR = F); panc8.con2 <- FilterConnectome(panc8.con, min.pct = 0.1, min.z = 0.25, remove.na = T)
```

## Evaluation signals

- CreateConnectome completes without error on a Seurat object with species-specific mapping (species='human' or 'mouse') and the mapped connectome gene set present in the scaled features
- Filtered edgelist only contains edges whose ligand and receptor z-scores are above the min.z threshold (e.g., 0.25) and min.pct threshold (e.g., 0.1) with no NA values retained
- Centrality rankings qualitatively match known tissue topology, e.g., endothelial cells rank as dominant receivers of VEGF- and ANGPT-family signals in the pancreas connectome
- Longitudinal validation: Vegfa centrality shows FAPs as dominant source at homeostatic timepoints and recruited monocytes/macrophages taking over by Day 2/5 in the DeMicheli muscle wound-healing dataset

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so candidate interactions cannot be checked for spatial proximity
- Outputs are only as good as the ground-truth database against which the original mapping takes place
- Any robust claim of signal transduction requires extensive wet-lab experimentation; computational prediction of intracellular transduction is confounded by intracellular signaling complexity

## Evidence

- [intro] Connectome is a multi-purpose tool designed to create ligand–receptor mappings in single-cell data, to identify non-random patterns representing signal, and to provide biologically-informative visualization of those patterns.: "Connectome is a multi-purpose tool designed to create ligand–receptor mappings in single-cell data, to identify non-random patterns representing signal, and to provide biologically-informative"
- [methods] CreateConnectome call on pancreas data with species, min.cells.per.ident and p.values parameters: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] FilterConnectome thresholds on percent expression and z-score: "panc8.con2 <- FilterConnectome(panc8.con,min.pct = 0.1,min.z = 0.25,remove.na = T)"
- [intro] Default FANTOM5 ligand–receptor database: "By default, Connectome uses the FANTOM5 database of ligand–receptor interactions13, formalizing the workflow first presented in1, but it also allows mapping against any user-provided ligand–receptor"
- [methods] Centrality function description: "Built into `Connectome` is a function called `Centrality`, which performs a graph-theory based analysis of the input network."
- [readme] FilterConnectome purpose per README: [evidence span withheld — anchor status: external_doc]
- [readme] CreateConnectome node definition per README: [evidence span withheld — anchor status: external_doc]
