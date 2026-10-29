---
name: spatial-transcriptomics-rgb-visualization
description: Use when you have a digital expression matrix with spatial coordinates
  (e.g., Seq-Scope Velocyto-format DGE from STARsolo alignment) and need to visualize
  marker gene sets or subcellular spliced/unspliced read patterns at sub-micrometer
  resolution.
when_to_use_negative:
- The analysis requires subcellular inference via sliding-window (MSSW) clustering
  — spatial segments must be larger than subcellular compartments (~10 μm), so subcellular
  analysis with MSSW is not feasible
- The dataset lacks per-barcode spatial coordinates (e.g., only an already-binned
  or clustered feature table is available), since un-binned barcode-level rendering
  is essential to retain sub-micrometer resolution
- The data is lower-resolution platform data where existing tools suffice and the
  goal is UMAP/spatial clustering rather than RGB marker rendering (Step V2 covers
  UMAP, spatial clustering, and feature violin plots instead)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: STtools
  role: 'Primary pipeline: quantifies marker geneset expression per spatial barcode
    without binning, renders <1 μm²/pixel RGB images, and generates Step V1 subcellular
    spliced/unspliced plots'
  repo: https://github.com/seqscope/STtools
- name: STARsolo
  role: Aligns cDNA sequences to the reference genome within STtools Step A3 to produce
    the DGE (including Velocyto-format output) consumed by the visualization
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:cd0b0ebb2544247b67e2130c100b3401b0d1f43339b210232afc76ff072c0769
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STtools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spatial-transcriptomics-rgb-visualization

## Summary

Render ultra-high-resolution RGB geneset plots and Step V1 subcellular (spliced/unspliced) plots from spatial transcriptomics data using STtools, preserving sub-micrometer barcode resolution (e.g., Seq-Scope at <1 μm²/pixel).

## When to use

Use when you have a digital expression matrix with spatial coordinates (e.g., Seq-Scope Velocyto-format DGE from STARsolo alignment) and need to visualize marker gene sets or subcellular spliced/unspliced read patterns at sub-micrometer resolution.

## When NOT to use

- The analysis requires subcellular inference via sliding-window (MSSW) clustering — spatial segments must be larger than subcellular compartments (~10 μm), so subcellular analysis with MSSW is not feasible
- The dataset lacks per-barcode spatial coordinates (e.g., only an already-binned or clustered feature table is available), since un-binned barcode-level rendering is essential to retain sub-micrometer resolution
- The data is lower-resolution platform data where existing tools suffice and the goal is UMAP/spatial clustering rather than RGB marker rendering (Step V2 covers UMAP, spatial clustering, and feature violin plots instead)

## Inputs

- Velocyto-format digital expression matrix (DGE) from STtools Step A3 STARsolo alignment
- Spatial coordinates .txt file for spatial barcodes (from Step A1)
- User-selected marker gene sets for the R, G, and B channels (e.g., periportal, pericentral, macrophage markers)
- Seq-Scope mouse liver data (e.g., GSM5212844 / SRR14082757) or equivalent externally supplied DGE with coordinates

## Outputs

- High-resolution RGB geneset PNG image at <1 μm²/pixel, one channel per marker geneset
- Step V1 subcellular plots showing spatial patterns of spliced vs. unspliced reads
- Barcode/HDMI density plots comparable to HE images for tissue boundary estimation (Step A2, optional context)

## How to apply

Load the digital expression matrix (Velocyto-format DGE produced by STtools Step A3 STARsolo alignment, or an equivalent externally supplied matrix) annotated with spatial coordinates for the sample, such as the Seq-Scope mouse liver dataset. Select arbitrary marker gene sets to encode in the R, G, and B color channels, and use STtools to quantify each geneset's aggregate expression per spatial barcode WITHOUT spatial binning, so the sub-micrometer barcode resolution (>1M barcodes/mm² for Seq-Scope vs ~120/mm² for VISIUM) is retained. Render the high-resolution RGB image at <1 μm² per pixel, mapping each channel's intensity to the normalized geneset expression, so cell-type differences (periportal vs. pericentral hepatocytes vs. macrophages) and subcellular differences (nucleus vs. mitochondria vs. macrophages) are both resolvable. Additionally generate Step V1 subcellular plots from the Velocyto-format DGE showing the spatial pattern of spliced versus unspliced reads. Note that MSSW-based spatial segments must be larger than subcellular compartments (~10 μm), so subcellular visualization relies on the un-binned barcode-level RGB rendering rather than sliding-window clustering.

## Related tools

- **STtools** (Primary pipeline: quantifies marker geneset expression per spatial barcode without binning, renders <1 μm²/pixel RGB images, and generates Step V1 subcellular spliced/unspliced plots) — https://github.com/seqscope/STtools
- **STARsolo** (Aligns cDNA sequences to the reference genome within STtools Step A3 to produce the DGE (including Velocyto-format output) consumed by the visualization) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --clustering False -o 'Sample' -c 2
```

## Evaluation signals

- Output PNG exists and depicts spatial structure at <1 μm²/pixel resolution
- Cell-type patterns (periportal vs. pericentral hepatocytes vs. macrophages) are visually discernible in the RGB image without sacrificing resolution
- Subcellular transcriptomic patterns (e.g., nucleus vs. mitochondria vs. macrophages) are resolvable in the rendered image
- Geneset aggregation was computed per spatial barcode without spatial binning, retaining >1M barcodes/mm² scale for Seq-Scope data
- Step V1 plots were generated from the Velocyto-format DGE and show distinct spliced vs. unspliced read patterns

## Limitations

- Spatial smoothing algorithms for robust inference from extremely sparse UMI counts per spatial barcode are not yet available, limiting truly subcellular inference
- Subcellular analysis with MSSW is not feasible; each spatial segment must be larger than subcellular compartments (~10 μm)
- UMI counts per spatial barcode are very low at sub-micrometer resolution, so tools assuming sufficient UMIs per barcode for clustering may perform poorly

## Evidence

- [abstract] visualizations include subcellular architectures: "It also generates various visualizations including transcriptome density, cell type mapping, marker gene highlighting, and subcellular architectures"
- [readme] Step V1 consumes Velocyto DGE: "**Step V1** takes DGE (Velocyto-format) from **Step A3** and generate subcellular plots showing pattern of spliced/unspliced reads"
- [intro] existing tools miss subcellular structure: "existing tools do not account for subcellular components in their analysis and visualization to accommodate the ultra-high-resolution from recent technologies"
- [discussion] MSSW subcellular infeasibility: "each spatial segment still needs to be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible"
- [intro] barcode density scales with resolution: "the number of spatial barcodes per mm2 rapidly increases as resolution increases (~120 for VISIUM, ~3K for Slide-seq, >1M for Seq-Scope)"
