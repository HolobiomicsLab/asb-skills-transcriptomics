---
name: pancreas-single-cell-data-processing
description: Use when building a single-tissue ligand–receptor connectome from the
  human pancreas scRNAseq data distributed by SeuratData, or when reproducing Connectome
  edgeweight columns (e.g. weight_sc) on a Seurat object.
when_to_use_negative:
- Input data is not single-cell RNA-sequencing data (e.g. bulk RNAseq or spatial transcriptomics),
  since Connectome mapping is designed for heterogeneous single-cell datasets
- The goal is differential connectomics between two tissue systems — use DifferentialConnectome
  on two pre-built connectomes instead of this single-tissue processing skill
- Fewer than 75 cells are captured per identity cluster, since the min.cells.per.ident
  = 75 filter excludes such clusters from the connectome
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: Connectome
  role: Creates the connectome edgelist (CreateConnectome), provides the FANTOM5 ligand–receptor
    reference (ncomms8866_human), and filters edges (FilterConnectome)
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Loads the panc8 dataset, sets identities to 'celltype', and performs NormalizeData
    and gene-restricted ScaleData to compute system z-scores
- name: SeuratData
  role: Distributes the panc8 cross-platform pancreas dataset used as input
- name: R
  role: Runtime language for the entire pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:311c56a31f2c5e69e2709ee7a760c52de47b4155b730ac0c1742be1d4b57d498
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# pancreas-single-cell-data-processing

## Summary

Processes the cross-platform panc8 human pancreas scRNAseq dataset from SeuratData — loading, normalizing, scaling ligand/receptor genes, and building a Connectome edgelist — as the reference single-tissue demonstration of ligand–receptor mapping in Connectome.

## When to use

Use when building a single-tissue ligand–receptor connectome from the human pancreas scRNAseq data distributed by SeuratData, or when reproducing Connectome edgeweight columns (e.g. weight_sc) on a Seurat object.

## When NOT to use

- Input data is not single-cell RNA-sequencing data (e.g. bulk RNAseq or spatial transcriptomics), since Connectome mapping is designed for heterogeneous single-cell datasets
- The goal is differential connectomics between two tissue systems — use DifferentialConnectome on two pre-built connectomes instead of this single-tissue processing skill
- Fewer than 75 cells are captured per identity cluster, since the min.cells.per.ident = 75 filter excludes such clusters from the connectome

## Inputs

- panc8 cross-platform human pancreas scRNAseq dataset distributed by SeuratData
- Seurat 3.0 object with a 'celltype' metadata column for identity assignment
- FANTOM5 ligand–receptor reference (Connectome::ncomms8866_human)

## Outputs

- Normalized and scaled Seurat object containing gene-wise z-scores for ligand and receptor genes
- Connectome edgelist connecting cell-type pairs via specific ligand–receptor mechanisms with edge attributes (pct, lig.scale, recept.scale, weight_norm, weight_sc)
- Optionally, a FilterConnectome-reduced edgelist (min.pct = 0.1, min.z = 0.25)

## How to apply

Load the panc8 cross-platform pancreas dataset via SeuratData and set cell identities to the 'celltype' metadata column so that identity classes define the network nodes. Normalize with Seurat's NormalizeData, then take the union of ligand and receptor genes from Connectome::ncomms8866_human that map to the dataset rownames and run ScaleData restricted to those genes to obtain gene-wise system z-scores. Run CreateConnectome with species='human', min.cells.per.ident=75, p.values=FALSE, calculate.DOR=FALSE, which limits analysis to identity clusters with > 75 cells captured and produces an edgelist with edge attributes including the weight_sc (weightscale) column. Optionally apply FilterConnectome (e.g. min.pct = 0.1, min.z = 0.25, remove.na = T) to retain only edges with ligand and receptor z-scores above 0.25. Validate by independently recomputing weightscale as the mean of ligand.scale and recept.scale z-scores per edge and confirming it matches CreateConnectome's weight_sc within numerical tolerance.

## Related tools

- **Connectome** (Creates the connectome edgelist (CreateConnectome), provides the FANTOM5 ligand–receptor reference (ncomms8866_human), and filters edges (FilterConnectome)) — https://github.com/msraredon/Connectome
- **Seurat** (Loads the panc8 dataset, sets identities to 'celltype', and performs NormalizeData and gene-restricted ScaleData to compute system z-scores)
- **SeuratData** (Distributes the panc8 cross-platform pancreas dataset used as input)
- **R** (Runtime language for the entire pipeline)

## Examples

```
panc8.con <- CreateConnectome(panc8, species = 'human', min.cells.per.ident = 75, p.values = F, calculate.DOR = F)
```

## Evaluation signals

- Independently computed weightscale (mean of ligand.scale and recept.scale z-scores per edge) must match CreateConnectome's weight_sc values within numerical tolerance for all shared edges
- The weightscale column must differ from the normalized-slot weight_norm values, demonstrating signal-specificity rather than raw connectivity weighting
- Every identity cluster in the connectome must have more than 75 cells (min.cells.per.ident = 75 threshold respected)
- All edges in the filtered connectome must have both ligand and receptor z-scores above 0.25 and non-NA values (remove.na = T)
- The ligand/receptor gene set passed to ScaleData must equal the union of ncomms8866_human ligands and receptors that map onto the panc8 rownames

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified
- Output quality is bounded by the ground-truth database (FANTOM5 / ncomms8866) against which the original ligand–receptor mapping takes place

## Evidence

- [methods] we run *Connectome* on the cross-platform Pancreas data distributed by SeuratData: "we run *Connectome* on the cross-platform Pancreas data distributed by SeuratData"
- [methods] CreateConnectome call: "panc8.con <- CreateConnectome(panc8,species = 'human',min.cells.per.ident = 75,p.values = F,calculate.DOR = F)"
- [methods] normalize/scale workflow: "Here, we normalize the data, identify the ligand and receptor genes that have mapped in the object, scale those genes"
- [methods] union of connectome genes: "connectome.genes <- union(Connectome::ncomms8866_human$Ligand.ApprovedSymbol,Connectome::ncomms8866_human$Receptor.ApprovedSymbol)"
- [methods] filter thresholds: "panc8.con2 <- FilterConnectome(panc8.con,min.pct = 0.1,min.z = 0.25,remove.na = T)"
- [methods] min cells rationale: "We also limit our connectomic analysis to only those clusters with > 75 cells captured"
- [readme] CreateConnectome role: [evidence span withheld — anchor status: external_doc]
- [methods] z-score filter: "filter the data to only include edges with a ligand and receptor z-score above 0.25"
