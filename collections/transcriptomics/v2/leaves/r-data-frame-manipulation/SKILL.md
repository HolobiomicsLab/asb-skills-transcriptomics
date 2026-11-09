---
name: r-data-frame-manipulation
description: Use when an edgelist-style data frame (e.g., a Connectome diff.sub table
  with ligand.norm.lfc and recept.norm.lfc columns keyed by source.ligand/target.receptor)
  must be transformed into per-edge derived metrics before visualization or export.
when_to_use_negative:
- The task is comparing two connectomes edge-by-edge rather than computing a derived
  per-edge magnitude — use DifferentialConnectome instead, since it requires exactly
  the same edges to be present so there is always a reference and test value
- The input is a raw Seurat object or a base Connectome edgelist rather than an already-differentiated,
  thresholded diff.sub table with normalized log-fold-change columns
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Connectome
  role: Supplies DifferentialConnectome and DifferentialScoringPlot, whose diff.sub
    output columns and thresholds define the data-frame transformation
  repo: https://github.com/msraredon/Connectome
- name: R
  role: Language in which the vectorized per-edge score computation (product of absolute
    normalized log2 fold-changes) and CSV export are implemented
- name: Seurat
  role: Provides the ifnb stim/control objects and FindMarkers-based differential
    expression feeding the connectomes
- name: dplyr
  role: Used for per-row (per-edge) transformation and filtering of the diff.sub data
    frame
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:8b5c920b152f10f9580e049f34db4ef64413018db16d5185f268bd5166a830c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - dplyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# r-data-frame-manipulation

## Summary

R data-frame manipulation as applied in the Connectome differential-connectomics workflow: deriving per-edge perturbation scores from the thresholded differential connectome edgelist (diff.sub) by multiplying the absolute normalized log2 fold-changes of ligand and receptor.

## When to use

Use when an edgelist-style data frame (e.g., a Connectome diff.sub table with ligand.norm.lfc and recept.norm.lfc columns keyed by source.ligand/target.receptor) must be transformed into per-edge derived metrics before visualization or export.

## When NOT to use

- The task is comparing two connectomes edge-by-edge rather than computing a derived per-edge magnitude — use DifferentialConnectome instead, since it requires exactly the same edges to be present so there is always a reference and test value
- The input is a raw Seurat object or a base Connectome edgelist rather than an already-differentiated, thresholded diff.sub table with normalized log-fold-change columns

## Inputs

- Thresholded differential connectome data frame (diff.sub) from DifferentialConnectome with columns ligand.norm.lfc, recept.norm.lfc, source.ligand, target.receptor
- Stimulated (ifnb stim) and control ifnb connectomes from SeuratData
- Visualization thresholds from DifferentialScoringPlot (min.score = 1, min.pct = 0.1, infinity.to.max = T)

## Outputs

- Data frame of per-edge perturbation scores (product of absolute ligand and receptor normalized log2 fold-changes), keyed by source cell, ligand, target cell, and receptor
- perturbation_scores.csv

## How to apply

Load the thresholded differential connectome (diff.sub) produced by DifferentialConnectome on the stim-vs-control ifnb SeuratData connectomes, confirming it carries ligand.norm.lfc, recept.norm.lfc and the source.ligand/target.receptor cell-gene annotations. For each edge, compute the perturbation score as the product of the absolute values of ligand.norm.lfc and recept.norm.lfc, using dplyr-style vectorized column operations so the score is always positive and ranges from 0 to Infinity regardless of whether ligand and receptor change in the same or opposite directions. Restrict the table to edges passing the DifferentialScoringPlot thresholds (min.score = 1, min.pct = 0.1, infinity.to.max = T) so computed scores match the perturbation-score heatmap panel. Write the resulting per-edge table, keyed by source cell, ligand, target cell, and receptor, to perturbation_scores.csv.

## Related tools

- **Connectome** (Supplies DifferentialConnectome and DifferentialScoringPlot, whose diff.sub output columns and thresholds define the data-frame transformation) — https://github.com/msraredon/Connectome
- **R** (Language in which the vectorized per-edge score computation (product of absolute normalized log2 fold-changes) and CSV export are implemented)
- **Seurat** (Provides the ifnb stim/control objects and FindMarkers-based differential expression feeding the connectomes)
- **dplyr** (Used for per-row (per-edge) transformation and filtering of the diff.sub data frame)

## Examples

```
diff.sub$perturbation.score <- abs(diff.sub$ligand.norm.lfc) * abs(diff.sub$recept.norm.lfc); write.csv(subset(diff.sub, perturbation.score >= 1), 'perturbation_scores.csv')
```

## Evaluation signals

- Every computed perturbation score must be non-negative (score >= 0), since it is a product of absolute values
- All retained edges must have both ligand.norm.lfc and recept.norm.lfc present (no NA) before scoring
- Scores range from 0 to Infinity, increasing with the log fold-change magnitude of ligand and receptor in either direction; minimally perturbed edges score zero
- Per-edge values must match the perturbation-score heatmap (third aligned panel) from DifferentialScoringPlot run with min.score = 1, min.pct = 0.1, infinity.to.max = T
- Output table rows must be uniquely keyed by source cell, ligand, target cell, and receptor

## Limitations

- Strongly paired ligand and receptor expression (and hence a high perturbation score) is not direct evidence of cell–cell communication
- Connectome outputs are only as good as the ground-truth ligand–receptor database (FANTOM5 by default) against which the original mapping takes place
- Edges with infinite log-fold-change require special handling (DifferentialScoringPlot's infinity.to.max = T) so they do not dominate the score distribution

## Evidence

- [results] Connectome computes a per-edge perturbation score as the product of the absolute log-fold changes of ligand and receptor between test and control conditions: [evidence span withheld — anchor status: not_found]
- [results] it ranges from 0 to Infinity and increases with the log fold-change of the ligand and receptor in either direction: [evidence span withheld — anchor status: not_found]
- [readme] CircosDiff Creates a CircosPlot of a differential connectome, leveraging an always positive 'perturbation score': [evidence span withheld — anchor status: not_found]
- [methods] Here, we do so using the function `DifferentialConnectome`.: "Here, we do so using the function `DifferentialConnectome`."
