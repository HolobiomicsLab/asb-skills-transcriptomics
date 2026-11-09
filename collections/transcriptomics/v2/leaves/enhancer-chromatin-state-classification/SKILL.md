---
name: enhancer-chromatin-state-classification
description: Use when you have genomic intervals (e.g., ATAC peaks from adata.var)
  plus H3K4me1 and H3K27ac ChIP-seq peaks and need to label each interval as a poised
  vs active enhancer to test priming.
when_to_use_negative:
- Input intervals are already annotated with chromatin state (e.g., ChromHMM segmentation
  output) — re-classification via ChIP overlap is redundant
- No matched histone modification ChIP-seq data exists for the cell type under study,
  since the poised/active labeling requires H3K4me1 and H3K27ac from the same cellular
  context
- The analysis goal is causal inference of peak-gene links rather than epigenomic
  annotation of already-prioritized pairs
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3173
tools:
- name: GrID-Net
  role: Provides the results_df of peak-gene pairs with Granger causality test statistics
    whose differentially prioritized pairs are annotated with enhancer states
  repo: https://github.com/alexw16/gridnet
- name: ChIP-Atlas
  role: Source of H3K4me1 and H3K27ac ChIP-seq peak coordinates for human neural progenitor
    cells and fetal neural cells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:f1c5a0bfe46d821e605e0004a6197dc778a84dc125d84204a1cf3b66e00e9f25
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - ChIP-Atlas
  - GrID-Net
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# enhancer-chromatin-state-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies ATAC-seq peaks as poised (H3K4me1-only) or active (H3K4me1+H3K27ac) enhancers by intersecting peak coordinates with ChIP-seq peak sets from ChIP-Atlas, enabling chromatin-priming assessment of prioritized regulatory pairs.

## When to use

Use when you have genomic intervals (e.g., ATAC peaks from adata.var) plus H3K4me1 and H3K27ac ChIP-seq peaks and need to label each interval as a poised vs active enhancer to test priming.

## When NOT to use

- Input intervals are already annotated with chromatin state (e.g., ChromHMM segmentation output) — re-classification via ChIP overlap is redundant
- No matched histone modification ChIP-seq data exists for the cell type under study, since the poised/active labeling requires H3K4me1 and H3K27ac from the same cellular context
- The analysis goal is causal inference of peak-gene links rather than epigenomic annotation of already-prioritized pairs

## Inputs

- ATAC-seq peak genomic coordinates (chr_no, start, end) from adata.var of a 10x Multiome ATAC AnnData object
- H3K4me1 ChIP-seq peak coordinates for matched cell type (ChIP-Atlas, human neural progenitor cells)
- H3K27ac ChIP-seq peak coordinates for matched cell type (ChIP-Atlas)
- Peak-gene pair sets per method, e.g., GrID-Net results_df (Granger causality statistics) and ranked pair lists from correlation-based baselines

## Outputs

- Per-peak enhancer state labels: poised (H3K4me1-only) vs active (H3K4me1+H3K27ac)
- Per-method poised-to-active enhancer ratio table (enhancer_priming_ratios.csv)
- Bar-chart figure comparing poised-to-active ratios across prioritization methods

## How to apply

Retrieve H3K4me1 and H3K27ac ChIP-seq peak coordinates for the relevant cell type (human neural progenitor cells and fetal neural cells from ChIP-Atlas in the reference paper) and extract ATAC peak coordinates (chr_no, start, end) from the 10x Multiome ATAC AnnData adata.var. Perform genomic interval intersection between each ATAC peak and the two ChIP-seq peak sets, labeling peaks overlapping H3K4me1 alone as poised enhancers and peaks overlapping both H3K4me1 and H3K27ac as active enhancers. Restrict the analysis to intervals associated with the pair sets of interest — in the reference workflow, peak-gene pairs differentially prioritized by GrID-Net versus correlation-based baselines. For each pair set, compute the poised-to-active enhancer ratio (count of H3K4me1-only peaks divided by count of H3K4me1+H3K27ac peaks), then compare ratios across methods to detect chromatin priming enrichment.

## Related tools

- **GrID-Net** (Provides the results_df of peak-gene pairs with Granger causality test statistics whose differentially prioritized pairs are annotated with enhancer states) — https://github.com/alexw16/gridnet
- **ChIP-Atlas** (Source of H3K4me1 and H3K27ac ChIP-seq peak coordinates for human neural progenitor cells and fetal neural cells)

## Examples

```
atac_peaks = atac_adata.var[['chr_no','start','end']]; poised = intersect(atac_peaks, h3k4me1_peaks) - intersect(atac_peaks, h3k27ac_peaks); active = intersect(intersect(atac_peaks, h3k4me1_peaks), h3k27ac_peaks); ratio = len(poised)/len(active)
```

## Evaluation signals

- enhancer_priming_ratios.csv exists and contains exactly one poised-to-active ratio per method comparison
- Each ATAC peak that overlaps a ChIP-seq set receives an unambiguous label (poised = H3K4me1-only, active = H3K4me1+H3K27ac)
- The GrID-Net pair set shows a higher poised-to-active enhancer ratio than every correlation-based baseline, reproducing Fig. 2f
- Interval intersection uses consistent genome build (GRCh38/hg38) and chromosome naming across ATAC and ChIP-seq peak files

## Limitations

- Chromatin state assignment depends on availability and quality of cell-type-matched H3K4me1/H3K27ac ChIP-seq; mismatched tissue or developmental stage biases the ratio
- The poised/active dichotomy ignores additional histone marks (e.g., H3K27me3) and other enhancer states, so the classification is coarse
- The inferred chromatin-priming pattern is a correlational annotation of causal hypotheses that still require perturbation-based validation

## Evidence
