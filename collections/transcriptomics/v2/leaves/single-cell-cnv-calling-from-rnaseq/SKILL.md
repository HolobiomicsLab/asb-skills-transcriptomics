---
name: single-cell-cnv-calling-from-rnaseq
description: Use when analyzing scRNA-seq data (e.g., 430-cell GBM dataset) to call
  large-scale CNVs per cell, identify mutually exclusive/co-occurring CNV subclones
  within a patient, and derive a CNV-based clonal phylogenetic tree.
when_to_use_negative:
- Input is already a DNA-seq copy-number call set or feature table — CaSpER is designed
  for CNV inference from RNA-seq signal, not re-analysis of existing CNV calls
- 3′-only sequencing data (e.g., DropSeq/10x-style) where BAF shift signal is adversely
  impacted because variants are measured only near 3′ ends of genes — pooled BAF signal
  may be too sparse
- Bulk RNA-seq cohort-level focal CNV discovery without single-cell resolution needs
  — the bulk variant of this skill applies instead
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_2228
tools:
- name: CaSpER
  role: R package performing multiscale smoothing, HMM-based CNV calling integrated
    with BAF allele-shift states, summarization, mutual-exclusivity testing, and clonal
    tree inference
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates B-allele frequency shift profiles from mapped RNA-seq reads (pileup
    then BAF profile) without a heterozygous variant call set; output feeds CaSpER's
    loh input
  repo: https://github.com/akdess/BAFExtract
- name: inferCNV
  role: Alternative method enabling only visual inspection of scRNA-seq expression
    profiles, cited for contrast
- name: HoneyBADGER
  role: Alternative scRNA-seq CNV detection method used in the paper for concordance
    checks of CaSpER calls
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:db86e0f22e8b1cc23af7e6513696e34b22aa2065285b0b085a0df859d7cf5f85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  - HoneyBADGER
  - inferCNV
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_2228
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-cnv-calling-from-rnaseq

## Summary

Infer single-cell copy-number variant (CNV) architecture — including mutually exclusive subclones and clonal trees — from scRNA-seq expression data by integrating multiscale-smoothed expression signal with patient-pooled BAF allele-shift signal using CaSpER.

## When to use

Use when analyzing scRNA-seq data (e.g., 430-cell GBM dataset) to call large-scale CNVs per cell, identify mutually exclusive/co-occurring CNV subclones within a patient, and derive a CNV-based clonal phylogenetic tree.

## When NOT to use

- Input is already a DNA-seq copy-number call set or feature table — CaSpER is designed for CNV inference from RNA-seq signal, not re-analysis of existing CNV calls
- 3′-only sequencing data (e.g., DropSeq/10x-style) where BAF shift signal is adversely impacted because variants are measured only near 3′ ends of genes — pooled BAF signal may be too sparse
- Bulk RNA-seq cohort-level focal CNV discovery without single-cell resolution needs — the bulk variant of this skill applies instead

## Inputs

- Single-cell RNA-seq expression matrix (e.g., scell_gbm, 430 cells, normalized counts, matrix.type='normalized')
- Per-patient pooled BAFExtract output files (B-allele frequency shift profiles generated from merged single-cell BAMs)
- hg19 cytoband and centromere coordinates
- Gene annotation (e.g., ensembl_gene_id mapping via generateAnnotation)
- loh.name.mapping file mapping each cell barcode to its patient-pooled BAF signal

## Outputs

- Chromosome-arm large-scale event matrix (finalChrMat from extractLargeScaleEvents, thr=0.75)
- Segment-level CNV summary (extractSegmentSummary with gamma=6)
- Cell-by-gene CNV matrix (gene.matrix)
- Mutually exclusive / co-occurring CNV event calls (Fisher's exact test)
- CNV-based clonal phylogenetic tree of cells
- Heatmaps: expression signal at smoothing scales, large-scale event heatmaps (e.g., sampleName='MGH31', chrs=c('5p','14q')), BAF shift plots

## How to apply

First merge single-cell BAM files per patient and run BAFExtract to generate per-patient pooled BAF shift profiles, then load them into R with readBAFExtractOutput (sequencing.type='bulk'-style per-sample BAF files) and map each cell to its patient-pooled BAF signal via a one-to-many loh.name.mapping. Build the casper object with CreateCasperObject (cnv.scale=3, loh.scale=3, method='iterative', matrix.type='normalized') supplying hg19 cytoband, gene annotation (generateAnnotation with ishg19=T), and centromere coordinates. Run runCaSpER(object, removeCentromere=T, cytoband=cytoband, method='iterative'), which performs HMM on all smoothed expression scales and integrates allele-frequency shift states; because cells lack their own variant calls, the patient-pooled BAF signal supplies the LOH information. Summarize calls across scales with extractLargeScaleEvents(final.objects, thr=0.75) for chromosome-arm events, extractSegmentSummary (gamma=6 consistency threshold) for segments, and gene.matrix for a cell-by-gene CNV matrix. Visualize and validate with plotHeatmap, plotBAFOneSample, plotSingleCellLargeScaleEventHeatmap, extractMUAndCooccurence (Fisher's exact test), and plotSCellCNVTree for the clonal tree.

## Related tools

- **CaSpER** (R package performing multiscale smoothing, HMM-based CNV calling integrated with BAF allele-shift states, summarization, mutual-exclusivity testing, and clonal tree inference) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates B-allele frequency shift profiles from mapped RNA-seq reads (pileup then BAF profile) without a heterozygous variant call set; output feeds CaSpER's loh input) — https://github.com/akdess/BAFExtract
- **inferCNV** (Alternative method enabling only visual inspection of scRNA-seq expression profiles, cited for contrast)
- **HoneyBADGER** (Alternative scRNA-seq CNV detection method used in the paper for concordance checks of CaSpER calls)

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"); finalChrMat <- extractLargeScaleEvents(final.objects, thr=0.75)
```

## Evaluation signals

- Consistency threshold: large-scale CNV calls must be consistent across ≥75% of the N×M pairwise scale comparisons (extractLargeScaleEvents thr=0.75, i.e., ≥7 of 9 scale pairs)
- Tumor cells show the expected GBM pattern: chromosome 7 amplification and chromosome 10 deletion, while normal oligodendrocytes cluster separately
- Reproduced mutually exclusive subclone pairs match the report: MGH31 (5q amp vs 14q del; 1p amp vs 13q del), MGH28 (8q:20p, 5q:19p), MGH30 (6p:7p)
- CNV-based clonal tree topology matches the reported topology from plotSCellCNVTree
- Figures from plotHeatmap, plotBAFOneSample, and plotSingleCellLargeScaleEventHeatmap reproduce the reported large-scale event matrix patterns

## Limitations

- If some cells have a substantially large number of reads, they may dominate the pooled BAF shift signal and create biases
- 3′-only sequencing (e.g., DropSeq) degrades BAF shift signal since BAF shift is measured only for variants located close to the 3′ ends of genes
- RNA editing, PCR duplication artefacts, and sequencing errors may bias CNV detection through complex effects on the BAF shift signal
- No BAF shift generation model accounts for the 3′ bias of sample-preparation assays; pooling reads is believed but not validated to soothe this effect

## Evidence

- [abstract] CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling.: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [methods] we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [methods] finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75): "finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75)"
- [discussion] Analyzing single-cell GBM RNA-seq data using CaSpER, unraveled mutual exclusive and co-occurring CNV subclones: "Analyzing single-cell GBM RNA-seq data using CaSpER, unraveled mutual exclusive and co-occurring CNV subclones"
- [discussion] If some cells have a substantially large number of reads, they may dominate the BAF shift signal and may potentially create biases.: "If some cells have a substantially large number of reads, they may dominate the BAF shift signal and may potentially create biases."
- [methods] Step 2. Extract BAF values from RNA-Seq bam files: "Step 2. Extract BAF values from RNA-Seq bam files"
