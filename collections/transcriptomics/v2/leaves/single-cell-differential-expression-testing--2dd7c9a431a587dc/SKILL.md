---
name: single-cell-differential-expression-testing
description: Use when comparing two conditions of a single-cell dataset (e.g., IFN-stimulated
  vs. control PBMCs) and you need per-cell-type significance of ligand/receptor perturbation
  before scoring differential signaling edges.
when_to_use_negative:
- There is only one condition (no test/control contrast), so DifferentialConnectome
  cannot produce a reference and test value per edge
- The goal is single-tissue network analysis of one connectome (use FilterConnectome/NetworkPlot/CircosPlot
  instead of differential testing)
- Either the ligand or receptor log2 fold change is missing for an edge, which violates
  the requirement that both LFCs be present before scoring
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Runs FindMarkers per cell type between stimulated and control identities to
    supply significant ligand/receptor log2 fold changes; built to work with Connectome
    from Satija Lab
- name: Connectome
  role: Consumes the two condition connectomes via DifferentialConnectome and provides
    DifferentialScoringPlot/CircosDiff logic for the always-positive perturbation
    score
  repo: https://github.com/msraredon/Connectome
- name: SeuratData
  role: Distributes the ifnb interferon-stimulated vs. control PBMC dataset and panc8
    pancreas data used for the differential workflow
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:3386f13d89ab0227b43418b5c9fc7de9a96113dd3a689949ed42630e72d630c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  - dplyr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-differential-expression-testing

## Summary

Statistical differential-expression testing (Seurat FindMarkers) that establishes which ligand and receptor genes are significantly perturbed between conditions, supplying the per-gene log2 fold changes and adjusted p-values that Connectome's DifferentialConnectome and perturbation scoring rely on.

## When to use

Use when comparing two conditions of a single-cell dataset (e.g., IFN-stimulated vs. control PBMCs) and you need per-cell-type significance of ligand/receptor perturbation before scoring differential signaling edges.

## When NOT to use

- There is only one condition (no test/control contrast), so DifferentialConnectome cannot produce a reference and test value per edge
- The goal is single-tissue network analysis of one connectome (use FilterConnectome/NetworkPlot/CircosPlot instead of differential testing)
- Either the ligand or receptor log2 fold change is missing for an edge, which violates the requirement that both LFCs be present before scoring

## Inputs

- Seurat 3.0 object with condition metadata (e.g., 'stim' column), such as the ifnb dataset from SeuratData
- Cell-type identity annotations (seurat_annotations) defining matched test/control identities per cell type
- Mapped ligand–receptor gene set from the Connectome object (FANTOM5 database by default)
- Two thresholded connectomes (control and stimulated) with identical edge sets for DifferentialConnectome

## Outputs

- Per-cell-type marker tables from FindMarkers with average log2 fold changes and adjusted p-values
- Differential connectome table with ligand.norm.lfc and recept.norm.lfc columns and source.ligand/target.receptor cell-gene annotations
- Per-edge perturbation scores (product of absolute ligand and receptor LFCs) keyed by source cell, ligand, target cell, receptor, written to diff_connectome_scored.csv / perturbation_scores.csv

## How to apply

Load the condition-paired Seurat object (e.g., the ifnb interferon-stimulated vs. control PBMC dataset from SeuratData) and split it by the 'stim' metadata column with SplitObject, normalizing and scaling each subset before CreateConnectome(species='human', p.values=F). For each cell type, run Seurat FindMarkers with ident.1 set to the stimulated identity and ident.2 to the matched control identity, restricted to the ligand–receptor gene set mapped by Connectome, and retain only genes with adjusted p-value < 0.05. Feed the resulting log2 fold changes into DifferentialConnectome(connect.ctrl, connect.stim), which requires the two connectomes to contain exactly the same edges so every edge has a reference and test value. Score each retained edge as the product of the absolute values of ligand.norm.lfc and recept.norm.lfc — always non-negative, ranging 0 to Infinity — so concordant and discordant ligand/receptor changes are treated equally. Apply the DifferentialScoringPlot thresholds (min.score = 1, min.pct = 0.1, infinity.to.max = TRUE) so the scored edges correspond to those visualized in the paper's differential figures.

## Related tools

- **Seurat** (Runs FindMarkers per cell type between stimulated and control identities to supply significant ligand/receptor log2 fold changes; built to work with Connectome from Satija Lab)
- **Connectome** (Consumes the two condition connectomes via DifferentialConnectome and provides DifferentialScoringPlot/CircosDiff logic for the always-positive perturbation score) — https://github.com/msraredon/Connectome
- **SeuratData** (Distributes the ifnb interferon-stimulated vs. control PBMC dataset and panc8 pancreas data used for the differential workflow)

## Examples

```
temp <- FindMarkers(ifnb, ident.1 = celltypes.stim[i], ident.2 = celltypes.ctrl[i])
```

## Evaluation signals

- Every retained gene passes the adjusted p-value < 0.05 significance criterion from FindMarkers
- Every computed perturbation score is non-negative (score >= 0) and scales with the magnitude of ligand and receptor log2 fold changes in either direction
- All retained edges have both ligand.norm.lfc and recept.norm.lfc populated, and both connectomes share exactly the same edge set before DifferentialConnectome
- Per-edge scores match the perturbation-score heatmap (third aligned panel) produced by DifferentialScoringPlot with min.score = 1, min.pct = 0.1, infinity.to.max = TRUE
- Output CSV exists with ligand, receptor, source, target, log fold change and perturbation-score fields populated and no empty score entries

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication, and suspension-based single-cell sequencing does not preserve spatial histologic information needed to verify proximity
- DifferentialConnectome requires exactly the same edges to be present in both connectomes, so edges absent from one condition cannot be compared
- Outputs are only as good as the ground-truth ligand–receptor database (FANTOM5 by default); a robust claim of signal transduction requires extensive wet-lab experimentation

## Evidence

- [methods] FindMarkers per cell type between stim and control identities: "temp <- FindMarkers(ifnb, 
                      ident.1 = celltypes.stim[i], 
                      ident.2 = celltypes.ctrl[i]"
- [methods] DifferentialConnectome compares the two condition connectomes: "Here, we do so using the function `DifferentialConnectome`."
- [methods] ifnb dataset source for the differential workflow: "we will use the interferon-stimulated vs. control PBMC data distributed by SeuratData"
- [readme] DifferentialConnectome edge-matching requirement: [evidence span withheld — anchor status: external_doc]
- [readme] DifferentialScoringPlot visualization counterpart of the scoring: [evidence span withheld — anchor status: external_doc]
