---
name: peak-gene-pair-differential-prioritization
description: Use when you have GrID-Net peak-gene results plus correlation-method
  rankings and want to show that causally-prioritized pairs are biologically distinct,
  e.g., enriched for poised enhancers.
when_to_use_negative:
- No matched ChIP-seq (H3K4me1/H3K27ac) data exists for the relevant cell type, so
  enhancer-state annotation cannot be performed
- The goal is causal link discovery itself rather than benchmarking GrID-Net against
  correlation methods
- The dataset is not a dynamic process with pseudotemporal context — GrID-Net requires
  multiomic data from a dynamic process such as differentiation to build the cell-state
  DAG
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: GrID-Net
  role: Produces the Granger-causal peak-gene pair rankings (results DataFrame with
    F-test statistics) whose differentially prioritized pairs are tested for enhancer
    priming; candidate pairs come from gridnet.identify_all_peak_gene_link_candidates
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations used to build the kNN graph (k=15)
    and pseudotime-directed DAG that underlies the GrID-Net run feeding this comparison
  repo: https://github.com/rs239/schema
- name: ChIP-Atlas
  role: Source of H3K27ac and H3K4me1 ChIP-seq peaks for human neural progenitor cells
    used to label ATAC peaks as poised or active enhancers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:4a9b92e8c5377dfcd359a8b6a8a620bcf359f6184f33bfe36c66369e16666911
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
  - Schema
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# peak-gene-pair-differential-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies peak-gene pairs differentially prioritized by GrID-Net's Granger causal inference relative to correlation-based baseline methods, then tests whether GrID-Net-specific pairs carry chromatin-priming signatures (poised H3K4me1-only vs active H3K4me1+H3K27ac enhancers).

## When to use

Use when you have GrID-Net peak-gene results plus correlation-method rankings and want to show that causally-prioritized pairs are biologically distinct, e.g., enriched for poised enhancers.

## When NOT to use

- No matched ChIP-seq (H3K4me1/H3K27ac) data exists for the relevant cell type, so enhancer-state annotation cannot be performed
- The goal is causal link discovery itself rather than benchmarking GrID-Net against correlation methods
- The dataset is not a dynamic process with pseudotemporal context — GrID-Net requires multiomic data from a dynamic process such as differentiation to build the cell-state DAG

## Inputs

- GrID-Net results DataFrame (peak-gene pairs annotated with Granger causality test statistics, e.g., from gridnet.gridnet_multimodal)
- Ranked peak-gene pair lists from correlation-based baseline methods
- ATAC-seq peak coordinates (chr_no, start, end) in 10x Multiome ATAC AnnData adata.var
- H3K4me1 and H3K27ac ChIP-seq peak coordinates for human neural progenitor cells from ChIP-Atlas

## Outputs

- Set of peak-gene pairs differentially prioritized by GrID-Net relative to each correlation baseline
- enhancer_priming_ratios.csv with one poised-to-active enhancer ratio per method comparison
- Bar-chart figure of poised-to-active enhancer ratios (analogous to Fig. 2f)

## How to apply

Run GrID-Net on the 10x Multiome human corticogenesis dataset to obtain a results DataFrame of peak-gene pairs with Granger causality F-test statistics, and rank the same candidate pairs with each correlation-based baseline. Determine the set of pairs differentially prioritized by GrID-Net relative to each baseline. Retrieve H3K4me1 and H3K27ac ChIP-seq peaks for human neural progenitor cells from ChIP-Atlas and intersect them with ATAC peak coordinates (chr_no, start, end from atac_adata.var), labeling peaks as poised enhancers (H3K4me1 only) or active enhancers (H3K4me1 + H3K27ac). For each method's pair set, compute the poised-to-active enhancer ratio and assemble a comparison table and bar chart; the expected result is that GrID-Net pairs show a higher poised ratio than every correlation baseline, consistent with Fig. 2f.

## Related tools

- **GrID-Net** (Produces the Granger-causal peak-gene pair rankings (results DataFrame with F-test statistics) whose differentially prioritized pairs are tested for enhancer priming; candidate pairs come from gridnet.identify_all_peak_gene_link_candidates) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations used to build the kNN graph (k=15) and pseudotime-directed DAG that underlies the GrID-Net run feeding this comparison) — https://github.com/rs239/schema
- **ChIP-Atlas** (Source of H3K27ac and H3K4me1 ChIP-seq peaks for human neural progenitor cells used to label ATAC peaks as poised or active enhancers)

## Examples

```
results_df = gridnet.gridnet_multimodal(rna_adata, atac_adata, root_cell_marker_gene='TOP2A', distance_thresh=1e6)
```

## Evaluation signals

- enhancer_priming_ratios.csv exists and contains exactly one poised-to-active enhancer ratio per method comparison
- The GrID-Net poised-to-active ratio exceeds every correlation-method ratio (the Fig. 2f chromatin-priming result)
- Peak-gene pairs in the comparison are restricted to peaks within 1 Mb of gene TSS (250 kb for 10x Multiome) and pass the 1% gene / 0.1% peak accessibility filters
- Interval intersection cleanly partitions overlapping ATAC peaks into H3K4me1-only (poised) and H3K4me1+H3K27ac (active) classes with no unlabeled overlaps among annotated peaks

## Limitations

- GrID-Net infers causal hypotheses from observational data that require perturbation-based validation, which was only mimicked via eQTL cross-referencing
- Reliance on multiomic single-cell data from matching tissue undergoing a dynamic biological process to provide pseudotemporal context
- Inferred causal interactions are likely indirect and mediated by additional biophysical events such as TF binding

## Evidence

- [results] poised-enhancer enrichment workflow: [evidence span withheld — anchor status: not_found]
