---
name: ploidy-relative-copy-number-conversion
description: Use when validating RNA-seq-derived CNV calls against matched DNA-seq
  absolute copy-number ground truth in single cells, so absolute copy numbers must
  be expressed relative to ploidy before computing TPR/FPR.
when_to_use_negative:
- Ground truth is already relative copy number or a binary loss/gain label — no ploidy
  conversion is needed
- No matched DNA copy-number data exist (e.g., MM135 10x cells, where exact ground
  truth CNV calls are unknown); rely on tool concordance instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CaSpER
  role: Calls gene-based and large-scale CNV events from single-cell RNA-seq that
    are compared against the ploidy-converted relative copy-number ground truth
  repo: https://github.com/akdess/CaSpER
- name: HoneyBADGER
  role: Comparison method run on the same seven DR-Seq cells to verify its reported
    event count against the converted ground truth
- name: BAFExtract
  role: Generates B-allele frequency shifts from mapped reads used by CaSpER when
    allelic shift information is integrated with expression HMM states
  repo: https://github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:c75e2a93c1e6215a65ddf0d4b95d5a4c2087b03ff751e00851f38aad85df7705
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  - HoneyBADGER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ploidy-relative-copy-number-conversion

## Summary

Converts absolute DNA copy-number ground truth (e.g., from DR-Seq simultaneous RNA/DNA measurements) into relative copy numbers via a ploidy formula, enabling direct comparison against RNA-seq-based CNV calls from CaSpER. This normalization is required because CaSpER reports relative (loss/heterozygous-loss/gain) states rather than absolute copy counts.

## When to use

Use when validating RNA-seq-derived CNV calls against matched DNA-seq absolute copy-number ground truth in single cells, so absolute copy numbers must be expressed relative to ploidy before computing TPR/FPR.

## When NOT to use

- Ground truth is already relative copy number or a binary loss/gain label — no ploidy conversion is needed
- No matched DNA copy-number data exist (e.g., MM135 10x cells, where exact ground truth CNV calls are unknown); rely on tool concordance instead

## Inputs

- DR-Seq single-cell RNA-seq normalized expression matrix for seven breast cancer cells
- Matched DNA-seq absolute copy-number ground truth per cell
- Gene annotation and cytoband files (hg19)
- BAF shift profile from BAFExtract (optional, for CaSpER integration)

## Outputs

- Per-cell relative copy-number ground-truth profile
- Gene-based CNV call matrix (cells x genes)
- Large-scale arm-level CNV event matrix (extractLargeScaleEvents output)
- TPR/FPR of deletion and amplification calls against ground truth

## How to apply

Load the DR-Seq single-cell RNA-seq normalized expression matrix for the seven breast cancer cells together with the matched DNA-seq absolute copy-number ground truth, and convert absolute copy numbers to relative copy numbers using the ploidy formula from the CaSpER Methods. Build the CaSpER analysis object with CreateCasperObject() supplying annotation, cytoband, loh.name.mapping, sequencing.type='single-cell', and control.sample.ids. Call CNVs with runCaSpER(object, removeCentromere=T, cytoband=cytoband, method='iterative'), which performs HMM on all expression scales and integrates HMM segment states with allele-frequency (BAF) shift information across pairwise scale comparisons. Summarize arm-level events with extractLargeScaleEvents(final.objects, thr=0.75), then compute true-positive and false-positive rates of the gene-based and large-scale CNV calls against the relative-copy-number ground truth at the default consistency threshold (75%, at least 7 of 9 scale comparisons) and at gamma=1.

## Related tools

- **CaSpER** (Calls gene-based and large-scale CNV events from single-cell RNA-seq that are compared against the ploidy-converted relative copy-number ground truth) — https://github.com/akdess/CaSpER
- **HoneyBADGER** (Comparison method run on the same seven DR-Seq cells to verify its reported event count against the converted ground truth)
- **BAFExtract** (Generates B-allele frequency shifts from mapped reads used by CaSpER when allelic shift information is integrated with expression HMM states) — https://github.com/akdess/BAFExtract

## Examples

```
object <- CreateCasperObject(raw.data=expr.matrix, annotation=annotation, cytoband=cytoband, loh.name.mapping=loh.mapping, sequencing.type="single-cell", control.sample.ids=controls); final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")
```

## Evaluation signals

- Default-parameter benchmarks reproduced exactly: 32% TPR / 1.6% FPR for large-scale deletions and 49% TPR / 3.8% FPR for large-scale amplifications
- Relaxed gamma=1 benchmarks reproduced: 45% TPR / 3% FPR (deletions) and 62.6% TPR / 8.6% FPR (amplifications)
- HoneyBADGER on the same seven cells reports zero CNV events
- Large-scale event calls pass the thr=0.75 consistency threshold (>=7 of 9 pairwise scale comparisons) before TPR/FPR computation

## Limitations

- Dynamic and highly non-uniform genome coverage by RNA-seq signal makes it hard to distinguish copy-number events from expression-level variation
- 3'-only sequencing (e.g., DropSeq) degrades the BAF shift signal because shifts are measured only for variants near 3' gene ends
- Cells with substantially large read counts may dominate pooled BAF shift signal and create biases in single-cell analyses

## Evidence

- [other] converting absolute copy numbers to relative copy numbers using the ploidy formula from Methods: [evidence span withheld — anchor status: not_found]
- [methods] HMM on all expression scales with BAF shift integration: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [methods] Casper object required for CNV analysis: "The casper object is required for performing CNV analysis on single-cell and bulk RNA-Seq."
- [methods] Large-scale event summarization threshold: "finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75)"
- [readme] CaSpER published reference: "CaSpER is published in Nature Communications: https://www.nature.com/articles/s41467-019-13779-x"
