---
name: edge-level-network-score-computation
description: Use when analyzing a DifferentialConnectome output (e.g., IFN-stimulated
  vs. control ifnb PBMCs) and you need a sign-independent per-edge magnitude of perturbation
  for ranking, filtering, or heatmap/circos visualization of differential ligand–receptor
  signaling.
when_to_use_negative:
- The input is a single-condition connectome or filtered edgelist (e.g., from FilterConnectome)
  rather than a differential connectome — there are no test/control log-fold-changes
  to multiply.
- You need a signed direction of change per edge rather than a magnitude-only, always-positive
  perturbation score.
- 'You need direct evidence of communication: strongly paired ligand–receptor expression
  changes are not direct evidence of cell–cell communication, and spatial proximity
  cannot be verified from suspension scRNA-seq.'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0085
tools:
- name: Connectome
  role: Provides DifferentialConnectome, DifferentialScoringPlot, and CircosDiff,
    which compute and visualize the always-positive perturbation score on differential
    connectomes
  repo: https://github.com/msraredon/Connectome
- name: Seurat
  role: Supplies the ifnb stim/control PBMC objects and per-cell-type FindMarkers
    log-fold changes feeding the differential connectome
- name: R
  role: Language in which the score computation (product of absolute ligand.norm.lfc
    and recept.norm.lfc) and dplyr-based filtering are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-07959-x
    title: Computation and visualization of cell–cell signaling topologies in single-cell
      systems data using Connectome
  source_content_hash: sha256:df6b182e3e8fb07b35826efe273387ffdf187b7ad32c436efa96f8ba1bce0bbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msraredon/Connectome
  tools:
  - Connectome
  - R
  - Seurat
  - dplyr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# edge-level-network-score-computation

## Summary

Computation of per-edge perturbation scores in a Connectome differential connectome, where each ligand–receptor edge between cell types is scored as the product of the absolute normalized log2 fold-changes of ligand and receptor between test and control conditions, yielding an always-positive magnitude of perturbation from 0 to Infinity regardless of direction of change.

## When to use

Use when analyzing a DifferentialConnectome output (e.g., IFN-stimulated vs. control ifnb PBMCs) and you need a sign-independent per-edge magnitude of perturbation for ranking, filtering, or heatmap/circos visualization of differential ligand–receptor signaling.

## When NOT to use

- The input is a single-condition connectome or filtered edgelist (e.g., from FilterConnectome) rather than a differential connectome — there are no test/control log-fold-changes to multiply.
- You need a signed direction of change per edge rather than a magnitude-only, always-positive perturbation score.
- You need direct evidence of communication: strongly paired ligand–receptor expression changes are not direct evidence of cell–cell communication, and spatial proximity cannot be verified from suspension scRNA-seq.

## Inputs

- Thresholded differential connectome (diff.sub) from DifferentialConnectome, with ligand.norm.lfc and recept.norm.lfc columns and source.ligand/target.receptor annotations
- Paired Seurat connectomes from stim and control conditions (e.g., ifnb SplitObject by 'stim')
- FANTOM5 ligand–receptor reference (default) or user-supplied ligand–receptor list

## Outputs

- perturbation_scores.csv with per-edge scores keyed by source cell, ligand, target cell, and receptor
- Filtered edge table matching the DifferentialScoringPlot panel (min.score = 1, min.pct = 0.1)

## How to apply

Run DifferentialConnectome on two matched connectomes (stim vs. control) built with CreateConnectome from the Seurat-split ifnb object, so the result carries ligand.norm.lfc and recept.norm.lfc columns plus source.ligand and target.receptor cell–gene annotations. For each edge, compute the perturbation score as the product of the absolute values of ligand.norm.lfc and recept.norm.lfc; because both factors are absolute, the score is always non-negative, ranges from 0 to Infinity, and rises with log-fold change in either the same or opposite directions — this is the same 'always positive perturbation score' leveraged by CircosDiff. Optionally restrict to edges passing the DifferentialScoringPlot thresholds (min.score = 1, min.pct = 0.1, infinity.to.max = T) so per-edge scores correspond to the perturbation-score heatmap panel. Write scores keyed by source cell, ligand, target cell, and receptor for downstream comparison with the plotted figures.

## Related tools

- **Connectome** (Provides DifferentialConnectome, DifferentialScoringPlot, and CircosDiff, which compute and visualize the always-positive perturbation score on differential connectomes) — https://github.com/msraredon/Connectome
- **Seurat** (Supplies the ifnb stim/control PBMC objects and per-cell-type FindMarkers log-fold changes feeding the differential connectome)
- **R** (Language in which the score computation (product of absolute ligand.norm.lfc and recept.norm.lfc) and dplyr-based filtering are implemented)

## Examples

```
scores <- transform(diff.sub, perturbation_score = abs(ligand.norm.lfc) * abs(recept.norm.lfc)); write.csv(subset(scores, perturbation_score >= 1), 'perturbation_scores.csv')
```

## Evaluation signals

- Every computed perturbation score is non-negative (score >= 0), since it is a product of absolute log-fold changes.
- All retained edges have both ligand and receptor normalized log2 fold-changes present (no NA values).
- Scores range from 0 to Infinity, with minimally perturbed edges scoring 0 and strongly perturbed edges scoring high regardless of ligand/receptor sign concordance.
- Per-edge values match the third aligned panel (perturbation-score heatmap) of DifferentialScoringPlot run with min.score = 1, min.pct = 0.1, infinity.to.max = T.
- The output table is keyed by the full edge identity: source cell, ligand, target cell, receptor.

## Limitations

- Strongly paired ligand and receptor expression (or strong perturbation thereof) is not direct evidence of cell–cell communication; robust transduction claims require wet-lab experimentation.
- The score is bounded by the ground-truth ligand–receptor database (FANTOM5 by default); outputs are only as good as that reference.
- Suspension-based single-cell sequencing does not preserve spatial information, so spatial plausibility of high-scoring edges cannot be verified.
- Edges with infinite fold-change require the infinity.to.max = T handling used in DifferentialScoringPlot to remain plottable.

## Evidence

- [readme] DifferentialScoringPlot Provides a comprehensive heatmap view of differences of interest between two connectomes.: [evidence span withheld — anchor status: not_found]
- [readme] CircosDiff Creates a CircosPlot of a differential connectome, leveraging an always positive 'perturbation score': [evidence span withheld — anchor status: external_doc]
- [readme] DifferentialConnectome Allows direct quantitative comparison of two connectomes. Requires exactly the same edges to be present: [evidence span withheld — anchor status: external_doc]
- [intro] By default, Connectome uses the FANTOM5 database of ligand–receptor interactions: "By default, Connectome uses the FANTOM5 database of ligand–receptor interactions13"
- [discussion] strongly paired ligand and receptor expression is not direct evidence of cell–cell communication: "strongly paired ligand and receptor expression is not direct evidence of cell–cell communication"
