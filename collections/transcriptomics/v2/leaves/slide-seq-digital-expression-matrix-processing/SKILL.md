---
name: slide-seq-digital-expression-matrix-processing
description: Use when input is a Slide-seq DGE plus spatial coordinates (e.g., STtools
  'example data 3') and the goal is cell-type clustering and spatial cluster maps
  at cellular scale, comparing simple aggregation vs sliding-window bins.
when_to_use_negative:
- Input is raw FASTQ from a full pipeline run (Seq-Scope/VISIUM) — alignment and preprocessing
  steps A1-A3 must run first before C1/C2 collapse barcodes into grids
- Goal is subcellular analysis of spliced/unspliced reads — MSSW sliding-window segments
  must be larger than subcellular compartments (~10 um), so subcellular analysis with
  MSSW is not feasible (use Step V1 on Velocyto-format DGE instead)
- Platform is VISIUM (~120 spots/mm2) where spot-level aggregation already matches
  cellular resolution and grid collapsing adds little
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STtools
  role: 'Main pipeline: Steps C1/C2 collapse the Slide-seq DGE into simple square
    grids and MSSW sliding-window grids; Step C3 does dimension reduction and clustering;
    Step V2 renders UMAP and spatial cluster maps'
  repo: https://github.com/seqscope/STtools
- name: Seurat
  role: Receives the collapsed DGE as Seurat objects and performs dimension reduction,
    cell-type clustering, and reference mapping of simple square grids as query
  repo: https://satijalab.org/seurat/articles/spatial_vignette.html
- name: STARsolo
  role: Only needed if starting from raw FASTQ rather than a precomputed DGE; aligns
    cDNA to the reference genome to generate the DGE (Step A3)
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:46a586a84ddb11811b1d4a972626ff5e2e97cf5ed992a49fa3ddcfeec2334d63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STtools
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# slide-seq-digital-expression-matrix-processing

## Summary

Process Slide-seq digital expression matrices (DGEs) with spatial coordinates through STtools' grid-collapsing, clustering, and visualization steps, comparing simple square-grid aggregation against the multiscale sliding-window (MSSW) strategy in Seurat.

## When to use

Use when input is a Slide-seq DGE plus spatial coordinates (e.g., STtools 'example data 3') and the goal is cell-type clustering and spatial cluster maps at cellular scale, comparing simple aggregation vs sliding-window bins.

## When NOT to use

- Input is raw FASTQ from a full pipeline run (Seq-Scope/VISIUM) — alignment and preprocessing steps A1-A3 must run first before C1/C2 collapse barcodes into grids
- Goal is subcellular analysis of spliced/unspliced reads — MSSW sliding-window segments must be larger than subcellular compartments (~10 um), so subcellular analysis with MSSW is not feasible (use Step V1 on Velocyto-format DGE instead)
- Platform is VISIUM (~120 spots/mm2) where spot-level aggregation already matches cellular resolution and grid collapsing adds little

## Inputs

- Slide-seq digital expression matrix (DGE, from STtools example data 3 Google Drive folder)
- Slide-seq spatial coordinates file (.txt, spatial barcode x/y positions)
- Seurat RDS objects from Step C1 (simple square grids) and Step C2 (sliding-window grids)

## Outputs

- Seurat object with collapsed DGE of simple square grids (Step C1)
- Seurat object with collapsed DGE of sliding-window square grids (Step C2)
- Dimension-reduced, clustered Seurat objects (Step C3)
- UMAP embeddings, spatial cluster maps, and feature violin plots (Step V2, e.g. slideseq_cluster_maps.png)

## How to apply

Download the Slide-seq digital expression data and spatial coordinates from the STtools example data deposit (Google Drive 'example data 3' folder referenced in the README). Because Slide-seq DGEs are already available, start at the clustering steps rather than alignment: run Step C1 to collapse spatial barcodes into non-overlapping equal-sized square grids (2500 um2 for Slide-seq, i.e. ~50 um squares) producing a Seurat object of the simple-aggregation DGE, and Step C2 to produce a Seurat object of overlapping sliding-window bins (100 um2, i.e. 10 um windows, the MSSW strategy). Run Step C3 on the RDS files from C1 and C2 to perform dimension reduction and clustering, using the simple square grids as query for reference mapping; expect low UMI counts per spatial barcode given Slide-seq's ~10 um resolution. Finally run Step V2 to generate UMAP, spatial clustering, and feature violin plots. Rationale: at Slide-seq resolution (~3K spatial barcodes per mm2, ~10 um beads) the MSSW benefit is limited, so the side-by-side comparison is the point of the exercise, not a guaranteed visual improvement.

## Related tools

- **STtools** (Main pipeline: Steps C1/C2 collapse the Slide-seq DGE into simple square grids and MSSW sliding-window grids; Step C3 does dimension reduction and clustering; Step V2 renders UMAP and spatial cluster maps) — https://github.com/seqscope/STtools
- **Seurat** (Receives the collapsed DGE as Seurat objects and performs dimension reduction, cell-type clustering, and reference mapping of simple square grids as query) — https://satijalab.org/seurat/articles/spatial_vignette.html
- **STARsolo** (Only needed if starting from raw FASTQ rather than a precomputed DGE; aligns cDNA to the reference genome to generate the DGE (Step A3)) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- Step C1 and C2 each yield a valid Seurat RDS object whose counts reflect collapsed barcodes over 2500 um2 (simple) and 100 um2 (MSSW) grids respectively
- Step C3 completes dimension reduction and clustering on both RDS inputs, with simple square grids used as the reference-mapping query
- Step V2 produces UMAP, spatial clustering, and feature violin plots rendered across the cerebellum tissue coordinates
- Side-by-side comparison of slideseq_cluster_maps.png reproduces the paper's Fig. 1E-H observation that the MSSW benefit is not visually pronounced for Slide-seq, confirmed by expert or LLM inspection

## Limitations

- For the Slide-seq mouse cerebellum dataset, the benefit of the MSSW strategy (100 um2 grids) was not visually pronounced compared to simple aggregation, primarily due to the low resolution of the Slide-seq technology (Fig. 1 F, G)
- Each spatial segment still needs to be larger than subcellular compartments (e.g. ~10 um), so subcellular analysis with MSSW is not feasible
- UMI count per spatial barcode is typically much lower at higher resolution, which can degrade tools that assume sufficient UMIs per barcode for cell-type clustering
- STtools requires a Linux operating system with STAR >= 2.7.5c, seqtk, R >= 4.0.0, Python >= 3.0, perl, and pigz installed

## Evidence

- [methods] Slide-seq example DGE and coordinates: "SlideSeq digital expression data and spatial coordinates are avaialbel at [example data 3]"
- [methods] Step C1 simple grids: "**Step C1** takes DGE from **Step A3** and output `Seurat` object with collapsed DGE of simple square grids."
- [methods] Step C2 sliding window: "**Step C2** takes DGE from **Step A3** and output Seurat object with collapsed DGE of square grids from sliding window strategy"
- [readme] Step C3 clustering: "**Step C3** takes in RDS file from **Step C1** and **Step C2** as input and performs dimension reduction, clustering and conducts refernece mapping with simple square grids as query"
- [methods] Step V2 visualization: "**Step V2** takes DGE (Velocyto-format) from **Step C3** and generate UMAP, Spatial clustering, and feature violin plots."
- [methods] Slide-seq platform support: "STtools is able to process spatial transcriptomics (ST) data from various platforms including, but not limited to, Seq-Scope, Slide-seq, and VISIUM."
