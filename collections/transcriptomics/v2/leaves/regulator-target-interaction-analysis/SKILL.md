---
name: regulator-target-interaction-analysis
description: Use when single-cell trajectory progressions and a reconstructed GRN
  edge list are available and you need to identify transcriptional regulators driving
  or mediating specific cell-fate transitions at the network level.
when_to_use_negative:
- The analysis goal is trajectory reconstruction itself (subpopulation identification,
  flow-network paths) rather than scoring regulators — CellRouter's trajectory and
  GRN-scoring steps are separate workflow stages
- No reconstructed gene regulatory network exists for the dataset; the score requires
  predicted target genes per regulator, not expression data alone
- The goal is general single-cell analysis with diffusion pseudotime (e.g. the theislab/scAnalysisTutorial
  approach), which does not use GRN-based regulator scoring
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellRouter
  role: Platform that reconstructs the gene regulatory network and multi-state trajectories
    whose progressions feed the GRN score; its R class (CellRouter_Class.R with libdir
    pointing to Java libraries) is the environment in which regulators driving or
    mediating cell-fate transitions are identified at the gene and network level
  repo: https://github.com/edroaldo/cellrouter
- name: R
  role: Language used to compute Spearman correlations between regulator/target-gene
    expression and trajectory progression, form the multiplicative GRN score, and
    write regulator_scores.csv; depends on packages such as reshape, reshape2, pheatmap,
    tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd
- name: igraph
  role: R package dependency installed as part of the CellRouter environment, supporting
    the network/graph operations underlying the gene regulatory network
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:5f7710e423535e9e45ab1d0a9b7de6c0c1ae82d7063c7160aed75551697eb24f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - R
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# regulator-target-interaction-analysis

## Summary

CellRouter's GRN score ranks transcriptional regulators of cell-state transitions by combining each regulator's activation status (Spearman correlation with trajectory progression), the mean correlation of its predicted target genes, and its number of regulated targets in a reconstructed gene regulatory network.

## When to use

Use when single-cell trajectory progressions and a reconstructed GRN edge list are available and you need to identify transcriptional regulators driving or mediating specific cell-fate transitions at the network level.

## When NOT to use

- The analysis goal is trajectory reconstruction itself (subpopulation identification, flow-network paths) rather than scoring regulators — CellRouter's trajectory and GRN-scoring steps are separate workflow stages
- No reconstructed gene regulatory network exists for the dataset; the score requires predicted target genes per regulator, not expression data alone
- The goal is general single-cell analysis with diffusion pseudotime (e.g. the theislab/scAnalysisTutorial approach), which does not use GRN-based regulator scoring

## Inputs

- Reconstructed gene regulatory network edge list (CLR/CellNet-style) listing regulator-target relationships
- Trajectory progression data for each cell-fate transition identified by CellRouter
- Regulator and target gene expression values from the single-cell RNA-seq dataset

## Outputs

- regulator_scores.csv: one row per transcriptional regulator, with transition-specific GRN scores (one score column per trajectory j)
- Ranked list of regulators driving or mediating cell-fate transitions

## How to apply

Load the CLR/CellNet-style reconstructed gene regulatory network edge list together with trajectory progression data for each cell-fate transition. For every regulator i and trajectory j, compute the Spearman correlation c(i,j) between the regulator's expression and the trajectory progression in R. For each regulator, compute the mean correlation m(t,j) of its predicted target genes with the trajectory progression and count the number of regulated targets n(j). Combine the three components multiplicatively as GRN(i,j) = c(i,j) * m(t,j) * n(j) to obtain a transition-specific score, then assemble all regulator scores into a table (regulator_scores.csv) for downstream identification of regulators driving or mediating cell-fate transitions. The rationale is that CellRouter integrates GRNs with multi-state transition trajectories and scoring schemes to capture transition-specific gene expression and putative regulatory interactions.

## Related tools

- **CellRouter** (Platform that reconstructs the gene regulatory network and multi-state trajectories whose progressions feed the GRN score; its R class (CellRouter_Class.R with libdir pointing to Java libraries) is the environment in which regulators driving or mediating cell-fate transitions are identified at the gene and network level) — https://github.com/edroaldo/cellrouter
- **R** (Language used to compute Spearman correlations between regulator/target-gene expression and trajectory progression, form the multiplicative GRN score, and write regulator_scores.csv; depends on packages such as reshape, reshape2, pheatmap, tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd)
- **igraph** (R package dependency installed as part of the CellRouter environment, supporting the network/graph operations underlying the gene regulatory network)

## Examples

```
GRN(i,j) <- c_reg[i,j] * m_targets[i,j] * n_targets[i,j]; write.csv(regulator_scores, 'regulator_scores.csv')
```

## Evaluation signals

- regulator_scores.csv exists and parses as a valid CSV with one row per regulator and one score column per cell-fate trajectory
- All scores are finite numeric values and equal the product of the three components: GRN(i,j) = c(i,j) * m(t,j) * n(j)
- Every regulator scored has at least one predicted target gene in the reconstructed GRN edge list (n(j) >= 1)
- Spearman correlations c(i,j) and m(t,j) fall within [-1, 1]; sign convention (activation vs. repression along the trajectory) is consistent across regulators
- Ranking of top regulators is transition-specific — the same regulator should not identically top every trajectory unless its expression and targets are similarly correlated with all trajectories

## Limitations

- Predicted regulatory relationships are putative — the GRN is reconstructed to identify putative gene regulatory relationships, so scores reflect inferred, not validated, targets
- Scores are trajectory-specific; a regulator may rank highly only for particular cell-fate transitions (e.g. HSPC differentiation to erythroid, myeloid and lymphoid lineages) and not generalize
- The method captures transition-specific gene expression and putative regulatory interactions but does not by itself validate regulatory function; CellRouter's authors defer functional characterization of identified genes to future work

## Evidence

- [methods] Identification of regulators driving or mediating cell-fate transitions: "Identification of genes implicated in the dynamic biological process under study, such as differentiation, and identification of regulators driving or mediating cell-fate transitions at the gene and"
- [methods] GRN reconstruction provides the predicted targets: "a gene regulatory network is reconstructed to identify putative gene regulatory relationships"
- [discussion] Integration of GRNs with trajectories and scoring schemes: "CellRouter addresses many of these challenges by integrating GRNs with multi-state transition trajectories and scoring schemes to capture transition-specific gene expression"
- [readme] GRNs give new insights into cell-state transitions: "CellRouter integrates subpopulation identification, multi-state trajectories, and gene regulatory networks (GRNs) to provide new insights into cell-state transitions during lineage diversification"
