---
name: graph-centrality-analysis
description: Use when a Seurat scRNA-seq object has been converted to a filtered Connectome
  edgelist and you need to identify cell types that dominate sending vs. receiving
  positions for specific signaling families (e.g. PDGF, EGF, VEGF) within a single
  tissue.
when_to_use_negative:
- The goal is differential signaling between two conditions/time points rather than
  single-tissue topology — DifferentialConnectome/CompareCentrality are the appropriate
  functions
- Strongly paired ligand–receptor expression is being claimed as direct evidence of
  cell–cell communication or spatially verified interaction; suspension scRNA-seq
  does not preserve spatial histologic information
- The dataset contains cell-type clusters with fewer than 75 cells captured, since
  the min.cells.per.ident=75 threshold will exclude them and distort node definitions
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Connectome
  role: R package providing CreateConnectome, FilterConnectome, and the Centrality
    function that performs the graph-theory based centrality analysis on the connectome
    edgelist
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Provides the Seurat 3.0 object whose active identity slot defines the network
    nodes; NormalizeData and ScaleData prepare ligand/receptor genes before connectome
    construction
- name: SeuratData
  role: Distributes the panc8 cross-platform pancreas dataset used as the single-tissue
    demonstration input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:274ef7a425c600872e0d66353e4cb63864ebcf798ba324cc9c8571fe6842c7c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# graph-centrality-analysis

## Summary

Performs graph-theory based centrality analysis on a single-cell ligand–receptor connectome (built with the Connectome R package) to rank cell types by cumulative incoming/outgoing edgeweight and Kleinberg hub/authority scores, identifying which cell types dominate production or reception of each signaling family.

## When to use

Use when a Seurat scRNA-seq object has been converted to a filtered Connectome edgelist and you need to identify cell types that dominate sending vs. receiving positions for specific signaling families (e.g. PDGF, EGF, VEGF) within a single tissue.

## When NOT to use

- The goal is differential signaling between two conditions/time points rather than single-tissue topology — DifferentialConnectome/CompareCentrality are the appropriate functions
- Strongly paired ligand–receptor expression is being claimed as direct evidence of cell–cell communication or spatially verified interaction; suspension scRNA-seq does not preserve spatial histologic information
- The dataset contains cell-type clusters with fewer than 75 cells captured, since the min.cells.per.ident=75 threshold will exclude them and distort node definitions

## Inputs

- Seurat 3.0 object (e.g. panc8 cross-platform pancreas dataset from SeuratData) with an active cell-type identity slot
- Ligand–receptor ground-truth database (FANTOM5/ncomms8866_human by default, or user-provided paired list)
- Filtration parameters for high-confidence edges (min.pct, min.z, remove.na)

## Outputs

- Per-family, per-cell-type centrality ranking table (e.g. centrality_table.csv) with incoming/outgoing edgeweight and Kleinberg hub/authority scores
- Paired centrality plots identifying cell types dominating production or reception of each signaling mode

## How to apply

First normalize the Seurat object (NormalizeData), subset to ligand/receptor genes from the ncomms8866_human database, and scale them with ScaleData. Build the connectome with CreateConnectome(species='human', min.cells.per.ident=75, p.values=FALSE, calculate.DOR=FALSE), which uses the active identity slot (here the 'celltype' annotation of the panc8 dataset) to define network nodes. Reduce the edgelist to high-confidence edges via FilterConnectome(min.pct=0.1, min.z=0.25, remove.na=TRUE), keeping only edges with ligand and receptor z-score above 0.25. Then run Centrality grouped by signaling family ('mode') with weight.attribute='weight_sc'; this subdivides the network into per-family sub-networks and ranks each cell type by cumulative incoming/outgoing edgeweight plus Kleinberg hub and authority scores. The rationale is that connectomic networks are directional, multi-modal, and weighted, so per-family centrality reveals receiver (authority) and sender (hub) roles, e.g. endothelial cells as dominant VEGF/ANGPT receivers in the human pancreas.

## Related tools

- **Connectome** (R package providing CreateConnectome, FilterConnectome, and the Centrality function that performs the graph-theory based centrality analysis on the connectome edgelist) — https://github.com/msraredon/Connectome
- **Seurat** (Provides the Seurat 3.0 object whose active identity slot defines the network nodes; NormalizeData and ScaleData prepare ligand/receptor genes before connectome construction)
- **SeuratData** (Distributes the panc8 cross-platform pancreas dataset used as the single-tissue demonstration input)

## Examples

```
panc8.con2 <- FilterConnectome(panc8.con, min.pct = 0.1, min.z = 0.25, remove.na = T); Centrality(panc8.con2, mode = 'angiopoietin', weight.attribute = 'weight_sc')
```

## Evaluation signals

- The script runs to completion and writes centrality_table.csv, which parses as a valid table of per-family, per-cell-type centrality ranks
- Each signaling family's sub-network yields ranked incoming (authority/receiving) and outgoing (hub/sending) centrality scores for every retained cell type
- Rankings qualitatively reproduce the reported pancreas topology: mesenchymal cells top receivers of PDGF, epithelial cells of EGF, immune cells of CSF/chemokine, and endothelial cells of VEGF/ANGPT signals
- All edges entering Centrality have passed FilterConnectome thresholds (ligand and receptor z-score > 0.25, min.pct = 0.1) and no NA values remain (remove.na = TRUE)
- Clusters with fewer than 75 cells are absent from the node set, consistent with min.cells.per.ident = 75

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication; robust claims of transduction require extensive wet-lab experimentation
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified
- Output quality is only as good as the ground-truth ligand–receptor database (FANTOM5/ncomms8866) against which the original mapping takes place
- Computational prediction of intracellular transduction is fundamentally confounded by the complexity of intracellular signaling networks

## Evidence

- [methods] Centrality is a graph-theory analysis function: "Built into `Connectome` is a function called `Centrality`, which performs a graph-theory based analysis of the input network."
- [methods] CreateConnectome parameterization on panc8: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] FilterConnectome thresholds for high-confidence edges: "panc8.con2 <- FilterConnectome(panc8.con,min.pct = 0.1,min.z = 0.25,remove.na = T)"
- [readme] Centrality identifies dominating sender/receiver cell types: [evidence span withheld — anchor status: external_doc]
- [intro] Connectomic networks are directional, multi-modal, weighted: [evidence span withheld — anchor status: not_found]
