---
name: connectome-output-consistency-verification
description: Use when independently reimplementing a Connectome edge statistic (e.g.
  weight_norm) and needing proof of numerical agreement with CreateConnectome output
  on a shared edge set.
when_to_use_negative:
- When the goal is biological interpretation of the connectome (e.g. filtering with
  min.pct/min.z or visualization) rather than validating the weight_norm arithmetic
- When the input object differs from the one fed to CreateConnectome (different normalization,
  identity slot, or min.cells.per.ident threshold) — edge sets will not match and
  the comparison is invalid
- 'When inferring actual cell–cell communication: the article warns that strongly
  paired ligand and receptor expression is not direct evidence of communication'
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Connectome
  role: Provides CreateConnectome (reference weight_norm edgelist), the ncomms8866_human
    ligand–receptor reference, and the weight_norm edge attribute that the reconstruction
    must reproduce
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Supplies NormalizeData and the active identity slot used to define source/target
    celltype nodes for the identically prepared input object
- name: SeuratData
  role: Distributes the cross-platform panc8 pancreas dataset used as the validation
    input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:741d21b4a56579660d97969861c2b814b5b59e76c6ca4c5e3ca05bfa003ff5a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# connectome-output-consistency-verification

## Summary

A verification skill that reconstructs the Connectome weight_norm (w1) edgeweight — the product of celltype-wise normalized ligand expression in the sender and normalized receptor expression in the receiver — outside the package, then numerically checks it against the weight_norm column emitted by CreateConnectome.

## When to use

Use when independently reimplementing a Connectome edge statistic (e.g. weight_norm) and needing proof of numerical agreement with CreateConnectome output on a shared edge set.

## When NOT to use

- When the goal is biological interpretation of the connectome (e.g. filtering with min.pct/min.z or visualization) rather than validating the weight_norm arithmetic
- When the input object differs from the one fed to CreateConnectome (different normalization, identity slot, or min.cells.per.ident threshold) — edge sets will not match and the comparison is invalid
- When inferring actual cell–cell communication: the article warns that strongly paired ligand and receptor expression is not direct evidence of communication

## Inputs

- Seurat 3.0 object with an active identity slot set to a celltype metadata column (e.g. SeuratData panc8 pancreas data after NormalizeData)
- Ligand–receptor reference table (Connectome::ncomms8866_human with Ligand.ApprovedSymbol and Receptor.ApprovedSymbol columns)
- CreateConnectome edgelist output containing a weight_norm edge attribute

## Outputs

- weight_norm_edges.csv — table of ligand–receptor pair, source celltype, target celltype, and independently computed weight_norm per edge
- A joined comparison of reconstructed vs. CreateConnectome weight_norm values for all shared edges

## How to apply

Prepare the input exactly as the reference run: load the SeuratData panc8 pancreas object, set the active identity slot to the celltype metadata column, and apply Seurat's NormalizeData, because CreateConnectome reads the active identity slot to define network nodes. Restrict genes to the union of Ligand.ApprovedSymbol and Receptor.ApprovedSymbol from Connectome::ncomms8866_human present in the rownames, and apply the same cluster-size filter (min.cells.per.ident = 75) that CreateConnectome uses. Compute mean normalized expression of each ligand per source celltype and each receptor per target celltype, and for every ligand–receptor pair and source–target combination write the product as weight_norm in weight_norm_edges.csv. Run CreateConnectome with species='human', p.values=F, calculate.DOR=F on the identically prepared object, then join the two tables on the edge key (ligand, receptor, source, target) and compare the weight_norm columns element-wise for all shared edges. Disable DOR and p-value calculation during validation to avoid extra columns or stochastic drift; any mismatch on shared edges indicates a divergence in normalization, identity-slot handling, or the celltype-wise averaging step.

## Related tools

- **Connectome** (Provides CreateConnectome (reference weight_norm edgelist), the ncomms8866_human ligand–receptor reference, and the weight_norm edge attribute that the reconstruction must reproduce) — https://github.com/msraredon/Connectome
- **Seurat** (Supplies NormalizeData and the active identity slot used to define source/target celltype nodes for the identically prepared input object)
- **SeuratData** (Distributes the cross-platform panc8 pancreas dataset used as the validation input)

## Examples

```
panc8.con <- CreateConnectome(panc8, species='human', min.cells.per.ident=75, p.values=F, calculate.DOR=F)
```

## Evaluation signals

- The weight_norm column in weight_norm_edges.csv numerically matches the weight_norm values emitted by CreateConnectome for all shared edges
- The reconstructed edge key set (ligand–receptor–source–target) is identical to the CreateConnectome edgelist after applying the same NormalizeData step and min.cells.per.ident = 75 filter
- The reference CreateConnectome call uses species = 'human', p.values = F, calculate.DOR = F, so comparison tables contain no extra DOR or p-value columns
- Both pipelines consume the same Seurat object with the active identity slot set to the celltype metadata column before any averaging

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication — matching weight_norm values validate the arithmetic, not the biology
- Outputs of Connectome are only as good as the ground-truth database (here FANTOM5/ncomms8866) against which the original mapping takes place
- Suspension-based single-cell sequencing does not preserve spatial information, so spatial proximity of candidate interactions cannot be verified even if weights agree

## Evidence

- [methods] CreateConnectome reference call and thresholds: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] Ligand–receptor gene restriction for reconstruction: "connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol,Connectome::ncomms8866_human$Receptor.ApprovedSymbol)"
- [methods] Cluster-size filter matching: "We also limit our connectomic analysis to only those clusters with > 75 cells captured"
- [readme] CreateConnectome inputs and node definition: [evidence span withheld — anchor status: not_found]
- [readme] Edgelist with weight_norm-like edge attributes: [evidence span withheld — anchor status: external_doc]
