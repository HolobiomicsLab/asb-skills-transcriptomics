---
name: ligand-receptor-pair-ranking
description: Use when you have factor loadings for ligand-receptor pairs from a Tensor-cell2cell
  decomposition and need to rank them (descending by loading value) to identify top
  communication pairs or feed factor-specific pre-ranked lists into GSEA.
when_to_use_negative:
- The input is raw per-context communication scores not yet factorized — build and
  decompose the 4D-communication tensor with Tensor-cell2cell first
- You need pathway-level scores rather than LR-pair rankings — tools like CellChat
  generate scores at the signaling pathway level instead of LR pairs
tools:
- name: cell2cell
  role: Provides the factor loadings and downstream analysis pipelines (cell2cell.analysis)
    used to rank LR pairs and interpret Tensor-cell2cell factors
  repo: https://github.com/earmingol/cell2cell
- name: GSEA
  role: PreRanked GSEA run on the descending-sorted LR-pair lists per factor, with
    999 permutations, min set size 15, and BH correction
- name: CellChat
  role: Source of the Human-2020-Jin ligand-receptor pair list used to define the
    LR pairs in the communication tensor
  repo: https://github.com/sqjin/CellChat
- name: Ligand-Receptor Pairs curated list
  role: Compendium repository of literature-curated ligand-receptor pair lists used
    as analysis inputs
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:b0b36d7c86292825e71a2698f52130c4386044cf3ee0edafaac3b116f1b6ad10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - GSEA
  - Ligand-Receptor Pairs curated list
  - cell2cell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ligand-receptor-pair-ranking

## Summary

Ranks ligand-receptor pairs by their loadings from Tensor-cell2cell factor decompositions (e.g., the 6-factor ASD or 10-factor BALF COVID-19 decompositions) to produce pre-ranked lists for downstream analyses such as factor-specific PreRanked GSEA against KEGG-derived LR pathway sets.

## When to use

Use when you have factor loadings for ligand-receptor pairs from a Tensor-cell2cell decomposition and need to rank them (descending by loading value) to identify top communication pairs or feed factor-specific pre-ranked lists into GSEA.

## When NOT to use

- The input is raw per-context communication scores not yet factorized — build and decompose the 4D-communication tensor with Tensor-cell2cell first
- You need pathway-level scores rather than LR-pair rankings — tools like CellChat generate scores at the signaling pathway level instead of LR pairs

## Inputs

- LR-pair loadings per factor from a Tensor-cell2cell decomposition (e.g., 6 ASD factors or 10 BALF COVID-19 factors)
- 22 KEGG-derived LR pathway gene sets (GMT format)
- Optional: curated LR pair lists such as Human-2020-Jin-LR-pairs.csv from CellChat (Jin et al. 2021)

## Outputs

- Pre-ranked LR-pair lists (one per factor, sorted descending by loading)
- Results table of NES and Benjamini-Hochberg-adjusted P-values for all factors x pathway sets (e.g., gsea_nes_pvalues.csv)
- Ranking of top ligand-receptor pairs per factor (e.g., APP/MDK/MIF/CD99, CCL2/CCL3/CCL8-CCR1/CCR2/CCR5, MIF-CD74/CD44)

## How to apply

Load the LR-pair loadings for each factor of the communication tensor decomposition (e.g., the six ASD factors or ten BALF COVID-19 factors). For each factor, sort LR pairs in descending order of loading value to build one pre-ranked list per factor. For functional interpretation, run PreRanked GSEA (e.g., gseapy prerank) per factor using KEGG-derived LR pathway gene sets with min set size 15 LR pairs and 999 permutations, then apply Benjamini-Hochberg correction to the P-values within each factor. For communication interpretation (e.g., the BALF severity factor), inspect the highest-loading entries to confirm top-ranked pairs such as APP/MDK/MIF/CD99, CCL2/CCL3/CCL8 with CCR1/CCR2/CCR5, and MIF-CD74/CD44. Curated LR pair lists (e.g., Human-2020-Jin from CellChat) are available at the LewisLabUCSD Ligand-Receptor-Pairs repository.

## Related tools

- **cell2cell** (Provides the factor loadings and downstream analysis pipelines (cell2cell.analysis) used to rank LR pairs and interpret Tensor-cell2cell factors) — https://github.com/earmingol/cell2cell
- **GSEA** (PreRanked GSEA run on the descending-sorted LR-pair lists per factor, with 999 permutations, min set size 15, and BH correction)
- **CellChat** (Source of the Human-2020-Jin ligand-receptor pair list used to define the LR pairs in the communication tensor) — https://github.com/sqjin/CellChat
- **Ligand-Receptor Pairs curated list** (Compendium repository of literature-curated ligand-receptor pair lists used as analysis inputs) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
import gseapy; gseapy.prank(rnk=ranked_lr_list, gene_sets='kegg_lr_sets.gmt', permutation_num=999, min_size=15, outdir=None)
```

## Evaluation signals

- GSEA output table contains a NES and adjusted P-value entry for each of the 6 factors x 22 KEGG-derived pathway sets
- All adjusted P-values fall within [0, 1] after Benjamini-Hochberg correction
- Top-ranked LR pairs by loading match expectations: APP/MDK/MIF/CD99, CCL2/CCL3/CCL8 with CCR1/CCR2/CCR5, and MIF-CD74/CD44 appear among the highest-loading entries
- Each pre-ranked list contains all LR pairs of the factor sorted strictly in descending loading order
- Distinct enriched functions emerge per factor (axon guidance, cell adhesion, ECM-receptor interaction, ERBB signaling, MAPK signaling) rather than identical sets across factors

## Limitations

- Only cell types shared across all samples were included in the COVID-19 and ASD case-study tensors, potentially missing rare cell types in the loadings
- The implemented tensor decomposition algorithm is not optimized for missing values, so LR pairs or contexts absent in some contexts cannot be fully represented
- Benchmarking and validating ranked communication predictions is challenging due to the lack of a ground truth

## Evidence

- [discussion] GSEA prerank setup: "such analyses included GSEA, clustering, visualization and statistical comparison of factors"
- [methods] cell2cell analysis module: "cell2cell.analysis : Includes simplified pipelines for running the analyses, and functions for downstream analyses of Tensor-cell2cell"
- [results] LR pair list source: "we use a list of LR pairs published with CellChat (Jin et al. 2021, Nature Communications)"
- [readme] LR pairs compendium: [evidence span withheld — anchor status: external_doc]
