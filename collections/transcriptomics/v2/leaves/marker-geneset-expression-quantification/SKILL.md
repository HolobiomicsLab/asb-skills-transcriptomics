---
name: marker-geneset-expression-quantification
description: Use when the analysis goal is to visualize spatial expression of specific
  marker gene sets at native sub-micrometer barcode resolution (e.g., Seq-Scope data
  with >1M barcodes/mm²), and you have an unaggregated digital expression matrix with
  spatial coordinates.
when_to_use_negative:
- Input DGE has already been collapsed into grid-level Seurat objects (Step C1/C2
  output), destroying the per-barcode spatial resolution needed for subcellular visualization
- The goal is cell-type clustering or reference mapping rather than marker geneset
  visualization — use STtools Steps C1–C3/V2 with the sliding window strategy instead
- Analysis is at VISIUM (~120 barcodes/mm²) or Slide-seq resolution where per-barcode
  RGB rendering is unnecessary; lower-resolution feature plotting suffices
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STtools
  role: 'Primary pipeline: quantifies per-barcode geneset expression without binning
    and renders RGB and subcellular (Step V1) plots from the STARsolo DGE'
  repo: https://github.com/seqscope/STtools
- name: STARsolo
  role: Alignment step (A3) that produces the digital expression matrix (DGE) in Velocyto
    format consumed by the visualization
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:da38d869779ccdc906ebbddbd7306064b7be9c63261aef42eae667e67d8b39a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STtools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# marker-geneset-expression-quantification

## Summary

Quantify user-selected marker gene sets in spatial transcriptomics data and encode their aggregate expression into R/G/B image channels rendered at <1 μm²/pixel, enabling STtools RGB plots that resolve cell-type (periportal vs. pericentral hepatocytes, macrophages) and subcellular (nucleus vs. mitochondria) patterns in Seq-Scope mouse liver data.

## When to use

Use when the analysis goal is to visualize spatial expression of specific marker gene sets at native sub-micrometer barcode resolution (e.g., Seq-Scope data with >1M barcodes/mm²), and you have an unaggregated digital expression matrix with spatial coordinates.

## When NOT to use

- Input DGE has already been collapsed into grid-level Seurat objects (Step C1/C2 output), destroying the per-barcode spatial resolution needed for subcellular visualization
- The goal is cell-type clustering or reference mapping rather than marker geneset visualization — use STtools Steps C1–C3/V2 with the sliding window strategy instead
- Analysis is at VISIUM (~120 barcodes/mm²) or Slide-seq resolution where per-barcode RGB rendering is unnecessary; lower-resolution feature plotting suffices

## Inputs

- Velocyto-format digital expression matrix (DGE) with spatial coordinates from STtools Step A3 STARsolo alignment (e.g., Seq-Scope mouse liver sample GSM5212844)
- User-defined marker gene sets for the R, G, and B color channels
- Spatial coordinates (.txt) file mapping barcodes to pixel positions

## Outputs

- High-resolution RGB PNG image (<1 μm²/pixel) with each channel's intensity proportional to normalized per-barcode geneset expression
- Per-spatial-barcode aggregate expression values for each marker geneset
- Optional Step V1 subcellular plots showing spliced/unspliced read spatial patterns

## How to apply

Start from the Velocyto-format or equivalent digital expression matrix (DGE) produced by the STARsolo alignment step (STtools Step A3), which retains per-barcode spatial coordinates at sub-micrometer resolution. Select the arbitrary marker gene sets to be mapped to the R, G, and B channels, then aggregate each geneset's expression per spatial barcode without collapsing barcodes into grids — the key decision point is skipping Steps C1/C2 grid-collapsing, since collapsing would sacrifice the sub-micrometer resolution that makes subcellular patterns (nucleus, mitochondria, macrophages) visible. Render the RGB image with each channel's intensity mapped to normalized per-barcode geneset expression, targeting <1 μm² per pixel. Optionally also run Step V1, which takes the Velocyto-format DGE from Step A3 to generate subcellular plots showing the spatial pattern of spliced versus unspliced reads.

## Related tools

- **STtools** (Primary pipeline: quantifies per-barcode geneset expression without binning and renders RGB and subcellular (Step V1) plots from the STARsolo DGE) — https://github.com/seqscope/STtools
- **STARsolo** (Alignment step (A3) that produces the digital expression matrix (DGE) in Velocyto format consumed by the visualization) — https://github.com/alexdobin/STAR

## Evaluation signals

- Output PNG exists and depicts spatial structure at <1 μm²/pixel resolution
- Cell-type-level patterns (periportal vs. pericentral hepatocytes vs. macrophages) are visually discernible in the RGB image
- Subcellular patterns (e.g., nucleus vs. mitochondria transcript distribution) are discernible, confirming no spatial binning was applied
- Each color channel intensity reflects the normalized aggregate expression of the corresponding marker geneset across spatial barcodes
- Step V1 subcellular plot shows distinguishable spliced vs. unspliced read spatial patterns consistent with the Velocyto-format DGE

## Limitations

- Subcellular analysis with the mean-shifting sliding window (MSSW) is not feasible; spatial segments must be larger than subcellular compartments (~10 μm), so subcellular inference relies on unbinned per-barcode visualization
- UMI count per spatial barcode is typically very low at sub-micrometer resolution, so spatial smoothing algorithms for robust inference from extremely sparse per-barcode profiles are not yet available in STtools
- No method to impute spatial expression profiles and no automatic overlay of histological images with spatial expression exist in current STtools

## Evidence

- [abstract] visualization capability: "It also generates various visualizations including transcriptome density, cell type mapping, marker gene highlighting, and subcellular architectures"
- [methods] Step V1 input: "**Step V1** takes DGE (Velocyto-format) from **Step A3** and generate subcellular plots showing pattern of spliced/unspliced reads"
- [readme] DGE production: "**Step A3** takes valid barcodes `whitelist.txt`, 2nd-seq `fastq.gz` files, and the STAR indices of reference genome as input to run `STARsolo` alignment; this step outputs digital expression matrix"
- [intro] resolution challenge: "existing tools do not account for subcellular components in their analysis and visualization to accommodate the ultra-high-resolution from recent technologies"
- [discussion] MSSW subcellular limitation: "each spatial segment still needs to be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible"
