---
name: celltype-identity-parcellation-for-network-nodes
description: Use when building a ligand–receptor edgelist from a Seurat 3.0 scRNAseq
  object (e.g., panc8) and the nodes must correspond to cell types rather than individual
  cells or clusters.
when_to_use_negative:
- The input has no cell-type annotations — without a celltype identity slot there
  are no biological nodes to parcellate, and Connectome cannot define source/target
  pairs
- Node-level analysis at single-cell or subcluster resolution is desired — this skill
  deliberately aggregates cells into cell-type nodes via the active identity slot
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Sets the active identity slot (cell type labels) that CreateConnectome uses
    to define network nodes; NormalizeData prepares the expression values
- name: Connectome
  role: CreateConnectome consumes the ident-labeled Seurat object and emits the celltype-level
    edgelist with weight_norm attributes
  repo: https://github.com/msraredon/Connectome
- name: SeuratData
  role: Distributes the panc8 pancreas dataset used as the ident-labeled input object
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:c69e91900ebb729064328e402150b5d7c6edb943e1ae3bfaa96581a10a4fde80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# celltype-identity-parcellation-for-network-nodes

## Summary

Defines the nodes of a signaling network by setting Seurat's active identity slot to a celltype metadata column before computing a connectome, so that CreateConnectome emits a directed celltype-to-celltype edgelist (source type producing ligand, target type presenting receptor) for the panc8 pancreas dataset.

## When to use

Use when building a ligand–receptor edgelist from a Seurat 3.0 scRNAseq object (e.g., panc8) and the nodes must correspond to cell types rather than individual cells or clusters.

## When NOT to use

- The input has no cell-type annotations — without a celltype identity slot there are no biological nodes to parcellate, and Connectome cannot define source/target pairs
- Node-level analysis at single-cell or subcluster resolution is desired — this skill deliberately aggregates cells into cell-type nodes via the active identity slot

## Inputs

- Seurat 3.0 object (e.g., SeuratData panc8 human pancreas dataset) with a celltype metadata column
- Cell type labels to set as active ident (Ident object / metadata column such as 'celltype')
- Ligand–receptor reference database (Connectome's ncomms8866_human / FANTOM5 default, or a user-provided list)

## Outputs

- Connectome edgelist with celltype-to-celltype source/target node pairs and edge attributes including weight_norm (w1) per ligand–receptor mechanism
- Optional weight_norm_edges.csv table of product-of-normalized-expression edge weights for validation against CreateConnectome output

## How to apply

Load the SeuratData panc8 pancreas object and set its active identity slot to the 'celltype' metadata column so every cell carries a source/target label for network analysis. Normalize expression with Seurat's NormalizeData, then restrict to ligand and receptor genes mapped from the ncomms8866_human reference (union of Ligand.ApprovedSymbol and Receptor.ApprovedSymbol found in rownames). Apply the CreateConnectome cluster-size filter min.cells.per.ident = 75 so only cell types with more than 75 captured cells become nodes. Run CreateConnectome(species = 'human', p.values = F, calculate.DOR = F); the function 'uses the active identity slot to define nodes for network analysis' and returns an edgelist whose source and target columns are cell types. The weightnorm (w1) edgeweight on each edge is the product of the celltype-wise normalized ligand expression in the sending type and the celltype-wise normalized receptor expression in the receiving type.

## Related tools

- **Seurat** (Sets the active identity slot (cell type labels) that CreateConnectome uses to define network nodes; NormalizeData prepares the expression values)
- **Connectome** (CreateConnectome consumes the ident-labeled Seurat object and emits the celltype-level edgelist with weight_norm attributes) — https://github.com/msraredon/Connectome
- **SeuratData** (Distributes the panc8 pancreas dataset used as the ident-labeled input object)

## Examples

```
panc8 <- SetIdent(panc8, value = 'celltype'); panc8 <- NormalizeData(panc8); panc8.con <- CreateConnectome(panc8, species = 'human', min.cells.per.ident = 75, p.values = F, calculate.DOR = F)
```

## Evaluation signals

- Every edge in the output edgelist has source and target values drawn from the celltype identity levels (no unassigned or per-cell node identifiers)
- All retained cell-type nodes have more than 75 captured cells (min.cells.per.ident = 75)
- weight_norm in a hand-computed table (normalized ligand expression in source type × normalized receptor expression in target type) numerically matches CreateConnectome's weight_norm for all shared edges
- The active ident slot of the input object resolves to the 'celltype' column before CreateConnectome is called
- Running CreateConnectome with p.values = F and calculate.DOR = F yields the same edge set as the manual celltype-wise computation

## Limitations

- Output quality is bounded by the ground-truth ligand–receptor database used for mapping (FANTOM5/ncomms8866 by default)
- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication
- Suspension-based scRNAseq does not preserve spatial information, so proximity of inferred celltype-to-celltype interactions cannot be verified

## Evidence

- [readme] CreateConnectome takes as input a Seurat 3.0 object and uses the active identity slot to define nodes for network analysis.: [evidence span withheld — anchor status: not_found]
- [methods] CreateConnectome call with ident filter: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] minimum cells per identity filter rationale: "We also limit our connectomic analysis to only those clusters with > 75 cells captured"
- [full_text] w1 edgeweight definition from source card workflow: [evidence span withheld — anchor status: not_found]
- [methods] reference database gene mapping: "connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol,Connectome::ncomms8866_human$Receptor.ApprovedSymbol)"
