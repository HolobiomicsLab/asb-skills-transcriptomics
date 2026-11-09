---
name: cross-scale-cell-type-label-transfer
description: Use when a sub-micrometer-resolution ST dataset (e.g. Seq-Scope, >1M
  spatial barcodes per mm2) has too few UMIs per spatial barcode for direct clustering,
  and cell-type labels from coarse bins must be transferred to finer overlapping sliding-window
  bins.
when_to_use_negative:
- 'Attempting subcellular-resolution label transfer: each spatial segment must be
  larger than subcellular compartments (~10 um), so subcellular analysis with MSSW
  is not feasible'
- The dataset already has sufficient UMIs per spatial barcode (e.g. VISIUM 100 um
  spots), so simple square-grid clustering without sliding-window transfer is adequate
- Only a single spatial aggregation scale exists — the method requires paired Step
  C1 simple grids and Step C2 sliding-window grids as query and reference
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STtools
  role: Runs Step C1 (simple square-grid collapse) and Step C2 (MSSW sliding-window
    collapse, --binsize 300 --window 150) whose outputs feed the label transfer; runs
    Step V2 visualization
  repo: https://github.com/seqscope/STtools
- name: Seurat
  role: Performs dimension reduction, cell-type clustering, and reference mapping
    in Step C3, transferring simple square-grid labels onto MSSW overlapping bins
  repo: https://satijalab.org/seurat/articles/spatial_vignette.html
- name: STARsolo
  role: Generates the DGE from raw FASTQ in Step A3, the input expression matrix that
    is collapsed at both scales
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:c0a0bd985a36969815dbd8eef83172d4c7b38b9bae597b13b7748ded3fc03c5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STtools
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Cross-scale cell-type label transfer

## Summary

Assign cell types at subcellular/super-cellular resolution by clustering STtools' multiscale sliding-window (MSSW) bins against labels learned on simple square-grid bins, using Seurat reference mapping. This overcomes the low UMI counts per barcode at sub-micrometer Seq-Scope resolution by transferring labels from coarser aggregation to finer overlapping grids.

## When to use

Use when a sub-micrometer-resolution ST dataset (e.g. Seq-Scope, >1M spatial barcodes per mm2) has too few UMIs per spatial barcode for direct clustering, and cell-type labels from coarse bins must be transferred to finer overlapping sliding-window bins.

## When NOT to use

- Attempting subcellular-resolution label transfer: each spatial segment must be larger than subcellular compartments (~10 um), so subcellular analysis with MSSW is not feasible
- The dataset already has sufficient UMIs per spatial barcode (e.g. VISIUM 100 um spots), so simple square-grid clustering without sliding-window transfer is adequate
- Only a single spatial aggregation scale exists — the method requires paired Step C1 simple grids and Step C2 sliding-window grids as query and reference

## Inputs

- Digital expression matrix (DGE) with spatial barcode coordinates from STARsolo alignment (Step A3 output), e.g. Seq-Scope mouse liver GSM5212844
- Seurat object with collapsed DGE of simple square grids (Step C1 RDS output)
- Seurat object with collapsed DGE of sliding-window (MSSW) square grids (Step C2 RDS output)

## Outputs

- Seurat object from Step C3 with dimension reduction, clustering, and reference-mapped cell-type labels on MSSW overlapping bins
- Spatial clustering plot (Step V2) comparing the simple-bin map versus the MSSW map (e.g. spatial_cluster_maps.png)

## How to apply

First run STtools Step C1 to collapse the DGE from Step A3 (STARsolo alignment output) into non-overlapping, equal-sized simple square grids large enough to capture sufficient UMIs for clustering, and run Step C2 to collapse the same DGE into overlapping square grids via the sliding window strategy (STtools example parameters: --binsize 300 --window 150). Then run Step C3 in Seurat: dimension reduction, clustering, and reference mapping with the simple square-grid bins as the query, producing cluster labels on the coarse bins. Finally transfer the simple-bin cluster labels onto the MSSW overlapping bins, which share spatial overlap with the coarse grids, yielding cell-type assignments at finer resolution. Generate spatial clustering plots via Step V2 and compare the simple-bin map side by side with the MSSW map to confirm the resolution gain.

## Related tools

- **STtools** (Runs Step C1 (simple square-grid collapse) and Step C2 (MSSW sliding-window collapse, --binsize 300 --window 150) whose outputs feed the label transfer; runs Step V2 visualization) — https://github.com/seqscope/STtools
- **Seurat** (Performs dimension reduction, cell-type clustering, and reference mapping in Step C3, transferring simple square-grid labels onto MSSW overlapping bins) — https://satijalab.org/seurat/articles/spatial_vignette.html
- **STARsolo** (Generates the DGE from raw FASTQ in Step A3, the input expression matrix that is collapsed at both scales) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- The MSSW spatial cluster map delineates cell-type boundaries at several-fold higher spatial resolution than the simple square-grid bin map, consistent with STtools' claim of several-fold higher downstream-analysis resolution
- Both Step C1 and Step C2 RDS outputs load as valid Seurat objects and Step C3 consumes both, with simple square grids used as the reference-mapping query
- Every MSSW overlapping bin receives a transferred cell-type label from the simple-bin clustering without unmapped bins
- Step V2 produces UMAP, spatial clustering, and feature violin plots from the Step C3 output
- Sliding-window parameters match the STtools example scale (--binsize 300 --window 150), and bins remain larger than ~10 um subcellular compartments

## Limitations

- Subcellular analysis with MSSW is not feasible: each spatial segment must be larger than subcellular compartments (e.g. ~10 um)
- Spatial smoothing algorithms for robust inference from extremely sparse expression profiles per spatial unit are not yet available in STtools, so labels transferred to very fine bins rely on overlap rather than expression-level imputation
- STtools provides no method to impute spatial expression profiles, which limits independent validation of fine-bin assignments

## Evidence

- [methods] Step C2 takes DGE from Step A3 and output Seurat object with collapsed DGE of square grids from sliding window strategy: "Step C2 takes DGE from Step A3 and output Seurat object with collapsed DGE of square grids from sliding window strategy"
- [readme] Step C3 reference mapping with simple square grids as query: "Step C3 takes in RDS file from Step C1 and Step C2 as input and performs dimension reduction, clustering and conducts refernece mapping with simple square grids as query"
- [abstract] Several-fold higher resolution claim: [evidence span withheld — anchor status: not_found]
- [discussion] MSSW subcellular infeasibility caveat: "each spatial segment still needs to be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible"
