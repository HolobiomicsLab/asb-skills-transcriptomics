---
name: preranked-gene-set-enrichment-analysis
description: Use when factor loadings from a Tensor-cell2cell decomposition (e.g.,
  6 ASD factors) need functional interpretation by ranking LR pairs and testing enrichment
  of pathway-level LR gene sets.
when_to_use_negative:
- The input is not a ranked list of LR pairs ordered by factor loadings (e.g., you
  only have raw expression counts without a Tensor-cell2cell decomposition)
- The pathway gene sets contain fewer than 15 LR pairs after intersecting with the
  ranked list, below the min set size cutoff
- The analysis targets a single context/sample rather than factor-specific loadings
  from a context-deconvolved communication tensor
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3307
tools:
- name: Tensor-cell2cell
  role: Source of the factor-specific LR-pair loadings that are ranked as pre-ranked
    GSEA inputs
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Suite providing downstream analysis functions for Tensor-cell2cell factors
    feeding into the GSEA step
  repo: https://github.com/earmingol/cell2cell
- name: GSEA
  role: Pre-ranked enrichment engine (or gseapy prerank equivalent) run per factor
    against KEGG-derived LR pathway sets
- name: CellChat
  role: Alternative LR-pair source for communication scores that can build the tensor
    whose loadings are enriched
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:43d0b6465aaa7ae4d9abb93d9238f724b73384c08c1020e643b85d4b4285818c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - GSEA
  - Tensor-cell2cell
  - cell2cell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# preranked-gene-set-enrichment-analysis

## Summary

Runs pre-ranked GSEA on ligand-receptor pair lists ordered by their loadings from each Tensor-cell2cell factor, against KEGG-derived LR pathway sets, to assign biological functions (e.g., axon guidance, ERBB, MAPK signaling) to individual ASD communication factors.

## When to use

Use when factor loadings from a Tensor-cell2cell decomposition (e.g., 6 ASD factors) need functional interpretation by ranking LR pairs and testing enrichment of pathway-level LR gene sets.

## When NOT to use

- The input is not a ranked list of LR pairs ordered by factor loadings (e.g., you only have raw expression counts without a Tensor-cell2cell decomposition)
- The pathway gene sets contain fewer than 15 LR pairs after intersecting with the ranked list, below the min set size cutoff
- The analysis targets a single context/sample rather than factor-specific loadings from a context-deconvolved communication tensor

## Inputs

- LR-pair loadings matrix from the 6 factors of a Tensor-cell2cell decomposition of the ASD communication tensor
- 22 KEGG-derived LR pathway sets used as gene sets

## Outputs

- gsea_nes_pvalues.csv: table of NES and BH-adjusted P-values for all 6 factors x 22 KEGG-derived pathway sets
- Per-factor list of significantly enriched biological functions (e.g., axon guidance, ERBB signaling, MAPK signaling)

## How to apply

Load the LR-pair loadings from the 6 factors of the Tensor-cell2cell decomposition of the ASD communication tensor and, for each factor, rank LR pairs in descending order of loading value to build 6 pre-ranked lists. Load the 22 KEGG-derived LR pathway sets as gene sets and run PreRanked GSEA (e.g., GSEA/gseapy prerank) separately per factor with min set size 15 LR pairs and 999 permutations. Apply Benjamini-Hochberg correction to the GSEA P-values across pathway sets within each factor. Assemble a results table of NES and adjusted P-values for all 6 factors x 22 pathway sets and save it as gsea_nes_pvalues.csv; distinct significant functions per factor (axon guidance, cell adhesion, ECM-receptor interaction, ERBB signaling, MAPK signaling) are the biological readout. This couples the factorization with downstream analysis methods, as the paper recommends for interpreting Tensor-cell2cell results.

## Related tools

- **Tensor-cell2cell** (Source of the factor-specific LR-pair loadings that are ranked as pre-ranked GSEA inputs) — https://github.com/earmingol/cell2cell
- **cell2cell** (Suite providing downstream analysis functions for Tensor-cell2cell factors feeding into the GSEA step) — https://github.com/earmingol/cell2cell
- **GSEA** (Pre-ranked enrichment engine (or gseapy prerank equivalent) run per factor against KEGG-derived LR pathway sets)
- **CellChat** (Alternative LR-pair source for communication scores that can build the tensor whose loadings are enriched) — https://github.com/sqjin/CellChat

## Examples

```
import gseapy as gp; res = gp.prerank(rnk=factor_loadings_df, gene_sets=kegg_lr_sets, min_size=15, permutation_num=999)
```

## Evaluation signals

- Output table gsea_nes_pvalues.csv contains a NES and adjusted P-value entry for each of the 6 factors x 22 KEGG-derived pathway sets (132 rows)
- All adjusted P-values fall within [0, 1]
- Each of the 6 factors yields distinct enriched functions consistent with the reported finding (axon guidance, cell adhesion, ECM-receptor interaction, ERBB signaling, MAPK signaling)
- GSEA runs used min set size 15 LR pairs and 999 permutations, and P-values were BH-corrected within each factor

## Limitations

- Only cell types shared across all samples were included in the ASD case-study tensor, so factor loadings and downstream GSEA miss communication members absent in some contexts
- The implemented tensor decomposition algorithm is not optimized for missing values in the communication tensor, affecting the loadings that drive the ranking

## Evidence

- [results] Each ASD factor was associated with distinct biological functions, including axon guidance, cell adhesion, extracellular-matrix-receptor interaction, ERBB signaling, and MAPK signaling.: [evidence span withheld — anchor status: not_found]
- [discussion] Downstream analyses including GSEA on ASD factors: "In our ASD case-study (Fig. 5), such analyses included GSEA, clustering, visualization and statistical comparison of factors"
- [discussion] Tensor-cell2cell results coupled with downstream analysis methods: "Tensor-cell2cell results can be coupled with downstream analysis methods to facilitate interpretation and provide further insights of underlying mechanisms."
- [full_text] LR pairs ranked per factor and run through GSEA with min set size 15 and 999 permutations, BH-corrected: [evidence span withheld — anchor status: not_found]
