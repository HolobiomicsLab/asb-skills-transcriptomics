---
name: gmm-based-baf-shift-threshold-estimation
description: Use when CNV calls from RNA-seq expression-based HMM segmentation need
  to be validated or corrected by an allelic shift (LOH) signal, and a threshold for
  calling a BAF shift 'significant' must be learned from the data rather than fixed
  a priori.
when_to_use_negative:
- No BAF/LOH signal is available (e.g., expression-only input without allele-frequency
  data) — states 2 and 4 cannot be gated by shift evidence and the GMM has nothing
  to fit.
- 3′-only sequencing (e.g., DropSeq) with sparse variants, where BAF shift is measured
  only near gene 3′ ends and the shift signal is adversely impacted.
- Samples where a few cells dominate the pooled BAF shift signal (very uneven read
  counts in scRNA-seq), biasing the mixture fit.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: CaSpER
  role: Computes LOH shifts per segment (calculateLOHShiftsForEachSegment), fits the
    BAF-shift signal with mclust, and integrates HMM segment states with allele-frequency
    shift information via assignStates within runCaSpER.
  repo: https://github.com/akdess/CaSpER
- name: mclust
  role: Gaussian-mixture-model fitting used to threshold allele-frequency (BAF) shifts
    learned from the segment-level shift distribution.
- name: BAFExtract
  role: Generates the B-allele frequency shift profiles from mapped RNA-seq reads
    (pileup + BAF profile) that feed the GMM thresholding, without requiring a heterozygous
    variant call set.
  repo: https://github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:86941b4bc469069a61d4295c632f789cb68d8f742bf52eb97b9133c2252e7be7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  - mclust
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# GMM-based BAF-shift threshold estimation

## Summary

Fits a Gaussian-mixture model (mclust) to the smoothed B-allele frequency (BAF) shift signal computed per HMM segment, deriving a data-driven threshold on allelic shift that gates which expression-derived HMM states receive final deletion/amplification calls in CaSpER.

## When to use

Use when CNV calls from RNA-seq expression-based HMM segmentation need to be validated or corrected by an allelic shift (LOH) signal, and a threshold for calling a BAF shift 'significant' must be learned from the data rather than fixed a priori.

## When NOT to use

- No BAF/LOH signal is available (e.g., expression-only input without allele-frequency data) — states 2 and 4 cannot be gated by shift evidence and the GMM has nothing to fit.
- 3′-only sequencing (e.g., DropSeq) with sparse variants, where BAF shift is measured only near gene 3′ ends and the shift signal is adversely impacted.
- Samples where a few cells dominate the pooled BAF shift signal (very uneven read counts in scRNA-seq), biasing the mixture fit.

## Inputs

- Smoothed BAF/LOH shift signal per segment (output of calculateLOHShiftsForEachSegment on BAFExtract-derived BAF profiles)
- 5-state HMM segmentation of multiscale smoothed expression signal (states 1–5: homozygous deletion, heterozygous deletion, neutral, one-copy amplification, multi-copy amplification)
- Casper object with gene annotation, cytoband, control sample ids, cnv.scale=3, loh.scale=3, matrix.type='normalized', expr.cutoff=4.5, method='iterative'

## Outputs

- GMM-derived BAF-shift threshold distinguishing shift-positive from background segments
- Final per-segment CNV calls (deletion/amplification/neutral) from assignStates integrating HMM state and BAF evidence
- Final.objects list over N×M scale comparisons feeding extractLargeScaleEvents, extractSegmentSummary, and gene.matrix

## How to apply

First compute LOH/BAF shifts for every HMM segment with calculateLOHShiftsForEachSegment on the smoothed BAF/LOH signal produced by BAFExtract (readBAFExtractOutput with sequencing.type set to 'bulk' or single-cell). Then fit a Gaussian-mixture model to the segment-level BAF-shift distribution using the mclust package, so that the mixture components separate background/no-shift segments from segments carrying a genuine allele-frequency shift; the fitted mixture supplies the shift threshold used by assignStates. Finally, combine states with the expression HMM results: segments in HMM state 2 (heterozygous deletion) or state 4 (one-copy amplification) are assigned a final deletion/amplification call only if their BAF shift exceeds the GMM-derived threshold, whereas states 1 and 5 (homozygous deletion and multi-copy amplification) are called directly without BAF evidence. This integration is executed for all N×M pairwise expression/BAF scale comparisons inside runCaSpER(object, removeCentromere=T, cytoband=cytoband, method='iterative') and harmonized with extractLargeScaleEvents(final.objects, thr=0.75) and extractSegmentSummary(final.objects) (gamma=6 consistency cutoff).

## Related tools

- **CaSpER** (Computes LOH shifts per segment (calculateLOHShiftsForEachSegment), fits the BAF-shift signal with mclust, and integrates HMM segment states with allele-frequency shift information via assignStates within runCaSpER.) — https://github.com/akdess/CaSpER
- **mclust** (Gaussian-mixture-model fitting used to threshold allele-frequency (BAF) shifts learned from the segment-level shift distribution.)
- **BAFExtract** (Generates the B-allele frequency shift profiles from mapped RNA-seq reads (pileup + BAF profile) that feed the GMM thresholding, without requiring a heterozygous variant call set.) — https://github.com/akdess/BAFExtract

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"); finalChrMat <- extractLargeScaleEvents(final.objects, thr=0.75)
```

## Evaluation signals

- Segments with HMM state 1 or 5 receive deletion/amplification calls regardless of BAF evidence, while state-2/state-4 calls appear only where the GMM shift threshold is exceeded.
- Chromosome-arm large-scale calls are consistent across at least thr=0.75 (i.e., ≥75%, 7 of 9) of the N×M expression/BAF scale comparisons in extractLargeScaleEvents.
- Segment-level calls are reported only where consistent calls exceed the gamma=6 cutoff out of the N×M comparisons in extractSegmentSummary.
- Gene-by-sample CNV matrix (gene.matrix) values are restricted to {0 neutral, 1 amplification, -1 deletion}.
- Fitted mclust mixture separates a background component from a shift-positive component of the segment BAF-shift distribution rather than collapsing to one component.

## Limitations

- 3′-only sequencing data degrade the BAF shift signal because shift is measured only for variants close to the 3′ ends of genes; the authors propose a 3′-bias-aware BAF-shift generation model as future work.
- In single-cell data, cells with substantially large numbers of reads may dominate the pooled BAF shift signal and create biases.
- RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal in complex ways and bias CNV detection.

## Evidence

- [methods] we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [abstract] CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling.: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [readme] BAFExtract generates the B-allele frequency shifts from mapped reads alone without the need for a heterozygous variant call set.: [evidence span withheld — anchor status: external_doc]
- [methods] final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"): "final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")"
- [discussion] the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes.: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
