---
name: housekeeping-gene-reference-set-mapping
description: Use when validating that a candidate ligand-receptor-derived gene set
  (e.g., eLRs) behaves like developmentally-required genes — enriched for essential/ZGA
  genes, depleted of housekeeping genes — relative to protein-coding and LR-pair baselines.
when_to_use_negative:
- The input is per-cell or pseudo-bulk expression data rather than a curated gene
  list to test for set-level overlap enrichment.
- The question is which LR pairs exist or are co-varying (use TimeTalk's |PCC|=0.1/SCC=0.2
  and Granger cutoffs instead), not whether an already-identified gene set is enriched
  for functional categories.
- The target gene set is unrelated to early embryo development, so the expected essential/ZGA-enriched,
  housekeeping-depleted signature provides no meaningful validation.
tools:
- name: R (version 3.6.3)
  role: Runtime for the one-tailed hypergeometric (Fisher-style overlap) enrichment
    tests of the eLR set against each reference gene set
- name: TimeTalk
  role: Source of the identified eLR gene list being validated against the reference
    gene sets
  repo: https://github.com/ChengLiLab/TimeTalk
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:e88df734c37d62647a362b6f0273f883cf3c0eae6712111bf7c5ec7958d162e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R (version 3.6.3)
  - TimeTalk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# housekeeping-gene-reference-set-mapping

## Summary

Maps a candidate gene list (e.g., the 430 TimeTalk eLR genes) against the HRT Atlas v1.0 housekeeping gene set, DEG 10 essential genes, and ZGA gene sets using one-tailed hypergeometric tests, expecting enrichment for essential/ZGA genes and depletion of housekeeping genes to support a role in organism fitness and early embryo development.

## When to use

Use when validating that a candidate ligand-receptor-derived gene set (e.g., eLRs) behaves like developmentally-required genes — enriched for essential/ZGA genes, depleted of housekeeping genes — relative to protein-coding and LR-pair baselines.

## When NOT to use

- The input is per-cell or pseudo-bulk expression data rather than a curated gene list to test for set-level overlap enrichment.
- The question is which LR pairs exist or are co-varying (use TimeTalk's |PCC|=0.1/SCC=0.2 and Granger cutoffs instead), not whether an already-identified gene set is enriched for functional categories.
- The target gene set is unrelated to early embryo development, so the expected essential/ZGA-enriched, housekeeping-depleted signature provides no meaningful validation.

## Inputs

- eLR gene list (430 genes from TimeTalk Figshare deposit)
- Housekeeping gene set from HRT Atlas v1.0
- Essential gene set from the DEG 10 database
- ZGA gene sets from Table S5 of ref. 18
- CellTalkDB mouse ligand-receptor pairs (2034 pairs) as all-LR baseline
- All-protein-coding gene universe (background)

## Outputs

- eLR_enrichment_pvalues.csv containing enrichment ratios and hypergeometric p-values per reference gene set and per background universe

## How to apply

Load the 430 eLR gene list from the TimeTalk Figshare deposit (https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498). Prepare reference sets: housekeeping genes from HRT Atlas v1.0 (http://www.housekeeping.unicamp.br/), essential genes from DEG 10, ZGA gene sets from Table S5 of ref. 18, and the 2034 CellTalkDB mouse ligand-receptor pairs as the all-LR baseline. In R (version 3.6.3), run one-tailed hypergeometric (Fisher-style overlap) tests of the eLR set against each reference gene set twice: once with the all-protein-coding gene universe as background and once with the all-LR-pair gene universe to obtain baseline-corrected enrichment. Housekeeping genes are the negative-control set — a correct result shows depletion (weak or absent enrichment) of eLRs among housekeeping genes, indicating these genes support organism fitness and early embryo development rather than individual cell survival. Assemble enrichment ratios and p-values into eLR_enrichment_pvalues.csv.

## Related tools

- **R (version 3.6.3)** (Runtime for the one-tailed hypergeometric (Fisher-style overlap) enrichment tests of the eLR set against each reference gene set)
- **TimeTalk** (Source of the identified eLR gene list being validated against the reference gene sets) — https://github.com/ChengLiLab/TimeTalk

## Examples

```
Rscript -e 'p <- phyper(length(intersect(eLR,housekeeping)), length(housekeeping), nProteinCoding-length(housekeeping), length(eLR), lower.tail=FALSE)'
```

## Evaluation signals

- file_exists and field_present checks on eLR_enrichment_pvalues.csv
- output_matches_reference: computed hypergeometric p-values match reported values 1.628e-111, 4.344e-14, 3.049e-85, 1.944e-47, 5.986e-16, 0.00279, 0.001159 within numerical tolerance (Fig. 3b-d / Supplementary Fig. 11)
- Housekeeping gene test shows depletion (no significant enrichment) of the eLR set, consistent with genes required for organism fitness rather than individual cell survival
- Enrichment direction is consistent across both background universes (all-protein-coding and all-LR pair), with baseline-corrected values reported for the all-LR universe

## Limitations

- Validation is in-silico only; hypergeometric overlap with essential/ZGA/housekeeping sets supports but does not functionally prove eLR involvement in early embryo development.
- Results depend on the chosen gene-set versions and the background universe; repeating the tests against the all-LR-pair gene universe is required to baseline-correct the enrichment.
- One-tailed tests assume the expected enrichment direction (enriched for essential/ZGA, depleted for housekeeping); a two-tailed design would not map directly onto these reported values.

## Evidence

- [discussion] Housekeeping reference-set source and version: "The housekeeping gene set was downloaded from the HRT Atlas v1.0 database26 (http://www.housekeeping.unicamp.br/)."
- [intro] eLR set being validated: "Our analysis identified 430 eLRs, including previously reported Fgf4-Fgfr1 and Fgf4-Fgfr2."
- [discussion] All-LR baseline gene pairs: "Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16 (http://tcm.zju.edu.cn/celltalkdb/download.php)."
- [discussion] Statistical environment: "All statistical analyses were performed using R (version 3.6.3)."
- [discussion] Essential gene reference set: "The essential gene set was downloaded from the DEG 10 database24 (http://origin.tubic.org/deg/public/index.php/index)."
