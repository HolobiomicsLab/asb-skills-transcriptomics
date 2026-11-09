---
name: ligand-receptor-gene-subsetting-against-fantom5
description: 'Use when preparing scRNA-seq data for Connectome ligand-receptor mapping:
  the expression matrix must be subset to the union of ligand and receptor genes from
  ncomms8866_mouse/human before NormalizeData/ScaleData and CreateConnectome.'
when_to_use_negative:
- The goal is unbiased gene-level analysis (e.g. FindMarkers differential expression
  on all genes) rather than ligand-receptor network construction — subsetting would
  discard non-connectome genes
- A researcher-hypothesized mechanism is not in FANTOM5 — the database must be customized,
  since extraneous or missing connections are bounded by the ground-truth list
- The dataset is already an edgelist/connectome output; this step operates upstream
  on raw expression data
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Connectome
  role: Provides the FANTOM5-derived ncomms8866_mouse/ncomms8866_human ligand-receptor
    databases and the CreateConnectome function that consumes the subset matrix
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Holds the scRNA-seq expression matrix whose features are subset to the mapped
    connectome genes before NormalizeData/ScaleData
- name: R
  role: Language in which the union of ligand/receptor approved symbols and matrix
    subsetting are performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:32db08b9f81d862b72c293bce8bb3fa7611ebbd22c7b1760b3ff5e0a8616a4c4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ligand-receptor gene subsetting against FANTOM5 (ncomms8866)

## Summary

Restricts a single-cell expression matrix to the union of ligand and receptor approved symbols drawn from the FANTOM5-derived ligand-receptor database (ncomms8866_mouse/human in the Connectome package) so that downstream CreateConnectome calls compute signaling edgelists only over known ligand-receptor mechanisms.

## When to use

Use when preparing scRNA-seq data for Connectome ligand-receptor mapping: the expression matrix must be subset to the union of ligand and receptor genes from ncomms8866_mouse/human before NormalizeData/ScaleData and CreateConnectome.

## When NOT to use

- The goal is unbiased gene-level analysis (e.g. FindMarkers differential expression on all genes) rather than ligand-receptor network construction — subsetting would discard non-connectome genes
- A researcher-hypothesized mechanism is not in FANTOM5 — the database must be customized, since extraneous or missing connections are bounded by the ground-truth list
- The dataset is already an edgelist/connectome output; this step operates upstream on raw expression data

## Inputs

- Single-cell expression matrix in a Seurat 3.0 object (e.g. from GSE143437_DeMicheli_MuSCatlas_rawdata.txt with cell_annotation identities)
- Connectome-bundled FANTOM5-derived ligand-receptor database: ncomms8866_mouse (or ncomms8866_human), with Ligand.ApprovedSymbol and Receptor.ApprovedSymbol columns
- Optionally, a user-provided list of paired ligand-receptor interactions

## Outputs

- Expression matrix subset to the union of mapped ligand and receptor genes (connectome.genes)
- Scaled/normalized connectome-ready Seurat object feeding CreateConnectome, yielding an edgelist of ligand-receptor mechanisms between cell-type nodes

## How to apply

Build the gene universe by taking the union of the Ligand.ApprovedSymbol and Receptor.ApprovedSymbol columns of the Connectome-bundled database (e.g. Connectome::ncomms8866_mouse for mouse data, ncomms8866_human for human data), both derived from the FANTOM5/Ramilowski 2015 (ncomms8866) ligand-receptor resource. Subset the Seurat object's expression matrix to only those mapped genes, then run NormalizeData and ScaleData with features restricted to the mapped connectome genes so CreateConnectome (species = 'mouse', p.values = F) scores only known mechanisms. If the species is not mouse or human, note that Connectome also ships mappings for rat and pig, or supply a user-provided ligand-receptor list. The rationale is to remove extraneous, non-mechanistic genes and keep network nodes/edges interpretable as ligand-receptor signaling events.

## Related tools

- **Connectome** (Provides the FANTOM5-derived ncomms8866_mouse/ncomms8866_human ligand-receptor databases and the CreateConnectome function that consumes the subset matrix) — https://github.com/msraredon/Connectome
- **Seurat** (Holds the scRNA-seq expression matrix whose features are subset to the mapped connectome genes before NormalizeData/ScaleData)
- **R** (Language in which the union of ligand/receptor approved symbols and matrix subsetting are performed)

## Examples

```
connectome.genes <- union(Connectome::ncomms8866_mouse$Ligand.ApprovedSymbol, Connectome::ncomms8866_mouse$Receptor.ApprovedSymbol); subset.matrix <- GetAssayData(seurat.obj)[connectome.genes, ]
```

## Evaluation signals

- The gene set used for subsetting equals union(ncomms8866_mouse$Ligand.ApprovedSymbol, ncomms8866_mouse$Receptor.ApprovedSymbol) — check no ligand or receptor symbols are missing from the subset matrix
- All genes feeding CreateConnectome map to at least one ligand-receptor mechanism in the chosen species database; no unmapped genes remain in ScaleData features
- Resulting connectome edgelists contain only recognized mechanism names (e.g. 'Vegfa - Flt1' within the VEGF mode), confirming mapping was against the FANTOM5 list
- Downstream validation: the longitudinal Vegfa analysis recovers FAP as dominant producer at Day 0/7, monocyte/macrophage at Day 2/5, and endothelial cells as dominant receivers
- Genes absent from the database (non-ligand, non-receptor) are excluded from the connectome computation, shrinking the feature space to the mapped connectome genes

## Limitations

- The outputs of Connectome, like any ligand-receptor mapping software, are only as good as the ground-truth database against which the original mapping takes place
- Strongly paired ligand and receptor expression is not direct evidence of cell-cell communication, and suspension-based single-cell sequencing does not preserve spatial proximity to verify candidate interactions
- Customization is needed to remove extraneous low-relevance connections or add newly-discovered/researcher-hypothesized mechanisms not present in the default database

## Evidence

- [methods] connectome.genes <- union(Connectome::ncomms8866_mouse$Ligand.ApprovedSymbol,Connectome::ncomms8866_mouse$Receptor.ApprovedSymbol): "connectome.genes <- union(Connectome::ncomms8866_mouse$Ligand.ApprovedSymbol,Connectome::ncomms8866_mouse$Receptor.ApprovedSymbol)"
- [intro] By default, Connectome uses the FANTOM5 database of ligand–receptor interactions13, formalizing the workflow first presented in1: "By default, Connectome uses the FANTOM5 database of ligand–receptor interactions13"
- [discussion] Customization allows removal of extraneous connections with low biologic relevance, or addition of newly-discovered or researcher-hypothesized mechanisms.: "Customization allows removal of extraneous connections with low biologic relevance, or addition of newly-discovered or researcher-hypothesized mechanisms."
- [methods] Here, we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes: "Here, we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes"
- [readme] Currently capable of creating mappings for human, mouse, rat, and pig, against the FANTOM5 ligand-receptor data found in Ramilowksi JA et al (2015): [evidence span withheld — anchor status: external_doc]
