---
name: oncogene-expression-quantification
description: Use when a gene fusion (e.g., IgH-WHSC1) has been detected per cell in
  scRNA-seq data and you must test whether the fusion partner oncogene is overexpressed
  in fusion-bearing cells.
when_to_use_negative:
- No fusion calls exist yet — scFusion (STAR, samtools, bedtools, statistical/deep-learning
  filters) must first produce per-cell fusion calls before expression stratification
  is meaningful
- The data are not from a Smart-Seq single-cell protocol or the fusion is not expressed
  in the sequenced cells
- The candidate oncogene lies far from the fusion breakpoint so coverage and expression
  cannot be linked to the fusion event
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scFusion
  role: Produces the per-cell IgH-WHSC1 fusion calls and breakpoint table that stratify
    cells into fusion-positive and fusion-negative groups
  repo: https://github.com/ZijieJin/scFusion
- name: R (>= 3.5) with stringr
  role: Quantifies and compares WHSC1 expression in fusion-positive versus fusion-negative
    cells
- name: STAR >= 2.7.2d
  role: Aligns single-cell libraries so WHSC1 per-base coverage downstream of the
    breakpoint can be profiled
  repo: https://github.com/alexdobin/STAR
- name: pyensembl
  role: Annotates fusion gene partners (gene_name, gene_type from the GTF) so the
    oncogene WHSC1 is correctly identified
  repo: https://github.com/openvax/pyensembl
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:189fb82e48a2bc74ee1e17d636798e16fe04e248c838cebea99dcd7f1c41d5e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Python
  - R
  - R (>= 3.5) with stringr
  - STAR >= 2.7.2d
  - TensorFlow
  - bedtools
  - pyensembl
  - pysam
  - samtools
  - scFusion
  - stringr
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# oncogene-expression-quantification

## Summary

Quantifies expression of an oncogene (e.g., WHSC1) in fusion-positive versus fusion-negative single cells, and profiles per-base sequencing coverage across the fusion breakpoint, to test whether a detected fusion drives oncogene overexpression as in scFusion's multiple myeloma IgH-WHSC1 analysis (Fig 7d-e).

## When to use

Use when a gene fusion (e.g., IgH-WHSC1) has been detected per cell in scRNA-seq data and you must test whether the fusion partner oncogene is overexpressed in fusion-bearing cells.

## When NOT to use

- No fusion calls exist yet — scFusion (STAR, samtools, bedtools, statistical/deep-learning filters) must first produce per-cell fusion calls before expression stratification is meaningful
- The data are not from a Smart-Seq single-cell protocol or the fusion is not expressed in the sequenced cells
- The candidate oncogene lies far from the fusion breakpoint so coverage and expression cannot be linked to the fusion event

## Inputs

- Per-cell fusion call table (fusion_results.csv) labeling IgH-WHSC1-positive vs -negative cells with breakpoints (1902353 in SMM0, 1905943 in RRMM2)
- Per-cell WHSC1 expression values from STAR-aligned single-cell RNA-seq counts
- BAM alignments (STAR output, samtools-processed) for per-base coverage profiling of WHSC1

## Outputs

- WHSC1 expression comparison between fusion-positive and fusion-negative cells (Fig 7d-e equivalent)
- WHSC1 sequencing-coverage profile showing increase downstream of the IgH-WHSC1 breakpoints
- WHSC1 expression summary appended to fusion_results.csv

## How to apply

First obtain per-cell fusion calls (fusion_results.csv from scFusion) so each of the single cells (e.g., 597 Smart-Seq cells from SMM0, RRMM2, etc.) is labeled fusion-positive or fusion-negative for the fusion of interest (IgH-WHSC1, breakpoints at 1902353 in SMM0 and 1905943 in RRMM2). In R (>= 3.5, using stringr), extract WHSC1 expression per cell from the aligned counts and stratify cells by IgH-WHSC1 status; compare WHSC1 expression between fusion-positive and fusion-negative cells to reproduce the overexpression shown in Fig 7d-e. Additionally, plot per-base sequencing coverage of WHSC1 from the STAR/samtools alignments to verify that coverage sharply increases downstream of the fusion breakpoints, the signature of IgH enhancer-driven overexpression. Treat the joint finding — significantly higher WHSC1 expression in fusion-positive cells plus a downstream coverage jump — as the validation that the fusion has transcriptional impact.

## Related tools

- **scFusion** (Produces the per-cell IgH-WHSC1 fusion calls and breakpoint table that stratify cells into fusion-positive and fusion-negative groups) — https://github.com/ZijieJin/scFusion
- **R (>= 3.5) with stringr** (Quantifies and compares WHSC1 expression in fusion-positive versus fusion-negative cells)
- **STAR >= 2.7.2d** (Aligns single-cell libraries so WHSC1 per-base coverage downstream of the breakpoint can be profiled) — https://github.com/alexdobin/STAR
- **pyensembl** (Annotates fusion gene partners (gene_name, gene_type from the GTF) so the oncogene WHSC1 is correctly identified) — https://github.com/openvax/pyensembl

## Examples

```
Rscript -e 'library(stringr); pos <- str_detect(fusion_results$fusions,"WHSC1"); t.test(WHSC1_expr[pos], WHSC1_expr[!pos])'
```

## Evaluation signals

- WHSC1 expression is significantly higher in IgH-WHSC1 fusion-positive cells than in other cells (Fig 7d-e equivalent)
- WHSC1 sequencing coverage shows a sharp increase downstream of the fusion breakpoints rather than uniform coverage
- The stratification matches the known breakpoints: position 1902353 in all 52 SMM0 cells and 1905943 in all 20 RRMM2 cells
- Overall fusion calls reproduce the expected summary: 38 total fusions with 94.7% involving immunoglobulin genes
- Expression and coverage summaries are written to fusion_results.csv alongside the fusion calls

## Limitations

- scFusion did not find WHSC1 fusions in single cells from RRMM2 — WHSC1 expression there may be activated by mechanisms other than WHSC1 fusions, or the fusion may have been missed
- RNA-based fusion and expression analysis has reduced sensitivity for fusions with low expression
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; more and deeper sequencing of single cells is needed

## Evidence

- [abstract] IgH-WHSC1 overexpression finding: "in a multiple myeloma dataset, scFusion detects the known recurrent fusion IgH-WHSC1, which is associated with overexpression of the WHSC1 oncogene."
- [intro] false positive control models: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [methods] R tooling: "python module: pyensembl (https://github.com/openvax/pyensembl)"
- [discussion] RRMM2 caveat: "It is likely that the expression of WHSC1 was activated in RRMM2 by mechanisms other than WHSC1 fusions or that the fusion was missed"
- [readme] scFusion scope: [evidence span withheld — anchor status: external_doc]
