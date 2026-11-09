---
name: multiscale-cnv-call-harmonization
description: Use when CaSpER (runCaSpER, method="iterative") has produced CNV calls
  from multiple smoothed expression scales paired with BAF shift evidence, and you
  must collapse N×M scale-wise calls into a single consistent set of chromosome-arm,
  segment, or gene-level CNV calls.
when_to_use_negative:
- 3′-only sequencing data (e.g., DropSeq) where the BAF shift signal is sparse because
  variants are measured only near gene 3′ ends — the BAF-evidence gating of HMM states
  2/4 will be unreliable
- Inputs lacking any BAF/allelic shift signal (expression-only matrices), since the
  harmonization relies on BAF-shift confirmation of intermediate HMM states
- Single-sample analyses needing no consensus, where only one scale is used and there
  is no N×M pairwise comparison to harmonize
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: CaSpER
  role: Performs the multiscale HMM segmentation, BAF-shift integration (assignStates),
    and cross-scale harmonization/summarization via runCaSpER, extractLargeScaleEvents,
    extractSegmentSummary, and gene.matrix
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates the B-allele frequency shift profiles from mapped RNA-seq reads
    that supply the allelic-shift evidence used to gate HMM states during harmonization
  repo: https://github.com/akdess/BAFExtract
- name: mclust
  role: Gaussian-mixture-model fit of the BAF-shift signal used to threshold allele-frequency
    shifts before combining with HMM segment states
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:7e7c2cd72c0431eb796bb38d7abea5a933be009c9a597e275af0558eb0ac4925
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  - mclust
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multiscale-cnv-call-harmonization

## Summary

Harmonize and summarize CNV calls that CaSpER produces from N×M pairwise comparisons of multi-scale smoothed expression signals and BAF shift signals, yielding consistent chromosome-arm, segment-level, and gene-level copy-number calls from bulk or single-cell RNA-seq.

## When to use

Use when CaSpER (runCaSpER, method="iterative") has produced CNV calls from multiple smoothed expression scales paired with BAF shift evidence, and you must collapse N×M scale-wise calls into a single consistent set of chromosome-arm, segment, or gene-level CNV calls.

## When NOT to use

- 3′-only sequencing data (e.g., DropSeq) where the BAF shift signal is sparse because variants are measured only near gene 3′ ends — the BAF-evidence gating of HMM states 2/4 will be unreliable
- Inputs lacking any BAF/allelic shift signal (expression-only matrices), since the harmonization relies on BAF-shift confirmation of intermediate HMM states
- Single-sample analyses needing no consensus, where only one scale is used and there is no N×M pairwise comparison to harmonize

## Inputs

- Normalized gene expression matrix (rows = genes with ensembl id or hgnc symbol, columns = samples/cells; e.g., yale_meningioma or scell_gbm 430-cell dataset)
- casper object created with CreateCasperObject (cnv.scale=3, loh.scale=3, matrix.type="normalized", expr.cutoff=4.5, method="iterative")
- Smoothed BAF/LOH signal from BAFExtract (via readBAFExtractOutput on per-sample BAF files), including a loh.name.mapping for one-to-many single-cell patient pooling
- hg19 cytoband table and centromere positions; gene annotation from generateAnnotation (id_type="ensembl_gene_id", ishg19=T)

## Outputs

- final.objects list containing per-scale HMM segments integrated with BAF shift states
- Chromosome-arm large-scale CNV matrix from extractLargeScaleEvents (thr=0.75), writable as large_scale_cnv_calls.tsv
- Segment-level CNV summary (extractSegmentSummary, gamma=6 consistency cutoff)
- Gene-by-sample CNV matrix (gene.matrix) with values 0=neutral, 1=amplification, -1=deletion
- Visualization objects for plotHeatmap, plotLargeScaleEvent, plotSingleCellLargeScaleEventHeatmap, and plotSCellCNVTree

## How to apply

Run runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative") on the casper object (cnv.scale=3, loh.scale=3) so that a 5-state HMM is fit on all smoothed expression scales and each HMM state is integrated with BAF allele-frequency-shift evidence — states 2 (heterozygous deletion) and 4 (one-copy amplification) only become calls when a BAF shift is detected, while states 1 and 5 are called directly. Each of the N×M pairwise expression/BAF scale comparisons yields a large-scale CNV call per chromosome arm; harmonize them with extractLargeScaleEvents(final.objects, thr=0.75) so an arm-level call is kept only if consistent in at least 75% (e.g., 7 of 9) comparisons. For finer resolution, summarize segments with extractSegmentSummary(final.objects) using a gamma=6 out-of-N×M consistency cutoff, and build the cell-by-gene CNV matrix with gene.matrix, encoding 0=neutral, 1=amplification, -1=deletion. The rationale is that multiscale smoothing plus cross-scale voting suppresses noise-driven calls while preserving recurrent large-scale events such as chr7 amplification/chr10 deletion in GBM.

## Related tools

- **CaSpER** (Performs the multiscale HMM segmentation, BAF-shift integration (assignStates), and cross-scale harmonization/summarization via runCaSpER, extractLargeScaleEvents, extractSegmentSummary, and gene.matrix) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates the B-allele frequency shift profiles from mapped RNA-seq reads that supply the allelic-shift evidence used to gate HMM states during harmonization) — https://github.com/akdess/BAFExtract
- **mclust** (Gaussian-mixture-model fit of the BAF-shift signal used to threshold allele-frequency shifts before combining with HMM segment states)

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"); finalChrMat <- extractLargeScaleEvents(final.objects, thr=0.75)
```

## Evaluation signals

- Large-scale chromosome-arm calls agree in at least 75% (thr=0.75, e.g., ≥7 of 9) of the N×M pairwise scale comparisons
- Segment and gene calls are reported only where consistent calls exceed gamma=6 out of the N×M comparisons
- Gene matrix values are restricted to {0, 1, -1} (neutral, amplification, deletion)
- Reproduced large-scale event matrix shows the expected patterns, e.g., chr7 gain/chr10 loss across GBM tumor cells and separation of normal oligodendrocytes
- A typical CaSpER run completes in about 5–30 minutes, so substantially longer runs indicate misconfigured scale parameters

## Limitations

- For 3′-only sequencing (e.g., DropSeq), the BAF shift signal is adversely impacted because shifts are measured only for variants near gene 3′ ends
- In single-cell studies, cells with substantially large numbers of reads may dominate the pooled BAF shift signal and create biases
- RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal and bias CNV detection in complex ways
- Ground-truth CNV calls are unknown for some validation datasets (e.g., MM135), so concordance is checked against other methods such as HONEYBADGER rather than truth

## Evidence

- [methods] runCaSpER call integrating HMM states with BAF shifts: "final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")"
- [methods] Harmonization across scales and pairwise comparisons: "Harmonization and Summarization of CNV calls from multiple scales and from multiple pairwise comparison of BAF and Expression Signals"
- [methods] Chromosome-arm threshold: "finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75)"
- [methods] HMM on all expression scales with BAF integration: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [abstract] Multiscale smoothing and allelic shift integration: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [readme] Typical runtime for a CaSpER run: "Each run generally takes about 5-30 minutes."
- [readme] BAFExtract supplies BAF shift without variant call set: [evidence span withheld — anchor status: external_doc]
