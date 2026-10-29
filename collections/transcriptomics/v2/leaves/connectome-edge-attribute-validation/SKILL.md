---
name: connectome-edge-attribute-validation
description: Use when auditing, extending, or re-deriving Connectome edgeweight attributes
  (weight_sc vs weight_norm) on a Seurat scRNA-seq object and you must prove how weight_sc
  is computed from scaled ligand/receptor z-scores.
when_to_use_negative:
- Your goal is downstream signaling analysis (NetworkPlot, Centrality, differential
  connectomics) rather than validating how the edgeweight itself is defined — use
  FilterConnectome-style workflows instead
- Your input is not a Seurat object with identifiable cell-type identities, or you
  lack a user-supplied/FANTOM5 ligand-receptor reference, so CreateConnectome cannot
  generate the reference weight_sc column to validate against
- You expect weightscale to measure absolute raw co-expression strength — it is a
  system z-score-based specificity metric, not a connectivity metric like weight_norm
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Connectome
  role: Provides CreateConnectome (generates the reference edgelist with weight_sc),
    the ncomms8866_human ligand-receptor gene list, and the FilterConnectome/EdgeDotPlot
    downstream context for edgeweight attributes
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Supplies NormalizeData and ScaleData to produce the gene-wise system z-scores
    (ligand.scale/recept.scale) that weightscale is computed from
- name: SeuratData
  role: Distributes the panc8 cross-platform pancreas dataset used as the validation
    input
- name: R
  role: Implementation environment in which the independent weightscale reconstruction
    and edge-by-edge comparison are coded
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:2643800072bad743122ca008b2f41e02ea9f6ec48bf7ecf6509554705b710b43
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# connectome-edge-attribute-validation

## Summary

Independently reconstructs Connectome's weightscale (weight_sc) edgeweight — the mean of gene-wise system z-scores (ligand.scale and recept.scale) of the ligand and receptor — and verifies it edge-by-edge against CreateConnectome output, confirming it encodes signal specificity rather than raw connectivity (weight_norm).

## When to use

Use when auditing, extending, or re-deriving Connectome edgeweight attributes (weight_sc vs weight_norm) on a Seurat scRNA-seq object and you must prove how weight_sc is computed from scaled ligand/receptor z-scores.

## When NOT to use

- Your goal is downstream signaling analysis (NetworkPlot, Centrality, differential connectomics) rather than validating how the edgeweight itself is defined — use FilterConnectome-style workflows instead
- Your input is not a Seurat object with identifiable cell-type identities, or you lack a user-supplied/FANTOM5 ligand-receptor reference, so CreateConnectome cannot generate the reference weight_sc column to validate against
- You expect weightscale to measure absolute raw co-expression strength — it is a system z-score-based specificity metric, not a connectivity metric like weight_norm

## Inputs

- Seurat 3.0 object (e.g. panc8 cross-platform pancreas data from SeuratData) with 'celltype' identities
- Ligand-receptor reference list: Connectome::ncomms8866_human (FANTOM5/Ramilowski 2015 pairs)
- Normalized and gene-scaled expression slot (z-scores for ligand and receptor genes)

## Outputs

- Connectome edgelist object with per-edge attributes including ligand.scale, recept.scale, weight_sc (weightscale) and weight_norm
- Independently computed weightscale column matched edge-by-edge to CreateConnectome's weight_sc

## How to apply

Load the cross-platform pancreas dataset (panc8) via SeuratData and set cell identities to the 'celltype' metadata column, then run Seurat's NormalizeData. Take the union of ligand and receptor genes from Connectome::ncomms8866_human that map to the dataset rownames and run ScaleData restricted to those genes to obtain gene-wise system z-scores — this is the key rationale, since system z-scores make weightscale sensitive to cell-type-specific expression roles across the whole tissue. Run CreateConnectome with species='human', min.cells.per.ident=75, p.values=FALSE, calculate.DOR=FALSE to produce the reference connectome containing the weight_sc column. Independently compute weightscale for each edge as the average of the ligand.scale and recept.scale z-score columns, then compare it to CreateConnectome's weight_sc values edge-by-edge. Validation passes when the reconstructed column matches weight_sc within numerical tolerance for all shared edges and demonstrably differs from the normalized-slot weight_norm values.

## Related tools

- **Connectome** (Provides CreateConnectome (generates the reference edgelist with weight_sc), the ncomms8866_human ligand-receptor gene list, and the FilterConnectome/EdgeDotPlot downstream context for edgeweight attributes) — https://github.com/msraredon/Connectome
- **Seurat** (Supplies NormalizeData and ScaleData to produce the gene-wise system z-scores (ligand.scale/recept.scale) that weightscale is computed from)
- **SeuratData** (Distributes the panc8 cross-platform pancreas dataset used as the validation input)
- **R** (Implementation environment in which the independent weightscale reconstruction and edge-by-edge comparison are coded)

## Examples

```
panc8.con <- CreateConnectome(panc8, species = 'human', min.cells.per.ident = 75, p.values = F, calculate.DOR = F)
```

## Evaluation signals

- Independently computed weightscale column matches CreateConnectome's weight_sc values within numerical tolerance for all shared edges
- The same edges' weight_norm values (computed from the normalized slot) differ from weight_sc, confirming signal-specificity vs raw connectivity weighting
- Connectome was built with min.cells.per.ident=75, p.values=FALSE, calculate.DOR=FALSE, so only identities with >75 captured cells contribute edges
- ScaleData was restricted to the union of ncomms8866_human ligand and receptor genes mapping to dataset rownames, so ligand.scale and recept.scale columns are populated for every edge

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell-cell communication
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified
- Outputs of Connectome are only as good as the ground-truth ligand-receptor database (FANTOM5 by default) against which the original mapping takes place

## Evidence

- [methods] CreateConnectome call with filters: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] normalize, map, and scale ligand/receptor genes: "Here, we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes"
- [methods] ligand-receptor gene union used for scaling: "connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol,Connectome::ncomms8866_human$Receptor.ApprovedSymbol)"
- [readme] edgelist with many edge attributes: [evidence span withheld — anchor status: external_doc]
- [readme] two edgeweights displayed on edges: [evidence span withheld — anchor status: external_doc]
