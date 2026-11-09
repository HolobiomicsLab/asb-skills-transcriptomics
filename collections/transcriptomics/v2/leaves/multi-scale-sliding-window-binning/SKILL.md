---
name: multi-scale-sliding-window-binning
description: Use when analyzing Seq-Scope/Slide-seq DGEs with spatial coordinates
  where simple square-grid aggregation is too coarse to resolve cell-type boundaries
  and finer, overlapping-bin clustering is desired.
when_to_use_negative:
- The dataset is already high enough resolution that simple square aggregation suffices,
  e.g. VISIUM spot data (~120 barcodes/mm²) where overlapping bins add no resolution
  benefit
- 'The goal is subcellular analysis: spatial segments must remain larger than subcellular
  compartments (~10 μm), so MSSW cannot be pushed to subcellular scale'
- The platform resolution is too low for overlapping-bin benefits to be visible, as
  shown for Slide-seq where the MSSW benefit at 100 μm² was not visually pronounced
  versus 2500 μm² simple aggregation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STtools
  role: Runs Step C2 (sliding-window grid collapsing) and Step C3 (dimension reduction,
    clustering, reference mapping) on the DGE
  repo: https://github.com/seqscope/STtools
- name: Seurat
  role: Provides Seurat objects and the reference mapping used to transfer simple-bin
    cluster labels onto MSSW overlapping bins
  repo: https://satijalab.org/seurat/articles/spatial_vignette.html
- name: BayesSpace
  role: Clustering backend leveraged within the sliding-window (Step C2) Seurat/BayesSpace-based
    object
  repo: https://www.nature.com/articles/s41587-021-00935-2
- name: STARsolo
  role: Generates the upstream DGE with spatial barcodes (Step A3) that Step C2 aggregates
    via sliding windows
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:e65750f89f09e5ac1fec3301df0c532cddfc77c517c6821f4c7699f01f94aee4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - BayesSpace
  - STARsolo
  - STtools
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multi-scale-sliding-window-binning

## Summary

Implements STtools' multiscale sliding window (MSSW) aggregation, which collapses an ultra-high-resolution spatial DGE into overlapping square bins (e.g. ~100 μm² at ~4 μm² steps for Seq-Scope) instead of non-overlapping simple square grids, enabling several-folds-higher-resolution cell-type cluster maps after label transfer.

## When to use

Use when analyzing Seq-Scope/Slide-seq DGEs with spatial coordinates where simple square-grid aggregation is too coarse to resolve cell-type boundaries and finer, overlapping-bin clustering is desired.

## When NOT to use

- The dataset is already high enough resolution that simple square aggregation suffices, e.g. VISIUM spot data (~120 barcodes/mm²) where overlapping bins add no resolution benefit
- The goal is subcellular analysis: spatial segments must remain larger than subcellular compartments (~10 μm), so MSSW cannot be pushed to subcellular scale
- The platform resolution is too low for overlapping-bin benefits to be visible, as shown for Slide-seq where the MSSW benefit at 100 μm² was not visually pronounced versus 2500 μm² simple aggregation

## Inputs

- Digital expression matrix (DGE) with spatial coordinates produced by STtools Step A3 (STARsolo)
- Simple square-grid Seurat object from Step C1 (e.g. binsize 300) used as clustering query/reference

## Outputs

- Seurat object (RDS) with DGE collapsed into MSSW overlapping sliding-window square grids (Step C2)
- Finer-resolution cell-type cluster annotations transferred onto MSSW bins (Step C3)
- Spatial clustering maps comparing simple-bin vs MSSW binning (Step V2, e.g. spatial_cluster_maps.png)

## How to apply

Starting from the Step A3 STARsolo digital expression matrix with spatial coordinates, first run Step C1 to collapse barcodes into non-overlapping equal-sized square grids (e.g. --binsize 300 in the Seq-Scope example) and build a Seurat object that serves as the clustering reference/query. Then run Step C2, which takes the same DGE and collapses barcodes into overlapping square grids produced by the multi-scale sliding window (MSSW) strategy (Seq-Scope example: --binsize 300 --window 150; Slide-seq comparison used 100 μm² MSSW grids vs 2500 μm² simple grids). In Step C3, perform dimension reduction, clustering, and Seurat reference mapping using the simple square grids as the query, then transfer simple-bin cluster labels onto the MSSW overlapping bins for finer-resolution annotation. Visualize with Step V2 (UMAP and spatial clustering plots) and compare MSSW vs simple-aggregation maps side by side. The rationale is that overlapping bins recover cell-type boundaries at several folds higher spatial resolution than non-overlapping aggregation, though the benefit depends on the underlying platform resolution.

## Related tools

- **STtools** (Runs Step C2 (sliding-window grid collapsing) and Step C3 (dimension reduction, clustering, reference mapping) on the DGE) — https://github.com/seqscope/STtools
- **Seurat** (Provides Seurat objects and the reference mapping used to transfer simple-bin cluster labels onto MSSW overlapping bins) — https://satijalab.org/seurat/articles/spatial_vignette.html
- **BayesSpace** (Clustering backend leveraged within the sliding-window (Step C2) Seurat/BayesSpace-based object) — https://www.nature.com/articles/s41587-021-00935-2
- **STARsolo** (Generates the upstream DGE with spatial barcodes (Step A3) that Step C2 aggregates via sliding windows) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --clustering False --lane-tiles 1_2106 --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- The Step C2 Seurat RDS object exists, loads as a valid Seurat object, and contains DGE collapsed into overlapping sliding-window square grids
- Side-by-side comparison of Step V2 spatial clustering maps shows MSSW bin boundaries at several folds higher resolution than the simple-bin map for Seq-Scope data
- On Seq-Scope mouse liver (GSM5212844), MSSW maps delineate cell-type boundaries finer than simple 100 μm² square-bin aggregation
- On Slide-seq mouse cerebellum, the expected (negative) outcome is that the MSSW benefit is not visually pronounced, matching Fig. 1 F, G
- The C1→C2→C3 chain runs end-to-end on the chained DGE and each named RDS object (C1, C2, C3 combined annotation) loads

## Limitations

- Subcellular analysis with MSSW is not feasible: each spatial segment still needs to be larger than subcellular compartments (~10 μm)
- For Slide-seq (10 μm resolution), the MSSW benefit at 100 μm² grids was not visually pronounced compared to 2500 μm² simple aggregation
- Low UMI counts per spatial barcode at ultra-high resolution and lack of spatial smoothing/imputation limit robustness of extremely sparse per-bin expression profiles

## Evidence

- [readme] Step C2 takes DGE from Step A3 and output Seurat object with collapsed DGE of square grids from sliding window strategy: "**Step C2** takes DGE from **Step A3** and output Seurat object with collapsed DGE of square grids from sliding window strategy"
- [readme] Step C3 reference mapping with simple square grids as query: "Step C3 takes in RDS file from **Step C1** and **Step C2** as input and performs dimension reduction, clustering and conducts refernece mapping with simple square grids as query"
- [readme] MSSW Seq-Scope example parameters binsize 300 window 150: "--binsize 300 --window 150"
- [abstract] STtools several-folds-higher-resolution claim: "runs downstream analyses at several folds higher resolution than existing methods"
- [discussion] Subcellular MSSW infeasibility constraint: "each spatial segment still needs to be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible"
