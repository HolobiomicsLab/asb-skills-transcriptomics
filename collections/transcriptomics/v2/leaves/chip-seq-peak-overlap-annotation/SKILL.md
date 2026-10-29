---
name: chip-seq-peak-overlap-annotation
description: Use when peak-gene pairs (e.g., GrID-Net vs correlation baselines on
  10x Multiome) must be validated for enhancer priming states, requiring H3K4me1/H3K27ac
  ChIP-seq overlap of ATAC peak coordinates.
when_to_use_negative:
- Input is already an annotated peak set with enhancer-state labels — re-intersection
  with ChIP-seq peaks is redundant
- No matched cell-type ChIP-seq data exists for the tissue/cell state under study
  (e.g., non-neural datasets like the sci-CAR A549 dexamethasone data with no neural
  progenitor histone marks)
- The goal is direct causal inference of peak-gene links — this skill is a downstream
  validation/annotation step, not the GrID-Net inference itself
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: GrID-Net
  role: Provides the results DataFrame of Granger-prioritized peak-gene pairs whose
    regulatory peaks are being annotated for enhancer state
  repo: https://github.com/alexw16/gridnet
- name: ChIP-Atlas
  role: Source of H3K27ac and H3K4me1 ChIP-seq peak coordinates for human neural progenitor
    cells and fetal neural cells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:f66edf5144d3d9a43dcd8850453eaef00c773e5926bb721a5cda13e682fd16df
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
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# chip-seq-peak-overlap-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates ATAC-seq peaks from single-cell multimodal data by genomic interval intersection with ChIP-seq peak sets (H3K4me1 and H3K27ac from ChIP-Atlas), classifying each peak as a poised (H3K4me1-only) or active (H3K4me1+H3K27ac) enhancer to test chromatin priming of prioritized peak-gene pairs.

## When to use

Use when peak-gene pairs (e.g., GrID-Net vs correlation baselines on 10x Multiome) must be validated for enhancer priming states, requiring H3K4me1/H3K27ac ChIP-seq overlap of ATAC peak coordinates.

## When NOT to use

- Input is already an annotated peak set with enhancer-state labels — re-intersection with ChIP-seq peaks is redundant
- No matched cell-type ChIP-seq data exists for the tissue/cell state under study (e.g., non-neural datasets like the sci-CAR A549 dexamethasone data with no neural progenitor histone marks)
- The goal is direct causal inference of peak-gene links — this skill is a downstream validation/annotation step, not the GrID-Net inference itself

## Inputs

- GrID-Net results DataFrame (peak-gene pairs annotated with Granger causality test statistics) from gridnet.gridnet_multimodal or gridnet.run_gridnet
- Ranked peak-gene pair sets from each correlation-based baseline method
- ATAC-seq peak coordinates (chr_no, start, end) from atac_adata.var of the 10x Multiome human corticogenesis AnnData
- H3K4me1 and H3K27ac ChIP-seq peak coordinates for human neural progenitor cell samples from ChIP-Atlas

## Outputs

- Per-peak enhancer-state labels (active = H3K4me1+H3K27ac, poised = H3K4me1-only) for intersected ATAC peaks
- Per-method poised-to-active enhancer ratio table (e.g., enhancer_priming_ratios.csv)
- Bar-chart figure of poised-to-active ratios across methods, reproducing Fig. 2f

## How to apply

Retrieve H3K27ac and H3K4me1 ChIP-seq peak coordinates for the relevant cell type (here, human neural progenitor cells) from ChIP-Atlas, and extract ATAC peak coordinates (chr_no, start, end) from the atac_adata.var DataFrame as required by gridnet. Perform genomic interval intersection between each ATAC peak and the two ChIP-seq peak sets: peaks overlapping both H3K4me1 and H3K27ac are labeled active enhancers, peaks overlapping H3K4me1 only are labeled poised enhancers. For each method's peak-gene pair set (GrID-Net and each correlation-based baseline), compute the poised-to-active enhancer ratio (count of H3K4me1-only peaks divided by count of H3K4me1+H3K27ac peaks). The rationale is that GrID-Net's causal (Granger-based) prioritization should capture chromatin-primed regulatory relationships, where a poised enhancer is present before gene activation, which correlational methods miss because they model only concurrent accessibility-expression association.

## Related tools

- **GrID-Net** (Provides the results DataFrame of Granger-prioritized peak-gene pairs whose regulatory peaks are being annotated for enhancer state) — https://github.com/alexw16/gridnet
- **ChIP-Atlas** (Source of H3K27ac and H3K4me1 ChIP-seq peak coordinates for human neural progenitor cells and fetal neural cells)

## Examples

```
import pandas as pd; active = peaks[peaks.overlap_h3k4me1 & peaks.overlap_h3k27ac]; poised = peaks[peaks.overlap_h3k4me1 & ~peaks.overlap_h3k27ac]; ratio = len(poised)/len(active)
```

## Evaluation signals

- enhancer_priming_ratios.csv exists and contains exactly one poised-to-active ratio per method comparison
- The GrID-Net poised-to-active ratio exceeds each correlation-method ratio, consistent with the reported chromatin-priming result (Fig. 2f)
- Peak labels partition cleanly: every intersected peak is either H3K4me1-only (poised) or H3K4me1+H3K27ac (active); H3K27ac-only overlaps should not occur or be handled explicitly
- Interval intersections use matching genome builds (GRCh38/hg38) for ATAC peaks and ChIP-seq coordinates
- ATAC peak coordinates match the atac_adata.var columns chr_no/start/end used by gridnet.identify_all_peak_gene_link_candidates

## Limitations

- Chromatin-priming evidence is correlational context; GrID-Net's locus-gene associations remain causal hypotheses that need perturbation-based validation
- The annotation depends on the availability and quality of cell-type-matched ChIP-seq data (human neural progenitor cells in ChIP-Atlas); results may not transfer to other tissues or datasets
- Corticogenesis data may explain only a subset of disease phenotypes, limiting generalization of the priming interpretation

## Evidence

- [discussion] ChIP-Atlas H3K4me1/H3K27ac retrieval: "We obtained H3K27ac and H3K4me1 ChIP-seq data for all samples of human neural progenitor cells and fetal neural cells in ChIP-Atlas35."
- [full_text] Primed-enhancer finding: [evidence span withheld — anchor status: not_found]
- [readme] ATAC peak coordinate schema in README: "the AnnData object for the ATAC-seq dataset must include annotations for the genomic positions of the start and end sites for all peaks in its adata.var DataFrame"
- [readme] Coordinate columns: [evidence span withheld — anchor status: not_found]
