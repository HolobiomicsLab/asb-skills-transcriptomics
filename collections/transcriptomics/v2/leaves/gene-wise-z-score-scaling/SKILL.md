---
name: gene-wise-z-score-scaling
description: Use when building a Connectome edgelist whose edgeweights must reflect
  cell-type-specific expression roles (weight_sc/weightscale) rather than raw connectivity
  (weight_norm), e.g. before CreateConnectome on scRNA-seq data.
when_to_use_negative:
- The goal is raw connectivity magnitude (weight_norm) rather than signal specificity
  — scaling introduces system-level z-score topology into the edgeweight
- Clusters have fewer than min.cells.per.ident cells (the pancreas workflow used 75),
  making per-identity scaling statistics unreliable
- The analysis compares two tissue systems where edge sets must match exactly (DifferentialConnectome
  requires identical edges), unless scaling is applied consistently to both
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Connectome
  role: Provides the ligand–receptor gene lists (ncomms8866_human/mouse), CreateConnectome
    for computing the weight_sc edgelist from scaled data, and FilterConnectome for
    downstream edge filtration
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: NormalizeData and ScaleData (restricted to connectome genes) produce the normalized
    and scaled slots that feed weight_norm and weight_sc respectively
- name: SeuratData
  role: Distributes the panc8 cross-platform pancreas and ifnb datasets used to exercise
    the scaling workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:0d3660599deb7ec8628ab3ca664a4ed180cbb9291e050d2d775302a1d3906a50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# gene-wise-z-score-scaling

## Summary

Scales ligand and receptor genes in a Seurat object to gene-wise system z-scores via ScaleData, restricted to connectome genes, before ligand–receptor mapping so that Connectome edge attributes capture signal specificity rather than raw expression connectivity.

## When to use

Use when building a Connectome edgelist whose edgeweights must reflect cell-type-specific expression roles (weight_sc/weightscale) rather than raw connectivity (weight_norm), e.g. before CreateConnectome on scRNA-seq data.

## When NOT to use

- The goal is raw connectivity magnitude (weight_norm) rather than signal specificity — scaling introduces system-level z-score topology into the edgeweight
- Clusters have fewer than min.cells.per.ident cells (the pancreas workflow used 75), making per-identity scaling statistics unreliable
- The analysis compares two tissue systems where edge sets must match exactly (DifferentialConnectome requires identical edges), unless scaling is applied consistently to both

## Inputs

- Seurat 3.0 object with active identity slot set to cell-type metadata (e.g. panc8 cross-platform pancreas data from SeuratData, identities set to 'celltype')
- Ligand–receptor reference list, e.g. Connectome::ncomms8866_human (FANTOM5/Ramilowski 2015)

## Outputs

- Seurat object with a scaled data slot containing gene-wise z-scores for the union of ligand/receptor connectome genes
- Connectome edgelist (from CreateConnectome) with weight_sc edgeweight column derived from ligand.scale and recept.scale z-scores, alongside weight_norm

## How to apply

Run NormalizeData on the Seurat object, take the union of ligand and receptor genes from the Connectome reference database (e.g. Connectome::ncomms8866_human$Ligand.ApprovedSymbol and Receptor.ApprovedSymbol) that map to the dataset rownames, and run ScaleData restricted to those connectome.genes to obtain gene-wise system z-scores. The article's pancreas workflow ('we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes, and generate the connectome') calls CreateConnectome with species='human', min.cells.per.ident=75, p.values=FALSE, calculate.DOR=FALSE. The scaled slot feeds the weight_sc (weightscale) column, defined as the mean of ligand.scale and recept.scale z-scores per edge, while the normalized (non-scaled) slot feeds weight_norm; because z-scores are computed gene-wise across the whole tissue system, weight_sc encodes signal specificity. Verify by independently averaging the ligand.scale and recept.scale columns per edge and matching CreateConnectome's weight_sc within numerical tolerance, while confirming it differs from weight_norm.

## Related tools

- **Connectome** (Provides the ligand–receptor gene lists (ncomms8866_human/mouse), CreateConnectome for computing the weight_sc edgelist from scaled data, and FilterConnectome for downstream edge filtration) — https://github.com/msraredon/Connectome
- **Seurat** (NormalizeData and ScaleData (restricted to connectome genes) produce the normalized and scaled slots that feed weight_norm and weight_sc respectively)
- **SeuratData** (Distributes the panc8 cross-platform pancreas and ifnb datasets used to exercise the scaling workflow)

## Examples

```
connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol,Connectome::ncomms8866_human$Receptor.ApprovedSymbol); panc8 <- ScaleData(panc8, features = connectome.genes); panc8.con <- CreateConnectome(panc8, species='human', min.cells.per.ident=75, p.values=F, calculate.DOR=F)
```

## Evaluation signals

- Independently computed weightscale (mean of ligand.scale and recept.scale per edge) matches CreateConnectome's weight_sc within numerical tolerance for all shared edges
- weight_sc values differ from weight_norm values on the same edges, demonstrating signal-specificity vs. connectivity weighting
- Scaled slot contains z-scores only for the union of ligand/receptor genes from ncomms8866 that map to dataset rownames (connectome.genes)
- All clusters retained in the connectome exceed the min.cells.per.ident threshold (75 in the pancreas example)
- FilterConnectome on the scaled edgelist retains only edges with ligand and receptor z-score above 0.25 when min.z = 0.25 is applied

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication; scaling does not prove signaling occurs
- Output quality is bounded by the ground-truth database (FANTOM5 / ncomms8866) against which ligand–receptor mapping is performed
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so scaled edge strength cannot verify spatial proximity
- No version or release number is stated in the article for Connectome, complicating exact reproduction (README cites v1.0.1)

## Evidence

- [methods] normalized vs scaled slot feeds weight_sc/weight_norm: "we run *Connectome* on the cross-platform Pancreas data distributed by SeuratData"
- [methods] normalize, identify ligand/receptor genes, scale, generate connectome: "Here, we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes"
- [methods] gene union from ncomms8866_human: "connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol,Connectome::ncomms8866_human$Receptor.ApprovedSymbol)"
- [methods] z-score filter threshold: "filter the data to only include edges with a ligand and receptor z-score above 0.25"
- [readme] CreateConnectome output edgelist with edge attributes: [evidence span withheld — anchor status: external_doc]
- [readme] edgeweights represent signal strength and signal specificity: [evidence span withheld — anchor status: external_doc]
