---
name: high-resolution-spatial-image-rendering
description: Use when you have a Seq-Scope (or similar sub-micrometer) spatial DGE
  with per-barcode spatial coordinates and want to visually resolve cell-type (e.g.,
  periportal vs. pericentral) or subcellular spatial patterns via marker-gene images.
when_to_use_negative:
- Your analysis requires subcellular MSSW (sliding-window) clustering — STtools spatial
  segments must be larger than subcellular compartments (~10 µm), so subcellular analysis
  with MSSW is not feasible
- The data has already been collapsed/binned into coarse grids and per-barcode sub-micrometer
  resolution is no longer recoverable
- You need automated histology image overlay or spatial expression imputation — these
  are not implemented in STtools
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STtools
  role: Aggregates marker geneset expression per spatial barcode without binning and
    renders the high-resolution RGB image and Step V1 subcellular spliced/unspliced
    plots
  repo: https://github.com/seqscope/STtools
- name: STARsolo
  role: Upstream Step A3 alignment that produces the Velocyto-format DGE consumed
    for the RGB and subcellular plots
  repo: https://github.com/alexdobin/STAR
- name: Seurat
  role: Handles the Seurat-object grid DGEs (Steps C1–C3) whose cell-type annotations
    contextualize the rendered spatial images
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:0b2b2e27636d2d2de4bbce062de112ed52e33d0c0506a4ca5af4d8d311d04c71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STtools
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# high-resolution-spatial-image-rendering

## Summary

Render ultra-high-resolution spatial transcriptomics visualizations with STtools, including RGB geneset plots that map three user-selected marker gene sets to R, G, and B channels at <1 μm² per pixel, plus Step V1 subcellular spliced/unspliced plots.

## When to use

Use when you have a Seq-Scope (or similar sub-micrometer) spatial DGE with per-barcode spatial coordinates and want to visually resolve cell-type (e.g., periportal vs. pericentral) or subcellular spatial patterns via marker-gene images.

## When NOT to use

- Your analysis requires subcellular MSSW (sliding-window) clustering — STtools spatial segments must be larger than subcellular compartments (~10 µm), so subcellular analysis with MSSW is not feasible
- The data has already been collapsed/binned into coarse grids and per-barcode sub-micrometer resolution is no longer recoverable
- You need automated histology image overlay or spatial expression imputation — these are not implemented in STtools

## Inputs

- Velocyto-format DGE from STARsolo Step A3 with per-barcode spatial coordinates (SeqScope mouse liver, e.g. GSM5212844 / SRR14082757)
- Three user-selected marker gene sets (one per R/G/B color channel)
- Optionally VISIUM or Slide-seq DGE with spatial coordinates (STtools example data 2 and 3)

## Outputs

- High-resolution RGB geneset PNG image at <1 μm² per pixel, one channel per marker geneset
- Step V1 subcellular plots showing spatial pattern of spliced/unspliced reads

## How to apply

Load the digital expression matrix annotated with spatial coordinates (the Velocyto-format DGE produced by the STARsolo alignment Step A3, or an equivalent externally supplied matrix). Select the arbitrary marker gene sets to encode in the R, G, and B channels of the visualization. Use STtools to quantify each geneset's aggregate expression per spatial barcode without spatial binning, so the sub-micrometer barcode resolution of Seq-Scope is retained. Render the high-resolution RGB image mapping each channel's intensity to the normalized geneset expression at <1 μm² per pixel. Additionally generate the Step V1 subcellular plots showing the spatial pattern of spliced versus unspliced reads from the Velocyto-format DGE.

## Related tools

- **STtools** (Aggregates marker geneset expression per spatial barcode without binning and renders the high-resolution RGB image and Step V1 subcellular spliced/unspliced plots) — https://github.com/seqscope/STtools
- **STARsolo** (Upstream Step A3 alignment that produces the Velocyto-format DGE consumed for the RGB and subcellular plots) — https://github.com/alexdobin/STAR
- **Seurat** (Handles the Seurat-object grid DGEs (Steps C1–C3) whose cell-type annotations contextualize the rendered spatial images)

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 liver_tile2106_sub_R1.fastq.gz --second-fq2 liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --clustering False --binsize 300 --window 150
```

## Evaluation signals

- Output PNG exists and depicts spatial structure at <1 μm² per pixel resolution
- Cell-type differences (periportal vs. pericentral hepatocytes vs. macrophages) are visually discernible in the RGB plot
- Subcellular patterns (e.g., nucleus vs. mitochondria vs. macrophages) are visually discernible without sacrificing resolution
- Each channel intensity corresponds to the normalized aggregate expression of its marker geneset quantified per spatial barcode with no spatial binning
- Step V1 output separates spliced from unspliced read patterns derived from the Velocyto-format DGE

## Limitations

- Spatial segments for sliding-window (MSSW) analysis must be larger than subcellular compartments (~10 µm); subcellular analysis with MSSW is not feasible
- No spatial smoothing or imputation methods are available in STtools for the extremely sparse expression profiles per spatial unit, limiting truly subcellular quantitative inference
- No joint clustering of cellular and subcellular components and no automatic overlay of histological images with spatial expressions

## Evidence

- [abstract] Visualizations including marker-gene RGB plots and subcellular architectures: "It also generates various visualizations including transcriptome density, cell type mapping, marker gene highlighting, and subcellular architectures"
- [readme] Step V1 subcellular plots from Velocyto-format DGE: "**Step V1** takes DGE (Velocyto-format) from **Step A3** and generate subcellular plots showing pattern of spliced/unspliced reads"
- [methods] STtools processes Seq-Scope, Slide-seq, and VISIUM data: "STtools is able to process spatial transcriptomics (ST) data from various platforms including, but not limited to, Seq-Scope, Slide-seq, and VISIUM."
- [discussion] MSSW segments must exceed subcellular compartment size: "each spatial segment still needs to be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible"
- [intro] Barcode density scales with resolution, motivating unbinned rendering: "the number of spatial barcodes per mm2 rapidly increases as resolution increases (~120 for VISIUM, ~3K for Slide-seq, >1M for Seq-Scope), and few tools seamlessly scale"
