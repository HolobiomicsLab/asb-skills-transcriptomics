---
name: figure-4-fraction-reproduction
description: Use when validating a network-based driver gene predictor by quantifying
  what fraction of its CGC-overlapping predictions are rarely-mutated (<0.05 SNV frequency)
  versus frequently-mutated across TCGA cancer types.
when_to_use_negative:
- The goal is to run WMDS.net itself on new expression matrices rather than evaluate
  its driver predictions against SNV mutation frequency and CGC overlap.
- Only one cancer type or no CGC reference list is available, so the per-cancer-type
  low/high fraction split of CGC-overlapping drivers cannot be computed.
- The mutation data lack non-synonymous SNV annotations (e.g. unannotated VCFs), making
  the 'fraction of tumor samples carrying non-synonymous somatic mutations' frequency
  uncomputable.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: WMDS.net
  role: Source of the predicted cancer driver gene lists for the 14 TCGA cancer types
    that are evaluated for SNV mutation frequency
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Environment used to implement WMDS.net (binary integer programming via intlinprog)
    and to compute per-gene mutation frequencies and the Figure 4 fractions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:67ef9c4ac8b70fd498f70dc57ebf9bcc7966e2fd662f0e47ca466155b283161f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# figure-4-fraction-reproduction

## Summary

Reproduces Figure 4 of the WMDS.net paper: per-cancer-type fractions of CGC-overlapping predicted driver genes that are low-frequency (<0.05) versus high-frequency (>0.05) SNV mutations across 14 TCGA cancer types, plus the ERBB3-in-LUAD low-frequency driver case.

## When to use

Use when validating a network-based driver gene predictor by quantifying what fraction of its CGC-overlapping predictions are rarely-mutated (<0.05 SNV frequency) versus frequently-mutated across TCGA cancer types.

## When NOT to use

- The goal is to run WMDS.net itself on new expression matrices rather than evaluate its driver predictions against SNV mutation frequency and CGC overlap.
- Only one cancer type or no CGC reference list is available, so the per-cancer-type low/high fraction split of CGC-overlapping drivers cannot be computed.
- The mutation data lack non-synonymous SNV annotations (e.g. unannotated VCFs), making the 'fraction of tumor samples carrying non-synonymous somatic mutations' frequency uncomputable.

## Inputs

- WMDS.net driver gene predictions for 14 TCGA cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC)
- CGC (Cancer Gene Census) gene list of 616 genes from COSMIC (https://cancer.sanger.ac.uk/cosmic)
- TCGA somatic SNV (MAF-style) mutation data for the same 14 cancer types from the Genomic Data Commons (https://portal.gdc.cancer.gov)

## Outputs

- Per-cancer-type table of low-frequency (<0.05) and high-frequency (>0.05) fractions among CGC-overlapping predicted driver genes, reproducing the Figure 4 bar chart
- Verification that ERBB3 is a low-frequency (<0.05) WMDS.net-predicted driver in LUAD

## How to apply

Load WMDS.net driver gene predictions for the 14 TCGA cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) and the CGC list of 616 genes from COSMIC. Download TCGA somatic SNV data for the same 14 cancer types from the Genomic Data Commons. For each predicted driver gene in each cancer type, compute SNV mutation frequency as the fraction of tumor samples carrying non-synonymous somatic mutations in that gene. Intersect predicted drivers with the CGC list per cancer type, split the overlap into low-frequency (<0.05) and high-frequency (>0.05) classes, and compute the fraction of each class to rebuild the Figure 4 bar chart. As a spot check, confirm ERBB3 is a WMDS.net-predicted driver in LUAD whose LUAD mutation frequency falls in the low-frequency (<0.05) class, consistent with COSMIC records of its rarity in LUAD.

## Related tools

- **WMDS.net** (Source of the predicted cancer driver gene lists for the 14 TCGA cancer types that are evaluated for SNV mutation frequency) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Environment used to implement WMDS.net (binary integer programming via intlinprog) and to compute per-gene mutation frequencies and the Figure 4 fractions)

## Examples

```
freq = mean(any(ms.arrayContains(geneSymbols),'1'),2); % fraction of LUAD tumors with non-synonymous SNVs per gene, then classify at 0.05
```

## Evaluation signals

- The per-cancer-type low/high-frequency fractions reproduce the Figure 4 pattern: most CGC-overlapping predicted drivers fall in the rarely-mutated (<0.05) class across all 14 TCGA cancer types.
- ERBB3 is present in the WMDS.net-predicted driver list for LUAD and its LUAD SNV mutation frequency is <0.05.
- Each gene's mutation frequency equals the fraction of tumor samples with non-synonymous somatic SNVs in that gene within the matching TCGA cancer type.
- The intersection of predicted drivers with the 616-gene CGC list is computed per cancer type, and every intersected gene is assigned to exactly one of the two frequency classes (threshold 0.05).

## Limitations

- The analysis depends on the CGC list of 616 genes downloaded from COSMIC, so conclusions are relative to that specific curated driver set.
- The 0.05 low/high-frequency cutoff is a fixed threshold; genes near the boundary are sensitive to sample counts and TCGA cohort updates from the Genomic Data Commons.
- ERBB3's rarity is asserted via COSMIC records of its rarity in LUAD, so external verification of low-frequency status relies on that external catalogue.

## Evidence

- [results] Most WMDS.net-predicted driver genes overlapping the CGC list were rarely mutated across different cancer types, indicating the method's strong ability to find driver genes with low-frequency mutations.: [evidence span withheld — anchor status: not_found]
- [methods] CGC 616 genes from COSMIC: [evidence span withheld — anchor status: not_found]
- [methods] TCGA SNV from GDC: [evidence span withheld — anchor status: not_found]
- [results] ERBB3 LUAD low-frequency driver: [evidence span withheld — anchor status: not_found]
- [intro] Matlab intlinprog R2020b: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
