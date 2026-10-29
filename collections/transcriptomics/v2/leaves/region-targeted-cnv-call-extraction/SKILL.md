---
name: region-targeted-cnv-call-extraction
description: Use when you need to determine whether specific focal or arm-level CNV
  events (e.g., an oncogene amplicon or a chromosome arm deletion) are recovered by
  CaSpER at particular expression smoothing scales in bulk or single-cell RNA-seq.
when_to_use_negative:
- When the goal is a genome-wide unsupervised CNV catalog rather than interrogation
  of predefined target regions — use full CaSpER summarization (extractSegmentSummary,
  extractLargeScaleEvents) instead
- When only DNA-seq (e.g., SNV array or WGS) data are available and no RNA-seq expression
  or BAF shift signal exists, since CaSpER relies on RNA-seq expression and BAF profiles
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: CaSpER
  role: Runs multiscale HMM on smoothed expression and BAF signals and returns per-scale
    final.objects from which region-targeted CNV calls are extracted
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates B-allele frequency shift profiles from mapped RNA-seq reads that
    are read in as LOH input for the casper object
  repo: https://github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:de247d923ac929d12524701d0e4047c495b1a8fc0dfc2259aa8a87863cfff243
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
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# region-targeted-cnv-call-extraction

## Summary

Extracts gene-level and segment-level CNV calls from CaSpER per-scale final.objects for specific genomic regions (e.g., PDGFRA at 4q12, chromosome 22q arm), tabulating how each expression smoothing scale and BAF scale pairwise comparison classifies each region as amp/del/neutral.

## When to use

Use when you need to determine whether specific focal or arm-level CNV events (e.g., an oncogene amplicon or a chromosome arm deletion) are recovered by CaSpER at particular expression smoothing scales in bulk or single-cell RNA-seq.

## When NOT to use

- When the goal is a genome-wide unsupervised CNV catalog rather than interrogation of predefined target regions — use full CaSpER summarization (extractSegmentSummary, extractLargeScaleEvents) instead
- When only DNA-seq (e.g., SNV array or WGS) data are available and no RNA-seq expression or BAF shift signal exists, since CaSpER relies on RNA-seq expression and BAF profiles

## Inputs

- CaSpER final.objects (list of per-scale objects from runCaSpER with cnv.scale=3 and loh.scale=3)
- hg19 cytoband and centromere annotation
- Normalized TCGA bulk RNA-seq expression matrix (e.g., TCGA-GBM)
- BAFExtract LOH output files loaded via readBAFExtractOutput(sequencing.type='bulk')
- List of target regions (cytoband coordinates, e.g., PDGFRA 4q12, chr22q arm)

## Outputs

- scale_region_cnv_calls.csv: matrix of amp/del/neutral calls per expression scale and BAF scale pairwise comparison for each target region
- Per-region gene-level and segment-level CNV call listings across scales
- finalChrMat from extractLargeScaleEvents (chromosome-arm-level calls at thr=0.75)

## How to apply

After running runCaSpER(object, removeCentromere=T, cytoband=hg19_cytoband, method='iterative'), iterate over the returned per-scale final.objects, which contain HMM segment states integrated with allele-frequency shift information at each expression scale. For each region of interest — identified via hg19 cytoband and centromere annotations (e.g., PDGFRA in 4q12, the 22q arm) — extract the gene-level and segment-level CNV calls from every expression scale and BAF scale pairwise comparison. Tabulate the emitted call (amp/del/neutral) per region per scale and write the scale-by-region matrix to a CSV (e.g., scale_region_cnv_calls.csv). The rationale is that CaSpER's multiscale framework recovers focal events at small (low) smoothing scales and broad arm-level events at higher scale lengths, so scale-resolved extraction reveals event scale-specificity. Complement region-level inspection with extractLargeScaleEvents(final.objects, thr=0.75) for arm-level calls and extractSegmentSummary(final.objects) for segment calls.

## Related tools

- **CaSpER** (Runs multiscale HMM on smoothed expression and BAF signals and returns per-scale final.objects from which region-targeted CNV calls are extracted) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates B-allele frequency shift profiles from mapped RNA-seq reads that are read in as LOH input for the casper object) — https://github.com/akdess/BAFExtract

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=hg19_cytoband, method="iterative")
```

## Evaluation signals

- scale_region_cnv_calls.csv exists and contains scale and region columns with rows for PDGFRA and 22q
- PDGFRA (4q12) focal amplification is called at small (low) expression smoothing scales
- Chromosome 22q arm-level deletion is called at higher scale lengths
- Calls transition from focal-region alterations at low smoothing scales to the arm-level 22q event at higher scales, consistent with CaSpER's multiscale behavior
- Every expression-scale × BAF-scale pairwise comparison yields a call in {amp, del, neutral} for each target region

## Limitations

- CNV calls from RNA-seq are affected by the dynamic and highly non-uniform coverage of the genome by RNA-seq signal, making it hard to distinguish copy number events from expression variation
- BAF shift signal may be adversely impacted in 3'-only sequencing (such as DropSeq) because variants are measured only close to the 3' ends of genes
- Factors such as RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal and bias CNV detection

## Evidence

- [methods] final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"): "final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")"
- [methods] we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [methods] finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75): "finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75)"
- [abstract] identification, visualization, and integrative analysis of focal and large-scale CNV events in multiscale resolution: "identification, visualization, and integrative analysis of focal and large-scale CNV events in multiscale resolution using either bulk or single-cell RNA sequencing data"
- [discussion] CaSpER discovered ssCNV regions in TCGA bulk RNA-seq data: "CaSpER discovered ssCNV regions in TCGA bulk RNA-seq data"
