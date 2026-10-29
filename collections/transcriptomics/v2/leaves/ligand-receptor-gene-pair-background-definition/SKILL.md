---
name: ligand-receptor-gene-pair-background-definition
description: Use when validating that an identified set of ligand-receptor genes (e.g.
  eLRs) is enriched or depleted for reference gene sets, and you need a defensible
  background universe — all protein-coding genes and/or an all-known-LR baseline.
when_to_use_negative:
- The query set is not composed of ligand-receptor genes — the LR-baseline correction
  is meaningless for unrelated gene panels.
- Reference gene sets or the protein-coding universe are unavailable or incompatible
  in gene identifier space, so overlap counts would be spuriously small.
- The goal is differential expression rather than set-overreach testing; hypergeometric
  background definition does not apply.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0204
tools:
- name: R (version 3.6.3)
  role: Platform for the one-tailed hypergeometric (Fisher-style overlap) enrichment
    tests of the eLR set against reference gene sets
- name: TimeTalk
  role: Source of the identified 430 eLR gene list and the bundled Mouse-2020-Shao-LR-pairs
    baseline file
  repo: https://github.com/ChengLiLab/TimeTalk
- name: CellTalkDB
  role: Provides the 2034 mouse ligand-receptor pairs used as the all-LR gene-universe
    baseline
- name: DEG 10 database
  role: Source of the essential gene reference set
- name: HRT Atlas v1.0
  role: Source of the housekeeping gene reference set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:9260be35f5feaf12ec0687c91124a439a35cc7bf7f7144fc472ffcba11cff4aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - CellTalkDB
  - DEG 10 database
  - HRT Atlas v1.0
  - R (version 3.6.3)
  - TimeTalk
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ligand-receptor-gene-pair-background-definition

## Summary

Defines the statistical universes (all-protein-coding genes and all known mouse ligand-receptor genes from CellTalkDB, 2034 pairs) against which the 430 TimeTalk eLR genes are tested for enrichment/depletion, using one-tailed hypergeometric tests in R. Proper background choice distinguishes LR-specific biology (e.g. essential-gene enrichment surviving LR-baseline correction) from genome-wide artifacts.

## When to use

Use when validating that an identified set of ligand-receptor genes (e.g. eLRs) is enriched or depleted for reference gene sets, and you need a defensible background universe — all protein-coding genes and/or an all-known-LR baseline.

## When NOT to use

- The query set is not composed of ligand-receptor genes — the LR-baseline correction is meaningless for unrelated gene panels.
- Reference gene sets or the protein-coding universe are unavailable or incompatible in gene identifier space, so overlap counts would be spuriously small.
- The goal is differential expression rather than set-overreach testing; hypergeometric background definition does not apply.

## Inputs

- Target gene list (e.g. 430 eLR genes from the TimeTalk Figshare deposit)
- Essential gene set from DEG 10 database
- Housekeeping gene set from HRT Atlas v1.0
- ZGA gene sets (Table S5, ref. 18)
- All-LR baseline: 2034 mouse ligand-receptor pairs from CellTalkDB (or Mouse-2020-Shao-LR-pairs.txt)
- All-protein-coding gene universe

## Outputs

- Table of enrichment ratios and hypergeometric p-values per reference set, written to eLR_enrichment_pvalues.csv
- Baseline-corrected enrichment results using the all-LR gene universe

## How to apply

Assemble the target gene list (e.g. the 430 eLR genes from the TimeTalk Figshare deposit) plus reference sets: essential genes from the DEG 10 database, housekeeping genes from HRT Atlas v1.0, and ZGA gene sets from Table S5 of ref. 18. Run one-tailed hypergeometric (Fisher-style overlap) tests against each reference set using the all-protein-coding gene universe as background in R (version 3.6.3), reporting enrichment ratios and p-values (e.g. 1.628e-111, 4.344e-14, 3.049e-85, 1.944e-47, 5.986e-16, 0.00279, 0.001159). Then repeat the identical tests using the all-LR-pair gene universe — the 2034 mouse ligand-receptor pairs from CellTalkDB (or the Mouse-2020-Shao-LR-pairs bundled with TimeTalk) — to obtain baseline-corrected enrichment that shows the signal is not generic to all communication genes. Rationale: enrichment that persists against the LR baseline supports the claim that eLR genes are required for organism fitness and early embryo development (ZGA) rather than individual cell survival.

## Related tools

- **R (version 3.6.3)** (Platform for the one-tailed hypergeometric (Fisher-style overlap) enrichment tests of the eLR set against reference gene sets)
- **TimeTalk** (Source of the identified 430 eLR gene list and the bundled Mouse-2020-Shao-LR-pairs baseline file) — https://github.com/ChengLiLab/TimeTalk
- **CellTalkDB** (Provides the 2034 mouse ligand-receptor pairs used as the all-LR gene-universe baseline)
- **DEG 10 database** (Source of the essential gene reference set)
- **HRT Atlas v1.0** (Source of the housekeeping gene reference set)

## Examples

```
LRpairs.df <- read.delim(file = "database/Ligand-Receptor-Pairs/Mouse/Mouse-2020-Shao-LR-pairs.txt", stringsAsFactors = F); p <- phyper(length(overlap)-1, length(ref.set), length(universe)-length(ref.set), length(eLR.genes), lower.tail=FALSE)
```

## Evaluation signals

- Computed hypergeometric p-values match the reported values (1.628e-111, 4.344e-14, 3.049e-85, 1.944e-47, 5.986e-16, 0.00279, 0.001159) within numerical tolerance (Fig. 3b-d / Supplementary Fig. 11).
- eLR_enrichment_pvalues.csv exists and contains enrichment-ratio and p-value fields.
- The all-LR baseline uses exactly the 2034 CellTalkDB mouse ligand-receptor pairs.
- Enrichment direction (essential/ZGA up, housekeeping down) is preserved after switching from the protein-coding universe to the all-LR universe.

## Limitations

- Results depend on the chosen universe: enrichment against all-protein-coding genes may reflect generic LR properties, hence the second pass against the 2034-pair LR baseline is required.
- Overlaps assume independence between the query set and reference databases; DEG 10, HRT Atlas, and ZGA sets have different annotation scopes that are not harmonized beyond gene-name matching.

## Evidence

- [discussion] Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16: "Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16"
- [discussion] The essential gene set was downloaded from the DEG 10 database24: "The essential gene set was downloaded from the DEG 10 database24"
- [discussion] The housekeeping gene set was downloaded from the HRT Atlas v1.0 database26: "The housekeeping gene set was downloaded from the HRT Atlas v1.0 database26"
- [discussion] The ZGA gene sets were obtained from Table S5 in ref. 18.: "The ZGA gene sets were obtained from Table S5 in ref. 18."
- [discussion] All statistical analyses were performed using R (version 3.6.3).: "All statistical analyses were performed using R (version 3.6.3)."
- [intro] Our analysis identified 430 eLRs, including previously reported Fgf4-Fgfr1 and Fgf4-Fgfr2.: "Our analysis identified 430 eLRs, including previously reported Fgf4-Fgfr1 and Fgf4-Fgfr2."
- [readme] file = "database/Ligand-Receptor-Pairs/Mouse/Mouse-2020-Shao-LR-pairs.txt": "LRpairs.df <- read.delim(file = "database/Ligand-Receptor-Pairs/Mouse/Mouse-2020-Shao-LR-pairs.txt",stringsAsFactors = F)"
