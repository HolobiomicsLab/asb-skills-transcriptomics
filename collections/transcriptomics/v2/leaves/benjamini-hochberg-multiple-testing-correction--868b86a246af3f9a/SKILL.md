---
name: benjamini-hochberg-multiple-testing-correction
description: Use when many hypothesis tests (e.g., GSEA on 22 KEGG-derived LR pathway
  sets per factor) produce raw P-values per condition, and FDR-controlled adjusted
  P-values are needed for a factor-by-pathway results table.
when_to_use_negative:
- Only one hypothesis test (single pathway set, single factor) is performed — no correction
  is needed
- GSEA P-values have already been FDR-adjusted or the GSEA tool already reports q-values
  you intend to use
- Input is enrichment scores (NES) without raw P-values — BH correction operates on
  nominal P-values, not on effect-size statistics
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0602
tools:
- name: Tensor-cell2cell
  role: Produces the 6-factor LR-pair loadings whose per-factor GSEA P-values are
    BH-corrected
  repo: https://github.com/earmingol/cell2cell
- name: GSEA
  role: Pre-ranked GSEA over KEGG-derived LR pathway sets generates the nominal P-values
    that Benjamini-Hochberg correction adjusts
- name: cell2cell
  role: Python suite containing downstream-analysis pipelines for factor interpretation,
    including enrichment workflows whose outputs are corrected
  repo: https://github.com/earmingol/cell2cell
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:ecfe86dfb8b3a9c87416a867c2226266edc383d168ff45e90acbf47590852651
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - GSEA
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg false-discovery-rate correction to raw P-values from pathway-level GSEA of Tensor-cell2cell factor loadings, yielding adjusted P-values that control the expected fraction of false positives across the multiple KEGG-derived LR pathway sets tested per factor.

## When to use

Use when many hypothesis tests (e.g., GSEA on 22 KEGG-derived LR pathway sets per factor) produce raw P-values per condition, and FDR-controlled adjusted P-values are needed for a factor-by-pathway results table.

## When NOT to use

- Only one hypothesis test (single pathway set, single factor) is performed — no correction is needed
- GSEA P-values have already been FDR-adjusted or the GSEA tool already reports q-values you intend to use
- Input is enrichment scores (NES) without raw P-values — BH correction operates on nominal P-values, not on effect-size statistics

## Inputs

- Raw GSEA P-values from pre-ranked GSEA of one factor's LR-pair loading list against 22 KEGG-derived LR pathway sets
- Normalized enrichment scores (NES) for each factor x pathway set pair

## Outputs

- BH-adjusted P-values within [0, 1] for each of the 6 factors x 22 KEGG-derived pathway sets
- Results table gsea_nes_pvalues.csv with NES and adjusted P-value entries for all 132 factor-pathway combinations

## How to apply

For each of the six ASD factors from the Tensor-cell2cell decomposition, rank ligand-receptor pairs in descending loading value and run pre-ranked GSEA (e.g., gseapy prerank) against the 22 KEGG-derived LR pathway gene sets with min set size 15 LR pairs and 999 permutations. Collect the raw GSEA P-values of all pathway sets within a single factor and apply Benjamini-Hochberg correction across pathway sets WITHIN each factor (not across all 6 x 22 tests at once), so each factor's multiple-testing burden is corrected independently. Assemble the corrected NES and adjusted P-values into a 6 factors x 22 pathway sets table saved as gsea_nes_pvalues.csv. The rationale is that each factor's enrichment scan tests 22 pathway sets simultaneously, so uncorrected P-values would inflate false-positive pathway calls.

## Related tools

- **Tensor-cell2cell** (Produces the 6-factor LR-pair loadings whose per-factor GSEA P-values are BH-corrected) — https://github.com/earmingol/cell2cell
- **GSEA** (Pre-ranked GSEA over KEGG-derived LR pathway sets generates the nominal P-values that Benjamini-Hochberg correction adjusts)
- **cell2cell** (Python suite containing downstream-analysis pipelines for factor interpretation, including enrichment workflows whose outputs are corrected) — https://github.com/earmingol/cell2cell

## Examples

```
gseapy.prerank(rnk=rnk_df, gene_sets='kegg_lr_sets.gmt', min_size=15, permutation_num=999); adj_p = statsmodels.stats.multitest.multipletests(res.res2d['pval'], method='fdr_bh')[1]
```

## Evaluation signals

- Output table contains a NES and adjusted P-value entry for each of the 6 factors x 22 KEGG-derived pathway sets (132 rows)
- All adjusted P-values fall within [0, 1]
- Every adjusted P-value is greater than or equal to its corresponding raw GSEA P-value
- Adjusted P-values are monotonically consistent with the BH procedure (rank-based step-up) computed within each factor separately, not pooled across factors
- Factors retain distinct biological-function associations (axon guidance, cell adhesion, ECM-receptor interaction, ERBB, MAPK signaling) after correction, matching the reported finding

## Limitations

- Correction is applied across pathway sets within each factor, so power differs per factor and results are not comparable to corrections pooled across all factors
- Only cell types shared across all samples were included in the ASD communication tensor, potentially missing rare cell types whose LR pairs would otherwise enter the ranked lists
- Benchmarking/enrichment conclusions rest on KEGG-derived pathway sets (min 15 LR pairs); pathway sets smaller than the minimum-size threshold are silently excluded from testing

## Evidence

- [full_text] Apply Benjamini-Hochberg correction to the GSEA P-values across pathway sets within each factor.: [evidence span withheld — anchor status: not_found]
- [discussion] GSEA listed as downstream analysis of Tensor-cell2cell factors: "such analyses included GSEA, clustering, visualization and statistical comparison of factors"
- [methods] Downstream analyses of factors in cell2cell: "cell2cell.analysis : Includes simplified pipelines for running the analyses, and functions for downstream analyses of Tensor-cell2cell"
- [full_text] Distinct biological functions per ASD factor after enrichment: [evidence span withheld — anchor status: not_found]
