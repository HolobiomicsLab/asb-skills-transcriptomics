---
name: zga-gene-set-enrichment-analysis
description: Use when a curated list of ligand-receptor pair genes (e.g. eLRs from
  early embryo scRNA-seq analysis) must be validated against reference gene sets (essential,
  housekeeping, ZGA) with over-representation tests.
when_to_use_negative:
- The goal is to identify new eLRs from scRNA-seq data rather than validate an already-curated
  gene list (use the TimeTalk pipeline instead).
- The input is not a gene set whose composition should be tested against reference
  sets (e.g. testing LR-TF regulatory relationships by Granger causality).
- Reference gene sets (essential, housekeeping, ZGA) or a defined background universe
  are unavailable, making the hypergeometric test uninterpretable.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: R (version 3.6.3) is used to run the one-tailed hypergeometric (Fisher-style)
    overlap tests and assemble the enrichment p-value table.
  repo: https://github.com/ChengLiLab/TimeTalk
- name: TimeTalk
  role: Source of the 430 eLR gene list being validated; code and data deposited on
    Figshare.
  repo: https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:3f0cef6c0e931cadbcfc99845bb68bce4d82741dbca60c713998d63121d3856d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - R (version 3.6.3)
  - TimeTalk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# zga-gene-set-enrichment-analysis

## Summary

One-tailed hypergeometric (Fisher-style) overlap tests in R that check whether the 430 TimeTalk-identified eLR genes are enriched for essential genes (DEG 10), ZGA genes (Table S5 of ref. 18), and depleted of housekeeping genes (HRT Atlas v1.0), against both all-protein-coding and all-LR-pair (2034 CellTalkDB pairs) background universes — the in-silico validation that eLR genes are required for organism fitness and early embryo development.

## When to use

Use when a curated list of ligand-receptor pair genes (e.g. eLRs from early embryo scRNA-seq analysis) must be validated against reference gene sets (essential, housekeeping, ZGA) with over-representation tests.

## When NOT to use

- The goal is to identify new eLRs from scRNA-seq data rather than validate an already-curated gene list (use the TimeTalk pipeline instead).
- The input is not a gene set whose composition should be tested against reference sets (e.g. testing LR-TF regulatory relationships by Granger causality).
- Reference gene sets (essential, housekeeping, ZGA) or a defined background universe are unavailable, making the hypergeometric test uninterpretable.

## Inputs

- 430 eLR gene list from the TimeTalk Figshare deposit (https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498)
- Essential gene set from the DEG 10 database (http://origin.tubic.org/deg/public/index.php/index)
- Housekeeping gene set from HRT Atlas v1.0 (http://www.housekeeping.unicamp.br/)
- ZGA gene sets from Table S5 of ref. 18
- 2034 mouse ligand-receptor pairs from CellTalkDB (all-LR baseline universe)
- All-protein-coding gene universe (background)

## Outputs

- eLR_enrichment_pvalues.csv — table of enrichment ratios and hypergeometric p-values per reference gene set under both background universes
- Enrichment/depletion summary comparable to Fig. 3b-d and Supplementary Fig. 11

## How to apply

Load the 430 identified eLR gene list from the TimeTalk Figshare deposit and assemble four reference sets: essential genes from DEG 10, housekeeping genes from HRT Atlas v1.0, ZGA genes from Table S5 of ref. 18, and the 2034 mouse ligand-receptor pairs from CellTalkDB as the all-LR baseline. In R (version 3.6.3), run one-tailed hypergeometric (Fisher-style overlap) tests of the eLR set against each reference set using the all-protein-coding gene universe as background, then repeat with the all-LR-pair gene universe as background to obtain baseline-corrected enrichment. The expected directionality is enrichment for essential and ZGA genes and depletion of housekeeping genes, since eLR genes should support organism fitness and early embryo development rather than individual cell survival. Report enrichment ratios and p-values per reference set/universe combination; the two-universe design is the decision point separating raw overlap significance from LR-specific enrichment.

## Related tools

- **R** (R (version 3.6.3) is used to run the one-tailed hypergeometric (Fisher-style) overlap tests and assemble the enrichment p-value table.) — https://github.com/ChengLiLab/TimeTalk
- **TimeTalk** (Source of the 430 eLR gene list being validated; code and data deposited on Figshare.) — https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498

## Examples

```
p <- phyper(length(intersect(eLR, ref)) - 1, length(ref), length(universe) - length(ref), length(eLR), lower.tail = FALSE)
```

## Evaluation signals

- Computed hypergeometric p-values match the reported values (1.628e-111, 4.344e-14, 3.049e-85, 1.944e-47, 5.986e-16, 0.00279, 0.001159) within numerical tolerance.
- Directionality check: eLR set is enriched for essential and ZGA genes but depleted of housekeeping genes under the all-protein-coding background.
- Baseline-corrected enrichment using the all-LR (2034 CellTalkDB pairs) universe is also computed and reported.
- File-exists and field-presence checks on eLR_enrichment_pvalues.csv (enrichment ratio and p-value columns present).
- Results should be visually comparable to Fig. 3b-d / Supplementary Fig. 11 of the TimeTalk paper.

## Limitations

- Validation is purely in silico; the inferred eLR involvement in early embryo development is not confirmed by wet-lab experiments.
- Results depend on the choice of background universe — significance under the all-protein-coding universe can differ from baseline-corrected enrichment under the all-LR-pair universe.
- eLR genes may reflect general organism fitness (essential genes) rather than embryo-specific function, so the ZGA/housekeeping contrast is required to interpret the enrichment.

## Evidence

- [discussion] The essential gene set was downloaded from the DEG 10 database24 (http://origin.tubic.org/deg/public/index.php/index).: "The essential gene set was downloaded from the DEG 10 database24 (http://origin.tubic.org/deg/public/index.php/index)."
- [discussion] The housekeeping gene set was downloaded from the HRT Atlas v1.0 database26 (http://www.housekeeping.unicamp.br/).: "The housekeeping gene set was downloaded from the HRT Atlas v1.0 database26 (http://www.housekeeping.unicamp.br/)."
- [discussion] The ZGA gene sets were obtained from Table S5 in ref. 18.: "The ZGA gene sets were obtained from Table S5 in ref. 18."
- [intro] Additionally, we conducted thorough in silico analyses to test the involvement of identified eLRs in early embryo development.: "Additionally, we conducted thorough in silico analyses to test the involvement of identified eLRs in early embryo development."
- [discussion] Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16 (http://tcm.zju.edu.cn/celltalkdb/download.php).: "Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16 (http://tcm.zju.edu.cn/celltalkdb/download.php)."
