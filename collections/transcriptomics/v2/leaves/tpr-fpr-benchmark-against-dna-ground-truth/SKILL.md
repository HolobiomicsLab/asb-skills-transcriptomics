---
name: tpr-fpr-benchmark-against-dna-ground-truth
description: Use when evaluating CNV calls from RNA-seq data against matched DNA-seq
  absolute copy-number ground truth, e.g. single cells with simultaneous RNA and copy-number
  measurements.
when_to_use_negative:
- No matched DNA-seq copy-number ground truth is available for the same cells (e.g.
  the MM135 10x multiple myeloma dataset, where ground truth CNV calls are unknown)
  — use concordance analysis instead of TPR/FPR
- The goal is de novo CNV discovery rather than caller accuracy assessment
- Input is bulk RNA-seq from a tumor where per-cell absolute copy number cannot serve
  as ground truth
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0203
tools:
- name: CaSpER
  role: RNA-seq CNV caller being benchmarked; produces gene-based and large-scale
    arm-level CNV calls via multiscale HMM and BAF shift integration
  repo: https://github.com/akdess/CaSpER
- name: HoneyBADGER
  role: Comparator CNV detector run on the same seven scRNA-seq cells to cross-check
    event counts
- name: BAFExtract
  role: Generates BAF shift profiles from mapped reads used by CaSpER's allelic shift
    integration
  repo: https://github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:d40a0051bb8856046b7d6aa6e1dbdc463b562660524f96b2f3e9641ab5d21315
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  - HoneyBADGER
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# tpr-fpr-benchmark-against-dna-ground-truth

## Summary

Benchmarks RNA-seq-based CNV caller output (CaSpER, with HoneyBADGER as a comparator) against matched DNA-seq copy-number ground truth by computing true-positive and false-positive rates for large-scale deletion and amplification events at multiple gamma/threshold settings.

## When to use

Use when evaluating CNV calls from RNA-seq data against matched DNA-seq absolute copy-number ground truth, e.g. single cells with simultaneous RNA and copy-number measurements.

## When NOT to use

- No matched DNA-seq copy-number ground truth is available for the same cells (e.g. the MM135 10x multiple myeloma dataset, where ground truth CNV calls are unknown) — use concordance analysis instead of TPR/FPR
- The goal is de novo CNV discovery rather than caller accuracy assessment
- Input is bulk RNA-seq from a tumor where per-cell absolute copy number cannot serve as ground truth

## Inputs

- DR-Seq single-cell RNA-seq normalized expression matrix for seven breast cancer cells
- Matched DNA-seq absolute copy-number ground truth for the same seven cells
- Gene annotation and cytoband definitions (hg19)
- LOH/BAF shift files from BAFExtract (readBAFExtractOutput)
- control.sample.ids list of normal/control cells

## Outputs

- Large-scale arm-level CNV event matrix (extractLargeScaleEvents output at thr=0.75)
- Gene-based CNV matrix (rows = cells, columns = genes)
- TPR and FPR values for deletion and amplification events at default and gamma=1 settings
- HoneyBADGER CNV event count for the same cells

## How to apply

Convert DNA-seq absolute copy numbers to relative copy numbers using the ploidy formula from the CaSpER Methods. Run the RNA-based caller (CreateCasperObject with sequencing.type='single-cell' and control.sample.ids, then runCaSpER(object, removeCentromere=T, cytoband=cytoband, method='iterative')) and summarize arm-level events with extractLargeScaleEvents(final.objects, thr=0.75). Score calls at the default consistency threshold (75%, i.e. at least 7 of 9 pairwise scale comparisons) and at gamma=1, computing TPR/FPR separately for large-scale deletions and amplifications against the relative-copy-number ground truth. Run HoneyBADGER on the same cells as a comparator and record its event count. Acceptance is matching the published benchmarks: default gamma 32% TPR / 1.6% FPR (deletion) and 49% TPR / 3.8% FPR (amplification); gamma=1 gives 45% TPR / 3% FPR (deletion) and 62.6% TPR / 8.6% FPR (amplification); HoneyBADGER reports 0 events.

## Related tools

- **CaSpER** (RNA-seq CNV caller being benchmarked; produces gene-based and large-scale arm-level CNV calls via multiscale HMM and BAF shift integration) — https://github.com/akdess/CaSpER
- **HoneyBADGER** (Comparator CNV detector run on the same seven scRNA-seq cells to cross-check event counts)
- **BAFExtract** (Generates BAF shift profiles from mapped reads used by CaSpER's allelic shift integration) — https://github.com/akdess/BAFExtract

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"); finalChrMat <- extractLargeScaleEvents(final.objects, thr=0.75)
```

## Evaluation signals

- Default-parameter TPR/FPR must match 32% TPR / 1.6% FPR (large-scale deletion) and 49% TPR / 3.8% FPR (large-scale amplification)
- gamma=1 run must reproduce 45% TPR / 3% FPR (deletion) and 62.6% TPR / 8.6% FPR (amplification)
- HoneyBADGER on the same seven DR-Seq cells must report exactly 0 CNV events
- extractLargeScaleEvents must be called with thr=0.75, corresponding to the default consistency of at least 7 of 9 pairwise scale comparisons
- Ground truth must be converted from absolute to relative copy numbers using the ploidy formula from Methods before scoring

## Limitations

- BAF shift signal is degraded for 3'-only sequencing (DropSeq-type) because BAF shift is measured only for variants near gene 3' ends
- Cells with a substantially large number of reads may dominate the pooled BAF shift signal and create biases in scRNA-seq
- RNA editing, PCR duplication artefacts, and sequencing errors may bias CNV detection through the BAF shift signal
- Ground truth CNV calls are unknown for some validation datasets (e.g. MM135), limiting TPR/FPR-style evaluation

## Evidence

- [methods] finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75): "finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75)"
- [methods] runCaSpER call: "final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")"
- [methods] ground truth conversion: "We assign a large-scale CNV call to every chromosome arm for each of the N×M pairwise scale comparisons."
- [discussion] 3' bias limitation: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
- [discussion] read-depth bias limitation: "If some cells have a substantially large number of reads, they may dominate the BAF shift signal and may potentially create biases."
- [discussion] HoneyBADGER comparator context: "Even though the exact ground truth CNV calls are unknown for this study, we show the concordance of CaSpER CNV calls with HONEYBADGER"
