---
name: baf-shift-integration-for-cnv-calls
description: Use when calling CNV events from bulk or single-cell RNA-seq where expression-based
  segmentation alone is ambiguous and BAF shift information from mapped reads (no
  heterozygous variant call set or SNV calls) should be integrated to confirm and
  consolidate calls at multiple scales.
when_to_use_negative:
- 3′-only sequencing protocols (e.g. DropSeq-style 10x data) with sparse variants
  near gene 3′ ends, where BAF shift signal is adversely impacted and pooling across
  cells is the only mitigation
- Analyses lacking BAM files or any variant-containing reads, so no BAF shift can
  be computed — expression-only CNV inference must be used instead
- Sample sets where a few cells carry a substantially large number of reads, which
  may dominate the pooled BAF shift signal and create biases in single-cell mode
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: CaSpER
  role: Runs multiscale HMM on the expression signal and integrates HMM segment states
    with BAF shift (allelic shift) information across pairwise scale comparisons;
    summarizes and visualizes the consolidated CNV calls
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates the B-allele frequency shift profile directly from mapped reads
    without a heterozygous variant call set; its output is read into CaSpER via readBAFExtractOutput
  repo: https://github.com/akdess/BAFExtract
- name: HoneyBADGER
  role: Expression-only CNV caller used as a comparison point; it does not use BAF
    shift integration and was shown to report zero CNV events on the DR-Seq benchmark
    cells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:210fd7697aad8b7fdf09781da92c27792aa477ae166653574a5bf8eb916776db
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# baf-shift-integration-for-cnv-calls

## Summary

Integrates B-allele frequency (BAF) shift signals (allelic shift measuring LOH) with multiscale HMM segmentation of genome-wide gene expression in CaSpER, so CNV calls are corrected and consolidated across pairwise comparisons of expression and BAF scales before large-scale and gene-based summarization.

## When to use

Use when calling CNV events from bulk or single-cell RNA-seq where expression-based segmentation alone is ambiguous and BAF shift information from mapped reads (no heterozygous variant call set or SNV calls) should be integrated to confirm and consolidate calls at multiple scales.

## When NOT to use

- 3′-only sequencing protocols (e.g. DropSeq-style 10x data) with sparse variants near gene 3′ ends, where BAF shift signal is adversely impacted and pooling across cells is the only mitigation
- Analyses lacking BAM files or any variant-containing reads, so no BAF shift can be computed — expression-only CNV inference must be used instead
- Sample sets where a few cells carry a substantially large number of reads, which may dominate the pooled BAF shift signal and create biases in single-cell mode

## Inputs

- RNA-seq BAM file(s) (bulk or merged single-cell)
- Genome chromosome list file (e.g. hg19.list / hg38.list from fetchChromSizes)
- Preprocessed genome FASTA pileup directory (BAFExtract -preprocess_FASTA output)
- Normalized gene expression matrix (rows=genes, columns=samples/cells)
- Gene annotation (e.g. ensembl_gene_id via generateAnnotation, ishg19=T), cytoband table, centromere coordinates
- Control sample IDs (control.sample.ids) and loh.name.mapping for BAF sample matching

## Outputs

- BAF/LOH shift profile file from BAFExtract (e.g. test.snp)
- Per-arm large-scale CNV matrix consolidated at thr=0.75 (finalChrMat)
- Gene-by-sample CNV call matrix (gene-based summarization)
- Segment-based CNV summary (extractSegmentSummary output)
- Visualizations: genome-wide expression heatmaps at multiple smoothing scales, large-scale CNV plots, BAF shift plots across all samples

## How to apply

First extract BAF values from RNA-seq BAM files with BAFExtract: build a compressed pileup per SAM stream (e.g. minimum mapping quality 50), then run get_SNVs_per_pileup against the preprocessed genome FASTA/pileup directory (hg19 or hg38) to produce a BAF profile, without any prior heterozygous variant call set. Load the BAF shift signal into the CaSpER object with readBAFExtractOutput (sequencing.type='bulk' or 'single-cell'), then run runCaSpER(..., method='iterative'), which performs HMM on all scales of the expression signal and integrates HMM segment states with allele frequency shift information across pairwise scale comparisons. Each chromosome arm receives a large-scale CNV call per N×M pairwise scale comparison; consolidate with extractLargeScaleEvents(final.objects, thr=0.75) so an event is kept when at least 75% of scale comparisons (e.g. ≥7 of 9) agree, and optionally derive gene-based and segment-based summaries. The rationale is that BAF shifts (LOH-like allelic imbalance) provide an orthogonal, expression-independent signal that corrects CNV calls made purely from smoothed expression, which is confounded by non-uniform RNA-seq coverage.

## Related tools

- **CaSpER** (Runs multiscale HMM on the expression signal and integrates HMM segment states with BAF shift (allelic shift) information across pairwise scale comparisons; summarizes and visualizes the consolidated CNV calls) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates the B-allele frequency shift profile directly from mapped reads without a heterozygous variant call set; its output is read into CaSpER via readBAFExtractOutput) — https://github.com/akdess/BAFExtract
- **HoneyBADGER** (Expression-only CNV caller used as a comparison point; it does not use BAF shift integration and was shown to report zero CNV events on the DR-Seq benchmark cells)

## Examples

```
samtools view SRR1295366.sorted.bam | ./bin/BAFExtract -generate_compressed_pileup_per_SAM stdin hg38.list test 50 0; ./bin/BAFExtract -get_SNVs_per_pileup hg38.list test ./hg38/ 20 4 0.1 test.snp
```

## Evaluation signals

- On DR-Seq breast cancer single cells with matched copy-number ground truth, integrated CaSpER calls should reach ~32% TPR / 1.6% FPR for large-scale deletions and ~49% TPR / 3.8% FPR for large-scale amplifications at default parameters (45%/3% and 62.6%/8.6% at gamma=1)
- The large-scale event matrix from extractLargeScaleEvents(thr=0.75) should contain arm-level calls supported by at least 7 of 9 scale comparisons, i.e. consistency ≥75%
- The BAFExtract output should contain variant positions with BAF values generated solely from the pileup, with no prior heterozygous variant call set supplied
- Concordance of CaSpER CNV calls with HoneyBADGER on datasets like MM135 (n=947 cells) serves as an independent sanity check when ground truth is unknown
- Per N×M pairwise scale comparisons, each chromosome arm should receive a large-scale CNV assignment before thresholding — a missing arm assignment indicates the integration step failed

## Limitations

- 3′-only sequencing (such as DropSeq) degrades the BAF shift signal because shifts are measured only for variants near the 3′ ends of genes; pooling effects that mitigate this are stated as a belief, not validated
- Cells with a substantially large number of reads may dominate the pooled BAF shift signal in single-cell mode, creating biases
- RNA editing, PCR duplication artefacts, and sequencing errors may bias CNV detection through complex effects on the BAF shift signal
- BAFExtract MAPQ filtering (example uses minimum mapping quality 50) can leave no surviving reads depending on the aligner's MAPQ conventions
- For MM135, the exact ground truth CNV calls are unknown, so concordance rather than accuracy is assessed

## Evidence

- [abstract] CaSpER integrates multiscale smoothing of expression signal and allelic shift signals (measuring LOH) for CNV calling: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [methods] Pairwise scale integration of HMM states with BAF shift information: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [methods] Per-arm large-scale CNV assignment for each pairwise scale comparison: "We assign a large-scale CNV call to every chromosome arm for each of the N×M pairwise scale comparisons."
- [readme] BAFExtract generates BAF shifts from mapped reads without a heterozygous variant call set: [evidence span withheld — anchor status: external_doc]
- [discussion] 3′-only sequencing degrades BAF shift signal: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
- [discussion] High-read cells may dominate pooled BAF shift signal: "If some cells have a substantially large number of reads, they may dominate the BAF shift signal and may potentially create biases."
