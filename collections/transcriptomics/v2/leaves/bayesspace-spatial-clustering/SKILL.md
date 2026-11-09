---
name: bayesspace-spatial-clustering
description: Use when a SeqScope/Slide-seq/VISIUM digital expression matrix with spatial
  coordinates has too few UMIs per spatial barcode for direct cell-type clustering
  and barcodes must be aggregated into grids before Seurat/BayesSpace clustering.
when_to_use_negative:
- 'The goal is subcellular-resolution analysis with MSSW: spatial segments must be
  larger than subcellular compartments (~10um), so sliding-window analysis below that
  scale is not feasible'
- Input is still raw FASTQ rather than the Step A3 digital expression matrix — run
  the alignment chain (A1–A3) first
- No spatial smoothing or imputation is needed and per-barcode UMI counts are already
  sufficient for direct single-barcode clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STtools
  role: Orchestrates the C1–C3 clustering chain that collapses the DGE into grid-based
    Seurat objects and combines cluster annotations
  repo: https://github.com/seqscope/STtools
- name: Seurat
  role: Holds the collapsed grid DGEs as Seurat objects and performs dimension reduction,
    clustering, and reference mapping in Step C3
  repo: https://satijalab.org/seurat/articles/spatial_vignette.html
- name: BayesSpace
  role: Spatial clustering backend used for the sliding-window (MSSW) grid object
    built in Step C2
  repo: https://www.nature.com/articles/s41587-021-00935-2
- name: STARsolo
  role: Produces the upstream digital expression matrix from FASTQ that feeds the
    C1/C2 grid collapsing
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:61d8ac69f65e25762ab08606edfdff145d71fc073510843be08aafe70df7216e
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# bayesspace-spatial-clustering

## Summary

Clusters cell types in ultra-high-resolution spatial transcriptomics data by collapsing the Step A3 STARsolo digital expression matrix into simple square-grid and multi-scale sliding-window (MSSW) Seurat/BayesSpace objects (Steps C1/C2), then combining dimension reduction, clustering, and Seurat reference mapping in Step C3.

## When to use

Use when a SeqScope/Slide-seq/VISIUM digital expression matrix with spatial coordinates has too few UMIs per spatial barcode for direct cell-type clustering and barcodes must be aggregated into grids before Seurat/BayesSpace clustering.

## When NOT to use

- The goal is subcellular-resolution analysis with MSSW: spatial segments must be larger than subcellular compartments (~10um), so sliding-window analysis below that scale is not feasible
- Input is still raw FASTQ rather than the Step A3 digital expression matrix — run the alignment chain (A1–A3) first
- No spatial smoothing or imputation is needed and per-barcode UMI counts are already sufficient for direct single-barcode clustering

## Inputs

- Digital expression matrix (DGE) with spatial coordinates from STtools Step A3 (STARsolo alignment output)
- Velocyto-format DGE from Step A3 for downstream Step V2 plotting
- Grid parameters: binsize for simple square grids (e.g. 300) and window size for sliding-window aggregation (e.g. 150)

## Outputs

- Step C1 Seurat object (RDS) with DGE collapsed into simple square grids
- Step C2 Seurat/BayesSpace object (RDS) with DGE collapsed into sliding-window square grids
- Step C3 combined cell-type cluster annotation after dimension reduction and reference mapping

## How to apply

Take the DGE and spatial coordinates produced by Step A3 STARsolo alignment and build two aggregations: first collapse spatial barcodes into non-overlapping, equal-sized square grids (e.g. binsize 300 in the SeqScope liver example) to make the Step C1 Seurat object; second collapse the same DGE with the multi-scale sliding-window strategy using overlapping bins (e.g. window 150) to make the Step C2 Seurat/BayesSpace object. Then run Step C3, which loads the C1 and C2 RDS files and performs dimension reduction, clustering, and Seurat reference mapping with the simple square grids as the query, yielding a combined cluster annotation. Choose grid/window sizes so each spatial segment stays larger than subcellular compartments (~10um); apply gene/segment filtering thresholds before clustering.

## Related tools

- **STtools** (Orchestrates the C1–C3 clustering chain that collapses the DGE into grid-based Seurat objects and combines cluster annotations) — https://github.com/seqscope/STtools
- **Seurat** (Holds the collapsed grid DGEs as Seurat objects and performs dimension reduction, clustering, and reference mapping in Step C3) — https://satijalab.org/seurat/articles/spatial_vignette.html
- **BayesSpace** (Spatial clustering backend used for the sliding-window (MSSW) grid object built in Step C2) — https://www.nature.com/articles/s41587-021-00935-2
- **STARsolo** (Produces the upstream digital expression matrix from FASTQ that feeds the C1/C2 grid collapsing) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --outdir $STOUT --genome $GENOMEINDEX --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- The C1 -> C2 -> C3 chain runs end-to-end on the Step A3 DGE without errors
- Each named RDS/Seurat object (C1 simple-grid, C2 sliding-window, C3 combined annotation) exists and loads as a valid Seurat object
- The C1 object was built with non-overlapping equal-sized square grids (e.g. binsize 300) while the C2 object used overlapping sliding-window bins (e.g. window 150)
- Step C3 reference mapping used the simple square grids as the query and returned a combined cluster annotation for all grid segments
- Genes and spatial segments were filtered with explicit thresholds before clustering

## Limitations

- Each spatial segment must be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with the MSSW sliding window is not feasible
- Spatial smoothing algorithms for robust inference from extremely sparse expression profiles per spatial unit are not yet available
- STtools provides no method to impute spatial expression profiles, and no joint clustering of cellular and subcellular components
- Thresholds to filter genes and spatial segments before clustering must be chosen by the user

## Evidence

- [methods] Step C1 takes DGE from Step A3 and output Seurat object with collapsed DGE of simple square grids: "**Step C1** takes DGE from **Step A3** and output `Seurat` object with collapsed DGE of simple square grids."
- [readme] Step C3 takes in RDS file from Step C1 and Step C2 as input and performs dimension reduction, clustering and conducts refernece mapping with simple square grids as query: "**Step C3** takes in RDS file from **Step C1** and **Step C2** as input and performs dimension reduction, clustering and conducts refernece mapping with simple square grids as query"
- [intro] The STtools pipeline includes preprocessing of raw sequence reads, alignment, collapsing barcodes into grids, clustering cell types, and high-resolution analysis with sliding window strategy: "The STTools pipeline includes preprocessing of raw sequence reads, alignment, collapsing barcodes into grids, clustering cell types, and high-resolution analysis with sliding window strategy."
- [intro] STtools leverages STARSolo, Seurat, BayesSpace, and Seqtk: [evidence span withheld — anchor status: not_found]
