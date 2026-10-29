---
name: single-cell-transition-regulator-identification
description: Use when single-cell trajectory data (e.g., CellRouter multi-state trajectories)
  and a CLR/CellNet-style GRN edge list are available and the goal is to identify
  transcriptional regulators driving or mediating specific cell-fate transitions.
when_to_use_negative:
- No reconstructed gene regulatory network is available — the score requires predicted
  regulator-target edges and cannot be computed from expression data alone
- The goal is trajectory reconstruction itself (subpopulation identification, multi-state
  trajectory detection) rather than downstream regulator identification
- Data are bulk or lack trajectory progression information for the transitions of
  interest, so per-cell Spearman correlations with progression cannot be computed
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellRouter
  role: Platform that reconstructs multi-state transition trajectories and gene regulatory
    networks used as the basis for the GRN score
  repo: https://github.com/edroaldo/cellrouter
- name: R
  role: Language in which the Spearman correlations, target-gene mean correlations,
    and multiplicative GRN score are computed
- name: igraph
  role: R package installed as a CellRouter dependency, used for network/community-detection
    operations underlying the subpopulation and GRN steps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:9dedce404568f945b410939ae7285a8081320219bb1b7f4d232d984c1914327e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-transition-regulator-identification

## Summary

Reconstructs CellRouter's GRN score, a multiplicative scoring scheme that ranks transcriptional regulators of cell-state transitions by combining the regulator's expression correlation with trajectory progression, the mean correlation of its predicted target genes, and the number of regulated targets in a reconstructed gene regulatory network.

## When to use

Use when single-cell trajectory data (e.g., CellRouter multi-state trajectories) and a CLR/CellNet-style GRN edge list are available and the goal is to identify transcriptional regulators driving or mediating specific cell-fate transitions.

## When NOT to use

- No reconstructed gene regulatory network is available — the score requires predicted regulator-target edges and cannot be computed from expression data alone
- The goal is trajectory reconstruction itself (subpopulation identification, multi-state trajectory detection) rather than downstream regulator identification
- Data are bulk or lack trajectory progression information for the transitions of interest, so per-cell Spearman correlations with progression cannot be computed

## Inputs

- Gene regulatory network edge list reconstructed in CellRouter (CLR/CellNet style), mapping regulators to predicted target genes
- Trajectory progression data for each cell-fate transition from CellRouter multi-state trajectories
- Single-cell expression matrix (e.g., normalized scRNA-seq counts, such as the Paul et al. 2015 mouse myeloid progenitor dataset)

## Outputs

- regulator_scores.csv — one row per transcriptional regulator with transition-specific GRN scores
- Ranked list of regulators driving or mediating each cell-fate transition

## How to apply

Load the CLR/CellNet-style reconstructed gene regulatory network edge list together with the trajectory progression data for each cell-fate transition. For every regulator i and trajectory j, compute in R the Spearman correlation c(i,j) between the regulator's expression and the trajectory progression. Then compute the mean correlation m(t,j) of the regulator's predicted target genes with the trajectory progression and count the number of regulated targets n(j). Combine the three components multiplicatively as GRN(i,j) = c(i,j) * m(t,j) * n(j) to obtain a transition-specific score per regulator, then assemble all regulator scores into regulator_scores.csv. The rationale, from the CellRouter paper, is that integrating GRNs with multi-state transition trajectories and scoring schemes captures transition-specific gene expression and putative regulatory interactions that drive or mediate cell-fate transitions.

## Related tools

- **CellRouter** (Platform that reconstructs multi-state transition trajectories and gene regulatory networks used as the basis for the GRN score) — https://github.com/edroaldo/cellrouter
- **R** (Language in which the Spearman correlations, target-gene mean correlations, and multiplicative GRN score are computed)
- **igraph** (R package installed as a CellRouter dependency, used for network/community-detection operations underlying the subpopulation and GRN steps)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- regulator_scores.csv exists and parses as a valid CSV with one row per regulator and score columns per trajectory
- All score entries are finite numeric values
- Each GRN(i,j) value equals the product of its three components: regulator-expression Spearman correlation c(i,j) × mean target-gene correlation m(t,j) × target count n(j)
- Scores are transition-specific: the same regulator receives distinct scores for different cell-fate transitions
- Top-ranked regulators are biologically consistent with known drivers of the studied transition (e.g., lineage-regulating transcription factors in HSPC differentiation)

## Limitations

- The GRN score relies on a reconstructed gene regulatory network of putative regulatory relationships, so regulator-target assignments are predictions, not validated interactions
- Spearman correlation with trajectory progression captures association, not causation; functional validation of identified regulators is a separate step (functional characterization of identified genes was noted as beyond the scope of the manuscript)
- CellRouter was tested on Unix-based systems (or Mac) and not on Windows; R dependencies must be installed (reshape, reshape2, pheatmap, tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd, Vennerable)

## Evidence

- [other] CellRouter employs a scoring scheme, termed the GRN score, that identifies transcriptional regulators of cell-state transitions based on their status of activation and their predicted target genes in a reconstructed gene regulatory network.: [evidence span withheld — anchor status: not_found]
- [other] Combine the three components multiplicatively as GRN(i,j) = c(i,j) * m(t,j) * n(j) to obtain the transition-specific score for each regulator.: [evidence span withheld — anchor status: not_found]
- [discussion] CellRouter addresses many of these challenges by integrating GRNs with multi-state transition trajectories and scoring schemes to capture transition-specific gene expression and putative regulatory interactions.: "integrating GRNs with multi-state transition trajectories and scoring schemes to capture transition-specific gene expression and putative regulatory interactions"
- [readme] Identification of genes implicated in the dynamic biological process under study, such as differentiation, and identification of regulators driving or mediating cell-fate transitions at the gene and network level.: "identification of regulators driving or mediating cell-fate transitions at the gene and network level"
- [methods] list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd'): "list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd')"
