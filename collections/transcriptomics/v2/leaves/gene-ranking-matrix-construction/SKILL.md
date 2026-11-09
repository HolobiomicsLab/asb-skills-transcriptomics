---
name: gene-ranking-matrix-construction
description: Use when a normalized UMI matrix from scRNA-seq data (e.g., SingCellaR
  Human_HSPC) must be converted into AUCell rankings to score lineage signature gene
  sets loaded from a GMT file.
when_to_use_negative:
- The signature gene sets for your system are not available or curated — the protocol
  notes the user would have to compile and curate customized gene sets; only the 75
  hematopoiesis sets are provided
- You only need differential expression between clusters (identifyDifferentialGenes)
  rather than enrichment-based per-cell signature scoring
- The object has not yet been normalized (normalize_UMIs not run) — rankings should
  be built on the normalized UMI matrix
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingCellaR
  role: R package whose Build_AUCell_Rankings / Build_AUCell_Rankings_Fast functions
    construct the per-cell gene ranking matrix consumed by Run_AUCell
  repo: github.com/supatt-lab/SingCellaR
- name: AUCell
  role: Underlying ranking/enrichment engine wrapped by SingCellaR for building rankings
    and computing AUC gene-set scores
- name: R
  role: Environment in which the SingCellaR AUCell ranking workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:b1eac8040cc15244dbd8a82a7c6ce4ad047a65cc021d86b429bc361761b8ca0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AUCell
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-ranking-matrix-construction

## Summary

Builds per-cell gene ranking matrices for AUCell-based signature scoring within SingCellaR, ranking genes by expression in each cell of the normalized UMI matrix so that gene-set enrichment (AUC) scores can be computed downstream for cell-type annotation.

## When to use

Use when a normalized UMI matrix from scRNA-seq data (e.g., SingCellaR Human_HSPC) must be converted into AUCell rankings to score lineage signature gene sets loaded from a GMT file.

## When NOT to use

- The signature gene sets for your system are not available or curated — the protocol notes the user would have to compile and curate customized gene sets; only the 75 hematopoiesis sets are provided
- You only need differential expression between clusters (identifyDifferentialGenes) rather than enrichment-based per-cell signature scoring
- The object has not yet been normalized (normalize_UMIs not run) — rankings should be built on the normalized UMI matrix

## Inputs

- SingCellaR object containing the normalized UMI matrix (from normalize_UMIs / get_normalized_umi)
- GMT file of lineage signature gene sets (75 pre-curated hematopoiesis gene sets in the protocol)
- Rankings output file path (AUCell_buildRankings.file)
- nCores, plotStats, splitByBlocks parameters (or block.cell.size for the fast variant)

## Outputs

- Cached AUCell rankings file (per-cell gene ranking matrix)
- Per-cell gene-set AUC enrichment score table (via Run_AUCell)
- Cell-type annotations retrievable with get_cells_annotation(object)

## How to apply

Load the integrated object with its normalized UMI matrix (retrieved via get_normalized_umi) into SingCellaR in R, and read the lineage signature sets with get_gmtGeneSets(gmt.file), confirming the expected 75 pre-curated hematopoiesis gene sets with listAvailableGeneSets(gmt.file). Build the rankings with Build_AUCell_Rankings(object, AUCell_buildRankings.file, nCores, plotStats, splitByBlocks), caching them to a rankings file that Run_AUCell consumes together with the GMT. The rationale is that AUCell ranks genes per cell rather than using raw expression, making enrichment robust to scaling differences across cells. For large-scale datasets, substitute Build_AUCell_Rankings_Fast with an explicit block.cell.size to avoid the running-time and memory issues documented for AUCell_buildRankings. Validate by checking that the rankings file and AUCell score table exist and cover all 75 gene sets before applying gene-set-specific AUC cutoffs for cell-type labeling.

## Related tools

- **SingCellaR** (R package whose Build_AUCell_Rankings / Build_AUCell_Rankings_Fast functions construct the per-cell gene ranking matrix consumed by Run_AUCell) — github.com/supatt-lab/SingCellaR
- **AUCell** (Underlying ranking/enrichment engine wrapped by SingCellaR for building rankings and computing AUC gene-set scores)
- **R** (Environment in which the SingCellaR AUCell ranking workflow is executed)

## Examples

```
Build_AUCell_Rankings(object, AUCell_buildRankings.file, nCores, plotStats, splitByBlocks); Run_AUCell(object, AUCell_buildRankings.file, geneSets.gmt.file, n.random.genes)
```

## Evaluation signals

- Rankings file and AUCell score table exist after the run and contain all 75 curated gene sets
- listAvailableGeneSets(gmt.file) confirms the expected gene-set names before Run_AUCell
- UMAP plot of cell types (plot_cells_annotation) renders cells colored by lineage annotations as in the protocol's Figure 7A
- HSC/MPP signature shows uniform enrichment across the majority of cells in cluster 1 of the Human_HSPC dataset
- For large datasets, the fast variant completes without the running-time and memory failures reported for AUCell_buildRankings

## Limitations

- AUCell_buildRankings has documented running-time and memory issues on large-scale datasets; use Build_AUCell_Rankings_Fast with block.cell.size
- Only 75 pre-curated hematopoiesis gene sets are provided; other systems require the user to compile and curate customized gene sets
- Plot rotations may differ from the protocol due to software version and seed number differences

## Evidence

- [discussion] running time and memory issues when performing the AUCell_buildRankings function for a large-scale dataset: "running time and memory issues when performing the AUCell_buildRankings function for a large-scale dataset."
- [discussion] SingCellaR requires signature gene sets to perform the cell type annotation analysis.: "SingCellaR requires signature gene sets to perform the cell type annotation analysis."
- [discussion] 75 gene sets curated from previous studies relevant to hematopoiesis: "the user would have to compile and curate customized gene sets for the relevant system of interest. In this protocol, we provide 75 gene sets curated from previous studies relevant to hematopoiesis."
- [methods] Build rankings and read GMT sets from the protocol workflow: [evidence span withheld — anchor status: not_found]
- [readme] Readme description of SingCellaR: "an integrative analysis tool for analysing single-cell RNA sequencing data"
