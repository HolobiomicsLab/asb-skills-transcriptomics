---
name: seurat-multimodal-reference-mapping
description: Use when a STARsolo-derived DGE has spatial barcodes too sparse (few
  UMIs each, e.g. SeqScope ~1M barcodes/mm2) for direct clustering, and cell-type
  labels must be transferred from a reference onto grid-collapsed spatial segments.
when_to_use_negative:
- The goal is subcellular-resolution analysis — sliding-window segments must be larger
  than subcellular compartments (~10um), so MSSW-based clustering is not feasible
  at subcellular scale
- Input is already a per-cell or per-spot dataset with sufficient UMI depth (e.g.
  standard VISIUM), where direct per-barcode clustering without grid collapsing suffices
- No suitable reference dataset exists for Seurat reference mapping, since Step C3's
  annotation step depends on transferring labels from a reference
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: STtools
  role: 'Pipeline orchestrating Steps C1–C3: grid collapsing, sliding-window aggregation,
    and clustering'
  repo: https://github.com/seqscope/STtools
- name: Seurat
  role: Dimension reduction, clustering, and multimodal reference mapping with simple
    square grids as query
  repo: https://satijalab.org/seurat/articles/spatial_vignette.html
- name: BayesSpace
  role: Spatial clustering backend for the sliding-window (MSSW) collapsed Seurat
    object in Step C2
  repo: https://www.nature.com/articles/s41587-021-00935-2
- name: STARsolo
  role: Produces the Step A3 digital expression matrix (DGE) that is the raw input
    to grid collapsing and clustering
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:d3deb77a57b87ccacbe762788030bcee7f7d484e28aa4557340c20fd9ae44d89
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# seurat-multimodal-reference-mapping

## Summary

Cell-type annotation of ultra-high-resolution spatial data (e.g. SeqScope) by clustering grid-collapsed DGEs in Seurat/BayesSpace and transferring labels via Seurat reference mapping, as implemented in STtools Steps C1–C3.

## When to use

Use when a STARsolo-derived DGE has spatial barcodes too sparse (few UMIs each, e.g. SeqScope ~1M barcodes/mm2) for direct clustering, and cell-type labels must be transferred from a reference onto grid-collapsed spatial segments.

## When NOT to use

- The goal is subcellular-resolution analysis — sliding-window segments must be larger than subcellular compartments (~10um), so MSSW-based clustering is not feasible at subcellular scale
- Input is already a per-cell or per-spot dataset with sufficient UMI depth (e.g. standard VISIUM), where direct per-barcode clustering without grid collapsing suffices
- No suitable reference dataset exists for Seurat reference mapping, since Step C3's annotation step depends on transferring labels from a reference

## Inputs

- Digital expression matrix (DGE) with spatial coordinates produced by STtools Step A3 (STARsolo)
- RDS/Seurat object with DGE collapsed into simple square grids (Step C1 output, e.g. binsize 300)
- RDS/Seurat object with DGE collapsed into sliding-window square grids (Step C2 output, e.g. window 150)
- Seurat reference object for multimodal reference mapping

## Outputs

- Combined Seurat cluster annotation (RDS) assigning cell types to spatial grid segments
- Seurat object with dimension-reduced embeddings and cluster labels for the simple square grid query

## How to apply

First collapse the Step A3 STARsolo digital expression matrix (DGE plus spatial coordinates) twice: once into non-overlapping equal-sized square grids (Step C1, e.g. binsize 300 in the SeqScope liver example) and once with the multi-scale sliding-window (MSSW) strategy of overlapping square bins (Step C2, e.g. window 150), saving each as an RDS Seurat object; Step C2 leverages BayesSpace for spatially-aware clustering. Then, in Step C3, load both RDS files and perform dimension reduction, clustering, and Seurat reference mapping using the simple square grids (C1) as the query and the sliding-window object (C2) as the reference context. The rationale is that individual SeqScope barcodes carry too few UMIs for cell-type calls, so aggregation into grids (simple and sliding-window) is required before clustering. The final output is the combined cluster annotation merging the two strategies, which downstream Step V2 uses for UMAP, spatial clustering, and feature violin plots.

## Related tools

- **STtools** (Pipeline orchestrating Steps C1–C3: grid collapsing, sliding-window aggregation, and clustering) — https://github.com/seqscope/STtools
- **Seurat** (Dimension reduction, clustering, and multimodal reference mapping with simple square grids as query) — https://satijalab.org/seurat/articles/spatial_vignette.html
- **BayesSpace** (Spatial clustering backend for the sliding-window (MSSW) collapsed Seurat object in Step C2) — https://www.nature.com/articles/s41587-021-00935-2
- **STARsolo** (Produces the Step A3 digital expression matrix (DGE) that is the raw input to grid collapsing and clustering) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- Each named RDS/Seurat object from the chain (C1 simple-square-grid object, C2 sliding-window object, C3 combined annotation) exists and loads as a valid Seurat object
- The C1→C2→C3 chain runs end-to-end on the chained Step A3 DGE without error
- The C1 and C2 objects were built from the same Step A3 DGE with the intended grid parameters (e.g. binsize 300, window 150)
- Step C3 reference mapping uses the simple square grids (C1 object) as the query, per the documented design
- The combined annotation covers all spatial segments present in the input DGE coordinates

## Limitations

- Each spatial segment must be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible
- UMI count per spatial barcode is typically much lower at higher resolution, so clustering quality depends on adequate grid collapsing; spatial smoothing algorithms for extremely sparse expression profiles per spatial unit are not yet available in STtools
- STtools currently provides no method to impute spatial expression profiles and no joint clustering of cellular and subcellular components

## Evidence

- [readme] Step C3 performs dimension reduction, clustering, and Seurat reference mapping with simple square grids as query: "Step C3 takes in RDS file from **Step C1** and **Step C2** as input and performs dimension reduction, clustering and conducts refernece mapping with simple square grids as query"
- [intro] The STtools pipeline includes preprocessing of raw sequence reads, alignment, collapsing barcodes into grids, clustering cell types, and high-resolution analysis with sliding window strategy: "The STTools pipeline includes preprocessing of raw sequence reads, alignment, collapsing barcodes into grids, clustering cell types, and high-resolution analysis with sliding window strategy"
- [readme] Step C1 takes DGE from Step A3 and outputs a Seurat object with collapsed DGE of simple square grids: "**Step C1** takes DGE from **Step A3** and output `Seurat` object with collapsed DGE of simple square grids."
- [readme] Step C2 takes DGE from Step A3 and outputs a Seurat object with collapsed DGE of square grids from sliding window strategy: "**Step C2** takes DGE from **Step A3** and output Seurat object with collapsed DGE of square grids from sliding window strategy"
