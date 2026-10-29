---
name: spearman-correlation-trajectory-progression
description: Use when you have a reconstructed gene regulatory network (CLR/CellNet-style
  edge list) plus per-cell trajectory progression values from CellRouter and want
  to rank regulators driving or mediating specific cell-state transitions.
when_to_use_negative:
- You have no reconstructed gene regulatory network linking regulators to predicted
  target genes — the multiplicative score requires regulator-target edges.
- You need trajectories between arbitrary cell states rather than regulator scores;
  CellRouter's flow-network trajectory reconstruction, not this scoring step, addresses
  trajectory detection.
- Trajectory progression values are unavailable (e.g., only raw counts without CellRouter
  trajectory inference) — correlations with progression cannot be computed.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellRouter
  role: Platform that reconstructs the gene regulatory network and multi-state trajectories
    whose progression values feed the Spearman-correlation-based regulator scoring
  repo: https://github.com/edroaldo/cellrouter
- name: R
  role: Environment in which Spearman correlations, target-gene aggregation, and GRN
    score computation are implemented
- name: igraph
  role: R package used within CellRouter for community detection and network representation
    of subpopulations and the gene regulatory network
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:ad36939c77e1e6fd4b264021876e6d32669c5808a543424f4ddee688936c5755
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# spearman-correlation-trajectory-progression

## Summary

Scores transcriptional regulators of cell-fate transitions in CellRouter by combining Spearman correlations of regulator expression and its predicted targets' expression with trajectory progression, multiplied by the number of regulated target genes (GRN(i,j) = c(i,j) * m(t,j) * n(j)).

## When to use

Use when you have a reconstructed gene regulatory network (CLR/CellNet-style edge list) plus per-cell trajectory progression values from CellRouter and want to rank regulators driving or mediating specific cell-state transitions.

## When NOT to use

- You have no reconstructed gene regulatory network linking regulators to predicted target genes — the multiplicative score requires regulator-target edges.
- You need trajectories between arbitrary cell states rather than regulator scores; CellRouter's flow-network trajectory reconstruction, not this scoring step, addresses trajectory detection.
- Trajectory progression values are unavailable (e.g., only raw counts without CellRouter trajectory inference) — correlations with progression cannot be computed.

## Inputs

- Gene regulatory network edge list (regulator-target predictions, CLR/CellNet-style, from CellRouter's GRN reconstruction step)
- Per-cell trajectory progression values for each CellRouter cell-fate transition (multi-state trajectories between subpopulations)
- Single-cell expression matrix used to compute regulator and target gene expression correlations

## Outputs

- regulator_scores.csv: one row per transcriptional regulator with transition-specific GRN scores per trajectory
- Ranked list of regulators driving or mediating each cell-fate transition

## How to apply

Load the gene regulatory network edge list (reconstructed CLR/CellNet-style within CellRouter) and the trajectory progression data for each cell-fate transition. For every regulator i and trajectory j, compute the Spearman correlation c(i,j) between the regulator's expression and trajectory progression in R. For each regulator, compute the mean correlation m(t,j) of its predicted target genes with the trajectory progression and count the number of regulated targets n(j). Combine multiplicatively as GRN(i,j) = c(i,j) * m(t,j) * n(j) to obtain a transition-specific score, then assemble all regulator scores into a table (e.g., regulator_scores.csv) for downstream identification of drivers of transitions such as HSPC differentiation to erythroid, myeloid, and lymphoid lineages. The rationale is that regulators whose own expression tracks progression AND whose predicted targets coordinately track progression are likely functional drivers of that transition.

## Related tools

- **CellRouter** (Platform that reconstructs the gene regulatory network and multi-state trajectories whose progression values feed the Spearman-correlation-based regulator scoring) — https://github.com/edroaldo/cellrouter
- **R** (Environment in which Spearman correlations, target-gene aggregation, and GRN score computation are implemented)
- **igraph** (R package used within CellRouter for community detection and network representation of subpopulations and the gene regulatory network)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- Output file exists, parses as a valid CSV with one row per regulator and one score column per trajectory.
- All GRN scores are finite numeric values equal to the product of the three components: regulator correlation, mean target-gene correlation, and target count (GRN(i,j) = c(i,j) * m(t,j) * n(j)).
- Regulator-target relationships used for n(j) and m(t,j) are consistent with the input GRN edge list (no targets counted for regulators absent from the network).
- Spearman correlations fall within [-1, 1], so per-component checks bound the score magnitude by n(j).
- Top-scored regulators for a given transition are biologically coherent with known lineage drivers (e.g., HSPC differentiation to erythroid, myeloid, and lymphoid lineages).

## Limitations

- The score depends on the quality of the reconstructed GRN; regulatory relationships are only 'putative gene regulatory relationships' identified by network reconstruction.
- CellRouter was tested in Unix-based systems (Unix or Mac) and was not tested in Windows, which may affect reproducibility of the R workflow.
- GRN scores capture transition-specific associations along trajectory progression; they do not by themselves prove causality, and functional characterization of identified genes (e.g., for generating engraftable HSC-like cells) extends beyond the analysis.

## Evidence

- [other] CellRouter employs a scoring scheme, termed the GRN score, that identifies transcriptional regulators of cell-state transitions based on their status of activation and their predicted target genes in a reconstructed gene regulatory network.: [evidence span withheld — anchor status: not_found]
- [discussion] GRN integrated with trajectories and scoring schemes: "CellRouter addresses many of these challenges by integrating GRNs with multi-state transition trajectories and scoring schemes to capture transition-specific gene expression"
- [methods] GRN reconstruction step: "a gene regulatory network is reconstructed to identify putative gene regulatory relationships"
- [methods] Regulator identification goal: "identification of regulators driving or mediating cell-fate transitions at the gene and"
- [readme] README overview of GRN step preceding trajectory and regulator identification: "a gene regulatory network is reconstructed to identify putative gene regulatory relationships. Step (2) Subpopulations are then identified by a combination of learning the structure of the data"
