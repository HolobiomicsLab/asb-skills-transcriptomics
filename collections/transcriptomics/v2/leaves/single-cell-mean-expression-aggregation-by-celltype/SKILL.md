---
name: single-cell-mean-expression-aggregation-by-celltype
description: Use when you must reproduce or de-novo compute the weight_norm edgeweight
  of every Connectome edge from per-celltype mean normalized expression, or when validating
  ligand×receptor product scoring on scRNAseq data.
when_to_use_negative:
- The analysis requires specificity (z-score) or percent-expressed (min.pct) edge
  scoring rather than the default co-expression magnitude weight — use FilterConnectome
  attributes instead
- The object has not been normalized (raw counts) — mean products of raw counts are
  not comparable to CreateConnectome's weight_norm
- Differential connectomics between two conditions is the goal — use DifferentialConnectome
  on two matching connectomes instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0085
tools:
- name: Seurat
  role: Normalizes the single-cell data (NormalizeData) and provides celltype identity
    slots used to group cells for mean expression aggregation
- name: SeuratData
  role: Distributes the panc8 pancreas dataset and ncomms8866 reference used as demonstration
    input
- name: Connectome
  role: Supplies the ncomms8866 ligand–receptor reference list and CreateConnectome,
    whose weight_norm output serves as the validation target
  repo: https://github.com/msraredon/Connectome
- name: R
  role: Implementation language for the aggregation and product computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:25dc752a3e1e47a71f9a5e44b5e3706978b9b16822454e808a74485b7b2a56ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-mean-expression-aggregation-by-celltype

## Summary

Computes celltype-wise normalized mean expression of ligand and receptor genes from a Seurat object so that every ligand–receptor edge in a Connectome edgelist can be weighted as the product of ligand mean (source celltype) × receptor mean (target celltype) — the default 'weightnorm' (w1) edgeweight.

## When to use

Use when you must reproduce or de-novo compute the weight_norm edgeweight of every Connectome edge from per-celltype mean normalized expression, or when validating ligand×receptor product scoring on scRNAseq data.

## When NOT to use

- The analysis requires specificity (z-score) or percent-expressed (min.pct) edge scoring rather than the default co-expression magnitude weight — use FilterConnectome attributes instead
- The object has not been normalized (raw counts) — mean products of raw counts are not comparable to CreateConnectome's weight_norm
- Differential connectomics between two conditions is the goal — use DifferentialConnectome on two matching connectomes instead

## Inputs

- Seurat 3.0 object with normalized expression (NormalizeData applied) and active identity slot set to a celltype metadata column
- Ligand–receptor reference table (e.g. Connectome::ncomms8866_human with Ligand.ApprovedSymbol and Receptor.ApprovedSymbol columns)

## Outputs

- Matrix/table of mean normalized ligand expression per source celltype
- Matrix/table of mean normalized receptor expression per target celltype
- Edgelist of ligand–receptor × source–target celltype pairs with product-based weight_norm values (e.g. weight_norm_edges.csv)

## How to apply

Load the Seurat object (e.g. panc8 from SeuratData), set the active identity slot to the celltype metadata column, and run Seurat's NormalizeData. Restrict to the ligand–receptor gene set mapped from the Connectome reference (union of Ligand.ApprovedSymbol and Receptor.ApprovedSymbol from ncomms8866_human, present in rownames), and apply the same cluster-size filter as CreateConnectome (min.cells.per.ident = 75) so only celltypes with >75 captured cells are included. For each ligand gene, compute the mean normalized expression within each source (sending) celltype; for each receptor gene, compute the mean normalized expression within each target (receiving) celltype. For every ligand–receptor pair in the reference database and every source–target celltype combination, form the edge weight as the product of these two means. The rationale is that weightnorm encodes signal strength as co-expression magnitude of the mechanism across the sending and receiving nodes, independent of z-score specificity.

## Related tools

- **Seurat** (Normalizes the single-cell data (NormalizeData) and provides celltype identity slots used to group cells for mean expression aggregation)
- **SeuratData** (Distributes the panc8 pancreas dataset and ncomms8866 reference used as demonstration input)
- **Connectome** (Supplies the ncomms8866 ligand–receptor reference list and CreateConnectome, whose weight_norm output serves as the validation target) — https://github.com/msraredon/Connectome
- **R** (Implementation language for the aggregation and product computation)

## Examples

```
panc8 <- NormalizeData(panc8); panc8.con <- CreateConnectome(panc8, species = 'human', min.cells.per.ident = 75, p.values = F, calculate.DOR = F)
```

## Evaluation signals

- Every shared edge in weight_norm_edges.csv numerically matches the weight_norm column emitted by CreateConnectome (species = 'human', p.values = F, calculate.DOR = F, min.cells.per.ident = 75)
- Only celltypes with more than 75 cells are present as source/target nodes in the aggregated table
- Ligand and receptor genes in the aggregation are exactly the intersection of the ncomms8866_human union list with the object rownames
- Each edge weight equals mean_ligand(source_celltype) × mean_receptor(target_celltype) to floating-point tolerance

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication
- Output quality is bounded by the ground-truth database against which ligand–receptor mapping is performed
- Clusters smaller than the min.cells.per.ident threshold are excluded from aggregation

## Evidence

- [methods] weightnorm definition via product of normalized ligand/receptor means: "Here, we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes"
- [methods] CreateConnectome parameters and cluster filter: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] Reference gene list construction: "connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol,Connectome::ncomms8866_human$Receptor.ApprovedSymbol)"
- [readme] CreateConnectome edgelist with many edge attributes: [evidence span withheld — anchor status: external_doc]
- [discussion] Biologic interpretation caveat: "strongly paired ligand and receptor expression is not direct evidence of cell–cell communication"
