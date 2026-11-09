---
name: multiscale-expression-signal-smoothing
description: Use when you have a normalized gene-by-sample RNA-seq matrix (e.g. TCGA-GBM
  bulk or single-cell GBM) and need to detect both focal CNVs (e.g. PDGFRA amplification)
  and large-scale chromosome-arm events from expression alone.
when_to_use_negative:
- You only need transcriptomic analysis (differential expression, clustering) without
  probing the CNV landscape — multiscale smoothing is specific to CNV event recovery
- You require an exact ground-truth SNV/heterozygous variant call set workflow — CaSpER
  deliberately avoids needing a high-quality heterozygous variant call set, so tools
  built around such call sets are a different fit
- Data are 3'-only sequencing (e.g. DropSeq) where BAF shift signal is adversely impacted
  because shifts are measured only for variants near 3' ends of genes
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3173
tools:
- name: CaSpER
  role: Implements multiscale smoothing of the expression signal (PerformMedianFilter,
    PerformMeanFilterByChr, CenterSmooth), 5-state HMM segmentation per scale, and
    integration of HMM states with BAF-shift evidence via runCaSpER
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates the B-allele frequency shift profile from mapped RNA-seq reads (without
    a heterozygous variant call set) that is read in via readBAFExtractOutput and
    integrated across scales
  repo: https://github.com/akdess/BAFExtract
- name: mclust
  role: Gaussian-mixture model used to threshold the BAF-shift signal on segments
    before combining with HMM states 2 and 4 via assignStates
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:b2f3d367468c2ddfe197af40de4b7a2908a7dab850dd1117577bd44463df4019
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  - mclust
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# multiscale-expression-signal-smoothing

## Summary

A signal-processing skill from CaSpER that smooths genome-wide RNA-seq expression signals at multiple scales (median/mean filtering) so that focal CNV events are recovered at small scales and broad arm-level CNV events at larger scales, prior to HMM segmentation and CNV calling.

## When to use

Use when you have a normalized gene-by-sample RNA-seq matrix (e.g. TCGA-GBM bulk or single-cell GBM) and need to detect both focal CNVs (e.g. PDGFRA amplification) and large-scale chromosome-arm events from expression alone.

## When NOT to use

- You only need transcriptomic analysis (differential expression, clustering) without probing the CNV landscape — multiscale smoothing is specific to CNV event recovery
- You require an exact ground-truth SNV/heterozygous variant call set workflow — CaSpER deliberately avoids needing a high-quality heterozygous variant call set, so tools built around such call sets are a different fit
- Data are 3'-only sequencing (e.g. DropSeq) where BAF shift signal is adversely impacted because shifts are measured only for variants near 3' ends of genes

## Inputs

- Normalized gene expression matrix (rows = genes with ensembl id or hgnc symbol, columns = samples), e.g. TCGA-GBM bulk RNA-seq
- BAFExtract LOH output files loaded via readBAFExtractOutput(sequencing.type="bulk")
- hg19 cytoband and centromere annotation objects
- Gene annotation (e.g. from generateAnnotation with id_type="ensembl_gene_id", ishg19=T)
- Control sample identifiers

## Outputs

- casper object with expression smoothed at multiple scales
- Per-scale final.objects containing HMM segments and CNV states for each expression/BAF scale pairwise comparison
- Gene-level and segment-level CNV calls per region and scale (e.g. PDGFRA 4q12, 22q arm)
- Scale-by-region CNV call matrix (e.g. scale_region_cnv_calls.csv)

## How to apply

Build a casper object with CreateCasperObject using cnv.scale=3 and loh.scale=3, matrix.type='normalized', method='iterative', and control sample identifiers, supplying gene annotation, cytoband (hg19), centromere annotations, and BAFExtract-derived LOH signal via readBAFExtractOutput. Apply multiscale recursive median filtering (PerformMedianFilter, default n=3 smoothing scales) plus chromosome-wise mean/center smoothing (PerformMeanFilterByChr, CenterSmooth) to generate an expression signal smoothed at multiple scales. Then run runCaSpER(object, removeCentromere=T, cytoband=cytoband, method='iterative'), which performs 5-state HMM segmentation (homozygous deletion, heterozygous deletion, neutral, one-copy amplification, multi-copy amplification) on all smoothed expression scales and integrates HMM segment states with allele-frequency shift information across all N×M pairwise expression/BAF scale comparisons. The rationale is that different smoothing scales expose CNV events of different genomic extents — focal events (e.g. 4q12 PDGFRA) emerge at low scales while arm-level events (e.g. 22q deletion) emerge at higher scale lengths.

## Related tools

- **CaSpER** (Implements multiscale smoothing of the expression signal (PerformMedianFilter, PerformMeanFilterByChr, CenterSmooth), 5-state HMM segmentation per scale, and integration of HMM states with BAF-shift evidence via runCaSpER) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates the B-allele frequency shift profile from mapped RNA-seq reads (without a heterozygous variant call set) that is read in via readBAFExtractOutput and integrated across scales) — https://github.com/akdess/BAFExtract
- **mclust** (Gaussian-mixture model used to threshold the BAF-shift signal on segments before combining with HMM states 2 and 4 via assignStates)

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")
```

## Evaluation signals

- Output per-scale final.objects exist for all N×M expression/BAF scale comparisons (default 3 expression × 3 BAF scales)
- Focal-region calls appear at low smoothing scales (e.g. PDGFRA 4q12 amplification) while the arm-level 22q event appears at higher scales, confirming scale-specificity
- Large-scale chromosome-arm calls are consistent in at least thr=75% (7 of 9) of the N×M scale comparisons via extractLargeScaleEvents(final.objects, thr=0.75)
- Segment-level calls only reported where consistent calls exceed gamma=6 out of the N×M comparisons in extractSegmentSummary
- Centromere regions are removed (removeCentromere=T) and calls are restricted to annotated hg19 cytobands

## Limitations

- BAF shift signal may be adversely impacted by 3'-only sequencing (e.g. DropSeq) because shifts are measured only for variants close to the 3' ends of genes
- If some cells have a substantially large number of reads, they may dominate the pooled BAF shift signal in scRNA-seq and create biases
- RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal in a complex manner and bias CNV detection
- MAPQ filtering settings (e.g. minimum mapping quality 50 in BAFExtract) could mean no reads survive depending on the aligner's MAPQ implementation

## Evidence

- [abstract] CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling.: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [methods] we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [methods] final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"): "final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")"
- [methods] a. plotHeatmap: Visualization of the genomewide gene expression signal plot at different smoothing scales: "a. plotHeatmap: Visualization of the genomewide gene expression signal plot at different smoothing scales"
- [readme] CaSpER documentation is [here: RMD file](https://rpubs.com/akdes/673120).: "CaSpER documentation is [here: RMD file](https://rpubs.com/akdes/673120)."
