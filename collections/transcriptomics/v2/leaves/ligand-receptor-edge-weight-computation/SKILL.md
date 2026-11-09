---
name: ligand-receptor-edge-weight-computation
description: Use when you need to reproduce, verify, or interpret the edge attributes
  Connectome emits for every celltype-to-celltype ligand–receptor mechanism, or when
  you must compute normalized-slot (w1) vs z-score-based (w2) weights independently
  from Seurat-normalized data.
when_to_use_negative:
- You need statistical confidence calls per edge — pass p.values = TRUE to CreateConnectome
  instead of computing only w1/w2 point estimates.
- Strongly paired ligand–receptor expression is being treated as direct evidence of
  cell–cell communication; the article explicitly states it is not, and suspension-based
  scRNAseq does not preserve spatial proximity.
- Downstream differential connectomics (DifferentialConnectome) is the goal and a
  completed connectome edgelist already exists — use filtering/differential functions
  rather than recomputing raw edgeweights.
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: Connectome
  role: Provides CreateConnectome which emits the reference weight_norm (w1) and weight_sc
    (w2) columns used to validate independently reconstructed edgeweights
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: NormalizeData and ScaleData supply the normalized expression and gene-wise
    z-scores that feed the w1 and w2 calculations
- name: SeuratData
  role: Distributes the panc8 cross-platform pancreas dataset used as the shared test
    input
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:4fad4b839d1477ddbbfedddc718c983851e568c1c1e733c15b886779c9939a4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ligand-receptor-edge-weight-computation

## Summary

Reconstruct how Connectome assigns edgeweights to ligand–receptor edges: weightnorm (w1), the product of celltype-wise normalized ligand and receptor expression, and weightscale (w2), the mean of gene-wise system z-scores which encodes signal specificity across the whole tissue system.

## When to use

Use when you need to reproduce, verify, or interpret the edge attributes Connectome emits for every celltype-to-celltype ligand–receptor mechanism, or when you must compute normalized-slot (w1) vs z-score-based (w2) weights independently from Seurat-normalized data.

## When NOT to use

- You need statistical confidence calls per edge — pass p.values = TRUE to CreateConnectome instead of computing only w1/w2 point estimates.
- Strongly paired ligand–receptor expression is being treated as direct evidence of cell–cell communication; the article explicitly states it is not, and suspension-based scRNAseq does not preserve spatial proximity.
- Downstream differential connectomics (DifferentialConnectome) is the goal and a completed connectome edgelist already exists — use filtering/differential functions rather than recomputing raw edgeweights.

## Inputs

- Seurat 3.0 object (e.g., SeuratData panc8 cross-platform pancreas data) with active identity slot set to celltype metadata
- Ligand–receptor reference database (Connectome::ncomms8866_human, or FANTOM5-derived list by default, or user-provided paired list)

## Outputs

- weight_norm_edges.csv: table with a weight_norm column per ligand–receptor edge per source–target celltype pair (normalized-expression product)
- weightscale column: per-edge mean of ligand.scale and recept.scale z-scores, comparable to CreateConnectome's weight_sc column

## How to apply

Load the SeuratData panc8 pancreas dataset, set active identities to the 'celltype' metadata column, and run Seurat's NormalizeData; restrict to genes in the union of Ligand.ApprovedSymbol and Receptor.ApprovedSymbol from the ncomms8866_human reference list that map to the rownames. For weightnorm (w1), compute mean normalized expression of each ligand within each source celltype and each receptor within each target celltype, then form the product per ligand–receptor pair per source–target combination. For weightscale (w2), additionally run ScaleData on the restricted gene set to obtain ligand.scale and recept.scale z-scores and average them per edge, since system z-scores incorporate the topology of the entire tissue system and capture signal specificity rather than raw connectivity. Generate the reference connectome with CreateConnectome(species='human', min.cells.per.ident=75, p.values=F, calculate.DOR=F) and compare the independently computed weight_norm/weightscale columns edge-by-edge against CreateConnectome's weight_norm and weight_sc values within numerical tolerance.

## Related tools

- **Connectome** (Provides CreateConnectome which emits the reference weight_norm (w1) and weight_sc (w2) columns used to validate independently reconstructed edgeweights) — https://github.com/msraredon/Connectome
- **Seurat** (NormalizeData and ScaleData supply the normalized expression and gene-wise z-scores that feed the w1 and w2 calculations)
- **SeuratData** (Distributes the panc8 cross-platform pancreas dataset used as the shared test input)

## Examples

```
connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol, Connectome::ncomms8866_human$Receptor.ApprovedSymbol); panc8.con <- CreateConnectome(panc8, species = 'human', min.cells.per.ident = 75, p.values = F, calculate.DOR = F)
```

## Evaluation signals

- weight_norm values in weight_norm_edges.csv numerically match CreateConnectome's weight_norm output for all shared edges
- Independently computed weightscale column matches CreateConnectome's weight_sc values within numerical tolerance edge-by-edge on the same input data
- Computed weightscale differs from weight_norm on shared edges, demonstrating signal-specificity (z-score-based) vs raw-connectivity (normalized-expression) weighting
- All clusters/identities below min.cells.per.ident = 75 cells are excluded from both the independent and reference computations
- Every edge corresponds to a ligand–receptor pair present in the ncomms8866_human reference list with both genes mapped in the dataset rownames

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication; robust claims of transduction require extensive wet-lab experimentation.
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified.
- Output quality is bounded by the ground-truth database (e.g., FANTOM5) against which ligand–receptor mapping is performed.
- weightscale's z-scores are system-dependent, so w2 values are not comparable across datasets prepared with different celltype compositions or preprocessing.

## Evidence

- [methods] min.cells.per.ident and species settings: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] gene restriction to reference list: "connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol,Connectome::ncomms8866_human$Receptor.ApprovedSymbol)"
- [methods] normalize-then-scale workflow: "Here, we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes"
- [readme] edgelist with many edge attributes: [evidence span withheld — anchor status: external_doc]
- [readme] two edgeweights: [evidence span withheld — anchor status: not_found]
