---
name: hmm-based-copy-number-segmentation
description: Use when the goal is to infer CNV/LOH events from raw RNA-seq signal
  profiles (bulk or single-cell) without a high-quality heterozygous SNV call set,
  e.g., tumor scRNA-seq clonal architecture questions.
when_to_use_negative:
- The goal is transcriptomic analysis only and no CNV/LOH inference is needed — the
  skill probes the genomic landscape, not expression signatures per se.
- Input data are already curated CNV calls or a feature table of copy numbers rather
  than raw RNA-seq signal profiles needing segmentation.
- Only 3′-end (e.g., DropSeq-style) data with very sparse variants are available and
  no pooling is possible, since BAF shift signal may be adversely impacted.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: CaSpER
  role: Performs the multiscale HMM segmentation of expression signal and integration
    with BAF allele-shift states; provides CreateCasperObject, runCaSpER, extractLargeScaleEvents,
    extractSegmentSummary, gene.matrix, and plotting functions.
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates B-allele frequency shift profiles from mapped RNA-seq reads alone
    (pileup + BAF profile) to feed the LOH/allelic-shift integration step without
    a heterozygous variant call set.
  repo: https://github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:c5736a6708633152da705b6619e9a9c84b5e6e296389239d35cb93945e74e70e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hmm-based-copy-number-segmentation

## Summary

Segment genome-wide RNA-seq signal into copy-number states with CaSpER's Hidden Markov Model, which runs on all multiscale-smoothed expression scales and integrates BAF allele-shift states to call focal and large-scale CNVs from bulk or single-cell RNA-seq.

## When to use

Use when the goal is to infer CNV/LOH events from raw RNA-seq signal profiles (bulk or single-cell) without a high-quality heterozygous SNV call set, e.g., tumor scRNA-seq clonal architecture questions.

## When NOT to use

- The goal is transcriptomic analysis only and no CNV/LOH inference is needed — the skill probes the genomic landscape, not expression signatures per se.
- Input data are already curated CNV calls or a feature table of copy numbers rather than raw RNA-seq signal profiles needing segmentation.
- Only 3′-end (e.g., DropSeq-style) data with very sparse variants are available and no pooling is possible, since BAF shift signal may be adversely impacted.

## Inputs

- genome-wide gene expression matrix (bulk RNA-seq counts or normalized single-cell matrix, e.g., matrix.type="normalized")
- BAFExtract output B-allele frequency files (read via readBAFExtractOutput, sequencing.type="bulk"-style per-sample BAF files)
- hg19 cytoband and centromere annotations
- gene annotation (generateAnnotation, e.g., id_type="ensembl_gene_id", ishg19=T)

## Outputs

- final.objects with HMM segment states per smoothing scale
- large-scale (chromosome-arm) CNV call matrix (extractLargeScaleEvents)
- segment-based CNV summary (extractSegmentSummary)
- cell-by-gene CNV matrix (gene.matrix)
- visualizations: plotHeatmap, plotLargeScaleEvent, plotBAFAllSamples/plotBAFOneSample, plotSingleCellLargeScaleEventHeatmap, plotSCellCNVTree

## How to apply

Build a casper object with CreateCasperObject (cnv.scale=3, loh.scale=3, method="iterative", matrix.type="normalized"), supplying hg19 cytoband, gene annotation, and for single-cell data a loh.name.mapping that maps each cell to its patient-pooled BAFExtract BAF signal (one-to-many). Run runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"), which performs HMM on all scales of expression signal and integrates HMM segment states with Allele Frequency Shift information. Summarize calls across the N×M pairwise scale comparisons: extractLargeScaleEvents(final.objects, thr=0.75) for chromosome-arm events (calls must agree in ≥7 of 9 pairwise comparisons), extractSegmentSummary (gamma=6 consistent-call threshold) for segments, and gene.matrix for a cell-by-gene CNV matrix. Visualize with plotHeatmap, plotLargeScaleEvent, plotBAFOneSample/plotBAFAllSamples, and for single-cell GBM further validate subclone structure with plotSingleCellLargeScaleEventHeatmap, extractMUAndCooccurence (Fisher's exact test), and plotSCellCNVTree.

## Related tools

- **CaSpER** (Performs the multiscale HMM segmentation of expression signal and integration with BAF allele-shift states; provides CreateCasperObject, runCaSpER, extractLargeScaleEvents, extractSegmentSummary, gene.matrix, and plotting functions.) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates B-allele frequency shift profiles from mapped RNA-seq reads alone (pileup + BAF profile) to feed the LOH/allelic-shift integration step without a heterozygous variant call set.) — https://github.com/akdess/BAFExtract

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"); finalChrMat <- extractLargeScaleEvents(final.objects, thr=0.75)
```

## Evaluation signals

- Large-scale CNV calls are consistent across at least 75% (thr=0.75, ≥7 of 9) pairwise scale comparisons.
- For the 430-cell single-cell GBM dataset, expected patterns reproduce: chr7 amplification/chr10 deletion across tumor cells, mutually exclusive MGH31 subclone pairs (5q amp vs 14q del; 1p amp vs 13q del), MGH28 (8q:20p, 5q:19p), MGH30 (6p:7p).
- Normal oligodendrocytes cluster separately from tumor cells in the large-scale event heatmap.
- Segment summarization with gamma=6 yields a coherent final segment set and the CNV-based clonal tree topology (plotSCellCNVTree) matches the reported architecture.
- Co-occurrence/exclusivity patterns are significant under Fisher's exact test (extractMUAndCooccurence).

## Limitations

- CNV inference from RNA-seq is challenging because dynamic, highly non-uniform genome coverage makes it hard to distinguish copy-number events from expression-level variation.
- With 3′-only sequencing (e.g., DropSeq), BAF shift signal may be adversely impacted since only variants near 3′ gene ends are measured; read pooling is believed but not validated to soothe 3′ bias.
- Cells with substantially large read numbers may dominate the pooled BAF shift signal and create biases; RNA editing, PCR duplication artefacts, and sequencing errors may also bias CNV detection via the BAF shift signal.

## Evidence

- [abstract] CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling.: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [methods] we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [methods] finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75): "finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75)"
- [methods] final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"): "final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")"
- [readme] BAFExtract generates the B-allele frequency shifts from mapped reads alone without the need for a heterozygous variant call set.: [evidence span withheld — anchor status: external_doc]
- [discussion] the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes.: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
