---
name: rna-seq-pileup-shape-change-interpretation
description: Use when a list of publicly reported splice-site mutation calls (e.g.
  TCGA HNSC MAF-style calls for TP53) must be cross-checked against SCISSOR shape-change
  outliers to flag likely mis-annotated calls.
when_to_use_negative:
- The goal is de novo shape-change detection from BAM/pileup data rather than interpretation
  of existing SCISSOR results against mutation calls (run Scissor via build_pileup/get_Ranges
  instead).
- The mutation calls are not splice-site annotated, or the gene/analysis cohort differs
  from the TP53 TCGA HNSC results object, so the GSC/LSC sample overlap is undefined.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SCISSOR
  role: Provides the ScissorOutput shape-change results (GSC/LSC outliers and outlyingness
    scores) that annotated splice-site mutations are cross-checked against; installed
    via install_github('hyochoi/SCISSOR')
  repo: https://github.com/hyochoi/SCISSOR
- name: R
  role: Analysis environment used to load the SCISSOR results object, filter public
    TCGA HNSC mutation calls, compute the association proportion, and write CSV outputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:1996b14a98d8de89cc8c996a906c5e84d074f2a364839d0654fda7dc007ece68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - R
  - SCISSOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# rna-seq-pileup-shape-change-interpretation

## Summary

Reproduce the SCISSOR-based discordance assessment of TP53 splice-site mutation calls in TCGA HNSC, checking which repository-annotated splice-site mutations coincide with statistically significant global (GSC) or local (LSC) shape changes in the base-level RNA-seq coverage shape.

## When to use

Use when a list of publicly reported splice-site mutation calls (e.g. TCGA HNSC MAF-style calls for TP53) must be cross-checked against SCISSOR shape-change outliers to flag likely mis-annotated calls.

## When NOT to use

- The goal is de novo shape-change detection from BAM/pileup data rather than interpretation of existing SCISSOR results against mutation calls (run Scissor via build_pileup/get_Ranges instead).
- The mutation calls are not splice-site annotated, or the gene/analysis cohort differs from the TP53 TCGA HNSC results object, so the GSC/LSC sample overlap is undefined.

## Inputs

- SCISSOR ScissorOutput results object for TP53 on TCGA HNSC (SC sample IDs, GSC$OS and LSC$OS outlyingness scores)
- Publicly reported TCGA HNSC mutation calls (per-sample mutation annotations including splice-site class)
- Set of SCISSOR-detected outliers: 26 global shape changes (GSCs) and 24 local shape changes (LSCs) among 452 analyzed samples

## Outputs

- splice_site_mutation_association.csv — per-mutation table with boolean flag of shape-change association
- splice_site_summary.csv — summary proportion of splice-site mutations associated with a statistically significant shape change (expected ~59%)

## How to apply

Load the SCISSOR TP53 results object from the TCGA HNSC analysis (452 analyzed samples) and extract shape-change sample IDs from ScissorOutput$SC, with outlyingness scores from ScissorOutput$GSC$OS and ScissorOutput$LSC$OS, corresponding to the 26 GSCs and 24 LSCs. Independently load the public TCGA HNSC mutation calls and filter to TP53 variants annotated as splice-site mutations. For each annotated splice-site mutation, flag whether its sample overlaps a SCISSOR-detected GSC or LSC sample, then compute the proportion with a significant shape change and compare against the reported 59% within rounding tolerance. Discordant cases (no shape-change flag) should be interpreted as likely incorrectly annotated mutations or cases with limited or no supporting reads for abnormal splicing. Write the per-mutation flag table and the summary proportion to CSV for validation.

## Related tools

- **SCISSOR** (Provides the ScissorOutput shape-change results (GSC/LSC outliers and outlyingness scores) that annotated splice-site mutations are cross-checked against; installed via install_github('hyochoi/SCISSOR')) — https://github.com/hyochoi/SCISSOR
- **R** (Analysis environment used to load the SCISSOR results object, filter public TCGA HNSC mutation calls, compute the association proportion, and write CSV outputs)

## Examples

```
devtools::install_github("hyochoi/SCISSOR"); library(SCISSOR); data(TOY_data)
```

## Evaluation signals

- The computed fraction of splice-site mutations with an associated significant shape change matches the published 59% within rounding tolerance.
- Every row of the per-mutation table carries a boolean shape-change association flag (no missing values).
- The set of SCISSOR shape-change sample IDs contains exactly the 26 GSCs and 24 LSCs from the 452 analyzed samples.
- Discordant (unflagged) mutations are individually reviewable as likely mis-annotations or low-support splicing events, consistent with the paper's interpretation.
- Both output files (splice_site_mutation_association.csv and splice_site_summary.csv) exist with the expected schema.

## Limitations

- LSC detection results depend on the window size choice, so the LSC sample set used for overlap may differ under alternative window sizes.
- GSC detection loses information about the order of base pairs, limiting localization of the underlying structural alteration.
- Variable fragment length and GC-content effects can introduce artifactual shape changes, so absence of a shape change does not prove a mutation call is wrong without read-level review (e.g. via IGV).

## Evidence

- [results] Reported discordance rate: "only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change"
- [results] GSC/LSC counts in TP53: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [methods] Package installation: "install_github("hyochoi/SCISSOR")"
- [readme] README overview of purpose: "aims for unsupervised screening of a range of structural alterations in RNA-seq data"
