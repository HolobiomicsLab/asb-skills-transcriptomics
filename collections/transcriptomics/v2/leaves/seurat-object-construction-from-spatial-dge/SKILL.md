---
name: seurat-object-construction-from-spatial-dge
description: Use when a DGE plus spatial coordinates at ultra-high resolution (e.g.
  Seq-Scope, >1M barcodes/mm2 with very low UMI per barcode) must be aggregated into
  grid-level Seurat objects before dimension reduction and clustering.
when_to_use_negative:
- Input is not a collapsed spatial DGE with coordinates (e.g. already a clustered
  Seurat object or a feature table) — no re-aggregation is needed
- Target spatial segments would be smaller than subcellular compartments (~10um);
  the article notes subcellular analysis with MSSW is not feasible at that scale
- Working with data already at cellular resolution (e.g. standard VISIUM spots) where
  collapsing barcodes into grids provides no benefit
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: STtools
  role: Implements Steps C1/C2 that collapse the DGE from Step A3 into simple square
    grids and sliding-window grids and emit the Seurat RDS objects
  repo: https://github.com/seqscope/STtools
- name: Seurat
  role: Object format used to store the grid-collapsed DGE; later used for dimension
    reduction, clustering, and reference mapping in Step C3
- name: BayesSpace
  role: Spatial clustering engine paired with the sliding-window (MSSW) collapsed
    object in the C-track clustering chain
- name: STARsolo
  role: Upstream producer of the DGE (Step A3) that feeds the Seurat object construction
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:93fedb8f665847d80827ade9dbfb65009f312a9e2d993b93a552e6f4a1945717
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - BayesSpace
  - STARsolo
  - STtools
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# seurat-object-construction-from-spatial-dge

## Summary

Constructs Seurat objects for cell-type clustering from a STARsolo digital expression matrix (DGE) with spatial coordinates by collapsing spatial barcodes into square grids — either non-overlapping bins (Step C1) or multi-scale sliding-window overlapping bins (Step C2) — as done in STtools for Seq-Scope/Slide-seq/VISIUM data.

## When to use

Use when a DGE plus spatial coordinates at ultra-high resolution (e.g. Seq-Scope, >1M barcodes/mm2 with very low UMI per barcode) must be aggregated into grid-level Seurat objects before dimension reduction and clustering.

## When NOT to use

- Input is not a collapsed spatial DGE with coordinates (e.g. already a clustered Seurat object or a feature table) — no re-aggregation is needed
- Target spatial segments would be smaller than subcellular compartments (~10um); the article notes subcellular analysis with MSSW is not feasible at that scale
- Working with data already at cellular resolution (e.g. standard VISIUM spots) where collapsing barcodes into grids provides no benefit

## Inputs

- Digital expression matrix (DGE) output by STARsolo in Step A3
- Spatial coordinate file for the spatial barcodes (.txt)
- Grid parameters: binsize (e.g. 300) for simple square grids, window (e.g. 150) for sliding-window grids

## Outputs

- Step C1 RDS file: Seurat object with DGE collapsed into non-overlapping simple square grids
- Step C2 RDS file: Seurat object with DGE collapsed into sliding-window (MSSW) square grids

## How to apply

Take the DGE produced by STARsolo alignment (Step A3) together with its spatial coordinate file, and aggregate spatial barcodes into equal-sized non-overlapping square grids (e.g. binsize 300 in the Seq-Scope mouse-liver example) to build the Step C1 Seurat object; this raises UMI counts per spatial unit above the sparse per-barcode level that defeats standard clustering. In parallel, build the Step C2 object by aggregating the same DGE with the multi-scale sliding-window (MSSW) strategy using overlapping square bins (e.g. window 150), which provides higher-resolution, redundant views that BayesSpace and Step C3 later exploit for dimension reduction, clustering, and reference mapping. Persist both objects as RDS files so Step C3 can load them and use the simple square grids as the query for Seurat reference mapping. Choose binsize/window so each spatial segment stays larger than subcellular compartments (~10um), since MSSW is not feasible below that scale.

## Related tools

- **STtools** (Implements Steps C1/C2 that collapse the DGE from Step A3 into simple square grids and sliding-window grids and emit the Seurat RDS objects) — https://github.com/seqscope/STtools
- **Seurat** (Object format used to store the grid-collapsed DGE; later used for dimension reduction, clustering, and reference mapping in Step C3)
- **BayesSpace** (Spatial clustering engine paired with the sliding-window (MSSW) collapsed object in the C-track clustering chain)
- **STARsolo** (Upstream producer of the DGE (Step A3) that feeds the Seurat object construction) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --clustering False --lane-tiles 1_2106 --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- Both RDS files (C1 and C2) exist and load as valid Seurat objects containing grid-collapsed expression data
- C1 grid collapse uses non-overlapping equal-sized squares (e.g. binsize 300) while C2 uses overlapping sliding-window bins (e.g. window 150) — verify geometry of the collapsed coordinates
- The C1 -> C2 -> C3 chain runs end-to-end on the chained DGE with no step errors
- Downstream Step C3 accepts both RDS files and produces a combined cluster annotation with simple square grids as the reference-mapping query
- Aggregated units show UMI counts sufficient for clustering despite the sparse per-barcode counts typical of Seq-Scope (>1M barcodes/mm2)

## Limitations

- UMI count per spatial barcode is typically much lower at higher resolution, so aggregation parameters (binsize/window) directly determine whether clustering is feasible
- Spatial segments must be larger than subcellular compartments (~10um), so subcellular analysis with MSSW is not feasible
- No spatial smoothing algorithms are yet available in STtools for robust inference from extremely sparse expression profiles per spatial unit

## Evidence

- [readme] Step C1 DGE collapse: "Step C1 takes DGE from Step A3 and output `Seurat` object with collapsed DGE of simple square grids."
- [readme] Step C2 sliding window: "Step C2 takes DGE from **Step A3** and output Seurat object with collapsed DGE of square grids from sliding window strategy"
- [readme] Step C3 consumes C1/C2 RDS: "Step C3 takes in RDS file from Step C1 and Step C2 as input and performs dimension reduction, clustering"
- [readme] Seq-Scope example parameters: "--binsize 300 --window 150 -l 20 -o 'Sample' -c 2"
- [intro] Why grids are needed: "the UMI count per spatial barcode is typically much lower due to the limited number of mRNAs that can be captured"
- [discussion] MSSW scale limit: "each spatial segment still needs to be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible"
