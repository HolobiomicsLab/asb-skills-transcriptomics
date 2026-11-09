---
name: normalized-expression-slot-edge-weighting
description: Use when reproducing or auditing Connectome-style ligand–receptor edgelists
  where each edge must carry a normalized-expression-derived weightnorm value computed
  from a Seurat object's active identity slot.
when_to_use_negative:
- When the goal is comparing two tissue systems rather than computing within-condition
  edge weights — DifferentialConnectome requires exactly the same edges to be present
  in both connectomes and is a separate step
- When only biologically/statistically interesting edges are needed downstream — FilterConnectome
  (e.g. min.pct = 0.1, min.z = 0.25) operates on the already-computed edgelist and
  is not part of weight calculation itself
- When spatial validation of interactions is required — the analysis does not preserve
  spatial histologic information, so weight_norm cannot confirm actual cell–cell communication
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3366
tools:
- name: Connectome
  role: Provides CreateConnectome, which computes the weight_norm edgeweight for every
    edge from celltype-wise normalized expression; supplies the ncomms8866_human ligand–receptor
    reference
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Hosts the single-cell object; its NormalizeData output and active identity
    slot define the celltype-wise normalized expression used for edge weighting
- name: SeuratData
  role: Distributes the panc8 pancreas dataset used to reproduce the weightnorm computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:054d1bca4bd13063f492ad9389da75bbe75e90f6a5bcf06dfe289395a53470ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# normalized-expression-slot-edge-weighting

## Summary

Computes the Connectome default 'weight_norm' edgeweight for every ligand–receptor edge as the product of the source cell type's mean normalized ligand expression and the target cell type's mean normalized receptor expression, reproducible from a Seurat object via CreateConnectome.

## When to use

Use when reproducing or auditing Connectome-style ligand–receptor edgelists where each edge must carry a normalized-expression-derived weightnorm value computed from a Seurat object's active identity slot.

## When NOT to use

- When the goal is comparing two tissue systems rather than computing within-condition edge weights — DifferentialConnectome requires exactly the same edges to be present in both connectomes and is a separate step
- When only biologically/statistically interesting edges are needed downstream — FilterConnectome (e.g. min.pct = 0.1, min.z = 0.25) operates on the already-computed edgelist and is not part of weight calculation itself
- When spatial validation of interactions is required — the analysis does not preserve spatial histologic information, so weight_norm cannot confirm actual cell–cell communication

## Inputs

- Seurat 3.0 object (e.g. panc8 pancreas dataset from SeuratData) with active ident set to a celltype metadata column and normalized data slot populated
- Ligand–receptor reference table (Connectome::ncomms8866_human, FANTOM5/Ramilowski 2015), used to subset connectome.genes

## Outputs

- Connectome edgelist (one row per ligand–receptor mechanism per source–target celltype pair) carrying a weight_norm edge attribute equal to normalized_ligand(source) × normalized_receptor(target)
- Optionally weight_norm_edges.csv for direct numerical comparison against CreateConnectome output

## How to apply

Start from a Seurat 3.0 object (e.g. panc8 from SeuratData) with the active identity slot set to a cell-type metadata column, and run Seurat's NormalizeData followed by ScaleData on the ligand/receptor genes mapped from the reference (union of ncomms8866_human Ligand.ApprovedSymbol and Receptor.ApprovedSymbol present in the rownames). Compute the mean normalized expression of each ligand gene within each source cell type and of each receptor gene within each target cell type. For every ligand–receptor pair in the reference database and every source–target celltype combination, define the edge weight as the product of the normalized ligand expression (sender) and normalized receptor expression (receiver); this is the weight_norm attribute Connectome emits by default. Apply the cluster-size floor via min.cells.per.ident = 75 so only clusters with > 75 captured cells contribute edges, and call CreateConnectome with species = 'human', p.values = F, calculate.DOR = F. The rationale is that pairing sender-side ligand abundance with receiver-side receptor abundance gives a direction-aware, celltype-resolved signaling strength without relying on p-values or differential-over-receptor (DOR) scoring.

## Related tools

- **Connectome** (Provides CreateConnectome, which computes the weight_norm edgeweight for every edge from celltype-wise normalized expression; supplies the ncomms8866_human ligand–receptor reference) — https://github.com/msraredon/Connectome
- **Seurat** (Hosts the single-cell object; its NormalizeData output and active identity slot define the celltype-wise normalized expression used for edge weighting)
- **SeuratData** (Distributes the panc8 pancreas dataset used to reproduce the weightnorm computation)

## Examples

```
panc8.con <- CreateConnectome(panc8, species = 'human', min.cells.per.ident = 75, p.values = F, calculate.DOR = F)
```

## Evaluation signals

- weight_norm values in the recomputed table must numerically match CreateConnectome's weight_norm for all shared (source, target, ligand, receptor) edges
- Every weight_norm value must equal the product of the sender cell type's mean normalized ligand expression and the receiver cell type's mean normalized receptor expression (multiplicative structure check)
- All edges must come from clusters passing the min.cells.per.ident = 75 filter — clusters with ≤ 75 cells should be absent
- Ligand and receptor genes must be restricted to those mapped from the ncomms8866_human reference (union of Ligand.ApprovedSymbol and Receptor.ApprovedSymbol) present in the data rownames
- All weight_norm values should be non-negative, since they derive from products of normalized expression means

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication; robust claims of transduction require extensive wet-lab experimentation
- Output quality is bounded by the ground-truth ligand–receptor database (FANTOM5/ncomms8866 by default) against which the original mapping takes place
- No Connectome version/changelog context is given in the article, so exact numeric reproducibility may depend on matching package versions

## Evidence

- [methods] CreateConnectome call and parameters: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] Normalization and gene mapping step: "we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes"
- [methods] Cluster-size filter rationale: "We also limit our connectomic analysis to only those clusters with > 75 cells captured"
- [readme] CreateConnectome input contract from README: [evidence span withheld — anchor status: external_doc]
- [discussion] Interpretation limitation: "strongly paired ligand and receptor expression is not direct evidence of cell–cell communication"
