---
name: spatial-barcode-grid-aggregation
description: Use when a STARsolo digital expression matrix with spatial coordinates
  has too few UMIs per spatial barcode for direct cell-type clustering (Seq-Scope,
  Slide-seq), and you need binned Seurat objects at super-cellular resolution.
when_to_use_negative:
- The input is raw FASTQ or an unaligned barcode file — aggregation requires a post-STARsolo
  DGE from Step A3 (preprocessing/alignment steps A1–A3 come first).
- Subcellular-resolution analysis is the goal — spatial segments must be larger than
  subcellular compartments (~10 μm), so MSSW cannot resolve subcellular structure.
- Each spatial barcode already has sufficient UMI depth for direct cell-type clustering
  (e.g. standard VISIUM-resolution data where aggregation adds little).
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: STtools
  role: Hosts Steps C1 and C2, which collapse spatial barcodes into simple square
    grids and sliding-window (MSSW) grids respectively; invoked via sttools.py with
    --binsize and --window parameters
  repo: https://github.com/seqscope/STtools
- name: Seurat
  role: Consumes the collapsed DGEs to build Seurat objects and later performs dimension
    reduction, clustering, and reference mapping (Step C3)
  repo: https://satijalab.org/seurat/articles/spatial_vignette.html
- name: BayesSpace
  role: Spatial clustering backend used with the sliding-window Seurat object from
    Step C2
  repo: https://www.nature.com/articles/s41587-021-00935-2
- name: STARsolo
  role: Produces the upstream DGE (Step A3) that both aggregation tracks consume
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_006
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:c6f6d89a6daf2b64092a33ab3f880ca28ac90f78c2cdad2241e4af0af1d81f2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - BayesSpace
  - STARsolo
  - STtools
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spatial-barcode-grid-aggregation

## Summary

STtools' two-track aggregation strategy that collapses sparse per-spatial-barcode DGEs into simple non-overlapping square grids (Step C1) and multi-scale sliding-window overlapping grids (Step C2) so that cell-type clustering becomes feasible on ultra-high-resolution platforms like Seq-Scope, Slide-seq, and VISIUM.

## When to use

Use when a STARsolo digital expression matrix with spatial coordinates has too few UMIs per spatial barcode for direct cell-type clustering (Seq-Scope, Slide-seq), and you need binned Seurat objects at super-cellular resolution.

## When NOT to use

- The input is raw FASTQ or an unaligned barcode file — aggregation requires a post-STARsolo DGE from Step A3 (preprocessing/alignment steps A1–A3 come first).
- Subcellular-resolution analysis is the goal — spatial segments must be larger than subcellular compartments (~10 μm), so MSSW cannot resolve subcellular structure.
- Each spatial barcode already has sufficient UMI depth for direct cell-type clustering (e.g. standard VISIUM-resolution data where aggregation adds little).

## Inputs

- Digital expression matrix (DGE) from STtools Step A3 (STARsolo alignment output)
- Spatial coordinates file for the spatial barcodes
- Step C1 RDS/Seurat object (simple square-grid collapsed DGE)
- Step C2 RDS/Seurat object (sliding-window collapsed DGE)

## Outputs

- Step C1 Seurat object (.rds) with DGE collapsed into non-overlapping equal-sized square grids
- Step C2 Seurat/BayesSpace object (.rds) with DGE collapsed into sliding-window square grids
- Cluster annotations transferred from simple grids onto MSSW overlapping bins for finer-resolution mapping

## How to apply

Take the Step A3 digital expression matrix (produced by STARsolo alignment) plus spatial coordinates and run Step C1 to collapse spatial barcodes into non-overlapping, equal-sized square bins — e.g. binsize 300 in the SeqScope example (100 μm² for Seq-Scope mouse liver, 2500 μm² for Slide-seq) — emitting a Seurat object with the collapsed DGE. In parallel, run Step C2 on the same DGE using the multi-scale sliding-window (MSSW) strategy with overlapping square bins (e.g. window 150), emitting a second Seurat/BayesSpace-based object. Choose bin sizes so each segment captures enough transcripts for clustering; note segments must remain larger than subcellular compartments (~10 μm). The two RDS objects are then consumed together by Step C3, where dimension reduction, clustering, and Seurat reference mapping use the simple square grids as query and transfer labels onto the finer MSSW bins.

## Related tools

- **STtools** (Hosts Steps C1 and C2, which collapse spatial barcodes into simple square grids and sliding-window (MSSW) grids respectively; invoked via sttools.py with --binsize and --window parameters) — https://github.com/seqscope/STtools
- **Seurat** (Consumes the collapsed DGEs to build Seurat objects and later performs dimension reduction, clustering, and reference mapping (Step C3)) — https://satijalab.org/seurat/articles/spatial_vignette.html
- **BayesSpace** (Spatial clustering backend used with the sliding-window Seurat object from Step C2) — https://www.nature.com/articles/s41587-021-00935-2
- **STARsolo** (Produces the upstream DGE (Step A3) that both aggregation tracks consume) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --clustering False --lane-tiles 1_2106 --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- Both the Step C1 and Step C2 RDS files exist and load as valid Seurat objects from the chained A3 DGE.
- Simple-grid bins are non-overlapping and equal-sized (e.g. --binsize 300), while MSSW bins are overlapping sliding windows (e.g. --window 150); verify the parameters in the invocation log.
- On Seq-Scope mouse liver, the MSSW map should delineate cell-type boundaries at several-fold higher resolution than the simple-bin map (side-by-side spatial_cluster_maps.png).
- On Slide-seq cerebellum (100 μm² MSSW vs 2500 μm² simple bins), expect the MSSW benefit NOT to be visually pronounced — consistent with Fig. 1 F, G — due to Slide-seq's low resolution.
- Step C3 runs end-to-end and yields a combined cluster annotation object with simple grids as the reference-mapping query.

## Limitations

- Subcellular analysis with MSSW is not feasible: each spatial segment must be larger than subcellular compartments (e.g. ~10 μm).
- UMI count per spatial barcode is typically much lower at higher resolution, and no spatial smoothing or imputation method exists in STtools to compensate for extremely sparse per-segment expression profiles.
- The MSSW advantage is technology-dependent: for low-resolution Slide-seq data the benefit over simple aggregation is not visually pronounced.

## Evidence

- [readme] Step C1 takes DGE from Step A3 and output Seurat object with collapsed DGE of simple square grids.: "**Step C1** takes DGE from **Step A3** and output `Seurat` object with collapsed DGE of simple square grids."
- [readme] Step C2 sliding window: "**Step C2** takes DGE from **Step A3** and output Seurat object with collapsed DGE of square grids from sliding window strategy"
- [readme] Step C3 combined input: "**Step C3** takes in RDS file from **Step C1** and **Step C2** as input and performs dimension reduction, clustering and conducts refernece mapping with simple square grids as query"
- [results] MSSW finer boundaries on Seq-Scope: [evidence span withheld — anchor status: not_found]
- [results] MSSW not pronounced on Slide-seq: [evidence span withheld — anchor status: not_found]
- [discussion] MSSW subcellular infeasibility: "each spatial segment still needs to be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible"
