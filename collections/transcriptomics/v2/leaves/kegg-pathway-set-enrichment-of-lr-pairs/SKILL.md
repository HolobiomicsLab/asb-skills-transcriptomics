---
name: kegg-pathway-set-enrichment-of-lr-pairs
description: Use when you have factor loadings for ligand-receptor pairs from a Tensor-cell2cell
  decomposition (e.g., the 6 ASD factors) and need to identify which KEGG-derived
  biological pathways each factor is enriched for.
when_to_use_negative:
- You need pathway-level communication scores directly rather than enrichment over
  factor loadings — the paper notes tools like CellChat already generate scores at
  the signaling pathway level instead of LR pairs.
- The tensor decomposition produced factors from a context different from the ASD
  dataset and no KEGG-derived LR pathway sets matching that organism/tissue exist.
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tensor-cell2cell
  role: Source of the factor loadings over LR pairs that are ranked and tested by
    GSEA; its ASD case-study results were coupled with GSEA for interpretation
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Suite containing Tensor-cell2cell and simplified pipelines plus functions
    for downstream analyses of the factors
  repo: https://github.com/earmingol/cell2cell
- name: GSEA
  role: Preranked gene set enrichment engine run per factor against the 22 KEGG-derived
    LR pathway sets
- name: CellChat
  role: Reference tool whose Jin et al. 2021 LR-pair list (Human-2020-Jin-LR-pairs.csv)
    underlies the communication scores decomposed by Tensor-cell2cell
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:2be221b0de08cbd1d611bffba7f6ed7c3451e635507a32b3d49bc0f1acaa4738
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - GSEA
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# kegg-pathway-set-enrichment-of-lr-pairs

## Summary

Factor-specific preranked GSEA that maps Tensor-cell2cell ligand-receptor pair loadings from the 6-factor ASD communication tensor onto 22 KEGG-derived LR pathway sets, yielding NES and BH-adjusted P-values that assign biological functions (axon guidance, cell adhesion, ECM-receptor interaction, ERBB and MAPK signaling) to each ASD factor.

## When to use

Use when you have factor loadings for ligand-receptor pairs from a Tensor-cell2cell decomposition (e.g., the 6 ASD factors) and need to identify which KEGG-derived biological pathways each factor is enriched for.

## When NOT to use

- You need pathway-level communication scores directly rather than enrichment over factor loadings — the paper notes tools like CellChat already generate scores at the signaling pathway level instead of LR pairs.
- The tensor decomposition produced factors from a context different from the ASD dataset and no KEGG-derived LR pathway sets matching that organism/tissue exist.

## Inputs

- LR-pair loadings from each of the 6 factors of the Tensor-cell2cell decomposition of the ASD communication tensor (ranked lists of LR pairs per factor)
- 22 KEGG-derived LR pathway sets (gene set file, e.g., GMT format)

## Outputs

- gsea_nes_pvalues.csv — results table of NES and BH-adjusted P-values for all 6 factors x 22 KEGG-derived pathway sets
- Factor-to-biological-function assignments (e.g., axon guidance, cell adhesion, ECM-receptor interaction, ERBB signaling, MAPK signaling)

## How to apply

Load the LR-pair loadings from each of the 6 factors of the Tensor-cell2cell decomposition of the ASD communication tensor and rank the LR pairs in descending order of loading value to build 6 pre-ranked lists. Load the 22 KEGG-derived LR pathway sets as gene sets and run PreRanked GSEA (e.g., GSEA/gseapy prerank) separately for each factor, using min set size 15 LR pairs and 999 permutations. Apply Benjamini-Hochberg correction to the GSEA P-values across pathway sets within each factor. This is a downstream interpretation step: the paper notes that Tensor-cell2cell results 'can be coupled with downstream analysis methods to facilitate interpretation', and in the ASD case study these included GSEA on factor-associated LR pairs. Assemble NES and adjusted P-values for all 6 factors x 22 pathway sets and save the results table for interpretation of factor-specific biology.

## Related tools

- **Tensor-cell2cell** (Source of the factor loadings over LR pairs that are ranked and tested by GSEA; its ASD case-study results were coupled with GSEA for interpretation) — https://github.com/earmingol/cell2cell
- **cell2cell** (Suite containing Tensor-cell2cell and simplified pipelines plus functions for downstream analyses of the factors) — https://github.com/earmingol/cell2cell
- **GSEA** (Preranked gene set enrichment engine run per factor against the 22 KEGG-derived LR pathway sets)
- **CellChat** (Reference tool whose Jin et al. 2021 LR-pair list (Human-2020-Jin-LR-pairs.csv) underlies the communication scores decomposed by Tensor-cell2cell) — https://github.com/sqjin/CellChat

## Examples

```
import gseapy; gseapy.prerank(rnk='factor1_lrs_ranked.csv', gene_sets='kegg_lr_pathways.gmt', min_size=15, permutation_num=999, outdir='gsea_factor1')
```

## Evaluation signals

- The output table gsea_nes_pvalues.csv contains a NES and adjusted P-value entry for every combination of 6 factors x 22 KEGG-derived pathway sets (132 rows/cells).
- All adjusted P-values fall within [0, 1].
- Each of the 6 pre-ranked LR lists is sorted in descending order of factor loading value before running prerank GSEA.
- Only pathway sets with at least 15 LR pairs (min set size 15) are tested; results derive from 999 permutations with Benjamini-Hochberg correction applied within each factor.
- Reproduced analyses recover distinct factor-specific functions (axon guidance, cell adhesion, ECM-receptor interaction, ERBB signaling, MAPK signaling) rather than all factors pointing to the same pathway.

## Limitations

- The ASD communication tensor included only cell types shared across all samples, so rare cell types and their LR-pair contributions are absent from the loadings being enriched.
- The implemented tensor decomposition algorithm is not optimized for missing values in the communication tensor, which limits which LR pairs can enter the ranking.
- Benchmarking/validating CCC-derived enrichment is challenging due to the lack of ground truth, so enriched functions are interpretive rather than validated interactions.

## Evidence

- [discussion] Tensor-cell2cell results can be coupled with downstream analysis methods to facilitate interpretation and provide further insights of underlying mechanisms.: "Tensor-cell2cell results can be coupled with downstream analysis methods to facilitate interpretation and provide further insights of underlying mechanisms."
- [discussion] ASD case-study downstream analyses include GSEA: "In our ASD case-study (Fig. 5), such analyses included GSEA, clustering, visualization and statistical comparison of factors"
- [methods] cell2cell package provides downstream analysis functions: "cell2cell.analysis : Includes simplified pipelines for running the analyses, and functions for downstream analyses of Tensor-cell2cell"
- [methods] Curated LR pair lists available from the authors' repository: "Find a curated list of ligand-receptor pairs for your analyses at our GitHub Repository (https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs)"
- [readme] cell2cell installation via pip: [evidence span withheld — anchor status: not_found]
