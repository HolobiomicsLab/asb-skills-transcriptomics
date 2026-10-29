---
name: essential-gene-overlap-analysis
description: Use when you have a defined gene list (e.g., TimeTalk's 430 eLR genes)
  and reference gene sets, and need to test whether the list is significantly enriched/depleted
  (e.g., essential/ZGA enriched, housekeeping depleted) against a gene-universe background.
when_to_use_negative:
- You need to test regulatory relationships between LR pairs and transcription factors
  (use Granger causality / RTN master-regulator analysis instead of gene-set overlap).
- The input is expression data rather than a defined gene list — first reduce to a
  concrete gene set before applying overlap tests.
- You need cell-type-specific differential expression, not gene-set-level enrichment
  against a fixed gene universe.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R (version 3.6.3)
  role: Environment in which the one-tailed hypergeometric overlap tests and CSV export
    are performed
- name: TimeTalk
  role: Source of the 430 eLR gene list being tested for enrichment/depletion
  repo: https://github.com/ChengLiLab/TimeTalk
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:9864429a5a12ac900222d6272c6fdebb6904456f7915066935932122423dcd2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R (version 3.6.3)
  - TimeTalk
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# essential-gene-overlap-analysis

## Summary

One-tailed hypergeometric (Fisher-style) overlap tests that check whether a gene set — such as the 430 early-embryo-development-related ligand-receptor (eLR) genes identified by TimeTalk — is enriched for essential genes (DEG 10), ZGA genes (Table S5, ref. 18), and depleted of housekeeping genes (HRT Atlas v1.0), relative to all-protein-coding and all-LR baselines.

## When to use

Use when you have a defined gene list (e.g., TimeTalk's 430 eLR genes) and reference gene sets, and need to test whether the list is significantly enriched/depleted (e.g., essential/ZGA enriched, housekeeping depleted) against a gene-universe background.

## When NOT to use

- You need to test regulatory relationships between LR pairs and transcription factors (use Granger causality / RTN master-regulator analysis instead of gene-set overlap).
- The input is expression data rather than a defined gene list — first reduce to a concrete gene set before applying overlap tests.
- You need cell-type-specific differential expression, not gene-set-level enrichment against a fixed gene universe.

## Inputs

- 430 eLR gene list from the TimeTalk Figshare deposit
- Essential gene set from the DEG 10 database
- Housekeeping gene set from HRT Atlas v1.0
- ZGA gene sets from Table S5 of ref. 18
- 2034 mouse ligand-receptor pair genes from CellTalkDB (all-LR baseline)
- All-protein-coding gene universe (background)

## Outputs

- eLR_enrichment_pvalues.csv (table of enrichment ratios and hypergeometric p-values)
- Per-reference-set overlap p-values comparable to Fig. 3b-d / Supplementary Fig. 11

## How to apply

Load the 430 eLR gene list from the TimeTalk Figshare deposit (https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498) and the reference sets: essential genes from the DEG 10 database, housekeeping genes from HRT Atlas v1.0, ZGA gene sets from Table S5 of ref. 18, and the 2034 mouse ligand-receptor pair genes from CellTalkDB as the all-LR baseline. In R (version 3.6.3), run one-tailed hypergeometric tests of the overlap of the eLR set against each reference set using the all-protein-coding gene universe as background, then repeat the tests using the all-LR pair gene universe as background to obtain baseline-corrected enrichment. Assemble enrichment ratios and p-values into eLR_enrichment_pvalues.csv. The expected result is enrichment for essential and ZGA genes and depletion of housekeeping genes, supporting that eLR genes are required for organism fitness and early embryo development rather than individual cell survival. Validate by checking computed p-values against the reported values (e.g., 1.628e-111, 4.344e-14, 3.049e-85, 1.944e-47, 5.986e-16, 0.00279, 0.001159) within numerical tolerance.

## Related tools

- **R (version 3.6.3)** (Environment in which the one-tailed hypergeometric overlap tests and CSV export are performed)
- **TimeTalk** (Source of the 430 eLR gene list being tested for enrichment/depletion) — https://github.com/ChengLiLab/TimeTalk

## Examples

```
p <- phyper(sum(eLR.genes %in% essential.genes)-1, length(essential.genes), nCoding-length(essential.genes), length(eLR.genes), lower.tail=FALSE)
```

## Evaluation signals

- File exists and schema check: eLR_enrichment_pvalues.csv is present with enrichment-ratio and p-value fields populated
- Output matches reference: computed hypergeometric p-values match reported values (1.628e-111, 4.344e-14, 3.049e-85, 1.944e-47, 5.986e-16, 0.00279, 0.001159) within numerical tolerance
- Directional invariant: eLR set is enriched (significant) for essential and ZGA genes and depleted of housekeeping genes
- Background check: both the all-protein-coding gene universe and the all-LR (2034 CellTalkDB pair) gene universe were used as backgrounds, yielding baseline-corrected enrichment

## Limitations

- Enrichment conclusions depend on the completeness and curation of the reference databases (DEG 10, HRT Atlas v1.0, Table S5 ZGA sets).
- The choice of background universe (all-protein-coding vs. all-LR genes) changes the p-values; both baselines should be reported.

## Evidence

- [intro] we conducted thorough in silico analyses to test the involvement of identified eLRs in early embryo development: "Additionally, we conducted thorough in silico analyses to test the involvement of identified eLRs in early embryo development."
- [discussion] The essential gene set was downloaded from the DEG 10 database24: "The essential gene set was downloaded from the DEG 10 database24 (http://origin.tubic.org/deg/public/index.php/index)."
- [discussion] The housekeeping gene set was downloaded from the HRT Atlas v1.0 database26: "The housekeeping gene set was downloaded from the HRT Atlas v1.0 database26 (http://www.housekeeping.unicamp.br/)."
- [discussion] The ZGA gene sets were obtained from Table S5 in ref. 18: "The ZGA gene sets were obtained from Table S5 in ref. 18."
- [discussion] Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB: "Mouse ligand-receptor gene pairs (2034 pairs) were downloaded from CellTalkDB16 (http://tcm.zju.edu.cn/celltalkdb/download.php)."
- [discussion] All statistical analyses were performed using R (version 3.6.3): "All statistical analyses were performed using R (version 3.6.3)."
