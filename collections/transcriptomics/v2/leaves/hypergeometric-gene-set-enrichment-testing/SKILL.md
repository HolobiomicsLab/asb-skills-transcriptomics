---
name: hypergeometric-gene-set-enrichment-testing
description: Use when you need to statistically validate that a candidate gene list
  (e.g., identified LR pairs) is over- or under-represented for known functional categories
  like essential, housekeeping, or developmental-stage genes.
when_to_use_negative:
- When the candidate list is not a subset of a defined gene universe — e.g., if the
  query genes come from an unfiltered or unknown-origin source, the hypergeometric
  background is not well-defined and results will be misleading.
- When you need pathway-level (GO/KEGG) enrichment rather than overlap with specific
  curated gene sets; this skill tests against named reference lists, not term-based
  ontology enrichment.
- When the gene list requires expression-based ranking (e.g., GSEA-style ordered statistics)
  rather than simple set membership overlap.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_0621
tools:
- name: R (version 3.6.3)
  role: Statistical environment in which the one-tailed hypergeometric (Fisher-style
    overlap) tests are computed against each reference gene set and background
  repo: https://github.com/ChengLiLab/TimeTalk
- name: TimeTalk
  role: Upstream framework that produced the 430 eLR gene list being validated; code
    and data deposited at Figshare
  repo: https://github.com/ChengLiLab/TimeTalk
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:87fdd3d7de9b409015e20d1d82ef0f1b030f769d7efaba4840a237e298d8f85a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R (version 3.6.3)
  - TimeTalk
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hypergeometric-gene-set-enrichment-testing

## Summary

One-tailed hypergeometric (Fisher-style) overlap tests to assess whether a candidate gene list (e.g., the 430 eLR genes identified by TimeTalk) is enriched/depleted for curated reference sets (essential genes, ZGA genes, housekeeping genes) against two backgrounds: all protein-coding genes and all ligand-receptor pair genes.

## When to use

Use when you need to statistically validate that a candidate gene list (e.g., identified LR pairs) is over- or under-represented for known functional categories like essential, housekeeping, or developmental-stage genes.

## When NOT to use

- When the candidate list is not a subset of a defined gene universe — e.g., if the query genes come from an unfiltered or unknown-origin source, the hypergeometric background is not well-defined and results will be misleading.
- When you need pathway-level (GO/KEGG) enrichment rather than overlap with specific curated gene sets; this skill tests against named reference lists, not term-based ontology enrichment.
- When the gene list requires expression-based ranking (e.g., GSEA-style ordered statistics) rather than simple set membership overlap.

## Inputs

- eLR gene list (430 genes, from TimeTalk Figshare deposit https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498)
- Essential gene set (DEG 10 database, http://origin.tubic.org/deg/public/index.php/index)
- Housekeeping gene set (HRT Atlas v1.0, http://www.housekeeping.unicamp.br/)
- ZGA gene sets (Table S5 of ref. 18)
- All-LR baseline gene universe (2034 mouse ligand-receptor pairs from CellTalkDB, http://tcm.zju.edu.cn/celltalkdb/download.php)
- All-protein-coding gene universe (background)

## Outputs

- eLR_enrichment_pvalues.csv (enrichment ratios and hypergeometric p-values per reference gene set and per background)
- Enrichment/depletion statistics matching Fig. 3b-d and Supplementary Fig. 11

## How to apply

Load the 430 identified eLR gene list from the TimeTalk Figshare deposit and prepare reference sets: essential genes from the DEG 10 database, housekeeping genes from HRT Atlas v1.0, ZGA gene sets from Table S5 of ref. 18, and the 2034 mouse ligand-receptor pairs from CellTalkDB as the all-LR baseline. In R (version 3.6.3), run one-tailed hypergeometric (Fisher-style overlap) tests of the eLR set against each reference gene set using the all-protein-coding gene universe as background. Repeat the tests using the all-LR pair gene universe as background to obtain baseline-corrected enrichment, since eLR genes are by definition drawn from LR pairs and would otherwise appear spuriously enriched. Assemble a table of enrichment ratios and p-values (expected values include 1.628e-111, 4.344e-14, 3.049e-85, 1.944e-47, 5.986e-16, 0.00279, 0.001159) and write to eLR_enrichment_pvalues.csv, verifying against Fig. 3b-d / Supplementary Fig. 11.

## Related tools

- **R (version 3.6.3)** (Statistical environment in which the one-tailed hypergeometric (Fisher-style overlap) tests are computed against each reference gene set and background) — https://github.com/ChengLiLab/TimeTalk
- **TimeTalk** (Upstream framework that produced the 430 eLR gene list being validated; code and data deposited at Figshare) — https://github.com/ChengLiLab/TimeTalk

## Examples

```
Rscript -e 'phyper(q, m, n, k, lower.tail=FALSE)'  # one-tailed hypergeometric overlap of 430 eLR genes vs. DEG10/HRT Atlas/ZGA/CellTalkDB reference sets
```

## Evaluation signals

- file_exists check on eLR_enrichment_pvalues.csv
- field_present check for enrichment ratio and p-value columns in eLR_enrichment_pvalues.csv
- output_matches_reference: computed hypergeometric p-values match reported values (1.628e-111, 4.344e-14, 3.049e-85, 1.944e-47, 5.986e-16, 0.00279, 0.001159) within numerical tolerance
- Results consistent with Fig. 3b-d / Supplementary Fig. 11 of the manuscript
- Directionality check: eLR genes enriched for essential and ZGA genes but depleted for housekeeping genes under both backgrounds

## Limitations

- Enrichment against all protein-coding genes alone is misleading because eLR genes are drawn from the LR pair universe; both backgrounds (all-protein-coding and all-LR) are needed for baseline-corrected interpretation.
- This is purely in-silico validation of set overlap; it does not confirm that individual eLR-to-TF regulatory relationships hold biologically — the article notes those 'should be confirmed by further wet experiments'.
- Reference gene sets are database-version dependent (DEG 10, HRT Atlas v1.0, CellTalkDB); updating the reference sets changes the hypergeometric p-values and would break the output_matches_reference check.

## Evidence

- [results] In-silico validation showed that the eLR pairs identified by TimeTalk are enriched with essential genes and ZGA genes but depleted of housekeeping genes: [evidence span withheld — anchor status: not_found]
- [methods] Mouse ligand-receptor gene pairs were downloaded from CellTalkDB: "Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16 (http://tcm.zju.edu.cn/celltalkdb/download.php)."
- [methods] The essential gene set was downloaded from the DEG 10 database: "The essential gene set was downloaded from the DEG 10 database24 (http://origin.tubic.org/deg/public/index.php/index)."
- [methods] The housekeeping gene set was downloaded from the HRT Atlas v1.0 database: "The housekeeping gene set was downloaded from the HRT Atlas v1.0 database26 (http://www.housekeeping.unicamp.br/)."
- [discussion] All statistical analyses were performed using R: "All statistical analyses were performed using R (version 3.6.3)."
- [methods] Code and data uploaded to Figshare: "we have uploaded our code and extensive data to Figshare (https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498)"
