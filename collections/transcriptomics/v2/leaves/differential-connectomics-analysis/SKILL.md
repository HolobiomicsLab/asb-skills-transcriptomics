---
name: differential-connectomics-analysis
description: Use when two single-cell RNA-seq conditions (e.g. ifnb interferon-stimulated
  vs. control PBMCs from SeuratData) must be compared at the ligand–receptor level,
  and each edge needs a magnitude-of-perturbation score that is sign-agnostic for
  ligand/receptor changes.
when_to_use_negative:
- Only a single tissue system is available with no comparison condition — use CreateConnectome
  with FilterConnectome, Centrality, or CircosPlot instead of DifferentialConnectome
- The goal is to claim direct cell–cell communication or intracellular signal transduction
  — the article states strongly paired ligand and receptor expression is not direct
  evidence of communication and requires wet-lab validation
- The two connectomes do not contain exactly the same edges — DifferentialConnectome
  requires identical edge sets so every edge has a reference and test value
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: Connectome
  role: Provides CreateConnectome, DifferentialConnectome, DifferentialScoringPlot,
    and CircosDiff to build condition-specific connectomes and compute the always-positive
    perturbation score on differential edges
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Supplies SplitObject (by 'stim') and FindMarkers (adjusted p-value < 0.05
    per cell type over the ligand–receptor gene set) to define significantly perturbed
    edges
- name: SeuratData
  role: Distributes the ifnb interferon-stimulated vs. control PBMC dataset used as
    the differential-connectomics example
- name: R
  role: Implementation language for the connectomics workflow and perturbation-score
    computation
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:70eb23c42735f5f1dd09443363af81facd71db021256fb9a5696a97d30ecf3cf
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# differential-connectomics-analysis

## Summary

Quantitatively compares two cell–cell signaling networks (connectomes) built from single-cell RNA-seq data, e.g. IFN-stimulated vs. control PBMCs, and scores each differential edge with an always-positive 'perturbation score' (product of the absolute ligand and receptor log-fold changes) so strongly perturbed edges surface regardless of change direction.

## When to use

Use when two single-cell RNA-seq conditions (e.g. ifnb interferon-stimulated vs. control PBMCs from SeuratData) must be compared at the ligand–receptor level, and each edge needs a magnitude-of-perturbation score that is sign-agnostic for ligand/receptor changes.

## When NOT to use

- Only a single tissue system is available with no comparison condition — use CreateConnectome with FilterConnectome, Centrality, or CircosPlot instead of DifferentialConnectome
- The goal is to claim direct cell–cell communication or intracellular signal transduction — the article states strongly paired ligand and receptor expression is not direct evidence of communication and requires wet-lab validation
- The two connectomes do not contain exactly the same edges — DifferentialConnectome requires identical edge sets so every edge has a reference and test value

## Inputs

- Seurat 3.0 object with a condition metadata column (e.g. 'stim' in the ifnb PBMC dataset) and active identity slot defining cell-type nodes
- Two condition-split, normalized/scaled Seurat objects processed by CreateConnectome (species='human', p.values=F)
- A thresholded differential connectome table with ligand.norm.lfc and recept.norm.lfc columns and source.ligand/target.receptor cell-gene annotations

## Outputs

- Differential connectome edge list from DifferentialConnectome with per-edge test vs. control values
- Scored differential edge list (e.g. diff_connectome_scored.csv or perturbation_scores.csv) with ligand, receptor, source, target, log fold changes, and non-negative perturbation scores
- Optional DifferentialScoringPlot heatmap (third aligned panel) of per-edge perturbation scores

## How to apply

Load the ifnb dataset from SeuratData, set identities to seurat_annotations, and SplitObject by the 'stim' condition; normalize and scale each subset and run CreateConnectome (species='human', p.values=F) on each to yield two connectomic networks. Call DifferentialConnectome(connect.ctrl, connect.stim), which requires exactly the same edges in both connectomes so every edge has a reference and test value. To restrict to significantly perturbed genes, run Seurat FindMarkers per cell type between stimulated and control identities over the mapped ligand–receptor gene set and keep genes with adjusted p-value < 0.05. For each retained edge compute the perturbation score as the product of the absolute values of ligand.norm.lfc and recept.norm.lfc (range 0 to Infinity), optionally filtering with DifferentialScoringPlot-style thresholds (min.score = 1, min.pct = 0.1, infinity.to.max = TRUE) so scores match the paper's differential scoring figures. Rank and export the scored edges, keyed by source cell, ligand, target cell, and receptor.

## Related tools

- **Connectome** (Provides CreateConnectome, DifferentialConnectome, DifferentialScoringPlot, and CircosDiff to build condition-specific connectomes and compute the always-positive perturbation score on differential edges) — https://github.com/msraredon/Connectome
- **Seurat** (Supplies SplitObject (by 'stim') and FindMarkers (adjusted p-value < 0.05 per cell type over the ligand–receptor gene set) to define significantly perturbed edges)
- **SeuratData** (Distributes the ifnb interferon-stimulated vs. control PBMC dataset used as the differential-connectomics example)
- **R** (Implementation language for the connectomics workflow and perturbation-score computation)

## Examples

```
library(SeuratData); data('ifnb'); ifnb.list <- SplitObject(ifnb, split.by='stim'); diff <- DifferentialConnectome(connect.ctrl, connect.stim)
```

## Evaluation signals

- Every computed perturbation score is non-negative (score >= 0), since it is a product of absolute log-fold changes
- All retained edges have both ligand.norm.lfc and recept.norm.lfc present and pass the adjusted p < 0.05 significance criterion from FindMarkers
- Output CSV (diff_connectome_scored.csv / perturbation_scores.csv) exists with ligand, receptor, source, target, log-fold-change and perturbation-score fields populated
- Per-edge scores match the perturbation-score heatmap (third aligned panel) from DifferentialScoringPlot run with min.score = 1, min.pct = 0.1, infinity.to.max = TRUE
- Minimally perturbed edges score zero and strongly perturbed edges score high regardless of ligand/receptor sign, consistent with the 0-to-Infinity score range

## Limitations

- Strongly paired ligand and receptor expression is not direct evidence of cell–cell communication
- Suspension-based single-cell sequencing does not preserve spatial histologic information, so spatial proximity of candidate interactions cannot be verified
- Any robust claim of transduction requires extensive wet-lab experimentation not supplied by the computational workflow
- Output quality is bounded by the ground-truth database (FANTOM5 by default) against which ligand–receptor mapping is performed

## Evidence

- [readme] DifferentialConnectome requires identical edges: [evidence span withheld — anchor status: external_doc]
- [readme] Perturbation score definition: [evidence span withheld — anchor status: external_doc]
- [methods] SplitObject on ifnb dataset: "ifnb.list <- SplitObject(ifnb,split.by = 'stim')"
- [methods] FindMarkers per cell type between conditions: "temp <- FindMarkers(ifnb, 
                      ident.1 = celltypes.stim[i], 
                      ident.2 = celltypes.ctrl[i]"
- [methods] DifferentialConnectome function: "Here, we do so using the function `DifferentialConnectome`."
