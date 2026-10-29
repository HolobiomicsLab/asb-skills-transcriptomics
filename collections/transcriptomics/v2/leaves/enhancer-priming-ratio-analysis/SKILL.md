---
name: enhancer-priming-ratio-analysis
description: Use when validating that causally prioritized peak-gene pairs (e.g.,
  GrID-Net Granger pairs) show epigenomic evidence of enhancer priming, by comparing
  H3K4me1-only vs H3K4me1+H3K27ac ChIP-seq overlap with ATAC peaks and comparing ratios
  across prioritization methods.
when_to_use_negative:
- When the biological system lacks dynamic pseudotemporal context or matching histone
  ChIP-seq data (the chromatin-priming readout depends on NPC H3K4me1/H3K27ac from
  ChIP-Atlas and differentiating tissue).
- When the goal is testing whether pairs correlate rather than whether a causal prioritization
  is epigenomically supported — this skill only compares precomputed pair sets, it
  does not infer the pairs.
- When ChIP-seq samples do not match the cell state of the single-cell dataset (e.g.,
  non-neural tissues), since the enhancer labels would not be biologically interpretable.
tools:
- name: GrID-Net
  role: Produces the causal peak-gene pair rankings whose differentially prioritized
    pairs are tested for poised-enhancer enrichment against correlation baselines
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learned multimodal RNA+ATAC cell representations used upstream to build the
    kNN graph, infer diffusion pseudotime, and orient the DAG for GrID-Net pair scoring
  repo: https://github.com/rs239/schema
- name: ChIP-Atlas
  role: Source of H3K27ac and H3K4me1 ChIP-seq peak sets for human neural progenitor
    cells used to label enhancer states
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:55b740af78ec9f703f662876968a1958b2261c257719ac17f0c2e42de54053bd
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# enhancer-priming-ratio-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies chromatin priming by computing the poised-to-active enhancer ratio (H3K4me1-only vs H3K4me1+H3K27ac ChIP-seq overlap) for peak-gene pairs differentially prioritized by GrID-Net versus correlation-based baselines on the 10x Multiome human corticogenesis dataset, reproducing the Fig. 2f result that GrID-Net-specific pairs are enriched for poised enhancers.

## When to use

Use when validating that causally prioritized peak-gene pairs (e.g., GrID-Net Granger pairs) show epigenomic evidence of enhancer priming, by comparing H3K4me1-only vs H3K4me1+H3K27ac ChIP-seq overlap with ATAC peaks and comparing ratios across prioritization methods.

## When NOT to use

- When the biological system lacks dynamic pseudotemporal context or matching histone ChIP-seq data (the chromatin-priming readout depends on NPC H3K4me1/H3K27ac from ChIP-Atlas and differentiating tissue).
- When the goal is testing whether pairs correlate rather than whether a causal prioritization is epigenomically supported — this skill only compares precomputed pair sets, it does not infer the pairs.
- When ChIP-seq samples do not match the cell state of the single-cell dataset (e.g., non-neural tissues), since the enhancer labels would not be biologically interpretable.

## Inputs

- GrID-Net results DataFrame annotating candidate peak-gene pairs with Granger causality test statistics (gridnet.gridnet_multimodal / run_gridnet output)
- Ranked peak-gene pair lists from correlation-based baseline methods
- H3K4me1 and H3K27ac ChIP-seq peak coordinates for human neural progenitor cell samples (ChIP-Atlas)
- ATAC peak coordinates from the 10x Multiome ATAC AnnData adata.var (columns chr_no, start, end)

## Outputs

- enhancer_priming_ratios.csv with one poised-to-active enhancer ratio per method comparison
- Per-peak enhancer state labels (active: H3K4me1+H3K27ac; poised: H3K4me1-only)
- Bar-chart figure of poised-to-active ratios per prioritization method (Fig. 2f analogue)

## How to apply

Load the GrID-Net results DataFrame of peak-gene pairs with Granger causality test statistics together with ranked pair lists from each correlation-based baseline, and define the pair sets differentially prioritized by GrID-Net. Retrieve H3K4me1 and H3K27ac ChIP-seq peaks for human neural progenitor cell samples from ChIP-Atlas and extract ATAC peak coordinates (chr_no, start, end) from the ATAC AnnData adata.var. Intersect each ATAC peak against the two ChIP-seq peak sets, labeling peaks overlapping H3K4me1 only as poised enhancers and those overlapping both H3K4me1 and H3K27ac as active enhancers. For each method-specific pair set, compute the poised-to-active ratio (count of H3K4me1-only peaks / count of H3K4me1+H3K27ac peaks), assemble a per-comparison ratio table and bar chart, and confirm the GrID-Net ratio exceeds every correlation baseline, as reported in Fig. 2f.

## Related tools

- **GrID-Net** (Produces the causal peak-gene pair rankings whose differentially prioritized pairs are tested for poised-enhancer enrichment against correlation baselines) — https://github.com/alexw16/gridnet
- **Schema** (Learned multimodal RNA+ATAC cell representations used upstream to build the kNN graph, infer diffusion pseudotime, and orient the DAG for GrID-Net pair scoring) — https://github.com/rs239/schema
- **ChIP-Atlas** (Source of H3K27ac and H3K4me1 ChIP-seq peak sets for human neural progenitor cells used to label enhancer states)

## Examples

```
results_df = gridnet.gridnet_multimodal(rna_adata,atac_adata,root_cell_marker_gene='TOP2A')
```

## Evaluation signals

- enhancer_priming_ratios.csv exists and contains exactly one poised-to-active ratio per method comparison (GrID-Net plus each correlation baseline)
- The GrID-Net poised-to-active ratio exceeds the ratio of every correlation-based baseline, matching the reported chromatin-priming result (Fig. 2f)
- Every ATAC peak is annotated with an interval overlap outcome against both the H3K4me1-only and H3K4me1+H3K27ac ChIP-seq sets, using chr_no/start/end from adata.var
- Bar-chart figure reproduces the qualitative ordering of Fig. 2f: GrID-Net pairs marked by more poised (H3K4me1-only) than active (H3K4me1+H3K27ac) enhancers, a pattern absent for correlation-prioritized pairs

## Limitations

- The priming readout requires multiomic single-cell data from matching tissue undergoing a dynamic process to supply pseudotemporal context; tissue matching is imperfect (corticogenesis data may explain only a subset of disease phenotypes such as SCZ).
- Enhancer labels are limited to the two histone marks available (H3K4me1, H3K4me1+H3K27ac) in ChIP-Atlas NPC and fetal neural samples; no additional chromatin modalities (Hi-C, CTCF topology) are incorporated.
- The comparison is between precomputed prioritizations, so a high poised ratio supports — but does not prove — causal prioritization, and GrID-Net associations remain causal hypotheses needing perturbation-based validation.

## Evidence

- [discussion] Candidate pair distance restriction for 10x Multiome: [evidence span withheld — anchor status: not_found]
