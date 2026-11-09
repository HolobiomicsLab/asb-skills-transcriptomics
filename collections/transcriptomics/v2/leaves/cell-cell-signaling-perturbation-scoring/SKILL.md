---
name: cell-cell-signaling-perturbation-scoring
description: Use when comparing cell–cell signaling between two tissue systems (e.g.,
  IFN-stimulated vs. control PBMCs from SeuratData 'ifnb') and you need to rank edges
  where ligand and receptor are both significantly perturbed, whether concordantly
  or discordantly.
when_to_use_negative:
- You are analyzing a single tissue/condition rather than comparing two systems —
  use single-connectome functions (FilterConnectome, NetworkPlot, CircosPlot) instead
- Your goal is to claim actual signal transduction — strongly paired ligand and receptor
  expression is not direct evidence of cell–cell communication and requires wet-lab
  validation
- Spatial context matters — suspension-based single-cell sequencing does not preserve
  spatial histologic information, so candidate interactions cannot be verified for
  spatial proximity
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: Connectome
  role: Provides CreateConnectome, DifferentialConnectome, DifferentialScoringPlot,
    and CircosDiff for building, differencing, and perturbation-scoring the signaling
    networks
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: SplitObject, NormalizeData/ScaleData, and FindMarkers supply the condition-split
    objects and per-cell-type differential expression statistics used to filter edges
- name: SeuratData
  role: Distributes the ifnb interferon-stimulated vs. control PBMC dataset used as
    the demonstration input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:3826d027e4cd56b2c233d9eca29b9d35ea39e77616bdba82630e2621955615a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - SeuratData
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cell-cell-signaling-perturbation-scoring

## Summary

Compute per-edge 'perturbation scores' in a differential connectome — the product of the absolute log-fold changes of the ligand and receptor — to rank strongly perturbed ligand–receptor edges between two conditions (e.g., IFN-stimulated vs. control PBMCs), regardless of the direction of change.

## When to use

Use when comparing cell–cell signaling between two tissue systems (e.g., IFN-stimulated vs. control PBMCs from SeuratData 'ifnb') and you need to rank edges where ligand and receptor are both significantly perturbed, whether concordantly or discordantly.

## When NOT to use

- You are analyzing a single tissue/condition rather than comparing two systems — use single-connectome functions (FilterConnectome, NetworkPlot, CircosPlot) instead
- Your goal is to claim actual signal transduction — strongly paired ligand and receptor expression is not direct evidence of cell–cell communication and requires wet-lab validation
- Spatial context matters — suspension-based single-cell sequencing does not preserve spatial histologic information, so candidate interactions cannot be verified for spatial proximity

## Inputs

- Seurat object containing two conditions (e.g., SeuratData 'ifnb' PBMC dataset with 'stim' metadata column)
- Ground-truth ligand–receptor database (FANTOM5 / Ramilowski et al. by default, or user-provided list)
- Two condition-specific connectome edgelists (from CreateConnectome) with identical edge sets

## Outputs

- Differential connectome edgelist from DifferentialConnectome with per-edge ligand/receptor log-fold changes
- Scored and ranked differential edge table (e.g., diff_connectome_scored.csv) with ligand, receptor, source, target, log fold change and perturbation-score fields, restricted to edges with adjusted p < 0.05 for both ligand and receptor
- Optional CircosDiff visualization of the differential connectome colored by the always-positive perturbation score

## How to apply

Load the SeuratData 'ifnb' interferon-stimulated vs. control PBMC dataset, set identities to seurat_annotations, and SplitObject by the 'stim' condition. NormalizeData and ScaleData each subset, then run CreateConnectome(species='human', p.values=F) on each to yield two connectomic networks whose edge sets align. Compute the differential connectome with DifferentialConnectome(connect.ctrl, connect.stim) — this function requires exactly the same edges in both connectomes so every edge has a reference and test value. In parallel, run Seurat FindMarkers per cell type between stimulated and control identities over the mapped ligand–receptor gene set and retain genes with adjusted p-value < 0.05. Filter the differential connectome to edges whose source.ligand and target.receptor cell-gene pairs are both significant, then score each edge by the product of ligand and receptor log-fold changes (DifferentialScoringPlot-style logic with min.score = 1, min.pct = 0.1, infinity.to.max = TRUE); rank and export the scored edge list (e.g., diff_connectome_scored.csv).

## Related tools

- **Connectome** (Provides CreateConnectome, DifferentialConnectome, DifferentialScoringPlot, and CircosDiff for building, differencing, and perturbation-scoring the signaling networks) — https://github.com/msraredon/Connectome
- **Seurat** (SplitObject, NormalizeData/ScaleData, and FindMarkers supply the condition-split objects and per-cell-type differential expression statistics used to filter edges)
- **SeuratData** (Distributes the ifnb interferon-stimulated vs. control PBMC dataset used as the demonstration input)

## Examples

```
ifnb.list <- SplitObject(ifnb, split.by = 'stim'); diff.con <- DifferentialConnectome(connect.ctrl, connect.stim); CircosDiff(diff.con, min.score = 1, min.pct = 0.1, infinity.to.max = TRUE)
```

## Evaluation signals

- diff_connectome_scored.csv exists with ligand, receptor, source, target, log fold change and perturbation-score fields populated
- Every retained edge passes the adjusted p < 0.05 significance criterion for BOTH the source.ligand and target.receptor cell-gene pairs
- Perturbation scores are always positive (product of absolute log-fold changes), as enforced by CircosDiff's 'always positive perturbation score'
- DifferentialConnectome ran without dropping edges — the two input connectomes must contain exactly the same edge set
- Filtered edges satisfy the DifferentialScoringPlot thresholds: min.score = 1, min.pct = 0.1, infinity.to.max = TRUE

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication; robust claims of transduction require extensive wet-lab experimentation
- Outputs are only as good as the ground-truth database (FANTOM5) against which ligand–receptor mapping is performed
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified
- No version or release number stated for Connectome in the article, complicating exact reproducibility

## Evidence

- [methods] Connectome assigns each edge a 'perturbation score' defined as the product of the absolute log-fold changes of the ligand and receptor, enabling identification of strongly perturbed edges regardless of the direction of ligand or receptor change.: [evidence span withheld — anchor status: not_found]
- [readme] DifferentialConnectome Allows direct quantitative comparison of two connectomes. Requires exactly the same edges to be present, so that there is always a reference value and a test value.: [evidence span withheld — anchor status: external_doc]
- [readme] CircosDiff Creates a CircosPlot of a differential connectome, leveraging an always positive 'perturbation score': [evidence span withheld — anchor status: not_found]
- [methods] we will use the interferon-stimulated vs. control PBMC data distributed by SeuratData: "we will use the interferon-stimulated vs. control PBMC data distributed by SeuratData"
- [discussion] strongly paired ligand and receptor expression is not direct evidence of cell–cell communication: "strongly paired ligand and receptor expression is not direct evidence of cell–cell communication"
