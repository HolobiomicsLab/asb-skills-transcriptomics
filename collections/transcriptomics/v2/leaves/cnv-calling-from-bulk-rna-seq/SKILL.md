---
name: cnv-calling-from-bulk-rna-seq
description: Use when normalized bulk RNA-seq expression matrices (with optional BAF
  shift files) must be mined for chromosome-arm, segment, and gene-level CNV events,
  or when RNA-derived CNV calls need TPR/FPR benchmarking against genotyping-array
  calls.
when_to_use_negative:
- '3′-only sequencing data (e.g., DropSeq): BAF shift will be measured only for variants
  near gene 3′ ends, degrading the BAF-shift signal CaSpER relies on'
- The input is single-cell RNA-seq (use the single-cell variant of this workflow with
  sequencing.type='single-cell'); bulk and single-cell parameterizations differ
- Only raw FASTQ without aligned BAMs is available, since BAFExtract requires mapped
  reads for pileup/BAF generation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: CaSpER
  role: 'Core multiscale HMM CNV caller: CreateCasperObject, runCaSpER, extractLargeScaleEvents,
    extractSegmentSummary, gene.matrix, calcROC applied to bulk RNA-seq expression
    plus BAF shift signal'
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates the B-allele frequency shift profile from mapped RNA-seq reads alone
    (pileup + BAF profile) without a heterozygous variant call set, as input to CaSpER
    LOH integration
  repo: https://github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_008
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:a84dbc85bd34c73add552e2a6790a556bd21352b3c6c33744191a1c4720a579e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  - HoneyBADGER
  - TCGABiolinks
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# cnv-calling-from-bulk-rna-seq

## Summary

Multiscale HMM-based CNV calling from bulk RNA-seq using CaSpER, which integrates smoothed genome-wide expression signal with BAF allelic-shift (LOH) signal to detect large-scale, segment-level, and gene-level amplifications/deletions, benchmarked against genotyping-array gold standards.

## When to use

Use when normalized bulk RNA-seq expression matrices (with optional BAF shift files) must be mined for chromosome-arm, segment, and gene-level CNV events, or when RNA-derived CNV calls need TPR/FPR benchmarking against genotyping-array calls.

## When NOT to use

- 3′-only sequencing data (e.g., DropSeq): BAF shift will be measured only for variants near gene 3′ ends, degrading the BAF-shift signal CaSpER relies on
- The input is single-cell RNA-seq (use the single-cell variant of this workflow with sequencing.type='single-cell'); bulk and single-cell parameterizations differ
- Only raw FASTQ without aligned BAMs is available, since BAFExtract requires mapped reads for pileup/BAF generation

## Inputs

- Normalized bulk RNA-seq gene expression matrix (e.g., yale_meningioma, TCGA-GBM n=160, TCGA-BRCA 150 randomly selected samples)
- BAFExtract B-allele frequency shift output files from RNA-seq BAMs
- hg19 cytoband information and centromere coordinates
- Gene annotation (generateAnnotation, hg19, ensembl_gene_id)
- Control sample identifiers
- Genotyping-array CNV call matrices as gold standard (for benchmarking)

## Outputs

- Large-scale chromosome-arm CNV matrix (finalChrMat from extractLargeScaleEvents)
- Segment-level CNV summary (extractSegmentSummary, gamma=6)
- Gene-level amplification/deletion matrices (gene.matrix)
- TPR/FPR (and segment-level PPV/TPR) benchmark tables written to CSV (e.g., tcga_gbm_cnv_accuracy_metrics.csv, tcga_brca_cnv_tpr_fpr.csv, meningioma_cnv_benchmark_results.csv)
- Scale-by-region CNV call matrices (e.g., scale_region_cnv_calls.csv) and CNV visualizations

## How to apply

Extract BAF shifts from aligned RNA-seq BAMs with BAFExtract (no heterozygous variant call set required) and load them via readBAFExtractOutput(sequencing.type='bulk'); build the hg19 gene annotation with generateAnnotation using centromere coordinates. Assemble the analysis container with CreateCasperObject(sequencing.type='bulk', cnv.scale=3, loh.scale=3, matrix.type='normalized', method='iterative', filter='median') supplying the normalized expression matrix, cytoband, annotation, and control sample IDs. Run the multiscale caller with runCaSpER(object, removeCentromere=T, cytoband=cytoband, method='iterative'), which performs HMM on all expression scales and integrates HMM segment states with allele-frequency shift information across N×M pairwise scale comparisons. Harmonize across scales: extractLargeScaleEvents(final.objects, thr=0.75) for chromosome-arm calls (75% consistency, at least 7 of 9 pairwise scale comparisons), extractSegmentSummary (gamma=6) for segment calls, and gene.matrix() for gene-level amplification/deletion matrices. Benchmark against matched genotyping-array CNV matrices using calcROC(chrMat, chrMat2) to obtain TPR/FPR per event category (e.g., TCGA-GBM: 79.5%/2% large-scale amplification, 88%/3.5% large-scale deletion; TCGA-BRCA: 60.3%/3.8% large-scale amplification; meningioma: 95%/0.3% large-scale deletion) and write results to CSV.

## Related tools

- **CaSpER** (Core multiscale HMM CNV caller: CreateCasperObject, runCaSpER, extractLargeScaleEvents, extractSegmentSummary, gene.matrix, calcROC applied to bulk RNA-seq expression plus BAF shift signal) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates the B-allele frequency shift profile from mapped RNA-seq reads alone (pileup + BAF profile) without a heterozygous variant call set, as input to CaSpER LOH integration) — https://github.com/akdess/BAFExtract

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")
```

## Evaluation signals

- calcROC TPR/FPR values fall within small tolerance of reported benchmarks: TCGA-BRCA large-scale amplification 60.3%/3.8% and deletion 79%/7.7%; TCGA-GBM large-scale 79.5%/2% amp and 88%/3.5% del; meningioma 95%/0.3% large-scale deletion
- Pearson correlations between BAF shift and smoothed expression signal match chr1p r2=-0.43 (P=0.0008), chr6q r2=-0.54 (P=0.0003), chr22q r2=-0.14 (P=0.43)
- Scale-specific behavior reproduces: focal events such as PDGFRA (4q12) amplification recovered at low expression scales, arm-level 22q deletion at higher scales
- Output CSVs exist with the expected schema (scale/region columns, PDGFRA and 22q rows, four TPR/FPR pairs)
- Large-scale calls use the documented 75% consistency threshold (at least 7 of 9 pairwise scale comparisons) across cnv.scale=3 / loh.scale=3 pairwise comparisons

## Limitations

- 3′-only library preparation (e.g., DropSeq) degrades the BAF shift signal because shifts are measured only for variants near the 3′ ends of genes
- RNA editing, PCR duplication artefacts, and sequencing errors may bias CNV detection through complex effects on the BAF shift signal
- Reproduced accuracy metrics are sensitive to the random sample draw (e.g., the 150-sample TCGA-BRCA subset), so only small tolerances around reported TPR/FPR are expected

## Evidence

- [abstract] CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling.: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [methods] HMM on all scales with BAF integration: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [methods] Core run call: "final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")"
- [methods] Large-scale harmonization threshold: "finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75)"
- [readme] BAFExtract needs no variant call set: [evidence span withheld — anchor status: external_doc]
- [discussion] 3′ bias limitation: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
